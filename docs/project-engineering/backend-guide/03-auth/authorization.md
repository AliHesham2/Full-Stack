# Authorization — What May the Caller Do

> **Part of:** Auth — Backend Project Guide
> **Read when:** Deciding who can access what; designing permissions, roles, policies, tenancy isolation; reviewing any endpoint for access control.
> **Related:** `authentication.md`, `../04-security/security-foundations.md`, `../11-specialized-concerns/multi-tenancy.md`, `../09-testing/testing-strategy.md`
> **Status:** Grounded in **OWASP API Security Top 10 2023** — verified as still the current edition, with **API1:2023 Broken Object Level Authorization at #1**. Corroborated by **OWASP Top 10:2025**, where **A01 Broken Access Control** also remains #1 · Stable principles

---

## 1. Position

Broken authorization — not broken authentication — is the **#1 API security vulnerability**
(OWASP API Security Top 10 2023: API1 BOLA, plus API5 Broken Function Level Authorization).
It is #1 on the web list too: **A01 Broken Access Control** held the top position in
**OWASP Top 10:2025**. Two independent lists, same verdict — this is the highest-yield place
to spend review effort. Authentication answers "who"; authorization answers "may this actor touch this object in this way" — and it must be answered **server-side, on every request, at object granularity**. The UI hiding a button is not authorization. A route being "internal" is not authorization. An ID being "unguessable" is not authorization.

## 2. Enforcement Placement

Authorization is checked at **two** places, and only these:

1. **Edge (coarse):** route-level guards — is this an authenticated route? Which role/claim floor applies? Cheap, fails fast, but NEVER sufficient alone.
2. **Use-case (fine, authoritative):** per-object, per-action checks inside the use case, where domain context lives ("does this actor own/operate this order in its current state?").

Rules:

- The fine check is **mandatory even when the coarse check passed** — an admin token still gets object-level checks (unless the action is genuinely global-admin by design, which is itself a policy to define).
- Load-then-check is normal (you need the object to know its owner) — but authorize **before** any side effect, and before returning any object data (returning the object to check its owner is the leak).
- Centralize policy definitions; **never scatter ad-hoc `if role === 'admin'` across handlers.** Policies are named, testable units ("canCancelOrder(actor, order)") with one home in the domain layer (`../02-domain-and-api/domain-logic.md`).

## 3. Model Selection

| Model | Use when | Cost / risk |
|---|---|---|
| **Resource ownership** (owner check) | Every user-scoped resource — the default floor | Must be applied EVERYWHERE; misses = BOLA |
| **RBAC** (roles → permissions) | Stable, enumerable job functions (admin, support, member) | Role explosion if abused; roles ≠ individuals |
| **ABAC / policy** (attributes, evaluated) | Contextual rules (tenant, region, record state, time) | Evaluation complexity; harder to audit |
| **ReBAC / relationship graphs** (e.g., team hierarchies, sharing) | Shared resources, nested orgs | Graph engine dependency — only at real need |

**Layered default:** ownership check everywhere + RBAC for functional tiers + ABAC-style conditions on the few rules that need context. Don't adopt a full ABAC/policy engine before a concrete rule demands it (`../01-foundation/complexity-control.md`).

### 3.1 Permission design rules

- Permissions are named by **action on resource** (`order:cancel`), not by UI affordance ("orders-page:visible") — UI changes faster than the domain.
- Roles are sets of permissions; users get roles; **checks test permissions, not role names** (`can(actor, 'order:cancel')` not `if user.role === 'admin'`) — role renames then don't break policy, and least-privilege audits are greppable.
- **Deny by default:** no permission grant ⇒ refuse. Whitelist semantics, never blacklist ("block the dangerous roles" omits the new role you add later).
- Superadmin is a real role used by real humans rarely, with its own audit trail and ideally step-up authentication — not an escape hatch sprinkled through code.

## 4. Object-Level Authorization (The BOLA Section)

The dominant real-world vulnerability pattern:

```
GET /api/orders/ord_9f2c...     ← attacker swaps ord_9f2c → ord_8a1b (someone else's)
```

Defense rules:

1. **Every object access verifies the relationship** between authenticated actor and the specific object: ownership, tenancy, sharing grant, or delegated authority. No exceptions for "GET" — reading other tenants' data is the breach, not just writing.
2. The check lives in the use case/repo query, not only in application memory: prefer queries that are scoped by construction (`WHERE id = $1 AND tenant_id = $2` / RLS) over fetch-then-check-then-hope — see §6.
3. **IDs are not capabilities.** UUIDs raise enumeration cost; they never replace authorization (logs leak IDs, referrers leak IDs, clients copy IDs).
4. Watch the **indirect object references**: the endpoint takes `invoiceId`, but the invoice links to an account — checking "owns invoice" while the invoice's `account_id` flows into another operation is boundary confusion. Follow the whole reference chain to the owning domain concept.
5. Bulk endpoints (`?ids=a,b,c`) authorize **each** object and return per-item outcomes — partial results must not silently include unauthorized reads.

## 5. Multi-Tenant Authorization

- Tenant context is derived from authenticated identity — **never from a client-supplied `tenant_id` header/param alone** (that's self-attested tenancy = no tenancy).
- Propagate tenant context explicitly through the call stack (request-scoped context object, not ambient globals — but see `../12-anti-patterns/anti-patterns.md` § global mutable state for the failure mode).
- Prefer **database-enforced isolation** as the backstop: PostgreSQL Row-Level Security policies keyed on the session's tenant setting (`../11-specialized-concerns/multi-tenancy.md` has the model selection; the Supabase/Postgres guide has mechanics).
- Cross-tenant operations (support staff "view as tenant") are explicit, permission-gated, audited, and time-bounded — never ambient.

## 6. Enforcement Layers (Defense in Depth)

| Layer | Mechanism | Catches |
|---|---|---|
| Edge guards | route middleware: authentication + role floor | unauthenticated traffic, gross role mismatches |
| Use-case policies | named policy functions (authoritative) | business-rule refusals, state-dependent access |
| Query scoping | tenant/owner predicates in every query | developers forgetting policy calls (yes, even good teams) |
| DB constraints / RLS | database-level row policies | application bugs, compromised app tier |
| Tests | denial tests per endpoint (§8) | regressions of all the above |

An authorization architecture that relies on only the first row is a breach with extra steps. Layers 3–4 exist because layer 2 is human/AI-written code, and code forgets.

## 7. Privilege Escalation — Review Vectors

When reviewing any user-data-mutation surface, check:

- **Horizontal:** actor A acting on actor B's objects (BOLA — §4)
- **Vertical:** actor escalating own rights — mass assignment (`role` field in an updatable DTO — see `../02-domain-and-api/validation.md` §3 strict schemas), permission-grant endpoints authorizing their own abuse ("user with `user:invite` granting themselves `admin`"), self-service flows reaching admin states
- **Stale-elevation:** privileges cached in tokens/sessions outliving their revocation (long TTLs, missing server-side session checks — `authentication.md` §2.1/§4)
- **Confused deputy:** a privileged endpoint acting on unvalidated instructions from a less-privileged source (server-side request forgery via parameter injection; `../04-security/security-foundations.md` § SSRF)

## 8. Testing (Non-Negotiable)

Every authorization-bearing endpoint ships **denial tests**:

- user A cannot read/write user B's object → 403/404
- role X cannot call admin endpoint → 403
- revoked session / expired token → 401
- cross-tenant access with a VALID token of another tenant → denied
- mass-assignment attempt (`role: "admin"` in body) → rejected
- pagination/list endpoints scoped (user A's list contains only A's rows — count assertions)

These are cheap to write at the API layer and catch the highest-impact class of real breaches. See `../09-testing/testing-strategy.md` § auth tests.

## 9. Failure Modes

- **Edge-guard-only authorization:** "we check the admin middleware" — object-level checks missing ⇒ BOLA by omission.
- **Owner-check-on-write-only:** reads unguarded ("it's just a GET") — data exfiltration endpoint.
- **Policy scatter:** `if (user.isAdmin || user.id === order.userId && order.status !== 'archived' ...)` inline in 12 handlers, each subtly different — centralize or accept eventual breach.
- **Self-attested tenancy:** `tenant_id` from the request body trusted because "the frontend always sends the right one."
- **Capability-by-obscurity:** "nobody can guess our IDs" (§4.3) — logs and referrers disagree.
- **Permanent superadmin tokens:** long-lived god-credentials for "ops scripts" — per-service identity + step-up instead (`authentication.md` §5).
