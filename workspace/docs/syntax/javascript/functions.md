# Functions: `this`, Closures & Memory

## `this` binding

`this` is determined by **how a function is called**, not where it's defined.

| Call form | `this` |
|---|---|
| `obj.method()` | `obj` |
| `method()` (plain call) | `undefined` (strict mode) / global object (sloppy) |
| `fn.call(x)` / `fn.apply(x)` / `fn.bind(x)` | `x` |
| `new Fn()` | the new instance |
| Arrow function | inherited lexically — cannot be rebound |

### The lost-this bug
```js
const logger = {
  prefix: '[app]',
  log(msg) { console.log(this.prefix, msg); }
};

logger.log('hi');           // ✓ works — called as method
const f = logger.log;
f('hi');                    // ✗ TypeError — this is now undefined
setTimeout(logger.log, 0);  // ✗ same bug via callback
```
Extracting a method or passing it as a callback severs it from its object.

### Fixes
```js
// ✓ Class field arrow — auto-bound, safe to pass around
class Handler {
  onClick = () => { this.save(); };
}

// ✓ Wrap in an arrow at the call site
setTimeout(() => logger.log('hi'), 0);

// ✓ bind — permanent bound function
const f = logger.log.bind(logger);
```

**Arrow rules:** Arrows for callbacks that must not have their own `this`; regular methods for object/class methods. Never use an arrow as an object method that reads `this`.

## Closures

A closure captures variable **bindings**, not values.

```js
function counter() {
  let n = 0;
  return () => ++n;   // closes over n; n stays alive between calls
}
const inc = counter();
inc(); // 1
inc(); // 2
```
Deferred callbacks capture per-iteration bindings only when declared with `let`.
→ [anti-patterns.md](./anti-patterns.md) § Closures capturing loop variables

## Memory leaks

Garbage collection frees objects only when unreachable. Common leaks:

### Forgotten listeners and intervals
```js
// ✗ Runs forever; retains owner even after it's "removed"
setInterval(this.refresh, 1000);
el.addEventListener('click', this.onClick);

// ✓ Clean up symmetrically...
clearInterval(id);
el.removeEventListener('click', this.onClick); // needs the SAME function reference

// ✓ ...or use the signal option — no stored reference needed
const controller = new AbortController();
el.addEventListener('click', handler, { signal: controller.signal });
controller.abort();   // removes all listeners registered with this signal
```

### Closures holding large objects
```js
// ✗ entire bigData retained just to read one field
function setup(bigData) {
  return () => bigData.summary;
}
// ✓ extract only what's needed
function setup(bigData) {
  const summary = bigData.summary;
  return () => summary;
}
```

### WeakMap/WeakSet — metadata without retention
```js
const metadata = new WeakMap();        // keys held weakly
metadata.set(domNode, { clicks: 0 });  // entry vanishes when domNode is GC'd
```

| Collection | Keys prevent GC? | Iterable? | Use for |
|---|---|---|---|
| `Map` | Yes | Yes | Real application data |
| `WeakMap` | No | No | Private state, caches keyed by object |
| `WeakSet` | No | No | Marking objects ("seen", "processing") |

**Rule:** Any cache/metadata map keyed by DOM nodes or class instances should be a `WeakMap`.
