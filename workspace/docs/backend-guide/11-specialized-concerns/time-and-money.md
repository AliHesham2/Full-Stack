# Time & Money — The Two Domains Where Naive Code Silently Corrupts

> **Part of:** Specialized Concerns — Backend Project Guide
> **Read when:** Storing timestamps, scheduling future events, computing durations, handling currencies, prices, balances, or rounding.
> **Related:** `../02-domain-and-api/validation.md`, `../05-data/data-architecture.md` §6
> **Status:** Stable rules · Discovered importance: these are the two domains where "simple-looking" naive implementations are always wrong, and wrong in ways that are legally or financially material

---

## 1. TIME

### 1.1 Storage & transport rules

- **Store UTC, always** — DB columns as `timestamptz` (instant-with-zone), never naive local times. A stored "2:30 PM" without a zone is a lie waiting to be interpreted.
- **APIs exchange ISO-8601 with explicit offset** (`2026-08-26T14:30:00+03:00` or `Z`); never bare epoch seconds in business payloads (opaque, sub-second-lossy, timezone-blind in logs) and never "local time strings."
- **Never trust client clocks:** timestamps of record are SERVER-side (or DB-default) generated (`../05-data/data-architecture.md` §6); client-supplied times are hints, validated against skew if used at all.
- **Durations are not timestamps:** store as intervals/seconds/millis — a "duration" stored as two timestamps is reconstructable; a duration stored as a timestamp is nonsense.

### 1.2 Timezone logic (where bugs live)

- Convert to a user's zone **only at presentation edges**; business logic reasons in UTC (or, for calendar-anchored rules, in the event's own zone — "meeting at 9 AM Cairo time" is a wall-clock rule, not a UTC rule).
- **DST is real:** "same time tomorrow" in wall-clock terms crosses a 23/25-hour day twice a year; recurring schedules must recompute per occurrence in the target zone (a stored UTC instant for "daily 9AM" drifts by an hour after DST transitions).
- **Named zones, not offsets:** `Africa/Cairo` carries the DST rules; `+02:00` doesn't. Schedules/queries stored with the zone name; conversions via the platform's tz database (kept updated — zone rules change politically).
- Server zone set to UTC (boot config) — local-timezone servers make every log correlation and "midnight job" timezone-ambiguous.

### 1.3 Clock-time computing

- **Ordering across machines:** DB-generated timestamps for ordering within the DB; for cross-system ordering use sequence numbers/ULIDs (UUIDv7 class — monotonic-ish, sort-safe, `../05-data/data-architecture.md` §4) — wall clocks skew and NTP steps backward.
- **"Now" is injectable:** all code paths that read the clock take a clock dependency (test seam) — no `Date.now()` scattered through domain logic; time-dependent tests then run deterministically (`../09-testing/testing-strategy.md` §3).
- Scheduling granularity: cron-in-UTC + tz-aware rendering, jittered starts (`../06-async-and-scale/background-jobs.md` §3), and idempotent overlap handling.

## 2. MONEY

### 2.1 Representation rules

- **Never binary floats** (`float`/`double`) for money: 0.1 + 0.2 !== 0.3 compounds into reconciliation mismatches that are someone's full-time job. Use:
  - **Integer minor units** (cents, piastres, fils, halalas) — the default; `1999` = 19.99 — with the currency code beside it, OR
  - **Exact decimal types** (Postgres `numeric`, language decimal class) where fractional minor units exist (gas, FX, interest math).
- **Currency code is part of the amount's type:** `Money(amount, currency)` — an amount without a currency is a bug deferred; never add/subtract/compare across currencies without an explicit conversion step (its own auditable operation).
- DB columns: `BIGINT` minor units + `CHAR(3)` ISO-4217 currency (or `NUMERIC(p,s)` for decimal flows); constraints on both (positive where domain requires).

### 2.2 Computation rules

- **Client totals are decoration, never data:** server recomputes every total from line items (`../02-domain-and-api/validation.md` §4.2 server-owned values); a trusted client price is the founding error of an entire vulnerability class.
- **Rounding is a business rule, explicit per context:** line-item rounding vs total rounding differ legally (tax regulations specify); pick per jurisdiction/processor, implement in ONE place (the money domain module), and test with adversarial values (0.5-cent boundaries, negative amounts, 3-decimal currencies like KWD).
- Percentage/discount math happens in exact decimals, rounded once at the boundary the business rule names; never round intermediate results.
- **External processors are the truth for their domain:** payment-provider amounts, fee structures, and exchange rates are fetched, not computed; store the provider's identifiers with every transaction for reconciliation (`../02-domain-and-api/domain-logic.md` §6 external-call pattern).

### 2.3 Consistency & audit

- Money mutations are **append-only ledgers** where possible (transaction records with running references), not UPDATE-in-place balances — balances are then derivable, auditable, and reconcilable (`../05-data/data-architecture.md` §6; full event-sourcing is overkill — a ledger table is the 80% win).
- Every money write is transactional with its business state change (`../05-data/transactions.md` §1: debit+credit+record = one transaction).
- Idempotency on money paths is mandatory, not recommended: payment intents, refunds, transfers all carry idempotency keys or natural dedup keys (`../05-data/concurrency-and-idempotency.md` §3.3) — double-charging is the defect class that ends trust permanently.
- Reconciliation job: nightly/aggregated comparison of internal ledger vs provider statements; drift alerts; the mismatch you find in reconciliation is always smaller than the one your users find.
- **Audit trail on money events** (append-only, who/what/when — `../08-observability/observability.md` §7): regulator-grade when required, common-decency grade always.

## 3. Failure Modes

- **Float money:** the classic silent corruption — cents vanish per transaction, books drift, reconciliation fails monthly with no localizable cause.
- **Zone-less storage:** "created_at 23:59:59" — whose midnight? Daily reports double-count or skip across DST; analytics quietly wrong forever after.
- **Client-clock trust:** "expires_at" set by the client — free license extensions for anyone with a system clock.
- **The daily-9AM-as-UTC-instant:** schedule stored as one UTC time; fires an hour off for half the year; nobody notices until the region with the other DST schedule complains.
- **Cross-currency arithmetic:** adding USD cents to EUR cents because "they're both numbers" — nonsense output, plausible-looking balance.
- **Round-everywhere:** rounding at each step of a discount+tax+fee chain; the sum of published parts ≠ the charged total; support tickets with spreadsheet receipts attached.
- **In-place balance mutation:** `UPDATE accounts SET balance = balance - X` without a ledger record — history-free money: unreconcilable, unauditable, un-debuggable when (not if) a bug fires.
