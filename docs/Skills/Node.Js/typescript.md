# TypeScript in Node.js

Node runs `.ts` natively via **type stripping** — no build step for erasable-syntax TS. Verified against [nodejs.org/api/typescript.html](https://nodejs.org/api/typescript.html) (v26.7.0 docs, fetched 2026-08) and first-party checks on 24.18.0.

## Version history (know it when reading old advice)

| Version | Type-stripping status |
|---|---|
| 22.6 | added, behind `--experimental-strip-types` |
| 22.18 / 23.6 | **enabled by default** |
| 24.3 / 22.18 | experimental warning removed |
| 24.12 / 25.2 | **Stable** |
| 26.0 | `--experimental-transform-types` **removed** |

## Rules

**R1 — Run TS directly with `node`; no ts-node/compile step for app code. [Stable]**
```bash
node server.ts        # works on Node ≥ 22.18 by default
node --test tests/**/*.test.ts   # general node:test knowledge; this project runs Vitest
```
Why: stripping replaces types with whitespace; no transformation, no source maps needed (line numbers stay correct). Node never type-checks — run `tsc --noEmit` separately in CI.

**R2 — Erasable syntax only. These FAIL with `ERR_UNSUPPORTED_TYPESCRIPT_SYNTAX`. [Stable]**
- `enum` declarations → use `const` objects + type unions, e.g. `const Color = { Red: 1 } as const;`
- namespaces containing runtime code (type-only `namespace` is fine)
- parameter properties (constructor `private x: string`)
- import aliases that emit (`import x = require(...)`)
- decorators (parser error; TC39 Stage 3, not transformed)
Why: Node only *strips* — anything needing code generation is refused. `--experimental-transform-types` (which once transformed enums etc.) was **removed in Node 26**, so this restriction is now permanent.
Need those features? Use `tsx`: `node --import=tsx file.ts` ([tooling.md](./tooling.md)).

**R3 — Mark type-only imports with `type`, or get runtime errors. [Stable]**
```ts
import type { Config } from './config.ts';       // correct
import { connect, type Options } from './db.ts'; // correct
import { Config } from './config.ts';            // RUNTIME ERROR if Config is a type
```
Why: without `type`, Node treats it as a value import of a name that doesn't exist at runtime. `verbatimModuleSyntax` in tsconfig enforces this at check-time.

**R4 — Recommended tsconfig for Node-run TS. [Stable]**
```json
{
  "compilerOptions": {
    "noEmit": true,
    "target": "esnext",
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "rewriteRelativeImportExtensions": true,
    "erasableSyntaxOnly": true,
    "verbatimModuleSyntax": true,
    "strict": true
  }
}
```
Source: official Node docs (recommends TS ≥ 5.8). `erasableSyntaxOnly` makes the compiler enforce R2; `rewriteRelativeImportExtensions` lets you author `.ts` import specifiers.

**R5 — Node ignores tsconfig.json at runtime. [Stable]**
- No `paths` aliases at runtime → use package.json `imports` (`#...`) instead ([module-system.md](./module-system.md) R9).
- No downleveling: `target: esnext` semantics are what you get; the runtime must support the syntax you write (Node 24 = V8 13.6).
- `--no-strip-types` disables stripping globally if you must force the old world.

**R6 — File semantics mirror JS extensions. [Stable]**
`.mts` → always ESM; `.cts` → always CJS; `.ts` → follows nearest package.json `type`. `.tsx` is **unsupported** (React/JSX assumption — use tsx/esbuild for that).
Extensions are mandatory in imports **and** in `require()`: `require('./file.ts')`, not `require('./file')` (first-party verified).

**R7 — Never publish `.ts` files in packages. [Stable]**
Node refuses to process TS inside `node_modules` (by design, to discourage shipping TS). Compile to `.js` + `.d.ts` for distribution. Dual `exports` conditions for library authors: [module-system.md](./module-system.md) R7–R8.

**R8 — REPL / `--eval` caveats. [Stable]**
TypeScript syntax works in `--eval` and STDIN (respect `--input-type`), but **not** in the REPL or under `--check`. For a TS REPL, use `node --import=tsx -i`.

## Decision guide

| Situation | Approach |
|---|---|
| App/script, erasable TS only | `node file.ts` + `tsc --noEmit` in CI (R1) |
| Need enums/decorators/legacy TS | `tsx` runner (R2) |
| Library for npm | compile to JS + types (R7) |
| Old project on ts-node | migrate to native stripping or tsx; ts-node is legacy-maintenance mode |
