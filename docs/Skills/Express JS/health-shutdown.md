# Health Checks & Graceful Shutdown

## Health Check Endpoint

Required for load balancer integration (Kubernetes, AWS ALB, etc.):

```ts
import { Router } from 'express';
import { db } from '../config/db.js';

const router = Router();

// Liveness: "is the process alive?" — lightweight, always 200
router.get('/live', (req, res) => {
  res.json({ status: 'ok' });
});

// Readiness: "can it handle requests?" — checks dependencies
router.get('/ready', async (req, res) => {
  try {
    await db.raw('SELECT 1');
    res.json({ status: 'ok', db: 'connected' });
  } catch {
    res.status(503).json({ status: 'unhealthy', db: 'disconnected' });
  }
});

export default router;
```

| Check | Purpose | Kubernetes equivalent |
|-------|---------|----------------------|
| `/health/live` | Process alive? Restart if fails | `livenessProbe` |
| `/health/ready` | Dependencies OK? Remove from LB if fails | `readinessProbe` |

> **Why the interval?** `closeIdleConnections()` only drops sockets that are idle *at
> that instant*. A socket still serving a request survives the call, then sits idle until
> `server.keepAliveTimeout` (default 5000ms) expires. Measured on Node 22, draining one
> 700ms request: a single call finished in **6737ms**, a 50ms interval in **730ms**.
> Under real keep-alive traffic the one-shot version hits your hard cap on every deploy.

**Rules**:
- Liveness checks must be lightweight (no DB calls, no external deps)
- Readiness checks should verify the app can actually serve requests
- Return 503 (not 500) when unhealthy — signals "temporary" to load balancers

Source: [Express Health Checks Guide](https://expressjs.com/en/advanced/healthcheck-graceful-shutdown)

## Graceful Shutdown

When deploying, the process manager sends `SIGTERM`. The app must:
1. Stop accepting new connections
2. Finish in-flight requests
3. Close DB connections / release resources
4. Exit

```ts
import { createServer } from 'node:http';

const server = createServer(app);

// Track open connections
let connections = [];
server.on('connection', (conn) => {
  connections.push(conn);
  conn.on('close', () => {
    connections = connections.filter(c => c !== conn);
  });
});

let shuttingDown = false;

function shutdown(signal) {
  if (shuttingDown) return;   // SIGTERM and SIGINT can both fire — run once
  shuttingDown = true;
  console.log(`${signal} received: closing server`);
  
  // Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');
    clearInterval(reaper);
    await closeDatabaseAndJobs();   // in-flight handlers are done — now release resources
    process.exit(0);
  });

  // Drop keep-alive sockets AS THEY GO IDLE — a single call is not enough
  const reaper = setInterval(() => server.closeIdleConnections(), 50).unref();

  // Force-close after timeout — exit 1 so the deploy is flagged
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    clearInterval(reaper);
    server.closeAllConnections();
    process.exit(1);
  }, 10000).unref();
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));

server.listen(config.port);
```

### Server Start Errors

`app.listen()` passes startup errors (e.g. `EADDRINUSE`) to its callback instead of throwing:

```js
const server = app.listen(config.port, (error) => {
  if (error) throw error;  // e.g. port already in use
  console.log(`Listening on ${server.address().port}`);
});
```

## Kubernetes Example

```yaml
spec:
  containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /health/live
        port: 3000
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 3000
      initialDelaySeconds: 5
      periodSeconds: 5
    terminationGracePeriodSeconds: 30
```

## Anti-patterns

1. **Heavy liveness probes** — keep `/live` response under 5ms
2. **No timeout on shutdown** — stuck connections prevent restarts
3. **Calling `closeIdleConnections()` once** — keep-alive sockets that go idle later still
   block `close()` for `keepAliveTimeout` (5s)
4. **Forgetting DB cleanup** — connection pools leak file descriptors
5. **Closing the DB pool in parallel with `server.close()`** — in-flight handlers still
   need it; close it *inside* the close callback
6. **Calling `process.exit(0)` before server closes** — drops in-flight requests
7. **Exiting 0 after a drain timeout** — hides dropped requests from deploy metrics
