# JavaScript Anti-Patterns

## Obsolete — no valid use case remains

### `var`
```js
// ✗
var x = 1;
// ✓
const x = 1;
```
**Why:** Function-scoped, hoisted, re-declarable. `const`/`let` provide block scoping and prevent accidental redeclaration.

### `for...in` on arrays
```js
// ✗
for (const key in arr) { ... }
// ✓
for (const item of arr) { ... }
```
**Why:** Iterates enumerable string keys including inherited ones, not numeric indices. Breaks if Array.prototype is extended.

### `arguments` object
```js
// ✗
function foo() { console.log(arguments[0]); }
// ✓
function foo(...args) { console.log(args[0]); }
```
**Why:** Not a real array (no map/filter), unavailable in arrow functions, slower in modern engines.

### `String.prototype.substr`
```js
// ✗
str.substr(1, 3)
// ✓
str.slice(1, 4)
```
**Why:** Deprecated (Annex B). `slice` handles negative indices and is universally supported.

### `escape()` / `unescape()`
```js
// ✗
escape("hello world")
// ✓
encodeURIComponent("hello world")
```
**Why:** Deprecated. Doesn't handle non-ASCII correctly.

### `setTimeout(fn, 0)` — know which one you actually want

`queueMicrotask` is **not** a faster `setTimeout`. They do opposite things, and
swapping one for the other is a real bug.

```js
// Goal: "run after the current call stack, before anything else"
queueMicrotask(() => doNext());          // ✓ microtask — no yield to the event loop

// Goal: "let the browser paint / let I/O run" (i.e. actually yield)
await scheduler.yield();                 // ✓ best, Chromium 129+ only — not Baseline
setTimeout(doNext, 0);                   // ✓ portable fallback
setImmediate(doNext);                    // ✓ Node-only, runs before timers
```

**Why it matters:** microtasks drain *completely* before the event loop advances, so
a chain of `queueMicrotask` calls starves timers, I/O and rendering forever — exactly
the starvation warned about in [async.md](./async.md) § Event loop ordering. Use
`queueMicrotask` for ordering, never for yielding.

```js
// ✓ Chunking long work so the UI stays responsive
for (const [i, item] of items.entries()) {
  process(item);
  if (i % 100 === 0) await (globalThis.scheduler?.yield?.() ?? new Promise(r => setTimeout(r)));
}
```

### `new Array(count)` without fill
```js
// ✗
const arr = new Array(3); // [empty × 3] — sparse
// ✓
const arr = Array.from({ length: 3 });
```
**Why:** Sparse arrays skip `map`, `forEach`, etc. on empty slots. Silent bugs.

### `__proto__`
```js
// ✗
obj.__proto__ = { ... };
// ✓
Object.setPrototypeOf(obj, { ... });
```
**Why:** Deprecated, performance hazard in all engines.

### `Array.prototype.sort()` without comparator
```js
// ✗
[10, 9, 8].sort() // [10, 8, 9] — lexicographic!
// ✓
[10, 9, 8].sort((a, b) => a - b) // [8, 9, 10]
```
**Why:** Default sort converts elements to strings. Produces wrong order for numbers.

### In-place mutation of shared arrays
```js
// ✗ sort/reverse/splice mutate the original — every other holder of the reference sees it change
const sorted = options.sort((a, b) => a.value - b.value);
// ✓ ES2023 copy variants leave the source untouched
const sorted = options.toSorted((a, b) => a.value - b.value);
// ✓ or copy manually
const sorted = [...options].sort((a, b) => a.value - b.value);
```
**Why:** Mutation aliases propagate bugs through every reference to the array.
→ [modern-features.md](./modern-features.md) § Copy-first array methods

### `with` statement
**Why:** Forbidden in strict mode. Unoptimizable and unreadable. No replacement needed.

## Dangerous — security or correctness hazards

### `eval()` / `new Function()`
```js
// ✗
eval(userInput);
new Function(userInput)();
```
**Why:** Arbitrary code execution. CSP blocks eval. → [security.md](./security.md)

### `innerHTML` with user input
```js
// ✗
el.innerHTML = userInput;
// ✓
el.textContent = userInput;
```
**Why:** XSS vector. → [security.md](./security.md)

### Modifying built-in prototypes
```js
// ✗
Array.prototype.myMethod = function() { ... };
```
**Why:** Breaks `for...in` on objects, conflicts with future standards, breaks third-party code.

### `Object.assign` with untrusted input
```js
// ✗
Object.assign({}, JSON.parse(userInput));
```
**Why:** Prototype pollution via `__proto__` key. → [security.md](./security.md)

## Common mistakes

### `==` with non-nullish values
```js
// ✗
if (count == "5") { ... }
// ✓
if (count === 5) { ... }
```
**Exception:** `x != null` is acceptable → [type-safety.md](./type-safety.md)

### `typeof x === 'undefined'` when optional chaining works
```js
// ✗
if (typeof obj.nested.val !== 'undefined') { use(obj.nested.val); }
// ✓
if (obj.nested?.val !== undefined) { use(obj.nested.val); }
```

### `Object.freeze` mistaken for deep immutability
```js
const config = Object.freeze({ db: { host: 'localhost' } });
config.db.host = 'evil';    // ✓ succeeds — freeze is SHALLOW
```
**Why:** `Object.freeze` locks only own properties. Nested objects stay mutable, and in
sloppy mode the write fails silently instead of throwing. Deep-freeze recursively or use
`structuredClone` at boundaries.

### JSON.stringify for deep comparison
```js
// ✗
JSON.stringify(a) === JSON.stringify(b)
// ✓
import { isDeepStrictEqual } from 'node:util';
isDeepStrictEqual(a, b);
```
**Why:** Fails on key order, `undefined` values (stripped), functions, symbols, sparse arrays, circular refs.

### Closures capturing loop variables in deferred callbacks
```js
// ✗ var — one shared binding for the whole loop
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // logs 3, 3, 3
}
// ✓ let — fresh binding per iteration
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // logs 0, 1, 2
}
```
**Why:** Callbacks run after the loop exits and read whatever the captured binding holds at that time.
→ [functions.md](./functions.md) § Closures

### Synchronous XHR
```js
// ✗
xhr.open('GET', url, false);
```
**Why:** Deprecated, blocks main thread. Use `fetch()`.

### Throwing strings
```js
// ✗
throw 'Something went wrong';
// ✓
throw new Error('Something went wrong');
```
**Why:** Strings have no stack trace, no `name`, no `cause`. Uncatchable by instance checks.

### Catching and swallowing errors
```js
// ✗
try { risky(); } catch (e) { /* silent */ }
// ✓
try { risky(); } catch (e) { logger.error('risky failed', e); throw e; }
```
**Why:** Silent swallowing makes debugging impossible.
