# Concurrency Model

Node executes JavaScript single-threaded on an event loop; concurrency comes from non-blocking I/O, worker threads, and child processes. Choose the right mechanism per workload.

## Mental model (why each tool exists)

| Mechanism | Parallelism | Use for | Overhead |
|---|---|---|---|
| Event loop (async I/O) | I/O concurrent, JS serial | network/disk/db calls | none — default path |
| `worker_threads` | true, in-process | CPU-bound JS (crypto, parsing, images) | thread + copy/SharedArrayBuffer |
| `child_process` | process-isolated | run external binaries, crash isolation | full process |
| `cluster` | process-isolated | multi-core HTTP servers (bare metal) | full process per core |

## Rules

**R1 — Never block the event loop with CPU work. [Stable]**
A synchronous computation >~10ms blocks *all* concurrent requests. Offload to a worker (R2), chunk with `setImmediate`, or use streams. Detect violations: `perf_hooks.monitorEventLoopDelay()`; diagnose with `node --inspect` + Chrome DevTools CPU profile, or [Clinic.js](https://clinicjs.org) (`clinic doctor -- node server.js`).

**R2 — `worker_threads` for CPU-bound JS. [Stable]**
```js
// main.mjs
import { Worker } from 'node:worker_threads';
import { fileURLToPath } from 'node:url';

const worker = new Worker(new URL('./heavy.mjs', import.meta.url), {
  workerData: { input: bigArray },
});
worker.on('message', resolve);
worker.on('error', reject);
```
```js
// heavy.mjs
import { parentPort, workerData } from 'node:worker_threads';
const result = crunch(workerData.input);
parentPort.postMessage(result);
```
Move large data with `ArrayBuffer` transfer (`postMessage(buf, [buf])`) or `SharedArrayBuffer` to avoid copy costs. Pool workers for repeated jobs (`piscina` is the standard pooler) — spawning a worker per request is a classic perf bug.

**R3 — `child_process` for external programs; pass args as an array of strings. [Stable]**
```js
import { spawn } from 'node:child_process';
const proc = spawn('ffmpeg', ['-i', inPath, '-vn', outPath]); // NOT spawn('ffmpeg', `-i ${inPath}`)
```
Why: array form avoids shell-injection and quoting bugs; Node 24 runtime-deprecated passing `args` as anything but an array of strings (or null/undefined) — [v24 release notes](https://nodejs.org/en/blog/release/v24.0.0). Prefer `execFile`/`spawn` over `exec` (which uses a shell). Always handle `error` event and non-zero `exit` codes.

**R4 — `cluster` only for bare-metal multi-core HTTP; not inside containers. [Context]**
Cluster scales one Node process across CPU cores behind a shared port. In Kubernetes/ECS/Nomad, horizontal replicas already do this job — clustering inside containers doubles memory and fights the orchestrator. Rule of thumb: cluster on VMs, replicas in orchestrators.

**R5 — Libuv thread pool: `UV_THREADPOOL_SIZE` for heavy fs/crypto/DNS. [Context]**
`fs`, `crypto` (pbkdf2/scrypt), and `dns.lookup` run on a **4-thread** libuv pool by
default. Saturating it stalls all of them — including DNS, which is why "the database got
slow" sometimes really means "someone added a hashing loop".

Measured (4 concurrent `pbkdf2`, 300k iterations, Node 22):

| `UV_THREADPOOL_SIZE` | Wall time |
|---|---|
| 4 (default) | ~330 ms — all four run in parallel |
| 1 | ~855 ms — serialized |

Raise the cap (`UV_THREADPOOL_SIZE=8 node app.js`, set **before** the process starts; it
is read once at startup) or move sustained CPU-crypto to workers. `crypto.subtle` is not
an escape hatch — most of its operations use the same pool. Measure saturation via
`perf_hooks` histograms or Clinic.js rather than private internals.

**R6 — AsyncLocalStorage carries context across awaits — but NOT into worker threads. [Stable]**
```js
als.run({ requestId: 'req-123' }, async () => {
  als.getStore();                 // { requestId: 'req-123' }
  await sleep(10);
  als.getStore();                 // { requestId: 'req-123' }  ✓ survives awaits
  new Worker('./w.mjs');          // inside w.mjs: als.getStore() === undefined  ✗
});
```
Verified on Node 22. A worker is a separate V8 isolate with its own module graph, so it
gets a **different `AsyncLocalStorage` instance** with an empty store — the context does
not travel. Same for `child_process`.

**Rule:** pass the correlation ID explicitly across the boundary (`workerData`, or on the
message payload) and re-enter `als.run()` inside the worker if downstream code reads it.

See [runtime-apis.md](./runtime-apis.md) R6. It is the recommended tracing primitive (used
by OpenTelemetry JS); do not build context passing with globals/WeakMaps by hand.

**R7 — Permission model interplay. [Context]**
Under `--permission`, spawning workers/child processes requires `--allow-worker` / `--allow-child-process`, and native addons are blocked by default (`ERR_DLOPEN_DISABLED` unless `--allow-addons`). See [security.md](./security.md).

## Diagnostics quick table

| Question | Tool |
|---|---|
| Is the event loop blocked? | `monitorEventLoopDelay`, `clinic doctor` |
| Where is CPU time going? | `node --inspect` + CPU profile; `--cpu-prof` |
| Memory leak? | `--heap-prof`, heap snapshots in DevTools |
| Why did it exit? | `process.exitCode`, `--trace-exit`, unhandled rejection ([error-handling.md](./error-handling.md)) |
