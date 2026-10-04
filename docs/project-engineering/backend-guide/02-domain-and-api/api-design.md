# API Design — Contracts, Resources, and Evolution

> **Part of:** Domain & API — Backend Project Guide
> **Read when:** Designing or changing any HTTP API surface: resources, status codes, errors, pagination, versioning, idempotency, rate limits.
> **Related:** `domain-logic.md`, `validation.md`, `../03-auth/authorization.md`, `../05-data/concurrency-and-idempotency.md`, `../11-specialized-concerns/webhooks.md`
> **Status:** Stable conventions · Grounded in current standards (RFC 9110 HTTP semantics; RFC 9745 `Deprecation` + RFC 8594 `Sunset` header fields; OWASP API Security Top 10 2023 — still the current edition, verified)

---

## 1. The Contract Mindset

An API is a **promise**, not an implementation detail. Every field you return will eventually be depended upon by a client you don't control (including your own frontend, other services, and increasingly AI agents consuming APIs programmatically). Design rules follow from this: be deliberate about what enters the contract, keep the contract boring and predictable, and change it only with explicit evolution strategy (§9).

## 2. Resource Design (REST Defaults)

- **Nouns, not verbs:** `POST /orders` not `POST /createOrder`. Actions that don't fit CRUD: model as state transitions on the resource — `POST /orders/{id}/cancellation` (creates a cancellation) — or, sparingly, a sub-resource action `POST /orders/{id}/refund` when the domain genuinely has a command-like operation.
- **Plural collections:** `/orders/{id}/items`.
- **Relationships via nesting only when ownership is strong and lifecycle is contained** (`/orders/{id}/items`). Otherwise, flat resources with filter links (`/items?order_id=X`) — deep nesting (`/users/{uid}/orders/{oid}/items/{iid}/discounts`) is URL-as-branch-logic.
- **IDs in URLs are opaque:** no sequential integers in public APIs (enumeration attack surface + coupling); prefer UUIDs — UUIDv7/ULID for index locality (`../11-specialized-concerns/time-and-money.md` § identifiers).
- **Consistent casing (snake_case or camelCase — pick once)** across paths, fields, and query params. Mixed conventions are a contract defect.

## 3. Status Codes — A Working Minimum

Use the smallest consistent set; clients must be able to branch on them:

| Code | Meaning in YOUR API | Notes |
|---|---|---|
| 200 | read/update succeeded | |
| 201 | created | with the new resource (or its location) |
| 204 | succeeded, no body | deletes, void actions |
| 400 | malformed request (shape/syntax) | never for business refusals |
| 401 | not authenticated / bad credentials | don't reveal whether the account exists |
| 403 | authenticated but not allowed | policy refusal — also the correct code for "not your object" |
| 404 | resource doesn't exist **or** you deliberately hide existence | prefer 404 over 403 for other-tenant objects when existence itself is sensitive |
| 409 | state conflict with current resource state | "already shipped" |
| 410 | gone (deliberately removed) | rare but useful for hard-deprecated endpoints |
| 422 | well-formed but semantically rejected | business-rule refusals (when not 409) |
| 423 | locked | rare: concurrent edit locks |
| 428 | precondition required | force clients into conditional requests for dangerous ops |
| 429 | rate-limited | MUST include `Retry-After` |
| 500 | our bug | logged with correlation ID; generic body |

Anti-rules: never 200-with-`{error:...}` body; never 4xx for internal failures; never vary semantics per endpoint. Payment-required (402) remains non-standard — don't use outside its narrow provider conventions.

## 4. Errors — One Shape, Everywhere

Adopt ONE error envelope for the whole API. In this project it is fixed (`../03-auth/authentication.md` §0), and Next.js's `ApiError` parses exactly this shape:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": { "items.0.qty": ["qty must be >= 1"] }
  }
}
```

A domain refusal (§3: 403/409/422) has no `details`, e.g. a 409:
`{ "error": { "code": "ORDER_ALREADY_SHIPPED", "message": "Order ord_123 was shipped and can no longer be modified." } }`

`details` appears only for validation errors (400). The request ID is not in the body: it travels in the `X-Request-Id` response header, and the logs are keyed by it.

Rules:

- **Machine-readable codes, human-readable message.** Clients branch on `code`, display `message`.
- **No internals in bodies, in any environment:** no stack traces, SQL, class names, or dependency names. Those go to logs keyed by the request ID.
- Validation errors are aggregated (all fields at once), field-addressed (the `details` keys are the field paths), and stable in code vocabulary.
- Map domain errors → HTTP **centrally** (`../07-reliability/error-handling.md` §4), never ad-hoc per handler.

## 5. Reading Data: Pagination, Filtering, Sorting

- **Everything list-shaped is paginated from day one.** Retrofitting pagination breaks clients and grows unbounded queries silently (`../12-anti-patterns/anti-patterns.md` § unbounded queries).
- **Offset vs cursor:** offset (`?limit=50&offset=100`) is fine for admin screens and small sets (≲100k rows). For large/growing collections or infinite-scroll UX, use opaque cursor tokens (keyset pagination): constant-time, stable under inserts. Research-grounded rule of thumb: above ~100k rows, offset degrades badly (deep pages scan everything they skip).
- **Cursors are opaque and signed/encoded** — never expose raw `WHERE` fragments via query params (injection surface).
- **Filtering whitelist:** `?status=shipped&created_after=2026-01-01` — the API defines the filterable vocabulary. Never forward arbitrary query params into queries.
- **Sorting whitelist** with a default (`sort=created_at:desc`) and a tiebreaker column to make pagination deterministic.
- **Sparse fieldsets** (`?fields=id,status,total`) only if payload size is a measured problem — it complicates caching and typing.

### Representation & content negotiation

- **JSON, UTF-8, one media type (`application/json`) is the default** — negotiate alternatives only when a real second representation exists (CSV export endpoints, binary downloads): a separate endpoint per format is simpler and cache-friendlier than `Accept`-switching on the same URL.
- **Versioned vendor media types vs plain JSON:** prefer plain JSON + additive evolution (§9); vendor media types (`application/vnd.acme.v2+json`) only for genuinely divergent representations — they complicate every client, cache, and proxy involved.
- **Locale:** `Accept-Language` is a hint for anonymous content; authenticated users' locale comes from their profile. Backend serves canonical machine formats (ISO-8601 UTC, ISO-4217, unformatted numbers — `../11-specialized-concerns/time-and-money.md`) and localized content as data fields with a fallback chain (`title_en`, `title_ar` → default). Presentation formatting (date/number rendering) is the client's job.
- **Compression** (gzip/brotli) negotiated at the edge for large payloads; measure before enabling per-endpoint (CPU trade on hot paths).

## 6. Writing Data

- **Create:** `POST /orders` with body → 201 + representation (or `Location` header). Validate at the edge (`validation.md`).
- **Partial updates: `PATCH` with a restricted patch vocabulary** (JSON-Merge-Patch semantics: fields present = set, absent = untouched). `PUT` = full replacement, and means it: missing fields are REMOVED — choose deliberately, document it.
- **Never accept server-owned values from clients:** prices, totals, permissions, tenant IDs, user IDs (unless the resource is genuinely client-owned). Recompute server-side; treat client values as untrusted hints (`validation.md` § trust boundaries).
- **Dangerous/irreversible operations** may require a precondition: `If-Match` on the current `ETag`/version — turns "are you sure" into a machine-checked guard, and maps cleanly to optimistic locking (`../05-data/transactions.md` §6).

### Idempotency

- **PUT/DELETE are idempotent by contract** — implement them so (retrying either leaves the same state; deleting a missing resource can return 404 or 204, pick and document).
- **POST needs help:** for operations where a retry must not double-execute (payments, order creation), accept an `Idempotency-Key` header: store key + request hash + response for a TTL; same key ⇒ same stored response; same key + different body ⇒ 422 error. This is the Stripe-proven pattern. Scope keys per (endpoint, authenticated caller). See `../05-data/concurrency-and-idempotency.md` for the storage/race mechanics.

## 7. When REST Is Not the Fit

| Need | Consider | Why |
|---|---|---|
| Server→client push | WebSockets/SSE (`../06-async-and-scale/realtime.md`) | HTTP request/response can't push |
| High-frequency small reads from own frontend | BFF endpoints / GraphQL read layer (measured need only) | N REST round-trips collapse; but GraphQL adds complexity + auth surface — justify it (`../01-foundation/complexity-control.md`) |
| Internal service-to-service procedure calls | RPC-style JSON or gRPC per ecosystem guide | Lower ceremony inside a trust boundary you control |
| Bulk operations | Batch endpoints (`POST /orders/batch`) with per-item results | N× single calls chatty + partial-failure ambiguity |
| Webhooks in/out | See `../11-specialized-concerns/webhooks.md` | Delivery semantics differ fundamentally |

Default remains plain REST. Each alternative is an explicit trade.

## 8. Abuse & Rate Limiting

- Rate limits are part of the contract: enforce per-identity (user, API key, IP fallback), return **429 with `Retry-After`**, and expose limit headers (`X-RateLimit-Remaining`) for well-behaved clients.
- Algorithm choice:

| Algorithm | How it works | Best for | Watch for |
|---|---|---|---|
| **Token bucket** | Bucket fills at fixed rate; each request consumes a token; burst = bucket size | APIs with burst tolerance (most APIs) | Burst size must be tuned; too generous = burst abuse; too strict = legitimate spikes rejected |
| **Sliding window log** | Timestamps of recent requests in a sorted set; count within window | Precise limits, low-burst traffic | Memory-heavy at high request rates (one entry per request); Redis sorted sets with TTL |
| **Sliding window counter** | Weighted average of current + previous window counts | Good precision, bounded memory | Approximate (±1 window worth of error); usually sufficient |
| **Fixed window** | Counter resets at fixed intervals (each minute) | Simple, low-traffic | Boundary burst: 100 requests at 23:59:59 + 100 at 00:00:00 = 200 in 1 second despite 100/min limit |

**Recommended default:** token bucket (most libraries implement it; burst-friendly; stateless per-request). Use sliding window only when fixed-window boundary burst is a real problem.

- Layered limits: coarse global + fine per-endpoint for expensive operations. Also enforce **concurrency** limits and payload size at the edge.
- Unknown-actor endpoints (login, signup, password reset) get stricter limits — they're the abuse magnets (credential stuffing).
- **Distributed rate limiting:** per-IP/per-user limits across multiple instances require a shared counter store (Redis, database). Local per-instance limits are not true limits at N instances — they're N× the intended limit. Decide whether approximate (local counters, slightly leaky) or exact (shared store, slight latency) is acceptable per endpoint.

## 9. Evolution & Versioning

- **Default: additive, non-breaking evolution.** Adding optional fields/params/endpoints needs no version bump. Clients MUST ignore unknown fields (state this in your docs; it makes YOUR future additions safe).
- **Breaking changes** (removing/renaming fields, changing types/semantics, tightening validation, altering error codes) require a version strategy:
  - Prefer **URL major version** (`/v2/orders`) — explicit, cacheable, greppable. Header/param versioning hides drift and breaks caches.
  - Deprecate loudly: the `Deprecation` header (**RFC 9745**, Standards Track) plus the
    `Sunset` header (**RFC 8594**) — these are **two different RFCs**, a distinction worth
    keeping straight because 8594 defines only `Sunset`. `Deprecation` says "this is on the
    way out"; `Sunset` gives the date it stops responding. Pair them with a changelog, usage
    metrics on the old version, a migration guide, and a dated removal. Deprecation without
    metrics is abandonment theater.
- **Internal vs public APIs:** internal APIs may evolve faster but STILL need contracts (your own services are clients too). Public APIs get the full compatibility discipline + versioned docs.
- Never make a breaking change and a migration mandatory in the same release as new features — sequence them (`../05-data/migrations.md` has the data-side analogue).

## 10. Review Checklist for Any New Endpoint

- [ ] Resource naming follows §2 conventions; casing consistent
- [ ] Status codes match §3 table; one error envelope; no internals leak
- [ ] List response paginated with whitelisted filter/sort
- [ ] Write accepts no server-owned values
- [ ] Authorization: role/ownership checked per-object (`../03-auth/authorization.md`) — BOLA is the #1 API vuln for a reason
- [ ] Validation at the edge with aggregated field errors
- [ ] Retriable POSTs (payment-creating?) take `Idempotency-Key`
- [ ] Rate limit + payload size bound applied
- [ ] Contract documented; change is additive or versioned per §9
- [ ] Tests: happy path + validation + authorization denial + conflict paths (`../09-testing/testing-strategy.md`)
