# Runtime APIs (Standard Library)

Which built-in APIs to reach for, and their 2026 stability. Globals verified first-party on Node 24.18.0. See [typescript.md](./typescript.md) for TS execution, [concurrency.md](./concurrency.md) for workers.

## Web-platform globals — use them instead of userland packages

| API | Status (Node 24) | Replaces |
|---|---|---|
| `fetch` (global) | Stable (undici 7; undici 8 in Node 26) | `axios`, `node-fetch`, `got` for most cases |
| `WebSocket` (global) | Stable since 22 | `ws` (client use) |
| `URL`, `URLSearchParams`, `URLPattern` (global since 24) | Stable | `url.parse()` [Avoid], path-to-regexp for simple patterns |
| `structuredClone` | Stable | `lodash.cloneDeep` |
| `BroadcastChannel`, `MessageChannel`, `Event`, `EventTarget` | Stable | EventEmitter for cross-context pub/sub |
| `crypto` (WebCrypto: `crypto.subtle`, `crypto.randomUUID`) | Stable | `uuid`, `node-uuid` |
| `AbortController` / `AbortSignal` | Stable | ad-hoc cancellation; pass `signal` into fetch/timers/fs ops |
| `performance` | Stable | `Date.now` benchmarking |
| Temporal | **Node 26+ only** | `Date`, `moment`, `luxon` for date/time ([deprecations.md](./deprecations.md)) |

**R1 — Prefer the global web APIs over npm equivalents. [Stable]**
Why: zero deps, consistent with browsers/Deno/Bun, maintained by core. Keep `axios` only for features undici's fetch lacks (interceptors, per-instance defaults); otherwise migrate.
Caveat: global `fetch` is undici — HTTP proxies and custom agents need `undici.setGlobalDispatcher()` / `ProxyAgent`, not the old `https.Agent` API.

**R2 — `node:sqlite` is a release candidate — fine for tools, use care for critical data. [RC]**
```js
import { DatabaseSync } from 'node:sqlite';
const db = new DatabaseSync(':memory:');
db.exec('CREATE TABLE t (id INTEGER PRIMARY KEY, v TEXT)');
const stmt = db.prepare('INSERT INTO t (v) VALUES (?)');
stmt.run('hello');
console.log(db.prepare('SELECT * FROM t').all());
```
Unflagged since 23.4/22.13; **Stability 1.2 (RC)** since 25.7. Synchronous API (by design). Use for: CLIs, local caches, small services. Prefer Postgres/MySQL drivers for high-concurrency server data. Source: [nodejs.org/api/sqlite.html](https://nodejs.org/api/sqlite.html).

**R3 — Streams: use Node streams for I/O plumbing; Web Streams at the boundaries. [Stable]**
- `stream.Readable/Writable/Duplex/Transform` (object mode, backpressure) — the workhorse for files/pipes/HTTP bodies.
- Web Streams (`ReadableStream` etc.) — where a dependency or spec expects them (e.g. WHATWG-compatible APIs).
- Convert at the boundary: `stream.Readable.toWeb(rs)` / `stream.Readable.fromWeb(ws)` (and `Writable` equivalents).
Why: Node streams remain substantially faster for heavy I/O; Web Streams win on portability. Both are Stable. Legacy `_stream_*` internal modules were **removed in Node 26** ([deprecations.md](./deprecations.md)).

**R4 — File system: use `node:fs/promises` by default. [Stable]**
```js
import { readFile, writeFile, rm, mkdir } from 'node:fs/promises';
await mkdir(dir, { recursive: true });
await rm(path, { recursive: true, force: true });   // replaces rimraf
```
`fs.rm` (with recursive) replaced `fs.rmdir` recursive long ago; promisified APIs removed callback error-first boilerplate. Use sync variants only in startup scripts/CLIs.

**R5 — Modern utilities that remove common dependencies. [Stable]**
```js
import { parseArgs } from 'node:util';          // replaces minimist/yargs for simple CLIs
import { setTimeout as sleep } from 'node:timers/promises'; // await sleep(100)
import { gzipSync } from 'node:zlib';           // compression
import { text } from 'node:stream/consumers';   // await text(stream)
```
`parseArgs` handles flags/options/positional args with strict mode — enough for most small CLIs; keep yargs/commander only for rich UX.

**R6 — `AsyncLocalStorage` for request-scoped context (correlation IDs, tracing). [Stable]**
```js
import { AsyncLocalStorage } from 'node:async_hooks';
const store = new AsyncLocalStorage();
store.run({ requestId: id }, () => handleRequest());
store.getStore()?.requestId; // anywhere downstream, including in awaited callbacks
```
Perf improved significantly in Node 24 (AsyncContextFrame is now the default implementation — from the [v24 release notes](https://nodejs.org/en/blog/release/v24.0.0)). Avoid the older `async_hooks` execution-async-resource APIs for new code.

**R7 — HTTP server: `node:http` is viable minimal; use a framework for real apps. [Context]**
Raw `node:http` for health endpoints/protocol work; [frameworks.md](./frameworks.md) for app routing/validation. `http.Server.prototype.writeHeader` was removed in 26 (use `writeHead`).

## Version-specific engine features (V8)

- **Node 24 (V8 13.6):** `Float16Array`, `RegExp.escape`, `Error.isError`, WebAssembly Memory64, explicit resource management (`using` declarations, `Symbol.dispose`).
- **Node 26 (V8 14.6):** Map upsert (`Map.prototype.getOrInsert`), `Iterator.concat`, Temporal.
Why this matters: with `target: esnext` and no downleveling ([typescript.md](./typescript.md) R5), these language features are runtime-gated — check [versions-and-releases.md](./versions-and-releases.md) before using them in code that must run on 24.
