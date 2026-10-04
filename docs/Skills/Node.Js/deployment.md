# Deployment & Operations

Docker, graceful shutdown, configuration, single-binary distribution. Sources: [Snyk Docker best practices](https://snyk.io/blog/10-best-practices-to-containerize-nodejs-web-applications-with-docker), [goldbergyoni/nodebestpractices](https://github.com/goldbergyoni/nodebestpractices), official [SEA docs](https://nodejs.org/api/single-executable-applications.html).

## Docker [Stable]

**R1 — Canonical Dockerfile for Node 24.**
```dockerfile
# ---- deps + build ----
FROM node:24-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build            # tsc emit for distributable JS (typescript.md R7)

# ---- runtime ----
FROM node:24-slim
ENV NODE_ENV=production
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev        # or COPY --from=build /app/node_modules for the lean path
COPY --from=build /app/dist ./dist
USER node
EXPOSE 3000
HEALTHCHECK CMD node dist/healthcheck.js
ENTRYPOINT ["node", "dist/server.js"]
```
Why each line:
- **multi-stage**: build tooling (devDeps, TS) never ships → smaller image, smaller attack surface.
- **`node:24-slim`** (or distroless): full images are for debugging only.
- **`npm ci`**: exact lockfile installs ([security.md](./security.md) R3).
- **`USER node`**: never run as root in containers.
- **`ENTRYPOINT ["node", ...]`**: direct exec form; exec syntax keeps Node as PID 1 — see R2's signal note.
- `NODE_ENV=production` skips dev warnings and enables dependency optimizations.

**R2 — Graceful shutdown: handle SIGTERM, drain, exit. [Stable]**

`server.close()` **returns immediately** — it stops accepting new connections and fires
its callback later, once in-flight requests finish. Measured on Node 22: with a request
still 700ms from completing, `close()` returned in 2ms and a `process.exit(0)` 200ms
later killed the request mid-flight. You must **await the callback**, not call and move on.

```js
import { once } from 'node:events';

const server = app.listen(3000);
let shuttingDown = false;

async function shutdown(signal) {
  if (shuttingDown) return;          // SIGTERM then SIGINT must not run this twice
  shuttingDown = true;
  log.info({ signal }, 'draining');
  setReady(false);                   // /readyz starts returning 503 (R4) before we stop accepting

  const closed = new Promise((resolve, reject) =>
    server.close(err => (err ? reject(err) : resolve())));

  // Keep dropping keep-alive sockets AS THEY GO IDLE — one call is not enough (see below)
  const reaper = setInterval(() => server.closeIdleConnections(), 50).unref();
  closed.finally(() => clearInterval(reaper));

  const timedOut = Symbol('timeout');
  const result = await Promise.race([
    closed.then(() => closeDatabaseAndJobs()),        // in-flight done → THEN release resources
    new Promise(r => setTimeout(() => r(timedOut), 10_000).unref()),
  ]);

  if (result === timedOut) {
    log.error('drain timed out — forcing close');
    server.closeAllConnections();
    process.exit(1);                 // nonzero: the deploy dropped requests, alert on it
  }
  process.exit(0);
}
process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT',  () => shutdown('SIGINT'));
```

Why the ordering matters:
- **Readiness off first**, so the load balancer stops sending new work while the old
  connections drain. Flipping this leaves a window of routed-but-refused requests.
- **`closeIdleConnections()` on an interval, not once.** A socket that is *busy* when you
  call it stays open, and once its response finishes it sits idle until
  `server.keepAliveTimeout` (default **5000ms**) expires. Measured drain of a single 700ms
  request on Node 22:

  | Strategy | `server.close()` completed |
  |---|---|
  | `closeIdleConnections()` once | **6737 ms** |
  | on a 50ms interval | **730 ms** |
  | `keepAliveTimeout = 1` + once | 1739 ms |

  A one-shot call turns a sub-second drain into a multi-second one on every deploy — and
  under real keep-alive traffic it reliably hits the hard cap instead.
- **Close DB pools *after* the server closes**, not in parallel — in-flight handlers still
  need the pool.
- **Exit 1 on timeout**, not 0. A drain that timed out dropped user requests; reporting
  success hides it from your deploy metrics.

Orchestrators send SIGTERM and wait `terminationGracePeriodSeconds` (default 30s) before
SIGKILL — keep the hard cap comfortably under that.

Caveats: if Node is not PID 1 (e.g. started via shell/npm), signals don't reach it — use
exec form, `node --run`, or an init (`tini`/`dumb-init`) as PID 1.

> The same pattern for Express, with connection tracking, is in
> `../Express JS/health-shutdown.md`.

## Memory limits in containers [Stable]

**R2b — Measure the runtime's container memory limits and leave room outside the heap.**

Node's default heap sizing depends on the runtime and detected cgroup limits. Inspect
`process.constrainedMemory()` and `v8.getHeapStatistics().heap_size_limit` inside the
deployed container. Do not assume a host measurement applies to a constrained container.
An explicit heap cap can help bound usage, but it limits only V8 old space: buffers,
native allocations, stacks, and other memory can still push total RSS over the container
limit and cause an OOM kill. Set the cap from load-test measurements and monitor total RSS.

```dockerfile
# Illustrative starting cap only; measure RSS under load before adopting. Leave room for
# buffers, native memory, and the stack (all of which live OUTSIDE the heap).
ENV NODE_OPTIONS="--max-old-space-size=384"     # for a 512Mi container
ENTRYPOINT ["node", "dist/server.js"]
```

```js
// Verify at boot — cheap insurance against a silent misconfiguration
import v8 from 'node:v8';
log.info({ heapLimitMB: Math.round(v8.getHeapStatistics().heap_size_limit / 2 ** 20) }, 'heap');
```

- `--max-old-space-size` caps only the **old space**. `Buffer`s, WASM memory, and native
  addon allocations are outside it, so the container limit must be meaningfully higher.
- Symptom map: **exit 137** = kernel OOM-kill (limit too low, or heap cap too high);
  `FATAL ERROR: Reached heap limit — JavaScript heap out of memory` = V8's own limit hit
  (a real leak, or a cap that is too low). They mean opposite things.
- `os.availableParallelism()` has the same problem in reverse — it reports host CPUs, not
  your CPU quota. Size worker pools from an env var, not from the machine.
- This is the one place `NODE_OPTIONS` earns its keep (R3 warns against it otherwise),
  because `--max-old-space-size` must be set before the process starts.

## Configuration [Stable]

**R3 — 12-factor: config from environment; fail fast at boot.**
```bash
node --env-file-if-exists=.env.local dist/server.js   # local override file
```
- Parse + validate `process.env` once with a schema ([security.md](./security.md) R5), export a frozen `env` object.
- **`--env-file` never overrides an already-set variable.** Verified: with
  `FOO=from_shell` exported and `FOO=from_file` in the file, `process.env.FOO` is
  `from_shell`. That is the safe precedence (orchestrator secrets beat a stray committed
  file), but it surprises people who expect `.env` to win locally — if a value "won't
  change", check for an exported shell variable first.
- Never read config lazily per-request (config drift mid-process; masks boot errors).
- `NODE_OPTIONS` is a foot-gun in prod: prefer explicit CLI flags in ENTRYPOINT.

**R4 — Health endpoints: liveness vs readiness. [Stable]**
- `/healthz` (liveness): process is sane — no dependency checks (a dead DB shouldn't get the pod killed-looping).
- `/readyz` (readiness): can serve — checks DB/queue connectivity. Return 503 while draining in shutdown (R2) so load balancers stop routing.

## Observability [Stable]

**R5 — Structured JSON logs (pino), one line per event; request-scoped context via AsyncLocalStorage.**
```js
import pino from 'pino';
const log = pino({ level: process.env.LOG_LEVEL ?? 'info', redact: ['req.headers.authorization'] });
```
Correlate with `AsyncLocalStorage` ([runtime-apis.md](./runtime-apis.md) R6); export Prometheus metrics (`prom-client`) or OTel traces. Console.log has no levels/context — not for production services.

## Single-binary / distribution [Context]

**R6 — SEA for standalone CLIs; containers still rule for services.**
`node --build-sea sea-config.json` bundles a JS app + runtime into one executable. Added
in **v25.5.0** (so present in 24? **no** — 25.5+ and 26+ only; on Node 24 LTS you still use
the older multi-step `postject` flow). SEA overall is **Stability 1.1 — Active
development**, not stable: expect the config format and flags to move. Use when distributing CLIs to users without Node. Alternatives: `pkg` (legacy), Bun `--compile`. For servers, containers remain simpler to operate. Caveat: SEA assets are injected into the binary — large assets make big binaries; cross-compilation per target platform required. Source: [nodejs.org/api/single-executable-applications.html](https://nodejs.org/api/single-executable-applications.html).

## Ops checklist (pre-prod review)

- [ ] Dockerfile follows R1 (multi-stage, slim, non-root, `npm ci`)
- [ ] SIGTERM handler with drain + hard cap (R2)
- [ ] Env schema-validated at boot; secrets via orchestrator (R3)
- [ ] `/healthz` + `/readyz` with 503-while-draining (R4)
- [ ] Structured logs with redaction; metrics wired (R5)
- [ ] Outbound timeouts everywhere ([frameworks.md](./frameworks.md) R4)
- [ ] Restart policy/alerting on nonzero exits ([error-handling.md](./error-handling.md) R5)
