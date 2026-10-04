# Multi-Tenancy — Isolation Models and Tenant Context

> **Part of:** Specialized Concerns — Backend Project Guide
> **Read when:** One deployment serves multiple customer organizations: SaaS, B2B platforms, or any data that must never cross customer boundaries.
> **Related:** `../03-auth/authorization.md` §5, `../05-data/data-architecture.md` §4, `../09-testing/testing-strategy.md`
> **Status:** Stable decision framework · Discovered importance: tenant leakage is the highest-severity SaaS defect class

---

## 1. Position

Multi-tenancy is an **architectural property decided early** (retrofitting tenancy onto a single-tenant schema is one of the most expensive rewrites in backend engineering) and a **security invariant enforced at every layer**. The core rule: tenant identity is derived from authenticated context, propagated explicitly, and enforced redundantly — client-asserted tenancy is not tenancy (`../03-auth/authorization.md` §5).

## 2. Isolation Model Selection

| Model | Isolation | Cost/ops | Fits |
|---|---|---|---|
| **Shared schema + `tenant_id`** (default) | logical (queries + RLS) | cheapest, one migration path | most B2B SaaS to substantial scale |
| **Schema-per-tenant** | stronger (namespace separation) | N× migration fan-out, connection/catalog overhead | hundreds (not thousands) of tenants with compliance-driven separation needs |
| **Database-per-tenant** | physical | per-tenant infra, backup/restore granularity, noisy-neighbor isolation | enterprise tier with contractual isolation, low tenant counts |
| **Region-per-tenant** | jurisdictional | full duplicated stack | data-residency-bound customers (see `privacy-compliance-audit.md`) |

Decision inputs: tenant count, compliance contracts, noisy-neighbor risk profile, per-tenant restore requirements, ops appetite. **Start shared-schema unless a signed requirement says otherwise** — the other models are escalations, and moving shared→separated is feasible while the reverse is not. Whatever the model: the application-level tenant-context discipline (§3) is identical.

## 3. Tenant Context Discipline (The Application Half)

- **Derive, don't assert:** tenant = attribute of the authenticated identity (claim/session/user record lookup) — never a header/body the client sends (that's self-attestation).
- **Explicit propagation:** request-scoped context object carried through the call stack — into repositories (query scoping), jobs (payload carries tenant), events (attribute on the envelope), and cache keys (`../06-async-and-scale/caching.md` §5: a key missing tenant scope = cross-tenant leak).
- **Query scoping by construction:** every tenant-scoped query includes the tenant predicate; better, repositories take the tenant context as a parameter so an unscoped query is a type error, not a code-review hope (`../03-auth/authorization.md` §4.2).
- **Background jobs re-establish context from payload** (never ambient state); admin/cross-tenant operations ("support views tenant X") are explicit, permissioned, audited, time-bounded (`../08-observability/observability.md` §7).

## 4. The Database Half — RLS as Backstop

On PostgreSQL-class engines, **Row-Level Security policies** keyed to a session tenant setting provide enforcement independent of application correctness:

- Policies: `USING (tenant_id = current_setting('app.tenant_id')::uuid)` — set per request/transaction from the derived context; with Supabase-class stacks, RLS is the recommended default on every exposed table (the policy syntax itself is in the Supabase guide / PostgreSQL docs).
- RLS is **defense-in-depth, not the only layer**: application-level scoping still matters (query plans, index usage — poorly-keyed RLS policies with non-indexed `tenant_id` filters scan; test performance, `../05-data/data-architecture.md` §5 composite indexes lead with tenant).
- Connection pooling interacts with session-based tenant settings (SET per transaction, reset on release) — pooler-aware implementation per the DB guide; this is a classic leak vector when done naively.

## 5. Noisy Neighbors & Fairness

- **Per-tenant rate limits** (not just per-user): one tenant's batch job or runaway integration must not consume everyone's latency budget (`../02-domain-and-api/api-design.md` §8 layered limits).
- **Per-tenant quotas** on storage/jobs/exports — enforced at issuance and reconciled by job (`../06-async-and-scale/file-and-object-storage.md` §6).
- **Per-tenant observability:** metrics/logs labeled by tenant (bounded cardinality: tenant ID, not tenant name) — "which tenant is hurting" must be answerable in seconds during incidents; per-tenant error rates catch tenant-specific data bugs (one tenant's weird import poisoning a shared pipeline).
- Queue fairness: separate queues or weighted scheduling when tenants share worker pools.

## 6. Testing Tenancy (The Only Real Proof)

- **Cross-tenant denial tests as a suite-level invariant:** valid token of tenant A against every object of tenant B → 403/404; list endpoints assert exact tenant scoping by count (`../09-testing/testing-strategy.md` §5.1). One parameterized test walker over endpoints catches entire classes of leaks.
- **Cache-key scoping tests:** tenant A populates, tenant B reads same logical key → miss/different data.
- **Job/event isolation:** tenant-scoped payloads processed under the right context, always.
- Migration fan-out tests (schema/database-per-tenant models): the migration chain must apply across tenant N in CI, not just the template.

## 7. Failure Modes

- **Self-attested tenancy:** `tenant_id` accepted from request/header because the frontend "always sets it" — full isolation bypass for anyone with curl.
- **The unscoped query:** one report query missing the tenant predicate returns the whole table; application-only enforcement fails exactly once, catastrophically — hence RLS backstop (§4).
- **Ambient-context bugs in async paths:** job reads tenant from a thread-local that doesn't exist in the worker → defaults to null/global → cross-tenant writes. Context lives in payloads, re-established at execution (§3).
- **Cache-key omission:** tenant-scoped data cached without the tenant dimension — user B gets user A's response from a "shared" cache; highest-frequency real-world SaaS leak (`caching.md` §7).
- **Pooled-session leakage:** RLS session setting surviving connection reuse — tenant A's setting serves tenant B's next query; reset discipline + tests.
- **Fairness blindness:** shared metrics look healthy (aggregate p95 fine) while one tenant burns in the tail — per-tenant slicing exists for this (§5).
