# Performance Engineering — Measure, Then Optimize

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Anything feels slow, before adding a cache, before "optimizing", when capacity planning, or when a latency budget needs setting.
> **Related:** `caching.md` (the decision AFTER measurement), `../05-data/data-access.md`, `../08-observability/observability.md`
> **Status:** Stable methodology · Avoids cargo-cult optimization by design

---

## 1. The Method (Non-Negotiable Order)

```
1. MEASURE       — where does time actually go? (tracing, EXPLAIN, profiling)
2. IDENTIFY      — name the dominant cost (usually: the N+1, the lock, the payload, the external call)
3. FIX THE CAUSE — usually a query, an algorithm, a boundary — NOT infrastructure
4. VERIFY        — re-measure against the same benchmark; no before/after numbers = no claim
```

Optimizing without measurement produces confident changes with random effects — and occasionally real regressions nobody can prove. "It feels faster" is not a measurement.

## 2. Latency Budgets

- Set an explicit budget per endpoint class before optimizing: e.g., p50 ≤ 100ms / p95 ≤ 500ms / p99 ≤ 1s for interactive reads; different budget for reports/exports.
- **Percentiles, never averages** — averages hide the tail, and the tail is where users and timeouts live (`../08-observability/observability.md` §4).
- Decompose the budget across the request path: network → edge → auth → query(s) → serialization. A budget makes "which component is over its share" a measurable question instead of a debate.
- SLO reality: p99 of a service feeding 100 sequential internal calls = the 1%-bad path amplified into near-certainty per user journey. Fan-out latency math is a design input, not a surprise.

## 3. The Usual Suspects (In Frequency Order)

1. **N+1 queries** (`../05-data/data-access.md` §4) — the default disease of ORM-shaped code
2. **Missing/wrong indexes** for the actual query set (`../05-data/data-architecture.md` §5)
3. **Unbounded payload/query** — SELECT without LIMIT, 5MB JSON responses, fetching all columns for a 3-field DTO
4. **Synchronous external calls** in the request path (payment, email, third-party APIs) — move out-of-path or parallelize with bounded concurrency (`background-jobs.md`)
5. **Serialization costs** — giant payloads built and parsed per request; over-fetching from the DB for fields nobody reads
6. **Connection/pool contention** — waits invisible in code, obvious in pool metrics (`../05-data/data-access.md` §5)
7. **Lock contention / hot rows** (`../05-data/transactions.md` §6)
8. **Chatty service-to-service** — 20 sequential internal HTTP calls per request (the distributed-monolith tax)
9. CPU-bound hot paths doing per-request what should be cached/precomputed — AFTER measurement says so (`caching.md` §1)

Memory/I/O on the radar: leak-shaped growth (unbounded in-process caches, listeners never removed), GC pressure from per-request allocation storms, sync I/O on async runtimes (event-loop blocking — see the Node guide for the runtime mechanics).

## 4. Profiling Toolkit (What to Actually Do)

The order matters — each layer narrows the search space:

**Layer 1: Distributed trace (production)**
One slow request's trace shows the span that ate the budget — 80% of backend latency diagnosis ends here. Find the slow span, then go to Layer 2 for that span.

**Layer 2: Database query analysis**
`EXPLAIN (ANALYZE, BUFFERS)` on the suspect query. Look for: sequential scans on large tables (missing index), nested loops with high row estimates (bad join order), sorts above work_mem (disk spill). Slow-query log with a threshold (e.g., >200ms) catches the candidates; `pg_stat_user_indexes` reveals dead-weight indexes and missing ones.

**Layer 3: Application-level profiling**
Async-profiler-class sampling in staging/load tests shows where CPU time actually goes (not where you assume). Allocation profiling for GC pressure symptoms (high allocation rate → frequent GC → latency spikes). Lock-contention profiling for concurrency bottlenecks. In Node.js: `--prof` or `clinic.js` for event-loop delay, CPU flame graphs.

**Layer 4: Load testing for capacity claims**
Realistic traffic shape (read:write ratios, payload sizes, endpoint mix — not `ab` on /health 10k times). Measure saturation points (the RPS where p95 doubles), not just throughput. Ramp gradually; observe the knee. Extrapolation from small datasets lies — test at production scale or accept the uncertainty explicitly.

**Key profiling anti-patterns:**
- Profiling in dev with 100 rows → misleading results; use production-shaped data
- Optimizing before profiling → you fix the wrong thing
- Profiling without a budget → no way to judge "is this fast enough"
- Only measuring p50 → the tail (p95/p99) is where users and timeouts live

## 5. Optimization Decision Table

| Diagnosis | Fix | NOT the fix |
|---|---|---|
| N+1 | join/batch the queries | cache the result (hides the bug, adds staleness) |
| missing index | add the right composite index | read replicas (wrong axis) |
| unbounded query | LIMIT + pagination (`../02-domain-and-api/api-design.md` §5) | "more RAM" |
| slow external call in path | async job / parallel bounded / pre-warm | retrying it more |
| huge payloads | trim DTOs, sparse fields, gzip/brotli | proxy upgrade |
| hot-row contention | shard counter / async recompute (`../05-data/transactions.md` §6) | bigger instance |
| CPU hot path | algorithm, then cache the COMPUTED result (with invalidation design — `caching.md`) | micro-optimizing string concat |

Each row's "NOT the fix" column is a real pattern observed in the wild — usually infrastructure purchased to avoid reading an EXPLAIN.

## 6. Performance Without Regressions

- Benchmarks live as repeatable artifacts (scripted load tests + recorded p50/p95/p99) — CI-optional, pre-release mandatory for hot paths.
- Every optimization ships with its before/after measurement and a rollback story (an optimization that complicates the code needs a bigger, proven win).
- Watch correctness interactions: query changes alter consistency windows, caching adds staleness, parallelism adds interleaving (`../05-data/concurrency-and-idempotency.md`). Performance changes are behavior changes.
- Capacity planning: know your saturation point (RPS where p95 doubles) and alert at a safety margin — capacity incidents should be scheduled events (scaling), not discoveries.

## 7. Anti-Patterns

- **Premature optimization:** optimizing code paths without load, evidence, or budget — complexity purchased for milliseconds nobody measured.
- **Cache-instead-of-fix:** `caching.md` §1 violations — the N+1 is still there; now there are two systems to debug and a staleness window.
- **Benchmark theater:** load tests against dev datasets of 100 rows, extrapolated to production claims.
- **The p50 monotheism:** reporting only averages/medians while users experience the tail.
- **Infrastructure as deflection:** "we'll scale out" as the answer to a quadratic algorithm — the algorithm wins eventually, always.
