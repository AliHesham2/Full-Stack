# Streaming, SSE & Large Responses

Anything that doesn't fit comfortably in memory must not go through `res.json()`.
Verified against express 5.2.1 on Node 22.

## Always use `pipeline`, never bare `.pipe()`

```js
import { pipeline } from 'node:stream/promises';
import { createReadStream } from 'node:fs';

app.get('/export.csv', async (req, res, next) => {
  res.type('text/csv').attachment('export.csv');
  try {
    await pipeline(rowsToCsv(query()), res);
  } catch (err) {
    next(err);            // headers may already be sent — the handler must check res.headersSent
  }
});
```

**Why `pipeline` and not `src.pipe(res)`**: `.pipe()` does not propagate errors and does
not destroy the source when the destination fails. A client that disconnects mid-download
leaves the source stream open — a file descriptor or DB cursor leak that only shows up
under load. `pipeline` destroys every stream in the chain on any failure.

Backpressure is handled by both, and that is the point of streaming at all: `res.write()`
returns `false` when the socket buffer is full, and a stream respects it. A `for` loop
calling `res.write()` without checking the return value buffers the whole response in
memory — the exact thing streaming was meant to avoid.

## Errors after the first byte

Once headers are sent you cannot change the status code. A stream that fails halfway
can deliver a **truncated 200**. A framing/connection error may expose truncation, but applications must not assume a 200 status proves the stream completed.

```js
app.use((err, req, res, next) => {
  if (res.headersSent) return next(err);   // delegate to Express's finalhandler → destroys the socket
  // ...normal JSON error response
});
```

**Rules**:
- Validate everything *before* writing the first byte — after that, the only honest
  signal is aborting the connection.
- For formats that can carry a trailing status (NDJSON, CSV with a footer), emit an
  explicit terminator so the client can detect truncation.
- Set `Content-Length` when known and preserve proper framing. Clients must handle stream/body read errors; for application-level completion, validate an expected length, checksum or terminator.

## Server-Sent Events

```js
app.get('/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache, no-transform',   // no-transform stops proxies buffering
    Connection: 'keep-alive',
    'X-Accel-Buffering': 'no',                   // tells nginx not to buffer
  });

  // Minimal bounded policy: disconnect a slow client instead of queuing more frames.
  // For lossless delivery, use a capped queue, drain handling and event IDs.
  const writeFrame = (frame) => {
    if (!res.write(frame)) res.destroy();
  };
  const send = (data, event) => {
    // Event names come from a server-controlled allowlist, never raw client input.
    const prefix = event ? 'event: ' + event + '\n' : '';
    writeFrame(prefix + 'data: ' + JSON.stringify(data) + '\n\n');
  };

  const heartbeat = setInterval(() => writeFrame(': ping\n\n'), 15_000);
  const unsubscribe = bus.subscribe(send);

  res.on('close', () => {            // outgoing response completed, disconnected or destroyed
    clearInterval(heartbeat);
    unsubscribe();
  });
});
```

**Rules**:
- **Always clean up on `res.on('close')`.** Without it every disconnected client leaves a
  live interval and subscription — the most common SSE memory leak.
- **Send a heartbeat comment** (`: ping\n\n`) every 15–30s or proxies and load balancers
  will kill an idle connection.
- **Disable compression on this route.** `compression()` buffers, so events arrive in
  batches or not at all:
  ```js
  app.use(compression({ filter: (req, res) => res.getHeader('Content-Type') !== 'text/event-stream' }));
  ```
- `Content-Type: text/event-stream` and the trailing blank line are both mandatory —
  a missing `\n\n` means the browser never dispatches the event.
- Browsers auto-reconnect and send `Last-Event-ID`; emit `id:` lines if resumption matters.

**SSE vs WebSocket**: SSE is one-directional HTTP and EventSource reconnects automatically.
Proxy buffering/timeouts still require configuration. Choose WebSocket for low-latency bidirectional traffic; ordinary client writes can use separate HTTP mutations.

IncomingMessage req.close describes completion of the incoming request on modern Node,
not the lifetime of an outgoing SSE response. Tie subscriptions to res.close. Bound per-client
buffers, handle write(false)/drain, and disconnect slow clients instead of queuing indefinitely.

## Downloads

```js
res.download('/abs/path/report.pdf', 'report.pdf', (err) => {
  if (err && !res.headersSent) next(err);
});
```
`res.download` / `res.sendFile` handle Range requests, ETags and `Content-Disposition`.
Never read a file into memory and `res.send()` it.

For high-volume static or large files behind nginx, hand off entirely — `X-Accel-Redirect`
(nginx) or `X-Sendfile` (Apache) lets the proxy stream the bytes while Express keeps only
the authorization decision. → [performance.md](./performance.md)

## Uploads

Streaming *in* has the mirror problem: multer buffers to memory or disk before your
handler runs. For large uploads, stream straight to object storage with `busboy` rather
than landing the file locally. Limits still apply first → [middleware.md](./middleware.md).

## Anti-patterns

1. **`src.pipe(res)`** without error handling — leaks the source on client disconnect.
2. **`res.write()` in a loop ignoring the return value** — unbounded memory growth.
3. **`res.json(hugeArray)`** — serializes the whole array into one string first. Use
   NDJSON (`application/x-ndjson`, one JSON object per line) for large result sets.
4. **No heartbeat on SSE** — connections die silently after the proxy idle timeout.
5. **`compression()` applied to SSE** — events never flush.
6. **Forgetting `res.on('close')`** — the leak scales with your traffic.
