# Events & Messaging — Decoupled Communication, Honestly Priced

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Considering an event bus / message broker / pub-sub, publishing events from transactions, or designing consumers for redelivery.
> **Related:** `background-jobs.md` (commands vs events), `../05-data/transactions.md` §7, `../01-foundation/complexity-control.md` (read BEFORE introducing a broker)
> **Status:** Stable semantics (delivery-guarantee vocabulary is durable); broker mechanics in the broker guide

---

## 1. Messages, Commands, Events — Get the Vocabulary Right

| | Command | Event |
|---|---|---|
| Semantics | "DO this" — imperative, one handler, sender needs it done | "this HAPPENED" — factual, past tense, any number of listeners |
| Failure | Sender's problem; must know outcome (error, retry) | Publisher doesn't care who consumes; consumers own their failures |
| Naming | `IssueRefund`, `SendEmail` | `OrderShipped`, `InvoicePaid` |
| Coupling | Sender knows receiver's contract | Sender knows only its own facts |

A "request/response over a queue" is a command with extra steps — if you need the answer, call the service (RPC) and save yourself the correlation machinery. Events are for **notification of facts**, not for asking questions.

## 2. Do You Actually Need a Broker? (The Honesty Test)

Introduce event-driven infrastructure ONLY when:

1. Multiple **independent consumers** must react to the same fact (billing + analytics + notifications on `OrderPlaced`), AND
2. They must be decoupled (deploy/availability/scale independently), AND
3. The outbox pattern (§4) is understood as part of the cost.

Anti-justifications that fail the test:

- "Async is modern" / "the tutorial architecture has Kafka" → cargo-cult (`../01-foundation/complexity-control.md`)
- One consumer that could be a direct call → just call it
- Hiding latency of work that could be synchronous → fix the latency
- "We might need it later" → you'll have better information later; add it then

**In-process pub/sub** (module A publishes, modules B/C subscribe within one deployable) delivers the decoupling benefit for a monolith with ZERO broker ops — the standard recommendation inside a modular monolith, provided events cross module boundaries through explicit contracts.

## 3. Delivery Semantics (Know What You Bought)

| Guarantee | Reality | What you must build |
|---|---|---|
| At-most-once | Message may be LOST | Loss-tolerant consumers; usually unacceptable for business facts |
| **At-least-once** (the practical default) | Message may arrive TWICE+ | **Idempotent consumers — mandatory, not optional** |
| Effectively-once | At-least-once + dedup/idempotency making duplicates invisible | That idempotency layer is YOURS to build (dedup table / state gates) |

Consumers are built for at-least-once, always: natural-key upserts, processed-message records, state-machine gates ("PAID ignores redelivery of InvoicePaid") — `../05-data/concurrency-and-idempotency.md` §3.4.

Ordering: most brokers offer per-partition/per-queue ordering only. **Design for unordered consumption** (consumers tolerate any order: `OrderShipped` before `OrderPlaced` after a redelivery/rebalance → state gate rejects it), and only where a true sequence requirement survives analysis, pin partition keys deliberately.

## 4. The Outbox Pattern (Write + Publish, Atomically)

The dual-write problem: `INSERT order` + `publish OrderPlaced` are two systems — either can fail alone (ghost event, or ghost order with no event). Fix:

```
-- same transaction:
INSERT INTO orders ...;
INSERT INTO outbox (id, topic, payload, headers) VALUES (...);
-- commit
```

A relay (poller or CDC/debezium-class) ships outbox rows to the broker, tracking offsets, retrying failures. Guarantees: events fire iff the write committed; at-least-once to the broker (consumers are idempotent anyway — §3). Cost: an outbox table + relay ops. This is the ONLY correct way to publish events from transactional state; "publish after commit, hope nothing crashes between" is a known data-loss design.

## 5. Event Design

- **Facts, named in past tense, business-level:** `OrderShipped` not `orders_table_updated`. Events are part of your public contract — stable, documented, versioned like APIs (`../02-domain-and-api/api-design.md` §9 discipline applies).
- **Payload = stable business facts + IDs** (not internal row dumps — those change schema constantly and leak internals).
- **Evolution is additive** (new optional fields), consumers tolerate unknown fields; breaking changes = new event name/version + a migration plan. Tolerant-reader discipline on the consumer side.
- **Schema registry/contract per topic** — machine-checked where the ecosystem supports it; versioned JSON schemas minimum.
- Enrichment: events carry enough context for typical consumers (avoid the "chase 5 more services per event" anti-pattern), but not so much that publishing couples you to every consumer's needs — balance, judged per event.
- Include: event ID (for dedup), correlation ID (trace propagation — `../08-observability/observability.md`), occurred-at (UTC, producer clock), producer identity.

## 6. Consumers (Where Event Systems Die)

- **Idempotent (§3), unordered-tolerant (§3), fast:** a consumer that takes 30s per message shapes the whole system (lag, redelivery storms) — offload heavy work to jobs (`background-jobs.md`): event consumer = thin validation + enqueue.
- **Own their failure handling:** retry with backoff, DLQ with alerting and a runbook (same discipline as jobs §3).
- **Lag is a first-class metric:** alert on consumer lag and DLQ depth; a silently lagging consumer = your "real-time" integration quietly becoming yesterday's data.
- Isolation: one failing consumer class must not block others (per-consumer queues, bulkhead semantics — `../07-reliability/reliability.md`).

## 7. Eventual Consistency — The UX Contract

Once events integrate systems, cross-system state is eventually consistent. This is a **product decision surface**, not just technical:

- Name the windows per flow ("payment status may lag ≤30s; UI polls or subscribes")
- Compensating UX: "processing" states, optimistic UI with correction, reconciliation guarantees
- Reconciliation sweeps: a job that periodically verifies projections against source-of-truth and repairs drift (every eventual-consistency system needs one, or drift accumulates forever)

Event sourcing specifically: full auditability + temporal queries at the cost of significant machinery (versioning, snapshots, projections). Justify per `../01-foundation/complexity-control.md` — audit-log + current-state tables (`../11-specialized-concerns/privacy-compliance-audit.md`) cover most audit needs at a fraction of the complexity.

## 8. Failure Modes

- **Dual-write ghosts:** publish-after-commit without outbox; crash window loses events; consumers get phantom "updated" events for rolled-back writes via non-transactional side channels.
- **Event-as-command confusion:** `SendWelcomeEmail` published to a bus with one consumer that must not fail — that's a command; a queue/job fits better (§1).
- **Row-dump events:** consumers break on every unrelated schema change; internals leak across boundaries. Business-fact payloads.
- **The unwatched DLQ:** consumer exhausts retries silently; integration "works" while losing facts (§6).
- **Ordering assumptions:** consumer logic requires arrival order; broker rebalance delivers redeliveries out of order; state machine gates were "too much work."
- **Distributed monolith via events:** every module subscribes to every event; no one can deploy without knowing the whole event graph — boundaries exist on paper only (`../01-foundation/architecture.md` §9).
- **Kafka-shaped resume:** a team of 4 runs a 3-broker cluster for 50k events/day — the operational tax is the whole engineering week, every week (§2 honesty test).
