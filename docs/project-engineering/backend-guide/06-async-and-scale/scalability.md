# Scalability — Growing Only When the Problem Demands It

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Capacity is (actually) running out, planning growth, or evaluating "should we scale out/shard/go multi-region" — and when auditing AI-generated "web-scale" designs.
> **Related:** `../01-foundation/complexity-control.md` (read FIRST), `performance.md` (usually the real answer), `../05-data/data-architecture.md`
> **Status:** Stable decision framework

---

## 1. Position

Scalability is a **cost-of-growth curve decision**, not a virtue. The engineering question is precise: *at what load does the current design fail, what fails first, and what is the cheapest change that moves the failure point beyond the foreseeable horizon?* Everything else — microservices "for scale", sharding on day one, multi-region for a single-country product — is complexity without a problem (`../01-foundation/complexity-control.md` §2).

The standard sequence that resolves ~95% of real scaling pressure:

```
1. Fix the hot path (indexes, N+1, payloads)        — 10–100× per-node capacity, usually
2. Scale VERTICALLY (bigger instance)                — 4–8× headroom, minutes of work
3. Scale OUT the stateless tier (replicas + LB)      — near-linear for the app tier
4. Read replicas / caching for read pressure         — read-side multiplication
5. Offload async work to workers                     — request-path relief
6. THEN the heavy machinery: partitioning, sharding, multi-region — each an explicit, evidence-gated project
```

Steps 1–2 are routinely skipped by enthusiastic designs and routinely solve the "problem."

## 2. Prerequisites for Horizontal Scaling

- **Statelessness of the request tier:** no in-process session state, no local-file assumptions, no in-memory caches holding business truth (`caching.md` §2 — in-process caches are per-instance staleness surfaces). Anything stateful moves to the shared tier (DB/cache/broker) or gets sticky routing as a deliberate, documented compromise.
- **Shared-state inventory:** enumerate what currently lives on one box (sessions, files, schedulers, singleton locks — §4) before adding a second instance; the second instance is the audit that finds them all.
- **Config via environment** (`../10-delivery/configuration.md`) — instances are interchangeable cattle.

## 3. The Database Is Always the Real Conversation

App tiers scale out trivially; data tiers don't. The escalation ladder, each step justified by measured saturation:

| Step | Solves | Cost / new failure modes |
|---|---|---|
| Index + query tuning | most "DB is slow" | none — always first (`performance.md`) |
| Connection pool right-sizing | pool exhaustion, not DB saturation | none |
| **Read replicas** | read-heavy load | replication lag semantics — route post-write reads to the leader (`../05-data/data-architecture.md` §7); stale-replica bugs |
| Caching hot reads | read amplification | staleness + invalidation (`caching.md`) |
| Table partitioning (time-based) | huge append-only tables (events, logs) | ops complexity; query routing |
| **Sharding** | write saturation beyond one node | LAST RESORT: cross-shard queries, transactions, rebalancing pain — effectively irreversible (`../01-foundation/complexity-control.md` §5) |

Before sharding: verify the write pressure isn't (a) an index/lock pathology, (b) a hot row (fix contention — `../05-data/transactions.md` §6), or (c) write amplification from redundant writes. Most "we need to shard" diagnoses are one of those three wearing a trench coat.

**Multi-region** requires: genuine geographic latency/compliance drivers, data-residency answers (`../11-specialized-concerns/privacy-compliance-audit.md`), and a conflict story for multi-writer data. A single-region product reading "multi-region architecture" blog posts doesn't qualify.

## 4. Hidden Shared State (Found at Instance #2)

| Component | Failure at 2+ instances | Fix |
|---|---|---|
| Cron/scheduled jobs | double-fire | distributed lock / leader election (`background-jobs.md` §3) |
| In-process queues/cache-as-truth | divergent per-instance state | shared queue/cache as system of record |
| Local file writes | file exists on instance A only | object storage (`file-and-object-storage.md` §1) |
| Local rate-limit counters | limits ×N | shared counter store (`../02-domain-and-api/api-design.md` §8) |
| WebSocket in-memory pub/sub | cross-instance messages lost | backplane (`realtime.md` §3) |
| Ad-hoc singletons (id generators, locks) | duplicates | DB sequences / UUIDv7 / distributed lock (`../05-data/concurrency-and-idempotency.md` §5) |

## 5. Load Balancing & Failure at Scale

- Health-checked LB + readiness gates (`../07-reliability/reliability.md` §2); instances join/leave without operator action.
- Per-instance concurrency limits + queue with backpressure — overload shows up as bounded latency, not collapse (thundering herds meet shed load: 503-with-Retry-After beats p99 of 60s).
- Retry storms are the scale-outage multiplier: client retries × instance count × downstream timeout = synchronized amplification. Backoff + jitter + retry budgets are scaling prerequisites, not polish (`../05-data/concurrency-and-idempotency.md` §6, `../07-reliability/error-handling.md` §5).
- Hotspots: one viral tenant/user/key concentrating traffic — per-tenant rate limits + hot-key strategies (`caching.md` §6); detect via per-dimension metrics (`../08-observability/observability.md` §4).

## 6. Decision Checklist for "We Need to Scale"

### Cost awareness (the dimension engineers forget)

Scaling steps are billed monthly, forever: bigger instances, replicas, a second region, broker clusters — each has a $/month line and an ops-hours line. Make both visible at decision time:

- **Unit economics check:** the cost per request/order/user at current scale vs post-change scale — scaling that triples infra cost for 2× capacity is a business decision, not an engineering reflex.
- **The cheaper levers first:** query fixes and payload trims are engineering hours once; replica/instance costs recur every month.
- **Idle-capacity audit:** autoscaled/sharded systems left oversized "just in case" quietly become the biggest line item; right-size against measured saturation (the checklist's first item), not peak-anxiety.
- **Data lifecycle costs scale too:** storage/backup/egress growth is a scaling dimension (`../05-data/data-architecture.md` §6, `../06-async-and-scale/file-and-object-storage.md` §6) — unbounded tables and undying objects are compounding subscriptions.

### The checklist

- [ ] Measured: which resource saturates first (CPU? pool? DB IOPS? bandwidth?) — with numbers
- [ ] Hot paths fixed (step 1) — N+1 and missing indexes eliminated
- [ ] Vertical headroom honestly evaluated (cost of bigger instance vs engineering week of sharding)
- [ ] Stateless-tier audit done (§2, §4) — the shared-state inventory is written down
- [ ] The chosen step moves the failure point ≥10× — otherwise you bought 3 months and permanent complexity
- [ ] New failure modes named and handled (every step in §3's table has a right-hand column — read it)
- [ ] Cost delta + unit-economics impact written next to the capacity gain
- [ ] Rollback/reversibility assessed (`../01-foundation/complexity-control.md` §5)
- [ ] ADR written with assumption + revisit trigger (`../10-delivery/documentation-and-decisions.md`)

## 7. Anti-Patterns

- **Shard-first design:** single-node traffic for the product's lifetime, plus cross-shard JOINs forever. The complexity is paid daily; the capacity is never used.
- **Microservices-as-scaling:** splitting a monolith whose actual bottleneck is ONE unindexed query — now the query is slow AND remote.
- **Replica-lag amnesia:** all reads to replicas including post-write confirmation reads; users see their own action "not happen" (read-your-writes violations — route by flow, `../05-data/data-architecture.md` §7).
- **The autoscaler that hid the leak:** scaling out to absorb a memory leak; the leak wins exponentially eventually, at the worst possible bill.
- **Scale theater:** multi-region diagrams for single-city products; interview-driven architecture; capacity planning by aspiration. The §6 checklist exists to price every one of these in engineering-hours before purchase.
