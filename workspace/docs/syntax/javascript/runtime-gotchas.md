# JavaScript Runtime Differences

Behavioral differences between Node.js, Bun, and Deno that cause real bugs.
State verified September 2026 (Node 24 Active LTS / 26 current, Bun 1.2+, Deno 2.x).
Module and interop rows re-checked by running the code, not read from docs.

## Module system

| Feature | Node.js | Bun | Deno |
|---|---|---|---|
| Named imports from CJS | Yes, via static lexing → [modules.md](./modules.md) | Yes | Yes |
| ESM default | No (`"type": "module"` or `.mjs` needed) | Yes | Yes (`.js`/`.ts` are ESM unless `"type": "commonjs"`) |
| CJS | Yes (default for `.js`) | Yes | Yes — `.cjs` always; auto-detection opt-in (`--unstable-detect-cjs`) |
| `require()` | CJS **and ESM** since 22.12 (no flag; throws `ERR_REQUIRE_ASYNC_MODULE` on top-level await) | Yes (works in ESM too) | CJS files; can require(ESM) |
| Extensionless local imports | Fails in ESM | Resolves | **Fails** — explicit `.js`/`.ts` required |
| `"imports"` in package.json | Yes | Yes | Yes |

**Key gotcha:** Code with extensionless imports works in Bun but fails in both Node.js ESM and Deno. Always use `.js` extensions. → [modules.md](./modules.md)

## Globals

| Global | Node.js | Bun | Deno |
|---|---|---|---|
| `process` | Yes | Yes | Yes (Deno 2+; or `import from 'node:process'`) |
| `Buffer` | Yes | Yes | **Not global** — `import { Buffer } from 'node:buffer'` |
| `globalThis` | Yes | Yes | Yes |
| `__dirname` / `__filename` | CJS only | CJS compat | Not defined → use `import.meta.*` |
| `import.meta.dirname` | 20.11+ / 21.2+ | Yes | 1.40+ |
| `window` | No | No | Removed in Deno 2 — use `globalThis` |

## `fetch` availability

All three runtimes have built-in `fetch`. Behavioral differences exist in edge cases:
- `Request`/`Response` objects aren't always interchangeable between runtimes
- Redirect handling in `manual` mode differs
- Bun's streaming behavior differs from Node's undici-based implementation

→ Error/timeout/retry rules: [async.md](./async.md) § HTTP with fetch

## TypeScript support

| Runtime | Execution | Type checking |
|---|---|---|
| Node.js | Native type stripping, unflagged since 23.6 (backported to 22.18); erasable syntax only | No — use `tsc --noEmit` |
| Bun | Native transpilation | No — use `tsc --noEmit` |
| Deno | Native, all versions | Yes (`deno check`) |

**Key points:**
- No runtime type checks by default — always run `tsc --noEmit` or `deno check` separately.
- Node's stripping handles only *erasable* syntax. Enums, namespaces, and parameter properties need `--experimental-transform-types`.
- → JSDoc alternative: [type-safety.md](./type-safety.md)

## Test runners

| Runner | Command | API |
|---|---|---|
| Node.js | `node --test` | `node:test` |
| Bun | `bun test` | `bun:test` |
| Deno | `deno test` | Built-in assert |

**APIs are completely different.** Test code is not portable. Abstract test helpers if multi-runtime support is needed.

## Temporal

| Runtime | Temporal |
|---|---|
| Node.js | 26+ unflagged; **not** in 24 LTS |
| Bun / Deno | Not yet |
| Browsers | Firefox 139+, Chrome/Edge 144+; **no Safari** |

Feature-detect and polyfill — see [datetime.md](./datetime.md).

## Timers

- Node/Bun: standard Web-compatible semantics.
- Deno: matches Node semantics since 2.8.
- Minimum delays differ slightly under load — never rely on sub-millisecond timer precision.

→ Event loop ordering guarantees: [async.md](./async.md) § Event loop ordering

## File system

```js
// Node.js & Bun
import { readFile } from 'node:fs/promises';

// Deno
const text = await Deno.readTextFile('./file.txt');
```
**Deno requires `--allow-read` permission** (and `-R`/`-E` for most npm/CJS packages). Node has an experimental permission model (`--permission`); Bun has none.

## Error stack traces

- **Node.js:** Most detailed, includes async context
- **Bun:** Shorter by default. Use `--verbose` for more.
- **Deno:** Includes `Deno.` namespace frames.

## Process exit

```js
// Node.js & Bun
process.exit(0);  // Immediate — skips pending async work

// Deno
Deno.exit(0);     // Similar behavior
```

**Rule:** Never call `process.exit()` / `Deno.exit()` in libraries. Let the caller decide.

## `console` differences

- Object formatting and colorization differ between all three
- `console.table` column selection syntax varies
- `console.time` precision differs

**Rule:** Don't parse `console` output programmatically. Use structured logging.
