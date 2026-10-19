# Performance & Production Reliability

## Code-level Optimizations

### Set NODE_ENV to "production"

**3x performance improvement** from view caching and reduced error verbosity.

```bash
# systemd unit file
Environment=NODE_ENV=production

# or in shell (dev only)
NODE_ENV=production node server.js
```

> `process.env` lookups hit a native getter on every read. Read it once at module load (`const isProd = process.env.NODE_ENV === 'production'`) rather than inside a request handler.

### Gzip Compression

```js
import compression from 'compression';
app.use(compression());
```

**Production**: Prefer compression at the reverse proxy (Nginx) level — offloads from Node.

### No Synchronous Functions in Handlers

```js
// BAD — blocks event loop
const data = fs.readFileSync('/large-file.json');

// GOOD
const data = await fs.promises.readFile('/large-file.json', 'utf-8');
```

Detect sync I/O during dev:
```bash
node --trace-sync-io server.js
```

### Worker Threads for CPU-bound Work

```js
import Piscina from 'piscina';
import path from 'node:path';

const pool = new Piscina({
  filename: path.resolve('./workers/heavy-task.js'),
});

app.post('/process', async (req, res, next) => {
  try {
    const result = await pool.run(req.body.data);
    res.json(result);
  } catch (err) {
    next(err);
  }
});
```

Don't spawn a Worker per request — use a pool (Piscina or manual). Workers are expensive to create.

Source: [Official performance best practices](https://expressjs.com/en/advanced/best-practice-performance)

## Environment-level Optimizations

### Use Latest Node.js LTS

Each Node.js version brings V8 improvements. Same code, more throughput.

### Process Manager

Use your OS init system (systemd) instead of PM2 when possible:

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Express App

[Service]
Type=simple
ExecStart=/usr/local/bin/node /app/index.js
WorkingDirectory=/app
User=nobody
Group=nogroup
Environment=NODE_ENV=production
LimitNOFILE=infinity
Restart=always

[Install]
WantedBy=multi-user.target
```

### Scaling across cores

One Node process uses one core. Two ways to use the rest:

1. **Container replicas** (preferred) — one process per container, let Kubernetes/ECS scale horizontally. Simplest to reason about, and it also survives a node failure.
2. **`node:cluster`** — fork one worker per core inside a single container, for VM/bare-metal deployments where there is no orchestrator.

```js
// cluster.js — only when there is no orchestrator doing this for you
import cluster from 'node:cluster';
import { availableParallelism } from 'node:os';

if (cluster.isPrimary) {
  for (let i = 0; i < availableParallelism(); i++) cluster.fork();
  cluster.on('exit', () => cluster.fork());
} else {
  await import('./index.js');
}
```

**Critical**: forked processes share no memory. In-process sessions, rate-limit counters, and caches all break — move them to Redis. See [security.md](./security.md) for the rate-limit store.

> PM2 still works, but a process manager inside a container duplicates what the orchestrator already does and hides crashes from it. Prefer systemd on VMs, replicas in containers.

### Reverse Proxy (Nginx)

Offload TLS, compression, static files, and load balancing:

```
upstream express_app {
  server 127.0.0.1:3000;
  server 127.0.0.1:3001;
}
server {
  listen 443 ssl http2;
  server_name example.com;

  ssl_certificate     /path/to/cert.pem;
  ssl_certificate_key /path/to/key.pem;

  location / {
    proxy_pass http://express_app;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

### HTTP Keep-Alive Timeouts (behind ALB/ELB)

The classic source of random 502s: the load balancer reuses a keep-alive connection that Node closed a moment earlier. Node's server `keepAliveTimeout` must be **longer** than the load balancer's idle timeout (AWS ALB defaults to 60s).

```js
const server = app.listen(config.port);
server.keepAliveTimeout = 65_000;   // > ALB idle timeout (60s)
server.headersTimeout   = 66_000;   // must exceed keepAliveTimeout
```

Nginx upstreams need the mirror setting (`keepalive_timeout`) plus `proxy_http_version 1.1;` and `proxy_set_header Connection "";`, or keep-alive to the upstream never happens at all.

### Observability

Metrics and traces are not optional in production. Instrument once, at the edge:

```js
// tracing.js — imported FIRST, before express
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

new NodeSDK({ instrumentations: [getNodeAutoInstrumentations()] }).start();
```

```bash
node --import ./tracing.js dist/index.js
```

- The auto-instrumentation patches Express, http, and most DB drivers — no per-route code.
- Correlate logs with traces by adding the trace ID to your pino context, or logs and traces stay two disconnected systems.
- Watch **event loop lag** (`perf_hooks.monitorEventLoopDelay`) as the primary saturation signal — CPU% lies for an I/O-bound Node process.

### Conditional requests (ETag)

Express computes a **weak ETag for every response body** by default and answers matching
`If-None-Match` with a bodyless 304. Verified on 5.2.1:

```
GET /data                          → 200  ETag: W/"6e-GbVyXkTQ/+vnwmdnVc+C1UEEfaQ"
GET /data  If-None-Match: W/"6e-…" → 304  (no body)
GET /data  If-None-Match: W/"6e-…"
           Cache-Control: no-cache → 200  (client explicitly opted out)
```

This saves bandwidth but **not** server work — the handler still runs and the body is
still generated; only the transfer is skipped. For expensive endpoints, compute a cheap
validator yourself and short-circuit before the work:

```js
app.get('/report', async (req, res) => {
  const version = await getReportVersion();     // cheap
  res.set('ETag', `W/"${version}"`);
  if (req.fresh) return res.status(304).end();  // skips the expensive part
  res.json(await buildExpensiveReport());
});
```

```js
app.set('etag', 'strong');   // hash-based; needed for Range requests
app.set('etag', false);      // disable — do this for streamed/large responses
```

Pair ETags with an explicit `Cache-Control` — without one, intermediaries guess:

| Content | `Cache-Control` |
|---|---|
| Fingerprinted asset (`app.3f9d.js`) | `public, max-age=31536000, immutable` |
| Personalized API response | `private, no-cache` (revalidate every time, still 304-able) |
| Secret / auth response | `no-store` |

### Caching

- **Response cache**: Varnish or Nginx for GET endpoints
- **Data cache**: Redis for expensive DB queries
- **CDN**: CloudFlare/Fastly for static assets

### Load Balancing

Multiple Express instances behind Nginx or HAProxy. Use sticky sessions only if needed (prefer stateless design with Redis sessions).

## Performance Anti-patterns

1. Running `console.log` in production (synchronous I/O)
2. Not setting `NODE_ENV=production`
3. Storing state in process memory (breaks with clustering)
4. Serving large files through Express (use Nginx + `X-Accel-Redirect`)
5. Heavy computation in request handlers (offload to workers)
6. Not enabling compression
7. `keepAliveTimeout` shorter than the load balancer idle timeout (intermittent 502s)
8. No event-loop-lag metric — you find out from users, not dashboards