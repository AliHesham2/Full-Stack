# Reliability — Timeouts, Health, Shutdown, and Graceful Degradation

> **Part of:** Reliability — Backend Project Guide
> **Read when:** Designing startup/shutdown, health checks, timeouts to dependencies, circuit breakers, degradation under partial failure, resource limits.
> **Related:** `error-handling.md` (classification), `../10-delivery/deployment.md`, `../08-observability/observability.md`
> **Status:** Stable principles

---

## 1. Reliability = Bounded Failure Behavior

A reliable backend is not one that never fails — dependencies fail, always. It's one whose failure behavior is **bounded, observable, and recoverable**: every operation has a timeout, every dependency has a degradation mode, every process starts and stops cleanly. The design unit is "what happens when X is down" for every X you call.

## 2. Health & Readiness

Two different questions, two endpoints (`../01-foundation/initialization.md` §2.5):

| Endpoint | Question | Checks | Used by |
|---|---|---|---|
| `/health` (liveness) | Should this process be **restarted**? | Process-internal only — event loop responsive, no DB queries | orchestrator liveness probes |
| `/ready` (readiness) | Should this instance **receive traffic**? | Critical deps reachable (DB ping, cache ping), migrations applied, warm-up complete | LB / deploy rollout gates |

Rules:

- Liveness must NEVER touch the database — a DB blip turning into a restart-storm of your whole fleet is a self-inflicted outage classic.
- Readiness failing ⇒ instance leaves rotation but is NOT restarted (transient dependency blips heal; restarts don't help them).
- Startup: process listens only AFTER readiness prerequisites pass (or fails fast on impossible prerequisites — missing config, unreachable DB at boot per config policy).
- Separate operational endpoints (`/metrics`, `/debug-*`): authenticated, internal-network-scoped (`../04-security/security-foundations.md` §6).

### 2.1 Response Format

Health endpoints should return structured, machine-parseable responses:

```json
// GET /health
{ "status": "ok", "uptime_s": 12345 }

// GET /ready
{
  "status": "ready" | "not_ready",
  "checks": {
    "database": { "status": "ok", "latency_ms": 2 },
    "migrations": { "status": "ok", "applied": 47 },
    "cache": { "status": "degraded", "error": "connection timeout" }
  }
}
```

Rules:
- **Status code** is the primary signal: `200` = healthy/ready, `503` = not ready. Never return `200` with `"status": "not_ready"` — load balancers read status codes, not JSON.
- **Liveness response** is minimal (no external calls). Readiness includes per-dependency check results for debugging.
- **Latency matters:** readiness checks should complete in <500ms total. Slow health checks delay deploy rollouts and autoscaling decisions.
- **Cache health check results briefly** (1–5s TTL) to avoid hammering dependencies on every probe — especially important when readiness is checked at high frequency during deploys.

## 3. Timeouts (The Default That Saves You)

**Every network call has an explicit timeout.** Default client timeouts (often infinite or 30–120s) are latency-bomb defaults: one slow dependency × your concurrency budget = full-instance exhaustion (threads/connections pinned on zombie requests).

- Per-dependency timeouts sized to actual latency profiles (p99 + margin): DB statement 5–30s by class, external API 3–10s, internal service calls tight (1–3s).
- **End-to-end coherence:** each callee must finish within the caller's remaining deadline, leaving time to return a response. Sequential calls share the total budget; parallel calls share a remaining deadline rather than adding their budgets. Propagate cancellation where supported.
- Timeouts on: HTTP clients, DB statements (`statement_timeout`), pool acquisition, cache gets (yes — a slow cache miss must not add unbounded latency), queue operations, graceful-shutdown waits (§5).
- A timeout firing on a non-idempotent write is ambiguous — treat as §5 of `error-handling.md` (idempotency or surface).

## 4. Resilience Patterns (Each With an "Only When")

| Pattern | What it does | Only when | Cost/abuse mode |
|---|---|---|---|
| **Retry + backoff + jitter** | Recovers transient blips | Retryable + idempotent operations (`error-handling.md` §5) | Retry storms amplify outages — bounded attempts, jitter mandatory |
| **Circuit breaker** | Stops hammering a dead dependency | The retry storm risk is real: dependency outages that persist | Half-open probing logic is subtle — use a library; breakers add state to debug |
| **Fallback / degradation** | Serves reduced answers when a dependency is down | A defined lesser answer exists (cache old data, hide a widget, disable a feature — `../11-specialized-concerns/feature-flags.md` kill switches) | Silently-stale-forever fallbacks hide the outage — mark degraded in observability |
| **Bulkhead** | One dependency's pool can't consume all resources | Dedicated pools per dependency under real contention | Pool sprawl; per-pool sizing math needed |
| **Load shedding / backpressure** | Reject excess load early (503 + Retry-After) rather than collapse at p99=60s | Sustained overload beyond capacity | Shed on the RIGHT dimension (per-tenant, per-class) or you shed your best customers |

### 4.1 Circuit Breaker Decision Framework

Not every slow dependency needs a breaker. Consider one when the first three conditions hold; a fallback improves the response but is not required to prevent cascading failure:

1. The dependency has **real outage history** (not theoretical)
2. Retries alone cause **measurable amplification** (retry storms under the outage)
3. The failure mode is **persistent** (minutes+), not transient (seconds)
4. Prefer a **fallback** acceptable during the outage window; otherwise fail fast with 503 and expose a retry policy.

If only (1) holds: retries with backoff suffice. If (1)+(2) hold but no fallback exists: the breaker opens and returns 503 — worse than the fallback, but better than cascading failure. The breaker is a **last resort before cascading failure**, not a first-line defense.

Circuit breaker configuration as a decision, not a default:
- **Failure threshold:** what fraction of requests must fail before opening? (e.g., 50% over 30s window — too low triggers on slow single requests; too high lets cascading failure start)
- **Open duration:** how long before half-open probe? (e.g., 30s — too short re-opens during the same outage; too long keeps healthy traffic degraded)
- **Half-open probe count:** how many test requests before deciding? (e.g., 3–5 — enough to detect recovery, not enough to flood a recovering service)

**When NOT to use a circuit breaker:**
- Between your own services in a modular monolith (fix the code, don't circuit-break it)
- On dependencies that are called once per day (not enough traffic to trigger; handle the failure directly)
- When traffic is too low for meaningful amplification. A breaker returning 503 can still prevent cascading resource exhaustion without a fallback

Discipline: these are **tools with failure modes, not virtues to sprinkle**. A circuit breaker around every internal call in a 3-service system is distributed-systems cosplay; one around a flaky third-party payment API is due diligence. Complexity bar applies (`../01-foundation/complexity-control.md`).

## 5. Graceful Shutdown (Design Once, Test Always)

Sequence on SIGTERM (deploy/autoscale kills):

1. Stop accepting NEW connections/requests (close listener, signal readiness=false)
2. Finish IN-FLIGHT requests within a bounded drain window (e.g., 10–30s)
3. Stop consuming new jobs/messages; finish or re-queue in-flight ones (respect visibility timeouts — `../06-async-and-scale/background-jobs.md` §3)
4. Close dependency clients cleanly (pools, brokers) — release connections the pool doesn't leak
5. Exit 0 within the orchestrator's kill grace period; else forced kill (test that you actually fit the window under realistic in-flight load)

The un-tested graceful shutdown is a deploy-time request-error spike every release. Test: kill -TERM under load in staging, assert zero dropped requests beyond the drain budget.

## 6. Resource Exhaustion (Slow Outages)

| Resource | Exhaustion signature | Bound it with |
|---|---|---|
| DB connections | pool-wait timeouts while DB is idle | pool size budget, acquisition timeout, no-connection-holding across awaits (§`../05-data/data-access.md` §5) |
| Memory | OOM restarts / GC thrash | payload caps, bounded caches/queues, leak profiling |
| FDs/sockets | EMFILE after connection creep | keep-alive hygiene, client reuse, ulimit awareness |
| Concurrent work | latency cliff past N concurrent | per-instance concurrency limit + bounded queue (§4 shedding) |
| Idle-in-transaction | locks/bloat with "low traffic" | idle timeout, transaction discipline (`../05-data/transactions.md` §2) |

Exhaustion is the failure class that monitoring must catch EARLY (saturation metrics, Utilization-Saturation-Errors — `../08-observability/observability.md` §4): by the time it's an outage, every probe looks guilty.

## 7. Failure Containment

- **Blast radius review per component:** if this crashes/hangs/returns garbage, what else falls? (Shared queue blocked by one job class → separate queues; one tenant's heavy export saturating workers → per-tenant fairness limits; one slow endpoint consuming all connections → per-route concurrency budgets.)
- **Fail static where possible:** config/reference data load at startup; if the config store dies at runtime, serve the last-known-good rather than 500-ing everything (`../10-delivery/configuration.md`).
- **Scheduled-work isolation:** cron/jobs on separate workers from the request tier — background load spikes must never add request latency.

## 8. Production Readiness Checklist (Per Service)

- [ ] Liveness/readiness split correctly (§2); no DB in liveness
- [ ] Every outbound dependency: explicit timeout + retry policy + (where warranted) breaker/fallback
- [ ] Graceful shutdown implemented AND load-tested within the kill window
- [ ] Resource bounds set: payload, concurrency, pools, queues, caches
- [ ] Degradation modes defined per critical dependency (what the product does when X is down)
- [ ] Saturation + error-rate alerts with runbook links (`../08-observability/observability.md` §5)
- [ ] Startup fail-fast on invalid config; no crash-loops on transient dependency blips (retry startup with backoff)
- [ ] Single-flight on singleton work (schedulers locked/leader-elected — `../06-async-and-scale/scalability.md` §4)

## 9. Failure Modes

- **Infinite-default timeouts:** the 30-minute hung request nobody can explain; fleet throughput eroded by zombies.
- **Liveness-touches-DB:** dependency blip → mass restarts → thundering reconnect herd on the recovering dependency.
- **The breaker that never opens / always open:** misconfigured thresholds nobody calibrated — a component providing the appearance of resilience.
- **Drain-less deploys:** every release = error spike; accepted as "normal deploy noise" (it's a shutdown bug, §5).
- **Degradation amnesia:** fallback path serves stale data for weeks because nothing marks degraded state; users file "wrong data" bugs while the dependency outage healed silently.
- **Restart-as-medicine:** crash-looping on a transient dependency instead of backoff-retry at startup — the orchestrator becomes the outage amplifier.
