# Error Handling — Taxonomy, Mapping, and Never Swallowing

> **Part of:** Reliability — Backend Project Guide
> **Read when:** Writing any catch/except, designing error responses, classifying retryability, or wiring error paths through layers.
> **Related:** `reliability.md` (timeouts, degradation), `../02-domain-and-api/api-design.md` §4 (error envelope), `../08-observability/observability.md`
> **Status:** Stable principles

---

## 1. The Taxonomy (Every Error Is Exactly One of These)

| Class | Definition | Examples | Response strategy |
|---|---|---|---|
| **Validation error** | Input violates a declared contract | bad shape, unknown fields, out-of-range | 400, field errors, no retry |
| **Domain/business error** | Valid input, refused by business rules in current state | out of stock, already shipped, insufficient funds | 403/409/422, domain error code, no retry (unless state changes) |
| **Authorization error** | Actor may not do this | not owner, wrong role, other tenant | 403/404, generic message, no retry, LOG for audit |
| **Operational (transient) error** | Environment failed, likely temporary | DB connection dropped, network timeout, dependency 503, 429 | 5xx/503, retryable WITH discipline (§5) |
| **Programmer error** | A bug — invariants violated, impossible states | TypeError, null deref, assertion failures | 500, alert, NO local handling — let the crash/error boundary catch it |

The two rules the taxonomy enforces:

1. **Programmer errors are never caught-and-continued.** A violated invariant means the process's state is unknown — catching it and carrying on is how data corrupts silently. Catch at the top boundary, log with full context, fail the request, alert. (In job workers: fail the job, let the DLQ catch it — same principle.)
2. **Operational errors are expected.** They are handled with retry/timeout/degradation policy (`reliability.md`), not treated as bugs and not swallowed.

## 2. The Error Model in Code

- **One error hierarchy per application** (or a tagged/result type in functional style) with a **stable machine code** per domain error (`ORDER_ALREADY_SHIPPED`) + context fields. Defined once, mapped once (§4).
- Domain/use-case layers throw/return **domain errors** — never HTTP concepts. A use case that raises `BadRequest` has leaked the edge into the domain (`../01-foundation/architecture.md` §4.3).
- **Errors carry context for logs, not for clients:** correlation ID, input identifiers (IDs not payloads), where-it-happened. The client-facing message is written separately (§3).
- Result-type vs exception styles: both work; pick the ecosystem-idiomatic one and be consistent. What matters: errors are **values you must confront** (typed, exhaustive handling at boundaries), not invisible control flow.

## 3. Client-Facing Errors (What Leaves the System)

- The one envelope (`../02-domain-and-api/api-design.md` §4): machine `code` + human-safe `message` (+ `details` for validation); the request ID travels in the `X-Request-Id` response header. Nothing else: no stack, no SQL, no driver text, no internal hostnames — error bodies are reconnaissance data (`../04-security/security-foundations.md` §6).
- Uniform across the API — one mapping layer (§4), not per-handler improvisation. Test the envelope shape on every error path.
- Messages help the caller ACT (what's wrong, what to change), not diagnose your internals. "qty must be ≥ 1" not "ValueError at line 47".

## 4. The Central Mapping Layer

```
domain error code  →  HTTP status + client message        (one table, tested)
operational error  →  503/504 + Retry-After where apt
programmer error   →  500 + correlation ID + alert fired
validation error   →  400 + field details (from the schema validator)
```

Implemented once at the edge error middleware. Every new domain error gets a table row and a mapping test. Consequences: adding an error never touches handlers; error semantics can't drift per endpoint; the envelope is testable as a contract.

### 4.1 Error Boundary Placement (Where Errors Are Caught)

Errors should be caught at the **layer that understands how to classify them**:

| Layer | Catches | Response |
|---|---|---|
| **Edge / middleware** | All unhandled throws from below; validation errors from schema libraries | Maps to HTTP envelope (§4). The ONLY place HTTP status codes are assigned. |
| **Use case / application** | Domain errors thrown by domain logic; authorization denials | Wraps with context (request IDs, domain identifiers). Does NOT map to HTTP. |
| **Domain** | Invariant violations as typed domain errors (e.g., `OrderAlreadyShipped`) | Throws/returns domain error objects. Never catches its own errors — lets them propagate. |
| **Infrastructure** | Connection failures, timeout errors, constraint violations from DB | Wraps as operational errors with retryability classification. Does NOT swallow. |
| **Process boundary** | Unhandled rejections, uncaught exceptions (programmer errors) | Logs full context, fires alert, exits or fails the request. Never continues. |

The critical rule: **programmer errors (invariant violations, null derefs, assertion failures) are caught ONLY at the process boundary**, not in per-request try/catch blocks. Catching them in application code and continuing is how silent data corruption happens — the process state is unknown after a violated invariant.

## 5. Retryability Classification (Decide Per Error, Never Blanket)

| Signal | Retry? |
|---|---|
| 4xx (validation, auth, domain refusal) | **No** — the same request fails identically |
| 429 / 503 with Retry-After | Yes, honoring the server's header |
| Connection refused / reset, timeout on **idempotent** operation | Yes — exponential backoff + jitter, bounded attempts (≤3–5), retry budget |
| Timeout on **non-idempotent** write (POST without idempotency key) | **No automatic retry** — outcome is ambiguous; surface to caller / require idempotency key (`../05-data/concurrency-and-idempotency.md` §3.3) |
| Serialization/deadlock DB errors | Yes (use-case level, tiny bounds) |
| Programmer error | No — alert and fix |

Never retry synchronously in the request path more than once or twice; sustained retries belong to jobs/queues (`../06-async-and-scale/background-jobs.md` §2).

## 6. Partial Failure (Multi-Step Operations)

When a use case's step 3 of 5 fails:

- **Transactional steps:** rolled back atomically — nothing to decide (that's why they're transactional).
- **Non-transactional side effects** (external API called, email sent, event published): the use case must define the recovery state — explicit `PENDING`/`FAILED` status persisted, a reconciliation/continuation path (job, manual runbook), and observability into the stuck states (`../02-domain-and-api/domain-logic.md` §6). "We'll notice eventually" is not a recovery design.
- Compensating actions (refund-if-charged) for flows that cannot resume forward: documented per flow, tested, idempotent themselves.

## 7. Logging & Alerting Interactions

- Every 5xx logs full context (request ID, route, params-safe, error chain) at `error` level; alert on error-RATE spikes, not single errors (noise kills alert trust — `../08-observability/observability.md` §5).
- Domain refusals (403/409/422) log at `info/warn` — they're business events, not incidents; authorization failures log at `warn` WITH the actor/object IDs (audit trail — `../03-auth/authorization.md`).
- Never log the same error at every layer it passes through (log once, at the boundary that handles it; wrap-and-annotate upward).

## 8. Failure Modes

- **The swallow:** `catch (e) { return null }` / empty catch / `catch` that logs-to-nowhere. The single most damaging pattern in generated code — failures become silent data corruption. An error you ignore is a decision you didn't know you made.
- **Catch-all middleware as a safety net for everything:** top boundary exists for PROGRAMMER errors (§1.1), not as an excuse to skip typed handling of operational/domain errors.
- **Error-flood logging:** same error logged at 4 layers × every middleware pass — the signal (and the on-call engineer's will to live) dies. Log once with context.
- **Retry-everything reflex / retry-nothing stubbornness:** both halves of §5 violated in different codebases, often the same codebase.
- **Leaky 500s:** driver errors serialized straight to the client because the mapping layer (§4) was bypassed with a thrown raw error.
- **Async error amnesia:** promise/callback paths with no rejection handler — the error vanishes into the runtime (unhandledRejection should crash in production: it's a programmer error by definition).
- **Generic-rethrow without context:** `catch (e) { throw new Error('failed') }` — destroys the diagnostic chain. Wrap WITH the original (cause chain) and add the operation's identifiers.
