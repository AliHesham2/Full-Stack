# Background Jobs — Async Work Done Right

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Any work that shouldn't block the HTTP response: emails, exports, integrations, scheduled tasks, retries, cleanup — or choosing/building a job system.
> **Related:** `../05-data/concurrency-and-idempotency.md`, `events-and-messaging.md`, `../08-observability/observability.md`
> **Status:** Stable principles · Queue mechanics (BullMQ-class libraries, SQL-based queues, broker choice) in the queue guide

---

## 1. When Work Goes Background (and When It Doesn't)

Move work to a background job when ANY of:

- It takes longer than ~1s (user-visible latency budget)
- It must be **retried** on failure (external APIs, email delivery)
- Its result isn't needed in the response ("fire and forget" side effects)
- It runs on a schedule (digests, sweeps, reconciliations)
- It must be rate-shaped (batch exports, third-party quota respect)

Keep work synchronous when: the caller needs the outcome to respond (validation of business rules, reads), the operation is fast and reliable, or the "async" version exists only to dodge a timeout that indicates a real performance bug. Don't background what should be fixed (`../01-foundation/complexity-control.md` — a queue is not a load-shedding bandage for an N+1).

## 2. Job Design Rules

1. **Idempotent by contract.** At-least-once delivery is the reality (worker crash mid-job ⇒ redelivery). Every job body must be safe to run twice: natural-key upserts, state-machine gates, dedup records (`../05-data/concurrency-and-idempotency.md` §3).
2. **Small and focused.** One job = one unit of work with one retry policy. A "process order" mega-job that emails, charges, and updates inventory has three different failure domains stitched into one retry loop — split it; chain with per-step jobs if sequencing matters.
3. **Self-contained payloads.** The job carries **references (IDs), not hydrated state**: fetch fresh data at execution time — the world changed since enqueue; serialized stale objects in payloads create time-travel bugs and bloat storage.
4. **Explicit retry policy per class:** max attempts, backoff (exponential + jitter — `../07-reliability/error-handling.md` §5), and which errors are retryable at all (validation failures aren't; transient 5xx/timeout are).
5. **Visible and bounded.** Every job type has: named queue, timeout, concurrency limit, and an owner. Anonymous cron-shaped "misc" queues are where incidents hide.

## 3. The Job Lifecycle (Design Each Stage)

```
enqueue (after commit!) → pick up → execute → succeed | fail → retry → … → dead-letter → alert & repair
```

- **Enqueue after commit** (the cardinal rule): a job enqueued inside a transaction that rolls back still fires (the queue isn't in your transaction) — emailing for a rolled-back order. Enqueue post-commit; if write+enqueue must be atomic, use the transactional outbox (`events-and-messaging.md` § outbox).
- **Execution:** acquire job (visibility timeout > expected runtime — jobs reappear if the worker dies mid-run: another reason for idempotency), heartbeat/progress for long jobs.
- **Long-running work:** chunk it (process 500 records per job, enqueue continuation) or checkpoint progress — never one 4-hour job whose crash restarts from zero.
- **Dead-letter queue (DLQ):** after max retries, park the job with full context and ALERT. The DLQ is an operational surface with a runbook (`../10-delivery/documentation-and-decisions.md`): inspect → fix cause → replay. An unmonitored DLQ is a silent data-loss bin.
- **Scheduling:** cron-style triggers are jobs too — idempotent (overlapping runs after worker restart), distributed-safe (lock/leader election so two instances don't double-fire), and jittered (all tenants' digests at 09:00:00 sharp = self-DDoS).

## 4. Build vs Buy

| Option | Fits | Doesn't fit |
|---|---|---|
| **Mature library over DB/Redis** (BullMQ-class) | Single-team products; the default | Needs multi-region, strict ordering, huge scale |
| **SQL-based jobs** (`FOR UPDATE SKIP LOCKED`) | Tiny scale, no infra appetite, low rate | High throughput (lock churn/vacuum pressure — `transactions.md` §6), sub-second scheduling |
| **Dedicated broker/platform** | Multiple services, heavy volume, strict SLAs | One service with modest volume — operational tax |

Rules: never hand-roll a queue over a `jobs` table with `SELECT ... WHERE status='pending'` polling races (use the SQL-queue pattern properly or a library); never accept at-most-once semantics for jobs with side effects.

## 5. Observability (Jobs Are Invisible Users)

A user sees the email fail. A job fails silently at 3AM. Minimum surfaces (`../08-observability/observability.md`):

- **Queue depth & age** (oldest pending job) — age is the early-warning signal
- **Per-queue: throughput, failure rate, retry count, DLQ count** (alerts on DLQ > 0 and age > threshold)
- **Per-job logs with the same correlation ID** as the originating request (propagate the ID into the payload at enqueue)
- Job-level tracing: enqueue → start → end, so "where is my email" queries are one trace lookup

## 6. Failure Modes

- **Enqueue-inside-transaction:** rolled-back writes fire their jobs (§3 cardinal rule). Symptom: emails about orders that don't exist.
- **The mega-job:** one body, five responsibilities, retry re-does everything each failure — partial progress thrash. Split per failure domain.
- **Payload time capsules:** enqueued job carries full serialized user/order state; world changed by execution time; job writes stale data back. Pass IDs.
- **Retry-blind handlers:** a 4xx "invalid recipient" retried 8 times with exponential backoff — classify errors, fail fast non-retryables.
- **The unbounded worker:** one job type blocks the shared queue (concurrency 1 for everything behind a slow exporter). Separate queues per latency/importance class: `critical` / `default` / `bulk`.
- **DLQ abyss:** max retries → vanish. No alert, no runbook, no replay. Data loss with a clean dashboard.
- **Schedule pile-up:** the 5-minute cron can't keep up; instances overlap; the system does the same work twice concurrently at growing lag — idempotency + overlap locks + alert on age.
