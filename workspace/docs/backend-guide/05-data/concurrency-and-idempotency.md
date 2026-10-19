# Concurrency & Idempotency — Races, Duplicates, and Safe Retries

> **Part of:** Data — Backend Project Guide
> **Read when:** Any operation that can run twice, in parallel, or interleaved: double-clicks, retries, job re-runs, concurrent edits, webhook redeliveries.
> **Related:** `transactions.md` (read together), `../02-domain-and-api/api-design.md` §6, `../06-async-and-scale/background-jobs.md`
> **Status:** Stable principles · Treated as first-class by this guide: duplication and interleaving are defaults of distributed operation, not edge cases

---

## 1. Position

Requests fail and get retried. Users double-click. Job runners re-execute after worker crashes. Message brokers redeliver. Load balancers replay. **"It will only run once" is the only wrong assumption.** Every mutating operation gets an explicit answer to two questions:

1. What happens if this runs **twice**?
2. What happens if this runs **concurrently with itself**?

## 2. The Race Taxonomy

| Race | Shape | Typical harm |
|---|---|---|
| Check-then-act | `if (stock >= qty) update` non-atomically | oversell; constraint violation later |
| Lost update | read-modify-write without version | silent data loss (last writer wins) |
| Double-submit | user retries/refresh mid-request | duplicate orders/charges |
| Duplicate delivery | webhook/job/event arrives twice | duplicate side effects (emails, rows) |
| Unique-check race | `if (!exists) insert` | duplicate accounts; crash on unique constraint |

## 3. Idempotency — The Core Discipline

An operation is idempotent when **running it N times leaves the same state as running it once**. Engineering options, cheapest first:

### 3.1 Natural idempotency (design for it)
- Absolute writes (`SET status='cancelled'`), not relative (`SET count = count + 1`) — retries safe by construction.
- Model increments as absolute updates where the domain allows ("set position to 3" vs "move by +1").
- UPSERT semantics (`INSERT ... ON CONFLICT DO NOTHING/UPDATE`) for create-if-absent.

### 3.2 Unique constraints as race armor
The database's serialized constraint check is the last word on uniqueness:

```
INSERT INTO orders (id, cart_id, ...) VALUES (...)
ON CONFLICT (cart_id) DO NOTHING;   -- one cart ⇒ at most one order
```

- Choose the **natural deduplication key** per operation (cart ID, external payment ID, `user_id+plan+period` for subscription charges).
- App-level existence checks are UX pre-checks, never the defense (§2 unique-check race).
- Handle the conflict result explicitly: 0 rows ⇒ fetch the existing record and return the SAME response shape as success (that's the idempotent contract).

### 3.3 Idempotency keys (client-declared)
For operations without a natural key (payment intents, order creation from generic payloads):

- Client sends `Idempotency-Key: <uuid>` (generated once per user intention, reused across retries — the Stripe-proven pattern, `../02-domain-and-api/api-design.md` §6).
- Server: `INSERT (key, endpoint, account, request_hash, status, response)` — the unique index on `(key, endpoint, account)` **is** the mutex.
  - Same key + same request hash: in-flight ⇒ wait/return current status; completed ⇒ replay stored response verbatim.
  - Same key + different body ⇒ 422 (key reuse mismatch — likely a client bug).
- TTL the records (24h typical) — this is retry-safety, not infinite memory.
- Keys are per-identity: user A's key never satisfies user B's retry (index includes account).

### 3.4 Idempotent consumers
Every event/webhook/job consumer assumes **at-least-once delivery** and deduplicates: message ID table, natural-key upserts, or state-machine gates ("only transition PENDING→PAID; redelivery of PAID is a no-op"). See `../06-async-and-scale/events-and-messaging.md` § semantics and `../11-specialized-concerns/webhooks.md` § inbound.

## 4. Atomic Check-and-Act Patterns

Replace read-check-write with single-statement forms (in escalating expressiveness):

```sql
-- guard in the write
UPDATE inventory SET qty = qty - $1
WHERE sku = $2 AND qty >= $1;              -- 0 rows ⇒ insufficient

-- unique as mutex
INSERT INTO subscriptions (user_id, plan, period) VALUES (...)
ON CONFLICT (user_id, plan, period) DO NOTHING;

-- optimistic version gate
UPDATE docs SET body=$1, version=version+1
WHERE id=$2 AND version=$3;                -- 0 rows ⇒ conflict (409)
```

When the invariant spans rows that must be READ first (transfer validation), that's `SELECT ... FOR UPDATE` territory — `transactions.md` §4 has the decision tree.

## 5. Distributed Locks (Use Sparingly)

- Real need: cross-resource mutual exclusion (two subsystems must not run maintenance simultaneously; cron leader election).
- Mechanics (per cache/coordination guide): SET-NX-with-TTL + token-checked release (or a lock library implementing fencing — prefer the library).
- **Fencing problem:** locks expire mid-operation; a correct design uses monotonic fencing tokens or re-checks the protected invariant atomically — the lock is an optimization, the constraint/atomic write is the correctness.
- Anti-rule: never use locks to make a non-idempotent operation "safe" — make it idempotent, then the lock becomes an optimization you may not need.

## 6. Retries (The Other Half)

Retrying is how duplicates are BORN — so retry design and idempotency design are one design:

- **Retry only idempotent-safe operations** (idempotent by design, key-protected, or read-only). Retrying a non-idempotent POST without a key is how double-charges happen.
- **Classify errors** (`../07-reliability/error-handling.md` §5): connection errors/429/5xx-with-Retry-After → retry with **exponential backoff + jitter**; 4xx validation/authorization → never retry; timeouts → ambiguous, treat as retryable ONLY with idempotency protection.
- **Bound everything:** max attempts (≤3–5), total retry budget, per-attempt timeout. Infinite retry loops are self-inflicted DDoS (retry storms amplify outages).
- **Concurrency-bounded parallelism** in batch paths: unbounded Promise.all against a rate-limited dependency = its own outage.

## 7. Concurrency Testing

- Parallel hammer tests on the invariants that matter: N concurrent submits of one cart ⇒ exactly one order; concurrent transfers ⇒ balance sums preserved (`../09-testing/testing-strategy.md` § concurrency).
- Assert **final-state exactness**, not "no exceptions" — races produce wrong-but-exception-free states.
- Chaos-lite in CI: inject duplicate deliveries of jobs/events; assert no duplicate side effects.

## 8. Failure Modes

- **"The UI prevents double-clicks":** the UI is not a participant in distributed-systems guarantees (retries, refresh, multiple tabs, curl).
- **Natural-key blindness:** dedup key chosen as a random UUID generated per attempt — unique by construction, dedupes nothing. The key must derive from the *intention* (cart/session/user+period), not the request instance.
- **Idempotency-record race:** two concurrent same-key requests both SELECT (no row), both INSERT — without the unique constraint the "idempotency" layer itself double-executes. The constraint IS the mechanism.
- **State-machine-less consumers:** webhook handler processes by "insert row then email" with no transition gate — redelivery re-emails. Model transitions; make repeated delivery a no-op.
- **Retry-the-500 reflex:** blanket retry middleware on everything including non-idempotent writes — amplified partial failures under exactly the outage conditions retries exist for.
