# Data Architecture — Ownership, Schema, and Lifecycle

> **Part of:** Data — Backend Project Guide
> **Read when:** Designing tables/keys/indexes, choosing a database model, deciding normalization/denormalization, or defining data lifecycle (deletion, archival, audit).
> **Related:** `data-access.md`, `transactions.md`, `../01-foundation/architecture.md` § boundaries, `../11-specialized-concerns/multi-tenancy.md`
> **Status:** Stable principles · Postgres-flavored where examples appear (18.x current: native uuidv7, virtual generated columns — verify per engine)

---

## 1. Position

Data outlives code. Schema decisions are the hardest to reverse (`../01-foundation/complexity-control.md` §5 reversibility ladder) and deserve more deliberation than any class hierarchy. This file covers how to *reason* about data; mechanics of queries live in `data-access.md`, of atomicity in `transactions.md`.

## 2. Data Ownership

- Every table has **exactly one owning module** (`../01-foundation/architecture.md` §4.2). Other modules read via the owner's contract, or (documented exception) via a read-only view — never write to another module's tables.
- Cross-module foreign keys: allowed for **integrity**, but the referencing module must not assume the referenced internals — treat the FK as an opaque ID owned elsewhere.
- One schema-change touchpoint per rule: if changing "orders" requires touching 4 modules' code and 3 other tables' triggers, ownership boundaries drifted.

## 3. Choosing a Data Model

**Default: relational (PostgreSQL-class).** It gives you transactions, constraints, and a 40-year tooling ecosystem — the boring, correct choice for the overwhelming majority of business systems. Justify deviations with access patterns, not fashion:

| Need | Alternative | Trade you're making |
|---|---|---|
| Flexible/sparse documents, single-document access | Document store | Cross-document transactions/joins get harder; constraint enforcement moves to app |
| Massive write throughput, append-only, key-only reads | Wide-column / LSM stores | Query flexibility drops; compaction/ops burden |
| Simple cache-shaped access | KV store (see `../06-async-and-scale/caching.md`) | Not a system of record — durability/consistency limited |
| Full-text/vector search alongside relational | Dedicated engine OR Postgres extensions | Extra system + sync problem; often Postgres built-ins suffice at real-world scale — measure first |
| Time-series at scale | TSDB | Only at genuine telemetry volumes |

Anti-rule: never choose "we'll use all of them" polyglot persistence for a CRUD app. Each additional store is a consistency problem, an operational surface, and a migration tax (`../01-foundation/complexity-control.md`).

## 4. Schema Design Rules

**Keys:**

- Surrogate primary keys by default; UUIDv7/ULID-style time-ordered IDs for anything exposed
  or index-hot. **Postgres 18 has a native `uuidv7()` function** (verified against the
  PostgreSQL 18 docs). Sequential integers only for internal, non-enumerable tables.

  *On the index-locality benefit:* time-ordered IDs insert near the right edge of the B-tree
  instead of scattering across it, so pages stay denser and write amplification drops. The
  effect is real and well documented; the exact magnitude depends on row width, fill factor and
  workload, so **benchmark your own table rather than quoting a percentage** — an earlier
  revision cited a specific figure this guide could not source.
- Natural keys as UNIQUE constraints alongside, not instead.
- Every FK that expresses a real relationship gets a constraint — "we enforce it in code" means "we don't enforce it."

### 4.1 ID Generation Strategy (Decision Framework)

| ID Type | Use when | Avoid when | Notes |
|---|---|---|---|
| **UUIDv7 / ULID** | Public APIs, distributed systems, anything exposed to clients or cross-service | Internal-only tables where sequential is simpler | Time-ordered: index-friendly, sortable, no coordination. The default for new projects in 2026. Postgres 18+ has native `uuidv7()`. |
| **Sequential integer (DB sequence)** | Internal tables, foreign keys between internal tables, counters | Anything exposed in URLs/APIs (enumeration attack surface) | Fastest index performance, smallest storage, but leaks creation order and count. |
| **Natural key** | Business-unique attributes (email, SKU, ISBN) as UNIQUE constraints | As primary key when the attribute might change | Changes to natural keys cascade through FKs; use as a secondary unique, not the PK. |
| **Composite key** | Junction tables where the combination IS the identity | When one column alone is sufficient | Rare in modern ORMs; prefer a surrogate PK + unique constraint on the natural combination. |

**Anti-rules:**
- Random UUIDv4 as PK on write-hot tables at scale: index locality destroys write performance (random inserts → page splits). Use UUIDv7.
- Sequential IDs in public APIs: enumerable, leaks count and creation rate. Use UUIDv7/ULID.
- "We'll use nanoid/ksuid/custom": every custom ID format is a maintenance burden and a future compatibility risk. UUIDv7 is the standard; deviate only with a written reason.

**Normalization strategy:**

- Start normalized (3NF-ish): one fact in one place. Normalization is a *write-correctness and change-safety* strategy, not an academic exercise.
- Denormalize **deliberately and measured**, for read performance or for aggregate consistency (running totals, materialized summaries): name the invariant the denormalized value maintains, define the update path for every write that affects it, and accept the consistency window. Denormalized-but-unmaintained fields are corrupt-by-design.
- Counters/totals: either maintain transactionally in the same row update (simple, contended) or recompute asynchronously (eventual — document it).

**Constraints are the last line of defense** (defense-in-depth, `../02-domain-and-api/validation.md` §5):

- `NOT NULL` by default — nullable is a design decision (does this absence carry meaning?)
- `UNIQUE` for business identity (email per tenant! — composite uniques, not just global ones)
- `CHECK` for enumerable domains and simple invariants (`qty > 0`)
- `ENUM` types or lookup tables for status vocabularies (see `migrations.md` for the evolution cost of enums)
- FK `ON DELETE` behavior chosen **deliberately**: RESTRICT by default (catches accidental orphaning), CASCADE only where child rows are genuinely owned artifacts (order → line items), never CASCADE across module boundaries.

**Multi-tenancy** (`../11-specialized-concerns/multi-tenancy.md`): `tenant_id` on every tenant-scoped table, composite indexes leading with it, RLS as the backstop.

## 5. Indexes

- Indexes serve **queries you have**, not tables: derive from the actual query set (slow log, EXPLAIN — `data-access.md` §5), not from column-importment guessing.
- Composite index column order: equality predicates first, then range/sort. A query filtering `tenant_id = ? AND status = ? ORDER BY created_at DESC` wants `(tenant_id, status, created_at)`.
- Partial indexes for hot subsets (`WHERE status = 'pending'` — small, fast, cheap to maintain).
- Covering/include columns when a small addition removes a table fetch.
- Every index slows writes and consumes cache: an index with no query served is pure liability. Audit `pg_stat_user_indexes`-equivalents for dead weight periodically.
- Unique indexes double as race-condition armor (`concurrency-and-idempotency.md` §4) — sometimes you create them for correctness, not speed.

## 6. Data Lifecycle

| Question | Default | When to deviate |
|---|---|---|
| Hard vs soft delete | Hard delete for true owned artifacts; **soft delete (`deleted_at`) only where audit/undo is a real requirement** | Soft delete adds a WHERE-clause tax on every read forever — adopt per table, not globally |
| Audit trail | Append-only event/audit records for state changes on regulated or business-critical entities (`../11-specialized-concerns/privacy-compliance-audit.md`) | Not every table needs audit — decide by blast radius |
| Archival | Define retention when data grows monotonically (logs, events, sessions); partition old data out | Ignoring this turns "slowest table" into an outage eventually |
| PII | Inventory + deletion story (GDPR-class rights — `../11-specialized-concerns/privacy-compliance-audit.md`) | — |
| Backups | Tested restores, not backup files (`../11-specialized-concerns/disaster-recovery.md`) | — |

Timestamps: `created_at`/`updated_at` on every table (UTC, DB-generated — `../11-specialized-concerns/time-and-money.md`); schema-level trigger or ORM discipline, but the DB is the authority.

## 7. Consistency Models (Orientation)

- **Strong (default):** reads-your-writes within transactions; what the relational model gives you. Choose it unless forced otherwise; most "we need eventual consistency" claims dissolve under questioning (`transactions.md`, `../06-async-and-scale/events-and-messaging.md` § when-not).
- **Read replicas:** same data, eventual replication lag — you gain read scale and lose read-your-write for sessions routed differently. Route: sticky sessions or leader-reads-after-write when freshness matters.
- **Eventual (cross-service/cross-system):** cache layers, event-driven projections, search indexes — accept staleness explicitly, bound it (TTLs, reconciliation jobs — see `../06-async-and-scale/caching.md` § invalidation and `events-and-messaging.md` § consumers).

Rule: name the consistency contract per data flow in the design doc. "It should be consistent" is not a contract; "replica lag ≤ 5s tolerated for listing pages, leader reads for post-write confirmation" is.

## 8. Read/Write Pattern Analysis (Do This Before Schema)

Before finalizing any non-trivial schema, write down:

1. The 5 most frequent queries (with filters + sorts) — these drive indexes
2. The write hotspots (which rows update at high rate — contention risk, `transactions.md` §6)
3. Growth rates per table (which tables need partitioning/archival planning)
4. Access ratios per query (read:write) — these drive caching decisions later
5. Cross-entity access patterns (which joins happen constantly — candidates for denormalization, carefully)

A schema designed without its query set is a guess with columns.

## 9. Failure Modes

- **Constraint-free schema:** nullable-everything, no FKs, no uniques — "the ORM validates" — until a bug, race, or manual migration script violates every invariant silently. The DB is the last defender; never send it home.
- **Premature denormalization:** duplicating fields "for performance" before any measurement — consistency bugs acquired, milliseconds not gained.
- **Soft-delete everywhere:** every query filters `deleted_at IS NULL`, one forgotten filter = zombie data leak; unique constraints fight soft-deleted rows (partial unique indexes solve it — know the mechanics).
- **Enum sprawl:** status enums accreting 14 values with no state machine behind them — model transitions explicitly in the domain (`../02-domain-and-api/domain-logic.md`).
- **God table:** `items` or `events` absorbing every half-related concept; 90 columns, 12 nullable groups — split by ownership when two change-cycles diverge.
- **ID entropy worship:** random UUIDv4 PKs on write-hot tables at scale (index locality — §4) or, opposite sin, sequential IDs exposed in public APIs (enumeration — `../02-domain-and-api/api-design.md` §2).
