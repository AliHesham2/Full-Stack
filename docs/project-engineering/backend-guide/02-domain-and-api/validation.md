# Validation — Boundaries, Layers, and Trust

> **Part of:** Domain & API — Backend Project Guide
> **Read when:** Handling ANY input from outside your process: request bodies, query params, headers, file uploads, webhook payloads, config, and data from other services.
> **Related:** `api-design.md`, `domain-logic.md`, `../04-security/security-foundations.md`, `../05-data/data-architecture.md` § constraints
> **Status:** Stable principles

---

## 1. Position

Validation is not a feature of endpoints; it is a property of **boundaries**. Every byte crossing into your process from a less-trusted zone gets validated. The question is never "should we validate" but "which kind, at which layer, with what authority."

## 2. The Two Kinds (Never Confuse Them)

| | **Syntactic (shape) validation** | **Business (semantic) validation** |
|---|---|---|
| Question | "Is this well-formed data?" | "Is this action correct/allowed in current state?" |
| Examples | email format; `qty` is int ≥ 1; ISO-8601 date; enum member | "enough stock"; "account not suspended"; "cancellation within window" |
| Where | Edge middleware, schema-declarative | Use cases / domain (`domain-logic.md`) |
| Mechanism | Schema library (zod/joi/pydantic/etc. per ecosystem guide) | Code + tests, often needs DB state |
| Failure | 400 + field errors | 403/409/422 + domain error code |
| Depends on state? | No | Usually |

Duplicating shape checks deep inside the domain is noise; skipping rule checks because "the endpoint validated it" is a breach. Each kind lives in exactly its layer, with the exceptions in §5.

## 3. Syntactic Validation — Mechanics

- **One schema library, one wrapper, applied at the edge for every input class:** body, query, params, headers where meaningful. No "trusted" internal endpoints — internal callers mis-send payloads too, and "internal" endpoints become external through proxies, gateways, or refactors.
- **Reject unknown/extra fields explicitly** (`strict()` mode or equivalent) — clients sending `{"role": "admin"}` into a signup body should get a 400, not a silent ignore that becomes a mass-assignment bug later. If you accept-and-strip instead, do it consciously and document it; rejection is the safer default.
- **Normalize after validating, not before:** validate the raw input (catch malformed UTF-8, control characters), then normalize (trim, lowercase emails, canonicalize phone numbers, timezone-coerce timestamps) and re-validate invariants if normalization can break them.
- **Types are contracts:** coerce deliberately (string `"5"` → 5 for query params) and bound everything: string lengths, array sizes, numeric ranges, nesting depth. Unbounded strings and arrays are DoS primitives.
- **Validate shapes once at the boundary, then trust your internal types.** Re-validating the same parsed object at every layer is ceremony without safety.

## 4. Trust Boundaries (Where Validation Is Mandatory)

Boundary rank (each may talk to the next-lower trust zone, and receives its data with suspicion):

```
Public clients (browsers, apps, 3rd-party consumers)
  ↘ validated
Your edge / API
  ↘ validated (contract-checked)
Your services / workers
  ↘ constraint-enforced (DB as last line)
Your database
```

Rules:

1. **Client input is hostile until validated** — including IDs (format AND existence AND ownership), enums, booleans, and anything that reaches a query, command, path, or redirect.
2. **Server-owned values are never read from the client:** prices, totals, discounts, role/permission fields, tenant identifiers, user IDs (derived from the authenticated session, not the body — unless the endpoint's domain is genuinely "act as user X," which is itself a privileged operation to authorize explicitly).
3. **Data from your own services still gets contract validation** at the consumer edge — schema drift is an outage class, and a compromised/misbehaving internal service is a real threat model (zero trust: `../04-security/security-foundations.md`).
4. **Config/environment is input too** — validated at startup (`../01-foundation/initialization.md` §2.2).
5. **Files and webhook payloads** are boundary data: see `../06-async-and-scale/file-and-object-storage.md` and `../11-specialized-concerns/webhooks.md`.

## 5. Defense-in-Depth Placement (Which Rule Lives Where)

A rule can have ONE authoritative home and several redundant enforcement points:

| Rule | Authoritative | Redundant enforcement (optional, deliberate) |
|---|---|---|
| "email must be unique" | use case + DB UNIQUE constraint | edge pre-check for fast UX (must handle race → still rely on constraint) |
| "qty ≥ 1" | edge schema | DB CHECK constraint |
| "total = Σ(line prices)" | domain computation | never trust client total |
| "status ∈ allowed set" | domain state machine | DB enum/CHECK |
| "user owns this object" | use-case authorization | RLS policy (defense layer, `../11-specialized-concerns/multi-tenancy.md`) |

The DB constraints in the right column are not "instead of" — they are the seatbelt behind the airbag. Schema-level constraints exist because application bugs and race conditions are assumptions, not possibilities (`../05-data/data-architecture.md` § constraints).

## 6. Error Design for Validation Failures

- **Aggregate:** report ALL failing fields in one response, not first-fail-only — clients (and humans) hate ping-pong validation.
- **Field-addressed:** `items[2].qty` (structured path), stable machine codes (`required`, `format`, `min`), human message last.
- **Say what would be valid** when cheap: `"expected ISO-8601 date, got '02/31/2026'"`.
- **Don't leak reconnaissance detail to attackers** on unauthenticated endpoints: "invalid credentials" (not "wrong password"), and consider generic 400s on signup-email-exists flows for sensitive products — a product decision, made explicitly.

## 7. Validation ≠ Sanitization ≠ Encoding

Three different jobs that AI agents routinely conflate:

- **Validation** (this file): reject or accept.
- **Sanitization/canonicalization:** transform input into a safe canonical form — needed for storage-bound HTML (rich text), filenames, redirect targets. Prefer allowlist-based transforms; "sanitizing" by stripping dangerous substrings is a losing arms race.
- **Output encoding:** escape FOR A TARGET CONTEXT at render/serialization time (HTML, URL, SQL identifier, shell) — the job of the template/query layer, not the validator (`../04-security/security-foundations.md` § injection family).

Validate at entry; encode at exit; sanitize only domain-specific formats that must persist transformed.

## 8. Testing Implications

- Every endpoint: one happy path + a battery of malformed inputs (type confusions, boundary values, oversized arrays/strings, unknown fields, unicode edge cases, deeply nested JSON).
- Property/fuzz tests on parsers for anything non-trivial.
- Denial tests: validation must reject what authorization would also reject (validation 400 should fire BEFORE a 500 from downstream type assumptions).

## 9. Failure Modes

- **Client-side validation as the only validation** — the classic; the server trusts the UI. Servers validate everything, always.
- **Mass assignment:** `updateUser(body)` binding whitelisted-free objects — extra fields flow into the ORM and become `role: "admin"`. Use explicit DTOs + strict schemas + field allowlists on write.
- **Validation-in-the-query:** building dynamic SQL whose shape changes with input "validity" — injection surface, not validation.
- **Double-validation drift:** the edge schema and a hand-written deep check disagree ("maxLength 100" vs 80); clients get random rejections. One home per rule.
- **Sanitize-on-input-forever:** storing HTML "sanitized" by an outdated allowlist, then rendering raw. Store the canonical form, encode at output, keep the sanitizer current (`../04-security/security-foundations.md`).
