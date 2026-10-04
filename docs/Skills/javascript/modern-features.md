# Modern JavaScript Features (ES2024+)

Shipped in current browsers/runtimes and worth adopting.
For proposals not yet shipped, see bottom.

> **How the ES year is assigned**: a proposal lands in the edition whose spec text
> was still open when it hit Stage 4. The cutoff is roughly March each year, so a
> Stage-4 approval in March 2026 publishes in **ES2027**, not ES2026. Years below
> follow [TC39 finished-proposals](https://github.com/tc39/proposals/blob/main/finished-proposals.md),
> which is authoritative — blog posts routinely get this wrong.

## Grouping

### `Object.groupBy()` / `Map.groupBy()` (ES2024)
```js
const grouped = Object.groupBy(items, i => i.category);
// { electronics: [...], books: [...] }

const byId = Map.groupBy(items, i => i.id); // non-string keys
```
**Replaces:** `reduce`-based grouping.
**Caveat:** `Object.groupBy` returns null-prototype objects (no inherited properties like `toString`).

## Set operations (ES2025)

```js
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

a.union(b);              // Set {1, 2, 3, 4}
a.intersection(b);       // Set {2, 3}
a.difference(b);         // Set {1}
a.symmetricDifference(b);// Set {1, 4}
a.isSubsetOf(b);         // false
a.isSupersetOf(b);       // false
a.isDisjointFrom(b);     // false
```
The first four return new Sets without mutating inputs. isSubsetOf, isSupersetOf and isDisjointFrom return booleans.

## Promise utilities

### `Promise.withResolvers()` (ES2024)
```js
const { promise, resolve, reject } = Promise.withResolvers();
```
**When:** Event-emitter or callback APIs needing external resolve/reject.
**Full patterns:** → [async.md](./async.md)

### `Promise.allSettled()` (ES2020)
```js
const results = await Promise.allSettled([fetchA(), fetchB()]);
```
→ [async.md](./async.md) for usage patterns.

## Array methods

### `Array.prototype.at()` (ES2022)
```js
arr.at(-1); // last element
arr.at(0);  // first
```

### `Array.prototype.findLast()` / `findLastIndex()` (ES2023)
```js
arr.findLast(x => x.id === target);
```

### Copy-first array methods (ES2023)
```js
const sorted   = arr.toSorted(cmp);      // sort() mutates in place!
const reversed = arr.toReversed();
const spliced  = arr.toSpliced(0, 1);
const replaced = arr.with(2, newValue);  // copy with index 2 replaced
```
Use these whenever the source array might be shared state.
→ [anti-patterns.md](./anti-patterns.md) § In-place mutation of shared arrays

### `Array.fromAsync()` (ES2026)
```js
const arr = await Array.fromAsync(asyncIterable);
```
Shipped well ahead of its spec edition — available in Node 22+, Chrome 121+, Firefox 115+, Safari 18.4+.

## Iterator helpers (ES2025)

```js
const result = items.values()
  .filter(x => x.active)
  .map(x => x.id)
  .take(10);
// Lazy — nothing executes until iteration begins
```
Works on any iterable via `Iterator.from()`. Available across all major engines since their 2024–2025 releases; verify older support targets before using.

## Object utilities

### `Object.hasOwn()` (ES2022)
```js
Object.hasOwn(obj, 'key');
// Replaces: Object.prototype.hasOwnProperty.call(obj, 'key')
```

### `structuredClone()` (2022, all environments)
```js
const clone = structuredClone(original);
```
**Supports:** Arrays, plain objects, Map, Set, Date, RegExp, ArrayBuffer, Transferables.
**Cannot clone:** Functions, DOM nodes (browser), symbols as keys.

Transfer instead of clone for performance:
```js
const clone = structuredClone(original, { transfer: [original.buffer] });
```

## String utilities

### `String.prototype.isWellFormed()` / `toWellFormed()` (ES2024)
```js
'surrogate\uDC00alone'.isWellFormed();  // false
'surrogate\uDC00alone'.toWellFormed();  // 'surrogate\uFFFDalone' — replaces the lone surrogate, does not delete it
```
**When:** Validating strings before APIs requiring well-formed UTF-16 (`TextEncoder`, `fetch`).

## Binary ↔ base64/hex (ES2026)

```js
const b64  = new Uint8Array([104, 105]).toBase64();   // 'aGk='
const back = Uint8Array.fromBase64('aGk=');

const bytes = new Uint8Array([0xde, 0xad]);
bytes.toHex();        // 'dead'
Uint8Array.fromHex('dead');  // Uint8Array [222, 173]

// UTF-8 text → base64 without btoa's Unicode crash:
const enc = new TextEncoder().encode('héllo').toBase64();
```
Replaces `btoa`/`atob`, which fail on non-Latin1 text and return awkward binary strings. Options support `alphabet: 'base64url'`.
**Support:** Chrome/Edge 140+, Firefox 133+, Safari 18.2+, Node 24+ — verify older runtime targets first (`typeof Uint8Array.prototype.toBase64 === 'function'`).

## Regular expressions

### `RegExp.escape()` (ES2025) + `v` flag (ES2024)
```js
const pattern = new RegExp(`${RegExp.escape(userInput)}\\d+`, 'v');
```
`RegExp.escape()` neutralizes special characters when interpolating dynamic values into patterns — prevents both broken regexes and syntax-injection surprises. The `v` flag supersedes `u` with Unicode property sets and set operations (`[\p{Letter}&&\p{ASCII}]`, `--` subtraction); POSIX classes such as `[[:alpha:]]` are not JavaScript syntax.
**Caveat:** `matchAll`/named groups remain the tools for extracting repeated matches — don't build parse-heavy logic on escaped dynamic patterns.

## Temporal API (Stage 4 March 2026 → **ES2027** edition)

Shipped: Firefox 139+ (May 2025), Chrome/Edge 144+ (Jan 2026), Node.js 26+ (unflagged).
Pending: **Safari/WebKit** — which also means every iOS browser. Bun, Deno.

> Many articles call Temporal "ES2026". It missed the ES2026 cutoff by weeks;
> [TC39 lists it under ES2027](https://github.com/tc39/proposals/blob/main/finished-proposals.md).
> The edition label does not affect availability — feature-detect instead.

Immutable date/time types with explicit timezones and calendar-safe arithmetic. Replaces `Date`.
→ Full rules and migration guidance: [datetime.md](./datetime.md)

## Shipped but niche

### `Float16Array` (ES2025)
Half-precision typed array for ML/audio workloads. All major engines support it; Node/Bun/Deno via V8/JSC builds from mid-2025 on. Verify runtime version before use.

## Also finished — commonly missed

### Explicit Resource Management: `using` / `await using` (ES2027)
```js
class Handle {
  [Symbol.dispose]() { this.close(); }
}
{
  using h = open();          // h.close() runs when the block exits — even on throw
}                            // ...replaces try/finally for cleanup

await using conn = await pool.acquire();   // needs [Symbol.asyncDispose]
```
V8 shipped it in Chrome 134 / Node 24. TypeScript 5.2+ downlevels it. Feature-detect with `typeof Symbol.dispose !== 'undefined'`.

### Import attributes + JSON modules (ES2025)
```js
import config from './config.json' with { type: 'json' };
const mod = await import('./data.json', { with: { type: 'json' } });
```
The old `assert { type: 'json' }` syntax was **removed** — code using `assert` throws.

### `Promise.try()` (ES2025)
```js
Promise.try(() => maybeThrowsSynchronously(x));   // sync throws become rejections
```
Removes the `new Promise(r => r(fn()))` dance when a function may throw before returning a promise.

### Small ES2026 additions
```js
Error.isError(x);                    // cross-realm-safe, unlike instanceof Error
Math.sumPrecise([0.1, 0.2, 0.3]);    // exact float summation, no drift
map.getOrInsert(key, defaultValue);  // "upsert" — replaces get-check-set
map.getOrInsertComputed(key, () => expensive());
JSON.parse(text, (k, v, ctx) => ctx.source);   // raw source text, preserves big-number precision
```

## Not shipped — do not use in production

| Feature | Stage | Notes |
|---|---|---|
| Decorators | 3 | Native in no engine yet. TypeScript `experimentalDecorators` is the *old* semantics — not the same thing. |
| Pattern Matching | 1 | Not shipped. |
| Composites | 1 | Successor to Records & Tuples; objects compared by value, no new primitives. |
| ~~Records & Tuples~~ | **WITHDRAWN** | Killed at the April 2025 TC39 plenary — no new primitives. Do not plan around it. Use `Object.freeze` / structural comparison, or watch Composites. |

**Rule:** Check [TC39 proposals](https://github.com/tc39/proposals) and [MDN compatibility](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference) before adopting any stage <4 feature.
