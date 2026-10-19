# Authentication — Who Is Calling

> **Part of:** Auth — Backend Project Guide
> **Read when:** Designing login, sessions, tokens, API keys, service-to-service identity, password handling, MFA, or account recovery.
> **Related:** `authorization.md` (a different concern — read both), `../04-security/security-foundations.md`, `../11-specialized-concerns/multi-tenancy.md`
> **Status:** Grounded in current standards: OWASP Session Management & Password Storage Cheat Sheets, NIST SP 800-63-4 (Rev 4, current), OAuth 2.1 draft (PKCE mandatory, implicit & ROPC removed), WebAuthn/passkeys adoption growing · Re-verify ecosystem specifics per project

---

## 0. API-only transport and documented auth baseline

Read [backend-only scope](../00-navigation/backend-only-scope.md). The server transport/default below guides real approved auth work; it does not require creating login endpoints for every API task, prove a configured provider or authorize a live session store. Missing material auth policy/limits must be settled in Planner's approved security plan before execution:

```
Authorized server caller (external project) → Express (this backend) → approved persistence/provider
```

- **Clients are authorized server-side callers in other projects**, not a fixed frontend framework. The current transport is server-to-server: no browser-facing API CORS policy or Express cookie session is implied. Use private networking and appropriate service credentials, plus user/resource/tenant authorization; CORS is not authentication. Direct browser/cookie support requires an explicit reviewed architecture change.
- **Only this backend owns database/provider SDKs and credentials.** Supabase remains the documented provider baseline; record actual approved provider/version before dependent work. No external client implementation or setup follows from this guide.
- **The session is opaque and server-side (§2.1 default).** On login, Express verifies the
  credentials with Supabase Auth, then creates its own session record. The record gets a random
  ID of at least 128 bits, stored hashed, and it maps to the user. Any Supabase tokens stay in
  that record, encrypted, and Express refreshes them itself. Express returns only the session
  ID. The authorized server client transmits it as `Authorization: Bearer <id>` under the approved contract. Any client cookie/storage implementation belongs to its separate project.
  Why not forward the Supabase JWT instead? It could not be revoked instantly, the refresh flow
  would leak into external clients and couple their implementation to the auth provider.

**Documented authentication contract for external callers, when auth is actually in scope:**

| Endpoint | Request | Success | Failure |
|---|---|---|---|
| `POST /auth/login` | `{ email, password }` | `200 { token }`: the session ID in the JSON body, **never** `Set-Cookie` | `401` with the error envelope. The message is always "invalid credentials" (§3, §6); 400 invalid shape, 429 throttling, 503/504 provider outage/timeout |
| `GET /auth/me` | `Authorization: Bearer <id>` | `200 { data: user }` | `401` if the session is missing, expired, or revoked |
| `POST /auth/logout` | `Authorization: Bearer <id>` | `204`, after deleting the session server-side | — |
| every other endpoint | `Authorization: Bearer <id>` | per endpoint | `401` unauthenticated, `403` authenticated but not allowed (`authorization.md`) |

- Every error uses the envelope `{ error: { code, message, details? } }` (`details` only for a
  400 validation error: `../02-domain-and-api/api-design.md` §4). External callers use this shape and status to distinguish "logged out" (401) apart from "service unavailable"
  (5xx or timeout). Returning 401 for anything that is not an auth failure logs users out by
  mistake.
- Express owns expiry/revocation; the complete approved policy records absolute/idle limits, refresh/revocation and client-visible outcomes. Do not infer a production TTL or client cookie implementation from examples. Operational failures must not be reclassified as invalid identity.

### 0.1 How Express Talks to Supabase

In this project only Express talks to Supabase (the Supabase reference, `workspace/docs/syntax/SupaBase/auth.md`,
shows the same setup), like this:

```
EXPRESS ↔ SUPABASE
├──▶ KEYS: SUPABASE_URL · SUPABASE_PUBLISHABLE_KEY (sb_publishable_…) · SUPABASE_SECRET_KEY
│    (sb_secret_…), all in Express's environment ONLY (04-security/secrets-management.md)
│
├──▶ TWO CLIENTS, both created with auth: { persistSession: false, autoRefreshToken: false,
│    │   detectSessionInUrl: false }. A server has no browser storage, and Express refreshes
│    │   tokens itself, per session (below)
│    ├──▶ USER client  = publishable key + global.headers.Authorization = "Bearer <that user's
│    │                   Supabase access token>". Created PER REQUEST, never shared between
│    │                   requests. RLS applies as that user → the default for all user data
│    └──▶ ADMIN client = secret key. Bypasses RLS entirely → only for work no user owns
│                        (background jobs, admin tasks). Never for a user's own request
│
├──▶ LOGIN — POST /auth/login { email, password }
│    1. validate the input (02-domain-and-api/validation.md)
│    2. a fresh client (publishable key) → auth.signInWithPassword({ email, password })
│       invalid credentials → 401 with generic "invalid credentials" (§3, §6)
│       network failure, timeout or provider 5xx → operational 503/504, NOT 401
│    3. create the Express session record (§0): a random id of ≥128 bits, stored HASHED →
│       the user id + Supabase access_token + refresh_token (ENCRYPTED at rest) + expires_at,
│       with absolute and idle timeouts
│    4. return 200 { token: <the session id> }. Never the Supabase tokens, never Set-Cookie
│
├──▶ EVERY REQUEST — Authorization: Bearer <session id>
│    1. hash it → look up the session → missing, expired or revoked → 401
│    2. the Supabase access token expired (or is about to) → auth.refreshSession({ refresh_token })
│       → store the NEW access and refresh token (the old refresh token is used up)
│       → invalid/revoked/expired refresh token → delete the session → 401
│       → network timeout, provider 5xx or rate limit → retain the session record and return
│         503/504 (or 429 with Retry-After); never use an expired access token
│       Refresh ONE AT A TIME per session (shared DB/Redis lock across instances, not an
│       in-process mutex); reload tokens after acquiring it and atomically store the result: two parallel requests refreshing with the
│       same refresh token race each other (05-data/concurrency-and-idempotency.md)
│    3. build the USER client with the access token → do the work
│
├──▶ GET /auth/me → the session's user. Call auth.getUser(accessToken) when it must be fresh:
│    it asks the Supabase Auth server, so the answer is authentic
├──▶ LOGOUT → delete the session record → 204. The stored refresh token dies with it
└──▶ SIGN-UP, password reset, OAuth → the same rule: Express calls Supabase Auth, and the
     client only ever receives Express's own session id
```

## 1. Authentication ≠ Authorization

Authentication establishes **who** the caller is. Authorization decides **what** they may do (`authorization.md`). Never conflate them: an authenticated admin token still gets per-object authorization checks, and "internal-only endpoint" is not authentication. Build them as separate layers with separate tests.

## 2. Model Selection — The First Decision

| Client type | Recommended model | Why |
|---|---|---|
| **Browser web app (first-party)** | **Server-side sessions + secure cookie** | Revocable instantly, no token payload to leak, CSRF manageable (§2.2), simplest correct story. OWASP recommends cookies for session exchange. |
| **Mobile / SPA with separate API** | Access token + refresh token (OAuth 2.1 flows via an IdP), or session cookie if same-site | Sessions still viable if the API lives on the same origin family; otherwise tokens |
| **Third-party / machine consumers** | API keys (with scopes + rotation) or OAuth client credentials | Auditable, revocable, rate-limitable identity |
| **Service-to-service (internal)** | mTLS, or short-lived tokens minted by a trusted issuer (token exchange) | Zero-trust: internal ≠ trusted blindly |

### 2.1 The sessions-vs-JWT decision (do not cargo-cult)

JWTs are the most over-prescribed pattern in modern backend tutorials. Decision rules:

- Choose **server-side sessions** when: same-site browser clients, you need instant revocation, you don't need cross-service identity propagation. This is the OWASP-recommended default for web apps.
- Choose **short-lived JWT access tokens** when: multiple services must verify identity statelessly, or you're integrating within an OAuth2/OIDC architecture with an established issuer.
- Never store long-lived bearer tokens where one XSS exfiltration = permanent account takeover; never put PII/authorization data inside a JWT payload (it's readable by anyone holding it, and stale until expiry — the "revocation problem").
- JWT access tokens: 5–15 min TTL + a **refresh** mechanism with rotation (§4).

### 2.2 Cookie mechanics (when sessions are chosen)

- `HttpOnly; Secure; SameSite=Lax` (or `Strict` where UX allows) — HttpOnly blocks XSS token theft; SameSite mitigates CSRF (partial protection only — keep CSRF tokens for state-changing browser flows on legacy-risk surfaces; `../04-security/security-foundations.md` § CSRF).
- Session ID: ≥128 bits of entropy from a CSPRNG, stored hashed server-side, regenerated on privilege change (login), with server-side absolute + idle timeouts.
- Prefer a session-per-user-invariant: one active session record per device, listed to the user, individually revocable ("log out other devices").

## 3. Passwords (If You Run Your Own Credential System)

Current guidance — NIST SP 800-63-4 (Rev 4) + OWASP Password Storage Cheat Sheet:

- **Hash with Argon2id** (OWASP's first recommendation) — memory-hard. **OWASP states a
  concrete floor, and this is the number to implement against: minimum 19 MiB memory,
  iteration count 2, parallelism 1** (verified against the current Password Storage Cheat
  Sheet). Tune *upward* from there toward ~100–500ms per hash on your production hardware —
  the latency target is how you scale above the floor, never a reason to drop below it. An
  earlier revision of this file gave only the latency target while citing OWASP; on fast
  hardware that can land under the stated minimum.
- **bcrypt** (work factor **≥ 10**) is acceptable for legacy systems. Two caveats OWASP
  attaches that are easy to miss:
  - **bcrypt truncates at 72 bytes.** Note the tension with the length policy below ("allow
    ≥64"): a 64-character password is fine, but multi-byte characters push past 72 bytes and
    everything beyond is silently ignored. Enforce a byte-length limit, or pre-hash.
  - If you pre-hash to work around that limit (`bcrypt(SHA-256(password))`), be aware of the
    **null-byte truncation** interaction — use a hex/base64-encoded digest, not raw bytes.
- **PBKDF2**, where required for compliance: ≥600,000 iterations with HMAC-SHA-256.
- MD5/SHA-*/fast hashes used directly are disqualifying defects.
- **Length policy:** minimum 15 characters for password-only authentication; minimum 8 only when the password is always part of MFA (allow at least 64), no composition rules (no "must contain !@#"), no forced periodic rotation (it degrades passwords — rotate only on evidence of compromise), screen against known-breached passwords at signup/password-change.
- **Transmit over TLS only**; compare in constant time (library concern, but verify); on failure respond generically ("invalid credentials") without revealing which factor failed.
- **Rate-limit** login attempts per-account AND per-IP/source (§6 + `api-design.md` §8), with escalating delays or temporary lockout — credential stuffing is the dominant real-world attack on login endpoints.
- Prefer established identity providers / auth libraries over hand-rolled crypto and flows whenever available (custom auth is security-critical code — the one place "build vs buy" almost always answers buy).

### 3.1 MFA

- Offer/require TOTP or **WebAuthn/passkeys** (phishing-resistant — the direction of travel; prefer passkeys for new consumer products where UX allows).
- Store TOTP secrets encrypted at rest; one-time backup codes generated at enrollment; recovery flows treated as MFA-equivalent sensitive paths (§7).
- Verify MFA **before** issuing the session (two-step: password check → MFA challenge → session), never "session with MFA-pending" that can access resources.

## 4. Token Architecture (When Tokens Are Chosen)

- **Access token:** short TTL (5–15 min), audience/scope-bound, verified by signature at each service.
- **Refresh token:** long-lived, stored server-side (revocable), **rotated on every use** — each refresh issues a new token and invalidates the old; reuse of a rotated refresh token is treated as theft (revoke the whole family, force re-auth, alert).
- Authorization-code flow with **PKCE for all clients**; the implicit grant and ROPC (password
  grant) are out. Exact redirect-URI matching.

  **Status precision:** OAuth 2.1 is still an **IETF Internet-Draft**, not a published RFC
  (checked against the RFC index — it is not there). That does *not* weaken the guidance: the
  same requirements are already normative in the **OAuth 2.0 Security Best Current Practice**,
  and every major provider has implemented them. Cite the BCP rather than "the OAuth 2.1 spec"
  when you need an authoritative reference, and expect the draft's section numbers to move.
- **Algorithm pinning** on JWT verification (`alg` confusion attacks are a classic: never accept `none` or unanticipated algorithms; prefer asymmetric verification where multiple services verify).

## 5. Service-to-Service & Machine Identity

- **mTLS** where the platform supports it (strongest: both sides present certs, short-lived, auto-rotated via the platform/spire/vault per ecosystem guide).
- Otherwise: short-lived tokens issued by a trusted internal authority (JWT with `aud` = target service), or cloud-native workload identity (IAM roles / workload identity federation) — never static shared secrets in config for high-privilege paths.
- Each service has its OWN identity (not one shared "backend" credential) so blast radius and audit trails are per-service.
- Zero-trust rule: an internal call still authenticates and still authorizes (`../04-security/security-foundations.md` § trust boundaries). "It came from inside the cluster" is not an authorization model.

## 6. Abuse Controls on Auth Surfaces

Auth endpoints are the highest-abuse surfaces: rate limits (per-account, per-IP, per-device fingerprint where available), generic error messages, uniform response timing for user-exists vs not (timing side channels), CAPTCHA/approved equivalents on repeated failures, and audit logging of failures (pattern detection: distributed stuffing shows as low-per-account, high-per-source aggregates).

## 7. Account Recovery (The Weakest Door)

Recovery bypasses MFA and passwords — treat it as the most security-sensitive flow you have:

- Recovery links/codes: single-use, short TTL (≤15 min), bound to the account, invalidated on use, and rate-limited.
- Verify possession of ≥1 strong factor before allowing credential reset; email-only reset for high-value accounts is a documented takeover pattern (email accounts get compromised).
- No leakage: "if an account exists" responses on recovery flows are enumeration leaks — same message regardless.
- Log recovery events for audit; notify the user (out-of-band) when recovery is initiated on their account.

## 8. Implementation Checklist

- [ ] §0 contract implemented exactly: `/auth/login` returns `{ token }` (no `Set-Cookie`), `/auth/me`, `/auth/logout`, 401 only for real auth failures
- [ ] Model chosen per client type (§2) — and the choice recorded with rationale
- [ ] Sessions: hashed server-side, regenerated at privilege change, absolute+idle timeouts
- [ ] Cookies: HttpOnly + Secure + SameSite; CSRF strategy decided
- [ ] Passwords: Argon2id, NIST-63-4 policy, breached-password screening
- [ ] MFA: phishing-resistant preferred; enforced before session issuance
- [ ] Tokens: short access TTL, rotating refresh, reuse detection, alg pinning
- [ ] S2S: per-service identity, short-lived credentials, no shared static secrets
- [ ] Auth endpoints: rate-limited, generic errors, timing-uniform, audit-logged
- [ ] Recovery: single-use short-TTL codes, out-of-band notification, enumeration-safe
- [ ] Tests: every auth path has a negative test (wrong password, expired token, revoked session, reused refresh) — see `../09-testing/testing-strategy.md` § auth tests

## 9. Failure Modes

- **JWT-as-session cargo-cult:** long-lived JWTs in localStorage, no revocation path, auth data staled inside payloads until expiry. Fix per §2.1.
- **Rotation theater:** refresh tokens rotated but old ones not invalidated server-side — rotation without a server-side invalidation list is decoration.
- **Enumeration leaks:** signup/login/recovery responses that differ for existing accounts — harvest-and-stuff pipeline gift.
- **MFA-bypass-by-flow:** endpoints reachable with "half-authenticated" state (password verified, MFA pending).
- **Custom crypto / custom flows:** invented challenge-response schemes, homebrew "encryption" of tokens. Use vetted libraries and standard flows; novelty here is a vulnerability, not a feature.
- **Silent deprecation blindness:** building new integrations on implicit grants or password grants in 2026 — both are removed in OAuth 2.1 for cause.
