# JavaScript Classes & Prototypes

## Rule: use `class` syntax, never manual prototype wiring

```js
class User {
  #id;                          // private field (ES2022) — hard privacy
  static nextId = 1;            // static property

  static {                      // static block (ES2022) — runs once, at class definition
    User.registry = new Map();
  }

  constructor(name) {
    this.name = name;
    this.#id = User.nextId++;
  }

  get id() { return this.#id; }
  toString() { return this.#format(); }
  #format() { return `${this.name}#${this.#id}`; }   // private method (ES2022)
}
```
**Why:** `class` is the standardized form. Manual `prototype` assignment and `Object.create` inheritance chains are legacy patterns that break tooling and readability.

**Note:** `#private` is enforced by the language, not by convention. Two different failures:

```js
p.#id;                       // ✗ SyntaxError at PARSE time — outside the class body it won't even load
class P { #id; static has(o) { return #id in o; } }
P.has({});                   // false — brand check, the only safe way to test from inside
```
Accessing `#id` on a non-instance *from inside* the class throws `TypeError`; accessing
it from outside is an early `SyntaxError`. TypeScript's `private` keyword is compile-time
only and erases to nothing — don't mix them up when reading code.

## Field initialization order

Instance fields initialize in declaration order, before the constructor body:

```js
class C {
  x = this.y;   // undefined — y not initialized yet
  y = 42;
}
```
**Rule:** A field referencing other state must be declared after that state — or compute lazily via a getter/method.

## Inheritance

```js
class Admin extends User {
  constructor(name, level) {
    super(name);            // REQUIRED before any `this` access — ReferenceError otherwise
    this.level = level;
  }
}
```
- Single inheritance only — prefer composition for anything deeper than one level.
- Arrow-function fields inherit the instance's `this`; regular methods bind per call site → [functions.md](./functions.md) § `this` binding
- `super.method(...)` calls the parent implementation; there is no implicit parent call.

## `instanceof` caveats

```js
user instanceof User   // walks prototype chain — same realm only
```
Fails across iframes/vm contexts and after `structuredClone` (prototype chain not copied — clone becomes a plain object). For cross-realm or cloned values, use duck typing or a tag field (`type: 'User'`). → [type-safety.md](./type-safety.md)

## Enumerability differences that bite

```js
class A { m() {} }
Object.keys(A.prototype).length;   // 0 — class methods are NON-enumerable
Object.keys({ m() {} }).length;    // 1 — object-literal methods ARE enumerable
```

Consequences:
- `{ ...instance }` copies **own instance fields only** — every method is left behind,
  and the result is a plain object. Same for `JSON.stringify`.
- Spreading also **invokes** getters and stores their return value as plain data:

```js
let n = 0;
const o = { get v() { return ++n; } };
const copy = { ...o };
Object.getOwnPropertyDescriptor(copy, 'v').get;   // undefined — it's a data property now
```

**Rule:** never round-trip a class instance through spread, `JSON`, or `structuredClone`
and expect behaviour to survive. Rehydrate through a constructor or a static
`fromJSON`. → [type-safety.md](./type-safety.md)

## Reading legacy prototype code

- `Object.create(proto)` / `setPrototypeOf` / `.call()` pseudo-inheritance: treat as read-only legacy.
- `class` IS prototypal inheritance underneath — methods live on the prototype, so `Object.hasOwn(obj, 'method')` is `false`. → [modern-features.md](./modern-features.md)
