# JavaScript Modules

## Rule: ESM only for new code

```js
// ✓ Named exports (preferred)
export function foo() { ... }
export const bar = 1;

// ✓ Default export (single-purpose modules only)
export default class MyClass { ... }

// ✗
// const foo = require('./mod');
// module.exports = { foo };
```
**Why:** Tree-shaking, top-level await, static analysis. All modern runtimes and bundlers support it.

ESM code always runs in strict mode — no `"use strict"` directive needed; sloppy-mode constructs (undeclared assignment, `with`, octal literals) simply throw.

## File extensions

Always use `.js` in import paths (required by Node.js ESM):

```js
// ✓
import { foo } from './utils.js';

// ✗ Fails in Node.js ESM
import { foo } from './utils';
```
**Caveat:** Bundlers (Vite, webpack) allow extensionless. Pick one convention, enforce with ESLint.
→ [runtime-gotchas.md](./runtime-gotchas.md) for cross-runtime behavior.

## CJS interop (legacy only)

### Default export from CJS
```js
// CJS: module.exports = function() { ... }
// ESM:
import factory from './legacy.cjs';          // ✓
import { default as factory } from './legacy.cjs'; // ✓ same
```

### Named exports from CJS — work, but only when statically detectable

Node runs `cjs-module-lexer` over the CJS file and synthesizes named exports from
assignments it can see. Verified on Node 22:

```js
// legacy.cjs — static, lexer sees both
module.exports.foo = 'FOO';
module.exports.bar = 'BAR';

import { foo, bar } from './legacy.cjs';   // ✓ works
```

```js
// dynamic.cjs — computed key, invisible to static analysis
const k = 'baz';
module.exports[k] = 'BAZ';

import { baz } from './dynamic.cjs';       // ✗ SyntaxError at link time
import mod from './dynamic.cjs';           // ✓ mod.baz === 'BAZ'
```

The rule is about the **form of the assignment**, not about how "dynamic" the package is.
Measured on Node 22:

| CJS source | `import { a }` |
|---|---|
| `module.exports.a = 1` | ✅ works |
| `exports.a = 1` | ✅ works |
| `Object.defineProperty(exports, 'a', {...})` | ✅ works |
| `module.exports = { a: 1, b: 2 }` | ❌ **fails** |
| `const o = { a: 1 }; module.exports = o` | ❌ fails |
| `module.exports = { a: 1 }; module.exports.b = 2` | ❌ fails — reassignment poisons the file |

The whole-object form is extremely common in published packages, so "named imports from
CJS work" is only half true.

**Rule:** for a dependency you don't control, default-import and destructure. The failure
is a **link-time SyntaxError**, not `undefined`, so it cannot hide — but it also cannot be
worked around at runtime.

```js
import mod from './legacy.cjs';
const { a, b } = mod;
```

### Requiring ESM from CJS

`require()` of an ES module works since Node 22.12 / 23.0 (no flag). The module must
have no top-level `await`, or `require` throws `ERR_REQUIRE_ASYNC_MODULE`.

```js
const mod = require('./esmmod.mjs');   // { __esModule: true, default: 42, hello: 'from esm' }
```
Advice to "always use dynamic `import()` from CJS" predates this and is now outdated
for Node — though bundlers and older runtimes may still need it.

## `__dirname` / `__filename` in ESM

These don't exist in ESM.

```js
// Node.js 20.11+ / 21.2+, Deno 1.40+
const __dirname = import.meta.dirname;
const __filename = import.meta.filename;

// Older Node.js
import { dirname } from 'node:path';
import { fileURLToPath } from 'node:url';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

## `import.meta.url`

```js
import { readFile } from 'node:fs/promises';
const data = await readFile(new URL('./data.json', import.meta.url), 'utf-8');
```

## Package.json `"type": "module"`

Required for Node.js to treat `.js` as ESM. Without it, `.js` is CJS.

```json
{ "type": "module" }
```

Use `.cjs` for CJS files in an ESM package, `.mjs` for ESM in a CJS package.

## Dual package hazard

Shipping the same module as both CJS and ESM loads it **twice** — two module instances,
so `instanceof` fails across the halves and singletons/caches silently fork.

**Solution: ship ESM only.** With `require(esm)` supported from Node 22.12, a CJS build
is no longer needed for Node consumers:

```json
{
  "type": "module",
  "exports": { ".": "./index.js", "./feature": "./feature.js" }
}
```

If you genuinely must ship both (old bundlers, Node <22.12), keep all state in one
format and make the other a thin re-export — never two independent builds:

```json
{
  "exports": {
    ".": { "import": "./index.js", "require": "./index.cjs" }
  }
}
```
and have `index.cjs` re-export rather than re-implement.

## Conditional exports

```json
{
  "exports": {
    ".": {
      "node": "./node-specific.js",
      "default": "./universal.js"
    },
    "./feature": "./feature.js"
  }
}
```
**Rule:** Always set `"exports"` in published packages. Unexported files are internal.
