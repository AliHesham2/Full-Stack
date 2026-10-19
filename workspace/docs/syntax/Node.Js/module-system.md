# Module System (ESM / CommonJS)

The single largest source of Node.js code errors. Rules verified against [nodejs.org/api/esm.html](https://nodejs.org/api/esm.html), [nodejs.org/api/packages.html](https://nodejs.org/api/packages.html), and [Joyee Cheung's require(esm) writeup](https://joyeecheung.github.io/blog/2025/12/30/require-esm-in-node-js-from-experiment-to-stability) (TSC member, 2025-12). See also [typescript.md](./typescript.md) for `.ts` module rules.

## Rules

**R1 — Default to ESM for new code. [Stable]**
```json
// package.json
{ "type": "module" }
```
Why: ESM is the standard; top-level await, static analysis, and modern tooling assume it. CJS remains supported indefinitely for existing code — do not rewrite working CJS just to rewrite it (conversion has real interop pitfalls, see R6).
Caveat: some older dependencies are CJS-only — that's fine, ESM can import them (R6).

**R2 — `require(esm)` is stable; module direction is no longer a blocker. [Stable]**
CJS can `require()` an ES module since 20.19.0 / 22.12.0 (unflagged, later marked stable — first-party verified on 24.18.0).
```js
// CJS file requiring an ESM file — works
const { x } = require('./lib.mjs');
```
Hard limit: the ESM module must be **synchronous** (no top-level await) — `require()` of a TLA-using module throws `ERR_REQUIRE_ASYNC_MODULE`. ESM importing CJS has no such restriction.

**R3 — Always use the `node:` prefix for core modules. [Stable]**
```js
import fs from 'node:fs/promises';
import { setTimeout as sleep } from 'node:timers/promises';
```
Why: prevents shadowing by same-named npm packages and makes core imports greppable. Lint it: `n/no-missing-import` or `unicorn/prefer-node-protocol`.

**R4 — File extensions are mandatory in relative imports. [Stable]**
```js
import './util.js';   // ESM: extension required (and it's the real path — see caveat)
import './util.ts';   // when importing TS under type stripping (typescript.md)
const u = require('./util.js'); // CJS: extension optional but recommended
```
Why: ESM resolution follows the browser model — no extension guessing. Errors look like `ERR_MODULE_NOT_FOUND`.
Caveat: the extension in ESM must match the **actual file on disk** in TS projects using `rewriteRelativeImportExtensions`; in plain JS projects, import the `.js` path even when the source is what you authored.

**R5 — In ESM there is no `__dirname` / `__filename` / `require`. [Stable]**
```js
import { dirname, join } from 'node:path';
import { fileURLToPath } from 'node:url';

const __dirname = import.meta.dirname;            // Node ≥ 20.11 / 21.2 (prefer this)
const __filename = import.meta.filename;
const require = (await import('node:module')).createRequire(import.meta.url); // createRequire only if you must load CJS dynamically
```
Why: agents migrating CJS→ESM hit `ReferenceError: __dirname is not defined` constantly; `import.meta.dirname` is the direct replacement.

**R6 — Know the interop shapes. Named imports from CJS depend on the export FORM. [Stable]**

Node runs `cjs-module-lexer` over the CJS source and synthesizes named exports from
assignments it can statically see. **Property assignment works; whole-object assignment
does not.** Measured on Node 22 — the difference is not documented anywhere obvious:

| CJS source | `import { a }` |
|---|---|
| `module.exports.a = 1` | ✅ works |
| `exports.a = 1` | ✅ works |
| `Object.defineProperty(exports, 'a', {...})` | ✅ works |
| `module.exports = { a: 1, b: 2 }` | ❌ **fails** |
| `const o = { a: 1 }; module.exports = o` | ❌ fails |
| `module.exports = { a: 1 }; module.exports.b = 2` | ❌ fails — the reassignment poisons the whole file |

```js
// legacy.cjs:  module.exports = { a: 1, b: 2 };
import { a } from './legacy.cjs';
// SyntaxError: Named export 'a' not found. The requested module is a CommonJS module,
// which may not support all module.exports as named exports.
```

The failure is a **link-time SyntaxError**, not `undefined` — it cannot hide, and it fires
before any of your code runs.

```js
// Always-safe shapes for the same file
import mod from './legacy.cjs';      // default = module.exports  →  { a: 1, b: 2 }
const { a, b } = mod;

import * as ns from './legacy.cjs';
ns.default;                          // { a: 1, b: 2 }
Object.keys(ns);                     // ['default']  ← ONLY default; no ns.a here
```

**Rule:** for a dependency you don't control, import the default and destructure. Reach for
named imports only after you've seen them work — a green local run is proof enough, since
the failure is deterministic.
Caveat: never rely on `__esModule` markers — Node ignores them. TypeScript's `export =`
compiles to whole-object assignment, so it lands in the ❌ row.

**R7 — Dual-package hazard: one package, two module instances. [Stable — avoid triggering it]**
If a dependency is loaded both via `require` and via `import`, Node can instantiate it twice → `instanceof` failures, split state (two `Set`s, two class identities).
- Library authors: prefer shipping **one** format (ESM-only is now a viable 2026 default for new libs; CJS consumers get it via `require(esm)` — R2).
- If you must dual-publish, use conditional `exports` with shared underlying state or the `"module-sync"` condition; test for the double-instance failure explicitly.

**R8 — Publish entry points via `exports`, not bare `main`. [Stable]**
```json
{
  "type": "module",
  "exports": {
    ".": "./dist/index.js",
    "./sub": "./dist/sub.js"
  }
}
```
Why: `exports` defines the public API surface — everything else becomes unreachable (`ERR_PACKAGE_PATH_NOT_EXPORTED`), which is a feature (no accidental deep imports). `"main"` is only a fallback for legacy tooling.
Conditions worth knowing: `"import"` / `"require"` (split formats), `"default"` (must be last), `"node-addons"`. Do not add redundant `node`/`browser` conditions — bundlers and Node provide those contexts themselves.

**R9 — Package-internal imports: use `imports` (subpath `#`), not tsconfig paths. [Stable]**
```json
{ "imports": { "#internal/*": "./src/internal/*.js" } }
```
Why: works at runtime with zero config, in both ESM and CJS, and is the only runtime-resolvable alias mechanism (Node ignores tsconfig `paths` — see [typescript.md](./typescript.md)).

## Quick diagnostics for agents

| Symptom | Cause | Fix |
|---|---|---|
| `ERR_MODULE_NOT_FOUND` with a bare relative path | missing extension | add `.js` (R4) |
| `ReferenceError: __dirname is not defined` | ESM scope | `import.meta.dirname` (R5) |
| `ERR_REQUIRE_ASYNC_MODULE` | `require()` of ESM with top-level await | switch caller to `import`, or drop TLA |
| `ERR_PACKAGE_PATH_NOT_EXPORTED` | deep import not in `exports` | import the documented subpath or ask maintainer |
| `SyntaxError: Named export 'x' not found` | CJS uses whole-object `module.exports =` | default-import + destructure (R6) |
| `default` is `{}` or `[Module: null prototype]` | CJS/ESM shape mismatch | check R6 shapes; log the namespace object |
| Two instances of a dep's class | dual-package hazard | R7 |
