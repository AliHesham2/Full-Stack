# Deprecations & Freshness Audit (2026-09)

What's deprecated, removed, or dead — for reviewing legacy code and for avoiding stale advice in training data. Sources: [official deprecations doc](https://nodejs.org/api/deprecations.html) (v26.7.0), [v24](https://nodejs.org/en/blog/release/v24.0.0) and [v26](https://nodejs.org/en/blog/release/v26.0.0) release notes.

## How Node deprecates (read before interpreting warnings)

| Kind | Effect |
|---|---|
| Documentation-only | docs only; no runtime warning (unless `--pending-deprecation`) |
| Application (non-node_modules only) | warns for YOUR code, silent inside dependencies |
| Runtime (all code) | warns everywhere |
| End-of-Life | removed / throws |

## Removed or End-of-Life (code that BREAKS on modern Node)

| API | Gone in | Replacement |
|---|---|---|
| `require('node:_stream_*')` internal stream modules | 26 (EOL) | public `node:stream` API |
| `http.Server.prototype.writeHeader()` | 26 | `writeHead()` |
| `--experimental-transform-types` flag | 26 | none — non-erasable TS is permanently unsupported ([typescript.md](./typescript.md) R2); use `tsx` |
| short GCM auth tags without `authTagLength` (DEP0182) | 26 (EOL) | pass `authTagLength` to `createDecipheriv` |
| `NODE_MODULE_VERSION` 137 → **147** | 26 (ABI break) | rebuild every native addon; prebuilt binaries for ≤ 24 will not load |
| `tls.createSecurePair` | 24 | `tls.TLSSocket` |
| `util.isArray` / `util.isBoolean` / … (`util.is*`) | removed in recent majors (deprecated since v4) | `Array.isArray`, `typeof`, etc. |

## Runtime-deprecated (warns now; removed later)

| API | DEP | Since | Replacement |
|---|---|---|---|
| `url.parse()`, `url.resolve()`, string-form `url.format()` | DEP0169/0170 | warns since 24 | `new URL(str)` — security-relevant: no CVEs for url.parse bugs |
| `module.register()` | DEP0205 | runtime in 26 (doc-only 25.9) | `module.registerHooks()` — sync, in-thread, all module types |
| passing `CryptoKey` to `node:crypto` functions | DEP0203 | runtime in 26 | pass `KeyObject` (use `crypto.createPublicKey` etc.) |
| `KeyObject.from()` with non-extractable CryptoKey | DEP0204 | runtime in 26 | will throw in a future version |
| stream API deprecation DEP0201 | DEP0201 | runtime in 26 | see the v26 release notes for the affected call |
| `SlowBuffer` | DEP0030 | runtime in 24 | `Buffer.allocUnsafe` |
| instantiating zlib/REPL/http classes without `new` | DEP0184/0185/0195 | runtime in 24 | always `new` |
| `spawn`/`execFile` args not array-of-strings | – | 24 | `spawn(cmd, ['--flag', value])` ([concurrency.md](./concurrency.md) R3) |
| `fs.rmdir(path, {recursive:true})` | DEP0147 | runtime (since 16) | `fs.rm(path, {recursive:true, force:true})` |
| `buffer.slice()` | DEP0158 | runtime | `buf.subarray()` |
| `node:punycode` | DEP0040 | doc-only, frozen | WHATWG `URL` handles IDNA; or npm `punycode` |
| `node:domain` | DEP0032 | doc-only, abandoned | `AsyncLocalStorage` ([runtime-apis.md](./runtime-apis.md) R6) |

## Flag renames (config drift, not code)

| Old | New |
|---|---|
| `--experimental-permission` | `--permission` (24) |
| `--experimental-strip-types` | default on (22.18+/23.6); `--no-strip-types` to disable |
| `--experimental-test-coverage` | still experimental on 24 (with `--test-coverage-lines/branches/functions` threshold options, first-party verified) |

## Dead / superseded npm packages (agent checklist for legacy repos)

| Package | Status | Use instead |
|---|---|---|
| `request` | dead since 2020 | global `fetch` ([runtime-apis.md](./runtime-apis.md) R1) |
| `moment` | legacy/maintenance | Temporal (Node 26+), `dayjs`, `luxon` — Temporal is the strategic answer |
| `node-uuid` | ancient | `crypto.randomUUID()` |
| `node-fetch` | obsolete on Node ≥ 18 | global `fetch` |
| `ts-node` | maintenance mode | native TS / `tsx` ([typescript.md](./typescript.md)) |
| `nodemon` (new setups) | superseded | `node --watch` ([tooling.md](./tooling.md)) |
| `rimraf` (new setups) | superseded | `fs.rm` recursive |
| `dotenv` (new setups) | superseded for dev | `node --env-file` |
| corepack bundling | not shipped from Node 25 | install managers explicitly ([tooling.md](./tooling.md) R2) |

## Legacy-repo audit procedure (for agents)

1. **Runtime**: `.nvmrc`/`engines`/Dockerfile base tag → EOL? ([versions-and-releases.md](./versions-and-releases.md))
2. **Grep removed APIs**: `writeHeader(`, `_stream_`, `createSecurePair`, `util.is[A-Z]`, `SlowBuffer`, `url.parse(`, `new url.URL` (legacy), `domain.`.
3. **Grep deprecated patterns**: `require('domain')`, `rmdir.*recursive`, `.slice(` on buffers, `spawn(` with string args.
4. **Dependencies**: the dead-packages table above; `npm ls` for them; check lockfile age + `npm audit`.
5. **Run with warnings visible**: `node --pending-deprecation app.js`; watch stderr for `DeprecationWarning: DEPxxxx`.
6. **Module sanity**: CJS/ESM mixing errors → [module-system.md](./module-system.md) diagnostics table.

## Freshness of THIS file

Deprecation state changes per major. Re-verify against [nodejs.org/api/deprecations.html](https://nodejs.org/api/deprecations.html) before acting on any "is X still deprecated" decision. Snapshot: 2026-09-02, verified against Node 26.0.0 release notes.

**Stability indexes worth knowing** (these are not deprecations, but they gate adoption):

| Feature | Index | Since |
|---|---|---|
| Type stripping (`node file.ts`) | **2 — Stable** | 24.12 / 25.2 |
| `require(esm)` | **2 — Stable** | 20.19 / 22.12 unflagged |
| `node:sqlite` | **1.2 — Release candidate** | 25.7 |
| Single Executable Apps (`--build-sea`) | **1.1 — Active development** | flag added 25.5 |
| `--permission` | experimental (flag renamed from `--experimental-permission` in 24) | 24 |
| `--experimental-test-coverage` | experimental | – |
