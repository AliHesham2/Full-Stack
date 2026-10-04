# Caching — When, What, and How to Invalidate

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Considering ANY cache (HTTP, query result, computed value, object cache), designing invalidation, or diagnosing staleness/stampede problems.
> **Related:** `../06-async-and-scale/performance.md` (measure BEFORE caching), `../05-data/data-architecture.md` §7 consistency, `../12-anti-patterns/anti-patterns.md`
> **Status:** Stable principles · Note: Redis Open Source (from Redis 8) is **tri-licensed — RSALv2 *or* SSPLv1 *or* AGPLv3**, at your choice; Valkey is the BSD-licensed fork. Only the AGPLv3 option is OSI-open-source, and choosing it carries copyleft obligations — a legal/product decision, not just a technical one (`../10-delivery/dependencies.md` §4)

---

## 1. Decision First: Should You Cache at All?

A cache is a **correctness risk purchased with latency/cost savings**. Add one only when:

1. A **measured** hot path shows meaningful cost (p95 latency, DB CPU, $ per request) — profiling first (`performance.md`)
2. The data tolerates **staleness** — you can name the acceptable window per datum ("product description: minutes", "account balance: zero")
3. The read:write ratio favors it (reads dominate; write-heavy data mostly punishes cache layers)

If any answer is "no," the correct cache is none. Caching as a reflex produces the two classic diseases: **consistency bugs** (users see stale data and file bugs you can't reproduce) and **opacity** (the DB is healthy, the cache is the problem, nobody's debugging it).

What to cache, in order of value: static/reference data → expensive computed reads → hot object lookups → session/identity lookups (with care). What not to cache: anything authorization-decision-critical unless invalidation is instant and tested (staleness = access-control violation), and anything write-heavy.

## 2. Placement Taxonomy

| Layer | Caches | Invalidation owner |
|---|---|---|
| HTTP/gateway | responses (CDN, proxies) | TTL + `Cache-Control` discipline; private data `no-store` (`../04-security/security-foundations.md` §4) |
| Application in-process | reference data, computed configs | process lifetime; weakest consistency (per-instance staleness) |
| Shared cache (Redis-class) | object/lookups across instances | explicit invalidation + TTL backstop (§4) |
| DB layer | engine buffers, materialized views | the engine / view refresh schedule |

Prefer the **highest layer that works**: if a CDN can serve it with correct privacy headers, an app-level cache is redundant machinery. In-process caches multiply consistency surfaces by instance count — fine for near-static data, wrong for business data.

## 3. Patterns

- **Cache-aside (default):** read-through with lazy fill — app checks cache, on miss reads DB and populates with TTL. Writes go to the DB; cache updated/invalidated after commit. Simplest correct default; the app owns correctness.
- **Read-through / write-through:** a library/proxy mediates all I/O; DB and cache change together. Cleaner invariants, more infrastructure coupling; use when the ecosystem already provides it — don't build it.
- **Write-behind:** async cache→DB write-behind — durability risk; only for loss-tolerant data (counters, analytics). Explicitly document the loss window.
- **Refresh-ahead:** proactively refresh hot keys before expiry — for the few keys where a miss is expensive; complexity is per-key, keep the set tiny.

## 4. Invalidation (The Actual Hard Part)

Strategies, combined:

1. **TTL as backstop (always):** every cached entry has one — the cache's memory of staleness fades even when events are lost. Choose per datum (§1.2); long TTLs for static, short for business-hot.
2. **Event-driven invalidation:** on write-commit, delete/refresh affected keys (after commit — `../02-domain-and-api/domain-logic.md` §6). Requires knowing the key-space a write affects — which requires deliberate key design (§5).
3. **Versioned keys / generation counters:** bump a generation in the key (`users:v{17}:1234`) — old entries age out by TTL; useful for bulk invalidation ("all product data changed") without enumerating keys.

Rules:

- **Delete, don't update, on invalidation** (lazy refill) — updating caches on write invites write-path races where two writers interleave and the cache ends up with the older value, durable until TTL.
- Multi-entity aggregates: invalidate every derived key (or version-bump the shared generation) — enumerate the derived key-space in the write path and TEST that enumeration (the forgotten key is the staleness bug).
- Accept and document the residual window: commit → invalidate is not atomic; concurrent readers may fill a stale value post-invalidate. Mitigations: short TTL floor, or a delayed double-delete — but honestly, the TTL floor usually suffices.

## 5. Key Design

- Namespaced, structured, versioned: `cache:{app}:{entity}:{version}:{id}` — you can reason about, enumerate, and bulk-evict by prefix.
- Include every scoping dimension of the VALUE (tenant! user! locale!) — a key missing tenant scope is a cross-tenant data leak, the highest-severity cache bug (test for it: `../09-testing/testing-strategy.md`).
- Deterministic serialization of key inputs (sorted params) — key collisions from unordered serialization produce wrong-data-for-request bugs.
- Size discipline: values bounded (no unbounded lists cached whole — cache pages, not collections), total memory budgeted with a maxmemory policy (see the Redis/Valkey guide for eviction semantics).

## 6. Stampede & Hot-Key Defense

Cache expiry under load ⇒ N concurrent misses all hit the DB to compute the same value.

- **Per-key single-flight (mutex/lock on miss):** one request recomputes, others wait/fallback — the standard fix; cache-client libraries often ship it.
- **TTL jitter** (±10–20%) so cohorts of keys don't expire in unison.
- **Stale-while-revalidate semantics** where the domain tolerates it: serve stale, refresh in background.
- **Logical early expiry** for the hottest keys (refresh before TTL).

Hot single key (celebrity row): shard its value across k subkeys, read one of k at random — a niche pattern for genuinely hot singletons; prefer fixing the access pattern first.

## 7. Security Considerations

- **Cache poisoning via unvalidated input:** keys built from raw user input can collide or be crafted — serialize + hash/normalize inputs into keys.
- **Cross-tenant/user leakage:** the §5 scoping rule; denial tests cover it.
- **Private responses in shared caches:** `Cache-Control: private/no-store` on authenticated payloads; CDN misconfiguration serving user A's response to user B is a recurring real-world breach class.
- Cache as a trust boundary: values in the cache were validated once at fill; if authorization data is cached, a permission revoke must invalidate — prefer NOT caching authorization decisions beyond seconds.

## 8. Observability & Testing

- Metrics per cache: hit rate, miss latency, eviction rate, invalidation count, error rate on the cache itself (a down cache must be a **fail-open-with-logging** degradation, never a hard outage — decide explicitly and test both modes).
- Alerts on hit-rate cliffs (a config change that silently disables invalidation shows up as hit-rate going UP while freshness complaints rise).
- Tests: hit/miss behavior, invalidation coverage (write ⇒ next read fresh), stampede (concurrent misses ⇒ one DB computation), cache-down behavior (degraded but serving), tenant-scoping (no cross-tenant reads).

## 9. Failure Modes

- **Cache-first architecture:** cache added before any measurement; hit rates ~30%; every bug investigation now has two systems to suspect. Remove or justify per §1.
- **The unenumerable key-space:** invalidation logic that must "delete everything related to X" with no key convention to enumerate it — permanent staleness bugs. Fix: versioned generations (§4.3).
- **Update-on-write races:** cache stores interleaved stale value until TTL; delete-on-write + TTL floor instead (§4 rules).
- **Caching the uncachable:** authorization decisions, balances at "zero staleness tolerance," write-through of every write — the cache becomes a second source of truth, and two sources of truth means no truth.
- **The immortal entry:** cached value with no TTL because "invalidation handles it" — invalidation has a bug (it always does eventually); TTL is the seatbelt.
