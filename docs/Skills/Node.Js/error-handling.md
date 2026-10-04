# Error Handling & Crash Semantics

How Node fails, and how to fail correctly. Cross-ref: [security.md](./security.md) R8 (fail fast), [deployment.md](./deployment.md) (graceful shutdown).

## Rules

**R1 — Unhandled promise rejections crash the process. [Stable]**
Since Node 15, the default is `--unhandled-rejections=throw`: an unhandled rejection is an uncaught exception → nonzero exit. Do not "fix" this by adding a swallowing handler:
```js
// ANTI-PATTERN — masks failures, leaves app in unknown state
process.on('unhandledRejection', () => {});
```
Correct responses: add `.catch()` at the call site, `await` inside try/catch, or let it crash and rely on the supervisor/orchestrator restart.

**R2 — One last-resort top-level handler that logs and exits. [Stable]**
```js
process.on('uncaughtException', (err) => {
  log.fatal({ err }, 'uncaught exception — shutting down');
  process.exit(1);            // skip graceful paths; state is unknown
});
```
Why: log for postmortem, exit nonzero so orchestrators alert/restart. Attempting to *continue* after an uncaught exception is undefined behavior.

**R3 — Use error `cause` for wrapping; keep the chain. [Stable]**
```js
try {
  await fetch(url);
} catch (err) {
  throw new Error(`payment service unreachable: ${url}`, { cause: err });
}
```
Why: preserves the original stack for diagnosis; supported natively since Node 16.9. Log with a serializer that prints `cause` chains (pino does).

**R4 — `error` events on EventEmitters throw if unhandled. [Stable]**
```js
stream.on('error', (err) => { /* required, or the process crashes */ });
```
Pipelines: prefer `stream.pipeline(src, transform, dst)` which forwards errors to a single callback instead of manual `on('error')` on each stream.

**R5 — Exit codes and signals: know the contract. [Stable]**
| Signal / code | Meaning | Your job |
|---|---|---|
| `SIGTERM` | orchestrator asks shutdown | graceful shutdown ([deployment.md](./deployment.md) R2) |
| `SIGINT` | Ctrl+C | same path as SIGTERM |
| exit 0 | success | — |
| exit 1 | uncaught exception / `process.exit(1)` | alert on it |
| exit 12+ | debug/invalid args | inspect `--trace-exit` output |

Never call `process.exit()` from library code; it fires instantly and skips pending I/O and `exit` flushing. Prefer setting `process.exitCode` and returning.

**R6 — Errors crossing async boundaries: `Error` objects only. [Stable]**
`postMessage`/structured clone serializes Error props (message/stack/cause) but drops custom prototype methods. For workers ([concurrency.md](./concurrency.md) R2), send plain `{ code, message }` payloads or re-instantiate typed errors on receipt. String errors (`reject('not found')`) destroy stack traces — reject with `new Error(...)` always.

**R7 — Debugging arsenal. [Stable]**
```bash
node --trace-exit app.js          # who called process.exit
node --trace-uncaught app.js      # full async stack of the crash
node --enable-source-maps app.js # mapped stacks (bundled/transpiled code)
node --inspect-brk app.js         # debugger
```
AggregateError (from `Promise.any`) and `err.errors[]` — remember to log the inner errors. `Error.isError` (V8 13.6, Node 24) for validating error-like objects crossing boundaries.

## Anti-patterns (agent review checklist)

- `try { ... } catch (e) {}` — empty catch blocks.
- `catch (e) { return null }` — converting crashes into null-propagation.
- String rejections / `throw 'string'`.
- `process.exit(0)` inside error handlers — masks failure from orchestrators.
- Domain module (`node:domain`) — dead for a decade; delete on sight ([deprecations.md](./deprecations.md)).
