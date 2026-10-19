# Transactions — Atomicity, Isolation, and Locking

> **Part of:** Data — Backend Project Guide
> **Read when:** Any multi-step write, anything that must not tear, or when choosing locking strategies and isolation levels.
> **Related:** `concurrency-and-idempotency.md` (read together), `data-access.md` §6, `../02-domain-and-api/domain-logic.md` §6
> **Status:** Stable principles · Isolation mechanics: PostgreSQL-flavored defaults (MVCC, read committed default) — verify per engine

---

## 1. What Transactions Are For

A transaction makes a group of writes **atomic** (all or nothing), **consistent** (constraints enforced at commit), and **isolated** (not interleaved catastrophically with concurrent ones). The decision rule is simple in shape:

> If a crash between two writes leaves the system in a state a human would call "wrong" (or a business rule would reject), those writes belong in one transaction.

Classic pairs that must be atomic: debit + credit; insert order + decrement stock; update state + write audit record. Classic pairs that must NOT be in one transaction: anything + an external API call; anything + sending email (§7).

## 2. Transaction Discipline

- **Keep them short.** A transaction holds locks and pins resources from BEGIN to COMMIT. No external HTTP calls, no email sends, no long computation, no `await` on user action inside.
- **One transaction boundary per use case**, opened by the use case (`data-access.md` §6), sized to the atomicity requirement — not "wrap the whole request just in case."
- **Retries for serialization failures** belong at the use-case level (bounded retries with backoff — see `concurrency-and-idempotency.md` §6), not silently inside repos.
- **Idempotent-ish rollback:** a rolled-back transaction leaves no partial rows, but side effects already emitted (logs, pre-transaction external calls) don't roll back — sequence side effects after commit (`../02-domain-and-api/domain-logic.md` §6).
- Set `lock_timeout`/`statement_timeout` per transaction class (§5) — a wedged lock is an outage; timeouts convert it into a retryable error.

## 3. Isolation Levels (Pragmatic Guide)

Default = READ COMMITTED (Postgres default): statements see committed data as of statement start. It prevents dirty reads; it does NOT prevent:

- **Non-repeatable reads** (same query, different rows across statements in one txn)
- **Phantoms** (new matching rows appear mid-transaction)
- **Lost updates** (read-modify-write races — §4)

| Level | When to reach for it | Cost |
|---|---|---|
| READ COMMITTED (default) | Everything routine | Baseline |
| REPEATABLE READ / SNAPSHOT | Multi-statement read consistency for report-style reads | More conflicts (serialization failures to retry) |
| SERIALIZABLE | Invariants spanning multiple rows/tables that can't be expressed with locks or constraints (e.g., complex availability rules) | Retry-mandatory design; conflict rates rise under contention |

Rule: **prefer explicit row locks (§4) or constraint-based defense (`concurrency-and-idempotency.md` §4) over blanket isolation upgrades.** SERIALIZABLE is correct but its retry storms under contention turn into latency incidents; targeted locking is usually clearer.

## 4. Locking Decisions

### Optimistic (version/condition-based) — default for user-facing flows

```
UPDATE orders SET status='shipped', version=version+1
WHERE id=$1 AND version=$2;      -- 0 rows updated ⇒ concurrent modification
```

- Use when: conflicts are rare, retry-by-user is acceptable, held connections are precious.
- Mechanics: `version` column (or `updated_at` with caveat — clock skew; version is safer), 0-rows-updated → map to 409 conflict (`../02-domain-and-api/api-design.md` §6) or re-read-and-retry in code.
- Powers the API-level `If-Match`/ETag pattern for dangerous operations.

### Pessimistic (row locks) — for hot invariants

```
SELECT ... FROM accounts WHERE id=$1 FOR UPDATE;   -- then read-check-write in txn
```

- Use when: the invariant genuinely spans rows (transfer: lock both accounts, ordered), conflict rate makes optimistic retry loops wasteful, or the operation is machine-driven (job) rather than user-retryable.
- **Always lock in consistent global order** (e.g., by ascending ID) — deadlock prevention is ordering discipline, not luck.
- `FOR UPDATE NOWAIT` or `lock_timeout` when queueing behind a stranger's lock is worse than failing fast.

Anti-rules: no `SELECT COUNT(*)` guards as locking substitutes (check-then-act race — `concurrency-and-idempotency.md` §3); no lock-and-call-HTTP (§1); no table locks where row locks suffice.

## 5. Timeouts & Failure Semantics

- `lock_timeout` (how long to wait for a lock — seconds, not minutes) and `statement_timeout` (per-statement bound) per connection/transaction class; a wedged lock is an outage; timeouts convert it into a retryable error.
- Serialization/deadlock errors (`40001`/`40P01` class): retry with backoff + jitter at the use-case level, bounded (≤3 attempts typical), then surface as 409/503. Unbounded retry loops are self-DDoS.
- Connection dropped mid-transaction: the DB rolls back — your job is to ensure the client knows (retryability story: idempotency keys, `concurrency-and-idempotency.md`).

## 6. Write Hotspots & Contention

- **Counter/summary rows** (balances, stock, seat counts) become lock queues under load: batch updates, move contention to async recomputation, or shard the counter — each with documented consistency windows (`data-architecture.md` §4).
- Long transactions + autovacuum = bloat feedback loops: monitor `idle_in_transaction` age; alert on it (`../08-observability/observability.md` §5).
- Queue-implementations-in-SQL (`FOR UPDATE SKIP LOCKED`) are legitimate for modest job queues — and a trap at scale (lock churn, vacuum pressure); know when to graduate to a real queue (`../06-async-and-scale/background-jobs.md` § build-vs-buy).

## 7. Distributed & Multi-System Transactions (The Honest Answer)

Two systems (DB + external API, two services) cannot share an ACID transaction. The options, in escalating complexity:

1. **Sequential with reconciliation (default):** write pending state → call external → write outcome → sweep stale pendings with a job. Boring, observable, self-healing. (`../02-domain-and-api/domain-logic.md` §6 pattern.)
2. **Transactional outbox:** event/write atomicity via an outbox table + relay (`../06-async-and-scale/events-and-messaging.md` § outbox). For "DB write + event publish" specifically.
3. **Saga (explicit compensations):** long-lived multi-service workflows with defined compensation per step. Only when multiple services genuinely own steps — and note this is a distributed-monolith detector: if all saga steps are YOUR services, reconsider the boundary (`../01-foundation/architecture.md` §9).

2-phase commit is not on this list for a reason: its operational failure modes (coordinator outages, heuristic decisions) cost more than the alternatives in almost every greenfield context.

## 8. Testing

- Concurrency tests for invariants that matter: two parallel transfers from one balance, double-submit of a state transition, concurrent unique-insert — assert final state exactly (`../09-testing/testing-strategy.md` § concurrency).
- Failure-injection: transaction rolled back ⇒ no partial writes visible; lock timeout ⇒ clean 409/503, no leaked connection.
- Test deadlocks never happen on your canonical lock-ordering paths (parallel test hammering).

## 9. Failure Modes

- **The giant transaction:** "begin" at request start, "commit" at request end, with email + payment inside — latency multiplies, lock queues form, rollbacks undo nothing external. Shrink to the atomic core.
- **Autocommit obliviousness:** multiple sequential statements assumed atomic without a transaction — a crash between them produces half-states (the most common AI-generated data bug).
- **Check-then-act everywhere:** `if (stock >= qty) update` — racy by construction; make the check atomic with the write (`WHERE stock >= $qty`).
- **Silent swallow of serialization failures** inside generic error handlers → user sees 500 for a condition the code should have retried or 409'd.
- **Lock-order roulette:** paths locking A→B and B→A — deadlocks at exactly the traffic level where you have no spare capacity to absorb retries.
