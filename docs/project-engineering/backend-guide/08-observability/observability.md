# Observability — Logs, Metrics, Traces, and What to Do With Them

> **Part of:** Observability — Backend Project Guide
> **Read when:** Deciding what to log/measure/trace, wiring correlation IDs, defining alerts and SLOs, or debugging production failures.
> **Related:** `../07-reliability/*`, `../11-specialized-concerns/privacy-compliance-audit.md` (audit logs are a different thing — §7)
> **Status:** Stable methodology · Exporter/platform syntax in the platform guide

---

## 1. Position

Observability is the difference between a system you debug and a system you guess at. It is designed WITH the feature, not bolted on at the incident. Three pillars, with distinct jobs:

| Pillar | Answers | Unit | Retention shape |
|---|---|---|---|
| **Structured logs** | What happened, in detail, once? | event (JSON line) | days–weeks, searchable |
| **Metrics** | How is the system behaving, in aggregate, over time? | counter/gauge/histogram | months, cheap |
| **Traces** | Where did time go across services/layers for THIS request? | span tree | days, sampled |

Rule of allocation: if a question is answerable by metrics, don't answer it with logs (cost); if it needs a specific request's story, that's traces+logs with a correlation ID.

## 2. Structured Logging (The Foundation)

- **JSON lines, one event per line**, fields not prose: `ts, level, msg, request_id, route, status, duration_ms, tenant_id, user_id, error.code`.
- **Levels as contracts:** `error` = a human should look (alertable); `warn` = degraded-but-handled (fallbacks, retries); `info` = lifecycle + significant business events; `debug` = detail, off in prod by default. If `info` floods, it's `debug` wearing a coat.
- **One log line per event, context attached** — not "entering function X" (that's tracing's job, and trace-level logging is how log budgets die).
- **Log at the boundaries** (edge: request outcome; job: start/end/fail; dependency call: outcome+duration) — interior layers log only what they ADD (domain refusals, decisions taken, state transitions).
- Correlation/request ID: generated at the edge (or accepted from a trusted upstream), attached to every log/trace in the request AND propagated into jobs/events/webhook calls (`../06-async-and-scale/background-jobs.md` §5). This single field is what turns five systems' logs into one story.

### 2.1 Correlation ID Propagation

The correlation ID must survive every context boundary:

| Boundary | Propagation method | Notes |
|---|---|---|
| **HTTP request** | `X-Request-ID` header (incoming + outgoing) | Accept from trusted upstreams; generate if absent. Return in response headers. |
| **Background job** | Stored in job payload at enqueue time | Job logs include this ID; it traces the request that triggered the job. |
| **Event / message** | Event metadata / headers (`correlation_id` field) | Every consumer log includes it; events without correlation IDs are orphaned stories. |
| **Outbound HTTP** | Forward as `X-Request-ID` header | When calling other services; do NOT generate a new ID (that breaks the trace). |
| **Webhook delivery** | Include in outbound webhook payload + headers | Customer-side logging can correlate their webhook receipt to your dispatch. |
| **Database** | Set via `SET app.request_id = '...'` at connection acquire (if using RLS-style session variables) | Optional but powerful: query logs in the DB show which application request triggered them. |

**Rules:**
- The ID format should be URL-safe, sortable, and contain no sensitive data: `req_<uuidv7>` or `req_<ulid>` works well.
- The ID is generated **once** at the edge and **never regenerated** — if you regenerate it mid-request, you lose the ability to correlate the full story.
- A missing/corrupted correlation ID in logs is an observability defect — log a warning, generate a replacement, but never let the absence block the request.
- **Never logged, ever:** passwords/tokens/secrets (even "temporarily"), full request bodies of sensitive endpoints (auth, payments), PII beyond identifiers needed for debugging, card data / credentials of ANY kind (`../04-security/secrets-management.md` §2). Redaction at the logging layer, tested by asserting secret fields absent in logs for flows that touch them.

## 3. Metrics (The Dashboard Layer)

Minimum backend metric set (RED per endpoint + USE per resource):

- **Rate** — requests/sec per route/class
- **Errors** — error rate by class (5xx vs domain refusals — different signals! `../07-reliability/error-handling.md` §7)
- **Duration** — latency histograms per route (p50/p95/p99 — percentiles, never averages: `../06-async-and-scale/performance.md` §2)
- Plus per-dependency: call rate, error rate, duration (external APIs, cache, DB)
- Plus saturation: pool utilization, queue depth, connection counts, event-loop lag (runtime-specific), memory/GC
- Plus business KPIs on a separate dashboard: signups, orders, payment success rate — technical dashboards are for incidents; business dashboards are for "is the product broken for users" (often the FIRST signal)

Label cardinality discipline: labels bounded (route names, error codes) — user IDs/emails as metric labels = cardinality explosion that kills the metrics backend.

## 4. Alerting (On Symptoms, With Runbooks)

- Alert on **user-visible symptoms** (error rate, p95 latency, queue age, DLQ depth), not causes (CPU high but everything fine at 3AM = a page that teaches people to ignore pages).
- Every alert links a runbook (`../10-delivery/documentation-and-decisions.md`): what it means, first three checks, escalation. An alert without a runbook is a guess with a pager.
- Severity honesty: page only for user-impacting NOW; everything else is a ticket. Alert fatigue is a reliability bug.
- SLO-lite as the framing layer: define per service "availability = 99.9% of requests succeed in <1s over 30 days" — alerts fire on error-budget burn, which converts "one number is high" into "users are hurting,
  proportionally."

  **Burn rate**, concretely: the rate at which you are consuming the error budget relative to
  the SLO window. A 30-day 99.9% SLO allows ~43 minutes of failure; burning budget 14× faster
  than sustainable exhausts it in ~2 days — that is a page. Burning 2× is a ticket. Two
  alert speeds (fast-burn: page; slow-burn: ticket) replace a dozen threshold alerts and
  express user pain proportionally.

  Adopt full SRE SLO machinery only at organizational scale — the concept scales down fine.

## 5. Tracing (Cross-Layer Time)

- Trace spans at: edge request → auth → use case → queries → external calls → job execution. Span attributes mirror log fields (correlation ID joins them).
- **Sampling** in high-volume systems: head-based (decision at start) for consistent coverage; tail-based (keep the slow/erroring ones) when the tail is what you hunt — platform-dependent, pick per guide.
- The debugging workflow traces enable: user complains → find request by ID (logs) → open trace → see the 2.1s external call / lock wait / N+1 chain of spans → done. Without traces this same investigation is hours of log grepping and guessing.

## 6. Debugging Production Failures (The Workflow)

1. **Scope:** business dashboard first — which users/flows/tenants, since when? (metric)
2. **Locate:** error-rate spike by route/dependency — which layer is failing? (metrics + error classification)
3. **Isolate:** sample failing request IDs → traces → dominant span
4. **Explain:** logs for those request IDs → the error chain with context
5. **Correlate:** what changed? deploys, migrations, config, dependency status — the change log IS observability (`../10-delivery/deployment.md` release records)
6. **Act:** mitigate (rollback/flag-off/`../11-specialized-concerns/feature-flags.md` kill switch) THEN fix — mitigation speed beats root-causing heroics while users hurt

The reverse order (grep logs first) is how incidents burn hours. Build the layers so the order is natural.

## 7. Audit Logs (Distinct From App Logs)

Audit logs answer **"who did what to which record, when"** — compliance and forensics, not debugging (`../11-specialized-concerns/privacy-compliance-audit.md`):

- Separate stream/table, append-only, tamper-evident storage, longer retention
- Events: auth events (login/failure/MFA/recovery), authorization-critical actions, data exports/deletes, admin/privileged ops, permission changes
- Content: actor ID, action, object ID, before/after state for critical fields, timestamp (UTC), request ID
- Never in the debug log stream; never with the sensitivity rules of app logs (audit NEEDS the identifiers; app logs minimize them — different jobs, different stores)

## 8. Failure Modes

- **String logging + grep archaeology:** unstructured prose logs, no correlation IDs — every incident starts with a 40-minute grep.
- **Log-everything insecurity:** debug-level detail enabled "to catch the bug," leaking payloads and secrets — volume is a security surface, not just a cost (`../04-security/secrets-management.md`).
- **Metric swamp:** 400 dashboards, no owner, no alert linkage; the interesting number is always one nobody charted.
- **Alert pager fatigue:** pages for non-symptoms; everyone mutes; the real page is missed. Ruthlessly cut (§4).
- **Averages-only latency:** p50 looks divine while p99 is 30s; users experience the tail (`../06-async-and-scale/performance.md` §2).
- **Correlation-ID dead-ends at async boundaries:** HTTP logs have the ID, job/webhook/event logs don't — the bug is in the async path, exactly where the ID chain broke. Propagate into payloads at enqueue (§2).
- **Observability retrofit:** feature ships un-instrumented; "we'll add metrics when it's a problem" — the first problem is always discovered by users, and you'll have nothing but a stack trace. Instrument at birth: request outcome, dependency calls, job lifecycle (§3 minimum set).
