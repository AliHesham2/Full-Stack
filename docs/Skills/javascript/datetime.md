# JavaScript Dates, Times & Timezones

The legacy `Date` API is a top bug source. Rules below prevent the classics;
Temporal (now shipping) replaces most of it.

## Legacy Date traps

### Months are 0-indexed
```js
new Date(2026, 8, 23)   // September 23, 2026 — NOT August
new Date(2026, 12, 1)   // January 2027 — month rolls over
```

### String parsing is inconsistent
```js
new Date('2026-08-23')           // UTC midnight (date-only ISO → Z)
new Date('2026-08-23T10:00:00')  // LOCAL time (ISO without offset)
new Date('2026-08-23 10:00:00')  // implementation-dependent — avoid entirely
new Date('08/23/2026')           // parser/locale dependent — avoid
```
**Rule:** Never parse dates from strings with `new Date(str)` except strict ISO including an explicit offset (`Z` or `±hh:mm`). Construct explicitly from parts: `new Date(y, m, d)`.

### Month overflow when mutating
```js
const d = new Date(2026, 0, 31);  // Jan 31
d.setMonth(1);                    // Mar 3 — Feb 31 doesn't exist, rolls forward
```
**Rule:** Clone before mutating: `const copy = new Date(original)`.

### `toISOString` is always UTC
```js
new Date(2026, 7, 23, 10).toISOString()  // '2026-08-23T07:00:00.000Z' on a UTC+3 machine
```
The `Z` suffix means zero-offset. Never slice it off and treat the rest as local time.

### `getTimezoneOffset` sign is inverted
```js
new Date().getTimezoneOffset()  // UTC+3 → returns -180 (minutes BEHIND UTC)
```

### Measuring elapsed time
```js
const t0 = performance.now();    // ✓ monotonic clock, sub-millisecond
doWork();
const ms = performance.now() - t0;

Date.now()                       // ✗ wall clock — NTP syncs can jump it backward
```

### DST arithmetic
Adding 24 hours ≠ next day near a DST transition (23- or 25-hour days).
```js
// ✗ Breaks twice a year
const tomorrow = new Date(now.getTime() + 24 * 60 * 60 * 1000);
```
Use calendar-aware libraries (date-fns, Luxon) or Temporal for date math.

## Temporal — the replacement (Stage 4 March 2026, **ES2027** edition)

Shipped: Firefox 139+ (May 2025), Chrome/Edge 144+ (Jan 2026), Node.js 26+ (unflagged).
NOT yet: **Safari/WebKit** — and every iOS browser is WebKit, so this is not a
"Safari desktop only" gap. Also missing in Bun and Deno.

Practical coverage is roughly 70% of browser traffic as of mid-2026: usable on the
server today, still polyfill-dependent on the web.

```js
Temporal.PlainDate.from({ year: 2026, month: 8, day: 23 });  // months are 1-indexed here!
Temporal.ZonedDateTime.from('2026-08-23T10:00[Africa/Cairo]');
Temporal.Now.instant();

d.add({ days: 1 });              // DST-safe calendar arithmetic
d.with({ hour: 0 }).toString();
```

Key properties: immutable value types, months 1-indexed, explicit timezones, calendar-aware arithmetic.

**Rule:** Feature-detect `typeof globalThis.Temporal !== 'undefined'`; fall back to `temporal-polyfill` (smaller, actively maintained) or Luxon until WebKit ships. Do not label Temporal "ES2026" — it missed that cutoff.
→ Status: [TC39 proposal-temporal](https://github.com/tc39/proposal-temporal)

## Formatting for users: `Intl`

Never hand-assemble dates or money with string slicing/manual symbols — locale conventions differ everywhere:

```js
new Intl.DateTimeFormat('ar-EG', { dateStyle: 'long' }).format(temporalOrLegacyDate);
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(5997 / 100);
new Intl.NumberFormat('de-DE', { maximumFractionDigits: 2 }).format(1234.5);  // '1.234,5'
```
Works with both `Date` and Temporal types. Convert integer minor units only here at display time (→ [type-safety.md](./type-safety.md) § Money). Create formatters once and reuse — construction is expensive.

## Quick decision table

| Task | Tool |
|---|---|
| Elapsed duration (monotonic) | `performance.now()` |
| Wall-clock timestamp | `Date.now()` |
| Date-only value (no timezone) | `Temporal.PlainDate`, else y/m/d integers |
| Point in time (absolute) | `Temporal.Instant`, else ISO string WITH offset |
| Scheduling future work | `setTimeout`, never date math |
| Serialization | ISO 8601 with explicit offset |
