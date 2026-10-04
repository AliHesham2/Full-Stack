# Webhooks — Inbound and Outbound Machine Contracts

> **Part of:** Specialized Concerns — Backend Project Guide
> **Read when:** Receiving webhooks from providers (payments, GitHub-class platforms) or sending webhooks to your customers/integrations.
> **Related:** `../06-async-and-scale/background-jobs.md`, `../05-data/concurrency-and-idempotency.md` §3.4, `../02-domain-and-api/api-design.md`, `../04-security/security-foundations.md`
> **Status:** Stable patterns · Grounded in current provider practice (HMAC signatures, timestamp validation, replay windows)

---

## 1. Position

A webhook is an **unauthenticated HTTP POST to your server from a third party** (inbound) or a machine contract you owe to customers (outbound). Both directions have failure modes that ordinary API design doesn't: you don't control the sender's retries, ordering, or duplication; your receiver must be skeptical, fast, and idempotent. Discovered importance: webhooks are the integration surface where correctness bugs destroy real money state (payment status!) silently.

## 2. Inbound (Receiving)

### 2.1 The processing contract

```
receive → verify signature → durably persist event + processing intent → acknowledge 2xx → process async
```

- **Respond fast (<5s, ideally instantly):** providers time out and RETRY (and some penalize/disable slow endpoints). The handler's job: verify, persist the raw event, enqueue processing, return 200. All parsing/business logic lives in the job (`../06-async-and-scale/background-jobs.md` §2 — the webhook handler is an edge, not a use case).
- **Return 2xx only after durable acceptance, not on processing success** — a 500 tells the provider "retry," which re-delivers an event your system already accepted: the duplicate storm multiplies. Transient processing failure is YOUR job to retry (the queued job retries), not the provider's. If durable persistence/enqueue fails before acceptance, return a retryable 5xx. Use an inbox/outbox or atomic DB queue to close the persist/enqueue crash gap; never acknowledge an event held only in memory.

### 2.2 Verification (Non-Negotiable)

1. **HMAC signature** over the raw body with the shared signing secret (header scheme per provider — Stripe `v1=`, GitHub `sha256=`): recompute, compare constant-time, on the **raw bytes before any parsing** (JSON re-serialization breaks the signature).
2. **Timestamp validation:** reject events older than a tolerance window — **5 minutes is the concrete default**, verified against Stripe’s webhook documentation (“Our libraries have a default tolerance of 5 minutes between the timestamp and the current time”). This is the replay-attack defense: an intercepted valid webhook re-sent later must fail. Keep your server clock NTP-synced, or a correct implementation rejects legitimate events.
3. **Configured secrets per endpoint**, stored in the secret manager (`../04-security/secrets-management.md`), rotatable with dual-validity windows.
4. TLS on the endpoint, obviously; and IP allowlists only as defense-in-depth (provider IP ranges change — never the sole mechanism).
5. Missing/invalid signature ⇒ **401/400 and LOG with full metadata** (you want to see attack attempts and provider misconfigurations in observability).

An endpoint that accepts unsigned webhooks is an unauthenticated write API wearing a friendly name.

### 2.3 Semantics (Assume the Worst Sender)

- **At-least-once delivery is a guarantee:** the same event WILL arrive twice (retries, provider at-least-once infra, your own 500s). Deduplicate on the event ID — a processed-events table with the provider's event ID as unique key (`../05-data/concurrency-and-idempotency.md` §3.2); duplicate delivery → look up → return 200 without reprocessing.
- **No ordering guarantees:** `payment.captured` may arrive before `payment.created` after a retry. Process events against current state with state-machine gates, not sequence assumptions. Providers say this explicitly — Stripe’s guidance is to not use the event `created` timestamp for ordering *or* for duplicate detection, and to **track event IDs** instead (which is exactly the dedup mechanism in §2.3 above).
- **Events ≠ current truth:** the payment provider's API is the source of truth; a webhook is a nudge. For critical transitions, fetch the current state from the provider API before acting (guards against out-of-order and lost events).
- **Schema drift is constant:** providers add fields, deprecate others — tolerant reader (ignore unknown fields), pin to documented contracts, changelog-watch critical providers (`../10-delivery/dependencies.md` §4 advisory-feed discipline applies to API providers too).

## 3. Outbound (Sending)

You are now the provider — honor the contract §2 demands of others:

- **Sign your payloads** (HMAC per-endpoint secret, timestamp header, documented scheme) and **publish a versioned payload contract** with additive-only evolution (`../02-domain-and-api/api-design.md` §9 discipline).
- **Delivery with retries:** exponential backoff with jitter over a window (e.g., seconds → hours across ~10–20 attempts), then **disable the endpoint** with notification — a dead customer endpoint that you hammer forever or silently drop both wrong; DLQ + alert + re-enable flow (`../06-async-and-scale/background-jobs.md` §3).
- **At-least-once + event IDs:** customers get an event ID to deduplicate with; document it loudly.
- **Dispatch reliability:** events publish from the outbox after commit (`../06-async-and-scale/events-and-messaging.md` §4 — the same dual-write problem, now your problem as sender).
- **Endpoint management UI/API:** customers set URLs, rotate secrets (dual-secret overlap windows), see delivery logs, request replays — this is product surface, budget it.
- **SSRF discipline on customer URLs:** your dispatcher fetches customer-supplied URLs — allowlist/blocklist policy (block internal ranges, metadata IPs — `../04-security/security-foundations.md` §3 SSRF row), response-size and redirect caps. Customer webhook URLs are a classic SSRF vector into your own infrastructure.
- Don't send secrets/PII in payloads you can't guarantee the receiver's handling of; sign-then-send, document retention.

## 4. Observability & Testing

- Inbound: per-provider metrics (delivery rate, verification failures, processing outcomes, duplicate rate, lag), raw-event retention window for debugging, alert on verification-failure spikes (attack or rotation drift).
- Outbound: per-endpoint delivery success/latency/final-failure, retry queues depth, DLQ alerts; customer-facing delivery log.
- Tests: signature verification (valid/invalid/replayed/rotated), duplicate delivery no-op, out-of-order processing, slow-processing timeout behavior (must still 200), dispatcher retry schedule, endpoint disable/re-enable. Failure-path battery applies fully (`../09-testing/testing-strategy.md` §5).

## 5. Failure Modes

- **The slow handler:** signature check + full business processing inline; provider times out; retries stack; duplicates pour into a system too busy to dedup. Verify-fast-process-async (§2.1).
- **Parse-before-verify:** JSON parsed and re-serialized before HMAC check — signature never matches, endpoint broken for everyone, or worse, verification silently skipped "because it kept failing."
- **500-on-processing-failure:** provider retries an already-accepted event; duplicate side effects compound the original failure (§2.1).
- **Replay-blind:** valid signature, stale timestamp — captured event re-delivered yesterday's state transition onto today's data (§2.2.2).
- **Sequence assumptions:** state machine missing gates; out-of-order redelivery un-captures a captured payment.
- **Webhook-as-truth:** acting on event payload without provider-API confirmation — the lost-event and out-of-order cases both corrupt state (§2.3.3).
- **Outbound hammer:** no terminal-failure state; dead endpoint retried forever; queue backs up; real deliveries delay for everyone (§3.2).
- **SSRF via customer URLs:** dispatcher happily POSTs to `http://169.254.169.254/…` because a "customer" registered it — internal metadata exfiltration through your own webhook sender.
