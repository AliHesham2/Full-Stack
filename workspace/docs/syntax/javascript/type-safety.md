# JavaScript Type Safety

## Coercion rule: `===` always

```js
if (x === null) { ... }
if (count === 0) { ... }
if (status === 'active') { ... }
```

### The one exception: nullish check
```js
if (x != null) { ... }  // true when x is neither null nor undefined
// Equivalent to: x !== null && x !== undefined
```
**Why safe:** `null == undefined` is `true` by spec, and no other value equals null with `==`.

**Do not extend** to other `==` comparisons:

| Expression | Result | Why |
|---|---|---|
| `0 == ''` | `true` | Both coerce to `0` |
| `'' == false` | `true` | Both coerce to `0` |
| `null == false` | `false` | null only == undefined |
| `[] == false` | `true` | Array → `''` → `0` |
| `[0] == false` | `true` | Array → `'0'` → `0` |

→ Full table: [ECMA-262 §7.2.15](https://tc39.es/ecma262/#sec-abstract-equality-comparison)

### Three equality algorithms, not two

`===` is not the strictest comparison. Verified in Node:

| | `===` | `Object.is` | SameValueZero (used by collections) |
|---|---|---|---|
| `NaN` vs `NaN` | `false` | `true` | `true` |
| `0` vs `-0` | `true` | `false` | `true` |

```js
[NaN].includes(NaN);   // true   — includes uses SameValueZero
[NaN].indexOf(NaN);    // -1     — indexOf uses ===
new Set([NaN, NaN]).size;          // 1  — deduped
new Map().set(-0, 'a').get(0);     // 'a' — -0 and 0 are the same key
```

**Rules**:
- Testing for `NaN`: `Number.isNaN(x)`, never `x === NaN`.
- Searching arrays for possibly-`NaN` values: `includes`, never `indexOf`.
- `-0` only matters where sign carries meaning (velocities, deltas, `1/x`); use
  `Object.is(x, -0)` to detect it.

## Type checking

### Primitives: `typeof`
```js
typeof x === 'string' | 'number' | 'boolean' | 'undefined' | 'symbol' | 'bigint'
```

### Arrays: `Array.isArray()`
```js
Array.isArray(x)  // ✓
x instanceof Array // ✗ fails across realms (iframes, vm modules)
```

### Class instances: `instanceof` (same realm only)
```js
if (err instanceof TypeError) { ... }
```

### Objects: duck typing
```js
if (x && typeof x.then === 'function') { /* thenable */ }
```
Don't use `typeof x === 'object'` for object checks — `typeof null === 'object'`.

## Numbers & precision

All JS numbers are IEEE-754 doubles. These rules prevent silent bugs.

### Never compare floats with `===`
```js
0.1 + 0.2 === 0.3                 // false!
```

`Math.abs(a - b) < Number.EPSILON` is the answer everyone repeats, and it is only
correct for values near 1. `Number.EPSILON` is an *absolute* gap sized for the range
[1, 2), so it misjudges both ends — verified in Node:

```js
Math.abs((1e10 + 0.1) - 1e10) < Number.EPSILON   // false — reports "different", diff is 0.1
Math.abs(1e-20 - 2e-20)       < Number.EPSILON   // true  — reports "equal", they differ 2×
```

```js
// ✓ Relative comparison — scales with magnitude
function almostEqual(a, b, rel = 1e-9) {
  if (a === b) return true;                     // same-signed Infinity
  if (!Number.isFinite(a) || !Number.isFinite(b)) return false; // NaN/opposite infinities
  return Math.abs(a - b) <= rel * Math.max(Math.abs(a), Math.abs(b));
}
```

**Better still: don't compare floats at all.** Money → integer minor units (below).
Anything with a fixed precision → integers plus a scale factor.

### Money: work in integer minor units
```js
// ✗ Floating cents drift
19.99 * 3                         // 59.97000000000001
// ✓ Integer cents (or BigInt)
1999 * 3                          // 5997 — format only at display time
```

### `toFixed` does not round the way you expect
```js
(1.005).toFixed(2);   // '1.00'  ← not '1.01'
(2.675).toFixed(2);   // '2.67'  ← not '2.68'
(1.5).toFixed(0);     // '2'
(2.5).toFixed(0);     // '3'
(1e21).toFixed(2);    // '1e+21' ← exponential, breaks any string parsing downstream
Math.round(-1.5);     // -1      ← rounds toward +Infinity, not away from zero
```
The stored double for `1.005` is slightly *below* 1.005, so rounding down is correct
arithmetic on the actual value — the display is what lies. `Intl.NumberFormat` uses a
different (decimal-correct) algorithm and returns `'1.01'` for the same input.

**Rules**:
- Never use `toFixed` for money. Keep integer minor units and format with `Intl`.
- Never use `toFixed` output as a number (`+x.toFixed(2)`) — you get the float back with
  a new rounding error on top.
- Above 1e21 `toFixed` switches to exponential notation.

### Safe integer range
```js
Number.MAX_SAFE_INTEGER           // 2^53 − 1 = 9007199254740991
JSON.parse('{"id": 9007199254740993}').id  // precision silently lost
// IDs above 2^53 (snowflake-style APIs) must travel as strings or BigInt
```

### parseInt needs a radix; `Number()` has its own traps
```js
parseInt('08', 10)   // ✓ always pass the radix
Number('08')         // 8 — but Number('') === 0 and Number(null) === 0!
```

### NaN and Infinity checks
```js
NaN === NaN            // false
Number.isNaN(x)        // ✓ true only for actual NaN
isNaN(x)               // ✗ coerces first: isNaN('abc') === true
Number.isFinite(x)     // ✓ no coercion
isFinite(x)            // ✗ isFinite('42') === true
```

### BigInt beyond 2^53
```js
9007199254740993n     // exact
// Cannot mix BigInt and Number in arithmetic; JSON.stringify throws on BigInt
```

## Sorting and comparison

`Array.prototype.sort` is **stable** since ES2019 — equal elements keep their input
order, so multi-key sorts can be layered.

Default sort compares **UTF-16 code units**, not letters:

```js
['b', 'a', 'Z'].sort();                              // ['Z', 'a', 'b']  — uppercase first
['z', 'ä'].sort();                                   // ['z', 'ä']       — ä after z
['b', 'a', 'Z'].sort((a, b) => a.localeCompare(b));  // ['a', 'b', 'Z']
['z', 'ä'].sort((a, b) => a.localeCompare(b, 'de')); // ['ä', 'z']
```

For user-visible lists, use a reused `Intl.Collator` — `localeCompare` constructs one
internally on **every comparison**, which is O(n log n) collator allocations:

```js
const collator = new Intl.Collator('ar', { numeric: true, sensitivity: 'base' });
names.sort(collator.compare);
```

`numeric: true` also fixes `'item10' < 'item9'` — the other classic string-sort bug.
→ [datetime.md](./datetime.md) for the rest of `Intl`.

## Nullish coalescing vs OR

```js
const value = input ?? fallback;  // fallback only if null/undefined
const value = input || fallback;  // fallback if any falsy (0, '', false)
```
**Rule:** Use `??` by default. Use `||` only when you intentionally treat falsy as empty (e.g., `displayName || username`).

## Optional chaining

```js
const city = user?.address?.city;
const name = arr?.[0]?.name;
const result = obj.method?.();
```
Don't over-chain — if the base is expected to exist, let it fail:
```js
// ✗ Masks bugs
definitelyExists?.maybeNot?.value;
```

## What `JSON.stringify` silently drops

```js
JSON.stringify({ a: undefined, b: () => {}, c: Symbol('s'), d: 1 });  // '{"d":1}'
JSON.stringify([undefined, , 1]);                                    // '[null,null,1]'
JSON.stringify({ toJSON: () => 'custom' });                          // '"custom"'
JSON.stringify({ n: 1n });                                           // TypeError
```

- Object keys with `undefined`/function/symbol values **vanish**; the same values inside
  an **array** become `null` instead, so array length is preserved but data is not.
- A `toJSON` method anywhere in the tree hijacks serialization — this is why `Date`
  becomes a string, and why a class instance may serialize unexpectedly.
- `BigInt` throws. Convert explicitly, or define `toJSON`.
- Prototype chain, non-enumerable properties, and `Map`/`Set` contents are all lost.

**Rule:** `JSON.parse(JSON.stringify(x))` is not a deep clone. Use `structuredClone`
→ [modern-features.md](./modern-features.md).

## JSDoc type checking

For projects not using TypeScript.

### Syntax check ≠ type checking
```bash
node --check file.js   # validates SYNTAX ONLY — parse errors, nothing about types
```

### Actual type checking of JSDoc annotations
```bash
tsc --noEmit --allowJs --checkJs src/    # full project type check of .js files
```
Editors (VS Code) surface these errors live through the built-in TypeScript server — no build step needed.

### Basic pattern
```js
// @ts-check
/**
 * @param {string} name
 * @returns {string}
 */
function greet(name) {
  return `Hello, ${name}`;
}
greet(42);  // Error: number not assignable to string
```

### Useful patterns
```js
/** @typedef {{ id: number; name: string }} User */

/** @type {Map<string, User>} */
const users = new Map();

/**
 * @param {User} user
 * @param {Partial<User>} updates
 * @returns {User}
 */
function updateUser(user, updates) {
  return { ...user, ...updates };
}
```

### JSDoc vs TypeScript

| Factor | JSDoc | TypeScript |
|---|---|---|
| Migration cost | Zero (add to .js) | Rename .ts, fix errors |
| Complex generics | Gaps | Full |
| IDE support | Good (VS Code) | Excellent |
| Build step | None (or optional tsc) | Required (tsc) |
| Best for | Adding types to existing JS | New projects, complex types |

**Rule:** New projects → TypeScript. Existing JS → JSDoc incrementally.
