# Security Foundations — Threats, Boundaries, and the Injection Family

> **Part of:** Security — Backend Project Guide
> **Read when:** Before designing any feature that touches input, output, external calls, files, or trust zones — and as the security checklist for every task's validation step.
> **Related:** `secrets-management.md`, `../03-auth/*`, `../02-domain-and-api/validation.md`, `../12-anti-patterns/anti-patterns.md`
> **Status:** Grounded in **OWASP Top 10:2025** (current edition — verified) and **OWASP API Security Top 10 2023** (still the current API edition — verified), plus current IETF/MDN guidance · Security is ecosystem-volatile: re-verify specifics for your stack per project

---

## 0. The Current OWASP Top 10 (2025) — and What It Changed

The web Top 10 was re-issued as **OWASP Top 10:2025**, replacing the 2021 edition. An earlier
revision of this file was anchored to 2021. The new list, verified from owasp.org:

| | 2025 category | Where this guide covers it |
|---|---|---|
| **A01** | **Broken Access Control** (still #1) | `../03-auth/authorization.md` — BOLA, object-level checks |
| **A02** | Security Misconfiguration | §4 headers, `../10-delivery/configuration.md`, `secrets-management.md` |
| **A03** | **Software Supply Chain Failures** *(new, elevated)* | §5 below, `../10-delivery/dependencies.md` |
| **A04** | Cryptographic Failures | `secrets-management.md` §5 |
| **A05** | Injection | §3 below |
| **A06** | Insecure Design | `../01-foundation/architecture.md`, this file's §2 trust boundaries |
| **A07** | Authentication Failures | `../03-auth/authentication.md` |
| **A08** | Software or Data Integrity Failures | §5, `../05-data/migrations.md` |
| **A09** | **Security Logging and Alerting Failures** *(renamed)* | `../08-observability/observability.md` §7 audit logs |
| **A10** | **Mishandling of Exceptional Conditions** *(new)* | `../07-reliability/error-handling.md` — the "never swallow" rule |

Two things worth noticing, because they are unusually direct confirmations of this guide's
structure:

- **Supply chain became its own top-3 category (A03).** The dependency policy in
  `../10-delivery/dependencies.md` is not defensive over-caution; it is now a named Top 10 risk.
- **A10 "Mishandling of Exceptional Conditions" is new** — swallowed errors, empty catch blocks,
  and failures that leave the system in an unknown state. That is precisely
  `../07-reliability/error-handling.md` §1, and it is the single most common defect class in
  generated code.

**Access control stayed at #1 across both lists** (web A01:2025 and API API1:2023 BOLA) — which
is why `../03-auth/authorization.md` is the file to read before shipping any endpoint.

## 1. The Security Mindset for Backend Work

Security is not a feature you add at the end; it is a property of **boundaries** and **defaults**. Every backend security control answers one of:

1. **Trust boundaries** — where does less-trusted data enter? (§2)
2. **Input attacks** — what does hostile input try to make you do? (§3)
3. **Output attacks** — what does your output enable downstream? (§4)
4. **Configuration & dependencies** — what does your supply chain expose? (§5)
5. **Information leakage** — what do errors, logs, and timing reveal? (§6)

Run these five questions over every feature (the agent workflow's validation step — `../00-navigation/agent-workflow.md` §2 step 6).

## 2. Trust Boundaries

- Draw the map first: public internet → edge/CDN/load balancer → your services → workers → database → third-party APIs. Data crossing each boundary **toward more-trusted zones** gets validated/contract-checked; identities crossing boundaries get re-authenticated (zero trust: your own services authenticate to each other — `../03-auth/authentication.md` §5).
- **The client application is part of the attack surface, not part of the defense.** Anything enforced only client-side (validation, hiding, "disabled" buttons, price computation) is an enforcement of nothing.
- Internal services are NOT automatically trusted: a compromised container, a misbehaving cron job, or a supply-chained dependency runs inside your perimeter. Least privilege applies to internal identities too.
- Every integration (webhooks in, OAuth partners, email links) is a new boundary with its own validation and authentication requirements.

## 3. Input Attack Family (Injection & Friends)

The whole family shares one root cause: **input crosses into an interpreter/context without encoding or parameterization.**

| Attack | Vector | Primary defense |
|---|---|---|
| SQL/NoSQL injection | input → query context | parameterized queries ONLY; never string-concat SQL; identifiers allowlisted (`../05-data/data-access.md`) |
| Command injection | input → shell | avoid shelling out with input; if unavoidable: strict allowlist + no `sh -c`, pass argv array |
| Template/HTML injection | input → template engine | auto-escaping engines; sandboxed templates; no user input in template *names* |
| Path traversal | input → filesystem path | allowlist names; resolve + prefix-check (`path.resolve` then verify it's inside the base dir); never raw `req.body.file` into paths |
| Unsafe deserialization | input → object graph | never deserialize untrusted data with native/object serializers (pick your JSON-only options); accept only schemas you validate |
| SSRF | input → your server makes requests | allowlist outbound destinations (scheme/host/port); block link-local/metadata IPs (169.254.169.254 & friends); never fetch user URLs server-side without an allowlist; disable redirect-following on user-supplied URLs |
| XXE (XML inputs) | input → XML parser | disable DTDs/external entities; prefer non-XML formats at boundaries |
| Log injection | input → log lines | structured logging with escaped values; no raw newline-laden input into logs (`../08-observability/observability.md`) |

Rules of engagement:

- **Parameterize or encode at the point of crossing** — the query layer parameterizes, the template layer encodes, the shell layer never exists. "Sanitize once at entry" does not survive context changes (input stored then rendered elsewhere crosses twice — `../02-domain-and-api/validation.md` §7).
- Mass assignment is injection's cousin: strict schemas reject unknown fields (`../02-domain-and-api/validation.md` §3).

### CSRF (browser-reachable state changes)

- State-changing browser-facing endpoints: `SameSite=Lax/Strict` cookies (partial protection — subdomain-adjacent tricks and older browsers remain) **plus** CSRF tokens (double-submit or synchronizer pattern) for high-risk surfaces.
- SameSite is not a CSRF defense for token-in-header APIs (no cookie ⇒ no CSRF, but then XSS token theft becomes the concern — HttpOnly cookies win that trade for browser clients; `../03-auth/authentication.md` §2.2).

## 4. Output-Side Attacks

- **XSS from backend content:** anything your API returns that a browser renders (HTML emails, rich-text fields, SPA-rendered user content) must be encoded at render time by the consumer — your job is to (a) store canonical data, (b) never generate HTML from untrusted input server-side without explicit escaping, (c) set a restrictive `Content-Type` (never `text/html` for user-controlled payloads) and `Content-Disposition` on file downloads.
- **Response headers** (set centrally, verify per framework guide): `Strict-Transport-Security` (relevant for API hosts too — browsers hitting your API over HTTPS should be pinned to it), `X-Content-Type-Options: nosniff`, `Cache-Control: no-store` on authenticated/private responses (proxxies and shared caches must never serve user A's data to user B — a classic CDN misconfiguration breach), and CSP for any HTML-serving surface.
- **Open redirects:** never redirect to client-supplied URLs without allowlisting destinations.
- **Error/output leakage:** see §6.

## 5. Supply Chain & Dependencies (2025–2026: Assume Hostility)

The npm ecosystem suffered **wormable supply-chain attacks in 2025–2026** (Shai Hulud, Mini Shai-Hulud, ChainDrop) — hundreds of popular packages compromised, in some cases even with signed provenance. Treat dependency selection as a security decision:

- Lockfiles committed and enforced; CI runs `npm audit`-equivalent + installs with `--ignore-scripts` semantics where possible (postinstall scripts are the primary worm vector).
- Minimal dependency count; every package earns its place (`../10-delivery/dependencies.md` has the full policy).
- Pin exact versions in CI; review lockfile diffs in PRs like code.
- Secrets never live where a malicious postinstall script can read them (build env hygiene; scoped CI tokens).

## 6. Information Leakage & Timing

- Errors: one envelope, no internals (`../02-domain-and-api/api-design.md` §4). Stack traces, SQL, driver names, internal hosts = reconnaissance gifts.
- Enumeration: uniform responses + uniform timing on existence-sensitive queries (login, recovery, signup).
- Logs and metrics: redact secrets and minimize PII (`../08-observability/observability.md` §6, `secrets-management.md` §4).
- Debug endpoints, `/metrics`, admin surfaces: authenticated, authorization-gated, internal-network-only — never exposed publicly by default.
- File metadata, sequential IDs, and verbose 404/403 distinctions leak existence — choose deliberately per `../02-domain-and-api/api-design.md` §3.

## 7. Rate Limiting & Abuse Prevention

- Per-identity limits at the edge (429 + `Retry-After`; algorithm and mechanics in `../02-domain-and-api/api-design.md` §8).
- Layer on **abuse patterns**: credential stuffing (per-account + per-source aggregate limits), scraping (velocity anomalies), resource exhaustion (payload caps, timeouts, query costs — big `OFFSET`/deep pagination is a DoS primitive).
- Failed-auth counters with escalation (lockout/backoff/CAPTCHA) and alerting — see `../03-auth/authentication.md` §6.

## 8. Feature Threat Checklist (Run Per Task)

- [ ] Where does input enter, and is it validated at the boundary (strict schema)?
- [ ] Does any input reach a query/command/template/path/URL context — parameterized/encoded/allowlisted?
- [ ] Is authorization enforced per-object server-side (`../03-auth/authorization.md` §4)?
- [ ] Are server-owned values (prices, roles, tenant IDs) derived, not accepted?
- [ ] Any new outbound request driven by user input → SSRF allowlist?
- [ ] Any file intake → size/type/extension validation + private storage (`../06-async-and-scale/file-and-object-storage.md`)?
- [ ] Authenticated responses `no-store`'d; security headers set?
- [ ] Secrets: none in code/logs/errors; new secret needs a management path (`secrets-management.md`)?
- [ ] New dependency justified + audited (`../10-delivery/dependencies.md`)?
- [ ] Failure paths tested — including what an attacker learns from each error response?

## 9. Security Anti-Patterns (High-Frequency AI-Generated)

- **"Sanitized" SQL string concatenation** ("we escaped the quotes") — parameterization is binary: you do or you don't.
- **Obfuscation as access control:** secret URLs, unguessable IDs, obfuscated endpoints (§2, `../03-auth/authorization.md` §4.3).
- **Trust-the-client fields:** `isAdmin: true` accepted in an update body; totals computed client-side.
- **Broad CORS as a fix:** `Access-Control-Allow-Origin: *` with credentials (that combination is rejected by browsers for cause — if you "made it work," something worse happened). In this project no browser origin is allowed at all and there is no `cors` middleware: only the Next.js server calls Express (`../03-auth/authentication.md` §0).
- **Crypto hand-rolling:** custom encryption schemes, ECB mode, non-CSPRNG tokens, homebrew password hashing — use vetted primitives through vetted libraries, keys managed per `secrets-management.md`.
- **Security theater tests:** "assert 401 without token" as the only auth test — object-level denial tests are the real coverage (`../09-testing/testing-strategy.md`).
