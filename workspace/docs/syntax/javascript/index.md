# JavaScript Knowledge System

Quick reference map. Each file is self-contained and cross-referenced.
Verified against primary sources (TC39 finished-proposals, MDN, runtime docs)
September 2026. Coercion, module-interop, prototype-pollution and float-comparison
claims were checked by running them, not read from docs.

| File | Scope |
|------|-------|
| [anti-patterns.md](./anti-patterns.md) | Obsolete, deprecated, and dangerous patterns to reject |
| [async.md](./async.md) | Async/await, event loop, fetch, cancellation, concurrency |
| [modules.md](./modules.md) | ESM rules, CJS interop, package exports |
| [type-safety.md](./type-safety.md) | Coercion rules, numbers/precision, JSDoc type checking |
| [datetime.md](./datetime.md) | Date pitfalls, timezones, Temporal |
| [functions.md](./functions.md) | `this` binding, closures, memory leaks |
| [classes.md](./classes.md) | Classes, private fields, inheritance, `instanceof` |
| [security.md](./security.md) | XSS, prototype pollution, injection, ReDoS, Trusted Types, supply chain |
| [regex.md](./regex.md) | `lastIndex` state, `matchAll` vs `match`, flags, dynamic patterns |
| [modern-features.md](./modern-features.md) | Shipped features worth adopting (ES2024+) |
| [runtime-gotchas.md](./runtime-gotchas.md) | Node/Bun/Deno behavioral differences |

## Core principles

1. **ESM only** for new code → [modules.md](./modules.md)
2. **`===` always** (one exception) → [type-safety.md](./type-safety.md)
3. **async/await always**; never trust `fetch` to reject on HTTP errors → [async.md](./async.md)
4. **Never put user input in DOM sinks** → [security.md](./security.md)
5. **`const` by default**; never mutate shared arrays/objects → [anti-patterns.md](./anti-patterns.md)
6. **Treat `Date` string parsing as broken**; prefer explicit construction or Temporal → [datetime.md](./datetime.md)
7. **Never reuse a `/g` regex for `test()`** — it carries state → [regex.md](./regex.md)
8. **Never use `toFixed` for money**; integer minor units + `Intl` → [type-safety.md](./type-safety.md)

## Deliberately out of scope

Excluded as low-value for an AI agent reference (per the research plan):
basic syntax tutorials, "clean code" platitudes, framework-specific content,
detailed API catalogs (MDN exists), polyfill guides, bundler configuration,
testing framework specifics, DOM API references, language history.

## Version policy

- Features are labeled by the **edition they publish in**, per
  [TC39 finished-proposals](https://github.com/tc39/proposals/blob/main/finished-proposals.md).
  Stage 4 in March of year N means edition N+1 — Temporal is ES2027, not ES2026.
- Spec edition ≠ availability. Several ES2026/ES2027 features shipped in engines years
  earlier; always feature-detect rather than trusting the year.
- Proposals below Stage 4 are explicitly flagged; withdrawn ones are kept only to stop
  agents planning around them.
- Runtime-specific behavior lives in [runtime-gotchas.md](./runtime-gotchas.md).
- Before adopting anything marked experimental, re-check the [TC39 proposals list](https://github.com/tc39/proposals) and [MDN compatibility tables](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference).
