# Data Access — Repositories, Queries, and the ORM Question

> **Part of:** Data — Backend Project Guide
> **Read when:** Writing any query or repository, choosing ORM vs raw SQL, diagnosing N+1 / slow queries, or managing connections.
> **Related:** `data-architecture.md`, `transactions.md`, `../06-async-and-scale/performance.md`
> **Status:** Stable principles · ORM specifics live in the ecosystem/ORM guides

---

## 1. The Repository Boundary

A repository is the module's **storage contract**: an interface expressed in domain terms (`findOpenOrdersForTenant(tenantId, cursor)`) implemented against the database. Its jobs:

1. Contain every SQL/query for the tables the module owns
2. Speak in domain concepts (in/out: entities or DTOs — never raw driver rows)
3. Be fakeable in tests (`../09-testing/testing-strategy.md`)

What it is NOT: a generic `BaseRepository<T>` with `findBy(field, op, value)` combinators — that's a type-unsafe query builder cosplaying as a boundary. Repositories with intention-named methods document the module's actual data needs; generic ones hide them and invite ad-hoc query sprawl.

## 2. ORM vs Raw SQL (Decision, Not Religion)

| | Use when | Watch for |
|---|---|---|
| **ORM/query builder** | CRUD-shaped access, boring mappings, migrations tooling | Lazy-loading N+1, leaky abstractions, surprising SQL, relation-modeling everything |
| **Raw/parameterized SQL** | Complex reads (reports, analytics), performance-critical paths, bulk ops, recursive/set operations | Parameterization discipline (§3), duplication of mapping code |
| **Mix (normal)** | ORM for 80% routine, raw SQL for the hard 20% | Two divergent migration/config paths — keep one schema source of truth |

Rules:

- ORMs optimize developer velocity for standard access; they are not a correctness or security mechanism. Injection safety comes from parameterization regardless of tool.
- An ORM row is a **storage representation**, not your domain entity — map at the boundary if the domain model is non-trivial; conflating them welds your domain to your schema (see `../02-domain-and-api/domain-logic.md` §4).
- Escape hatches are a feature: when the ORM fights you on a query the DB could express in 10 elegant lines, drop to parameterized SQL at the repository layer with a comment explaining why.

## 3. Query Correctness Rules

- **Parameterize, always** (`../04-security/security-foundations.md` §3): placeholders for values, allowlists for identifiers (column/table names can't be bound — map `sort=created_at` through a constant object).
- **Explicit column lists** in SELECTs that leave the repository: `SELECT *` in shared code is schema-coupling and a leak surface (new sensitive column silently flows into API responses). `SELECT *` inside tightly-mapped repo internals is tolerable; anywhere else, list columns.
- **Always bounded:** every list query has LIMIT (pagination is mandatory at the query layer — `../02-domain-and-api/api-design.md` §5); every statement has a statement timeout (§5).
- Deterministic ordering for anything paginated (tiebreaker column) — without it, page boundaries shuffle rows between requests.
- Tenant/owner scoping in the query itself, not post-filtering memory (`../03-auth/authorization.md` §4): `WHERE id = $1 AND tenant_id = $2` cannot forget to authorize.

## 4. The N+1 Family

Fetching a list, then per-item fetching a related record (often invisibly via ORM lazy relations). Detection: query logs/counters in dev & tests (a request doing 51 queries for 50 items is the signature), EXPLAIN on suspects.

Fixes in order of preference:

1. Join / single query with the right shape
2. Batched IN-queries (`WHERE order_id IN ($1...$50)`) + in-memory stitch
3. Eager-loading directive (ORM's batched load), verify it actually batches
4. Data-access restructure (denormalize only as last resort and per `data-architecture.md` §4)

The same disease has variants: N+1 on writes (loop of single INSERTs → batch), N+1 on external APIs (loop of per-item HTTP calls → batch endpoint or parallel bounded concurrency).

## 5. Connections & Pools

- Pool with explicit, sized budget: roughly `connections_available_to_DB / instances`, leaving headroom for migrations and ops. Oversized pools on autoscaled fleets exhaust DB connections before CPU — a classic outage shape.
- **Statement timeouts** (e.g., 5–30s by query class) and **idle-in-transaction timeouts** at the pool/driver level — a forgotten transaction holds locks and vacuum backlogs.
- Acquire late, release early: no business logic between pool acquire and query dispatch; never hold a connection across an external HTTP call or an `await` of user input.
- Health checks use a separate lightweight probe, not a full query against a hot table (`../07-reliability/reliability.md` §2).
- Serverless/edge runtimes: connection pools don't survive process churn — use HTTP-transaction proxies or platform poolers (per platform guide).

### 5.1 Pool Sizing Heuristic

The naive formula `pool_size = CPU_cores × 2 + disk_spindles` is a starting point, not a rule. The actual constraint is: **total connections across all instances ≤ DB max_connections minus headroom for migrations/ops/monitoring**.

Practical sizing process:
1. **Know the DB's connection limit** (PostgreSQL default: 100; cloud: varies by tier)
2. **Subtract operational headroom** (~10–20 connections for migrations, monitoring, psql access)
3. **Divide by expected instance count** — this is your per-instance pool max
4. **Measure before tuning:** connection wait time (should be near zero; if >5ms average, pool is too small or instances are too many); active connections vs idle (a pool that's always full is undersized; a pool that's always idle is oversized)
5. **Set acquisition timeout** (e.g., 5s) — a request waiting >5s for a connection is already a degraded experience; fail fast with 503

**Warning signs:**
- `connection_timeout` errors under normal load → pool too small OR instance count grew without pool adjustment
- DB shows high `idle` connections but app reports timeouts → pool too large (holding connections you don't use, starving other services)
- Cloud-managed pools (PgBouncer, Supabase pooler): session-mode vs transaction-mode pooling interacts with per-session settings (RLS, `SET` commands) — know which mode you're in

## 6. Transactions & Repositories (Interaction)

- Repositories expose **transaction-aware operations**; the use case owns the boundary: `repo.transaction(async tx => { ... })` — see `transactions.md` for what may live inside.
- Never hide transactions inside repository methods (silent nested/savepoint surprises); the orchestration layer decides atomicity — it's a business decision, not a data-access detail.
- One use case = one transaction scope as the default; multiple scopes need explicit reconciliation design (`transactions.md` §7).

## 7. Testing Implications

- Integration tests run repositories against a **real database engine** (containers) — mocking the ORM proves nothing about actual SQL correctness (`../09-testing/testing-strategy.md` § DB tests).
- Repo test coverage: each intention-named method, each filter/sort branch, constraint violations surface as typed errors.
- Query-count assertions on hot endpoints (fail the test if an endpoint exceeds its query budget) — the CI answer to N+1 regressions.

## 8. Failure Modes

- **Leaky repository:** handlers receive driver rows and read `.rows[0].user_id` — the boundary exists in folders but not in types.
- **The 5,000-line `queries.ts`:** one file accumulating every module's SQL; merge-conflict magnet, ownership blur — split along module lines.
- **ORM-relation everything:** every FK modeled as a lazy relation, every list endpoint triggers lazy cascades; the data model becomes an object graph with accidental I/O. Model relations you actually traverse.
- **Post-filter authorization:** fetch full table page → filter by tenant in app code → paginate the remainder: correct-looking, wrong (rows leak into logs/errors, unbounded reads, offset math breaks). Scope in the query.
- **Connection leakage under error paths:** early returns between acquire and release without `finally`/scope semantics — pool exhaustion only under production error rates (test the error paths).
