# JavaScript Async Patterns

## Defaults

- Use `async/await`. Never use raw `.then()` chains.
- Use `try/catch` for errors, not `.catch()` chains.
- Top-level `await` is available in ESM modules.

## Event loop ordering

Execution order, guaranteed by spec:

1. Run synchronous code to completion.
2. Drain the **microtask queue**: promise callbacks, `queueMicrotask`, `await` resumptions — until empty.
3. Take one **macrotask**: timers (`setTimeout`/`setInterval`), I/O callbacks, UI events.
4. Repeat — microtasks drain again after every macrotask.

```js
setTimeout(() => console.log('timeout'));
queueMicrotask(() => console.log('microtask'));
console.log('sync');
// Output: sync → microtask → timeout
```

**Rule:** Microtasks starve macrotasks. A recursive microtask chain blocks timers and I/O forever.

### Node adds `process.nextTick` — and its priority depends on module system

`process.nextTick` has its own queue, drained *around* the microtask queue. Which one
wins is not the same in CJS and ESM. Verified on Node 22 with identical code:

```js
setTimeout(() => console.log('setTimeout'));
setImmediate(() => console.log('setImmediate'));
Promise.resolve().then(() => console.log('promise'));
queueMicrotask(() => console.log('queueMicrotask'));
process.nextTick(() => console.log('nextTick'));
console.log('sync');
```

| | CJS (`.cjs`) | ESM (`.mjs`) |
|---|---|---|
| 1 | sync | sync |
| 2 | **nextTick** | promise |
| 3 | promise | queueMicrotask |
| 4 | queueMicrotask | **nextTick** |
| 5 | setTimeout | setTimeout |
| 6 | setImmediate | setImmediate |

ESM module evaluation already runs inside a promise job, so the microtask queue keeps
draining before control returns to the nextTick checkpoint.

**Rule:** Never write logic that depends on `nextTick` vs promise ordering — the same
file converted from CJS to ESM silently reorders. Use `queueMicrotask` when you need a
microtask, and reserve `process.nextTick` for library internals that must run before
any I/O.

## Error handling

### Throw Error instances, not strings
```js
// ✗
throw 'not found';
// ✓
throw new Error('not found');

// ✓ With cause chaining (ES2022)
throw new Error('Failed to fetch user', { cause: fetchError });

// ✓ Custom error classes for domain errors
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}
```

### Catch at the boundary
```js
// ✓ At the request handler / worker / top level
app.get('/api/data', async (req, res, next) => {
  try {
    const data = await fetchData(req.params.id);
    res.json(data);
  } catch (err) {
    next(err); // the ONE central error handler logs it and sends the one envelope
               // { error: { code, message, details? } } (the backend project's error handler)
  }
});

// ✗ Don't wrap every individual await
```

### Unhandled rejection — log, then let it kill the process

Since Node 15 an unhandled rejection **terminates the process by default**. A handler
that only logs silently restores the old swallow-and-continue behaviour and leaves the
process in an unknown state.

```js
// Node.js — ✓ log for diagnosis, then still die
process.on('unhandledRejection', (reason) => {
  logger.fatal({ err: reason }, 'Unhandled rejection');
  process.exit(1);          // let the supervisor restart a clean process
});

// ✗ Never do this — turns a crash into a corrupted-state server
process.on('unhandledRejection', (reason) => logger.error(reason));
```

```js
// Browser — reporting only; there is no process to restart
window.addEventListener('unhandledrejection', (event) => {
  errorReporter.report(event.reason);
  // preventDefault() only silences the console — use it deliberately, not by habit
});
```

## HTTP with fetch

### fetch does NOT reject on HTTP errors
```js
const res = await fetch(url);
if (!res.ok) {                          // REQUIRED — 404/500 resolve normally
  throw new Error(`HTTP ${res.status}: ${res.statusText}`);
}
const data = await res.json();
```
Only network failures (DNS, offline, aborted request) reject. Every non-2xx response arrives as a fulfilled promise.

### Consume or release the body
```js
const data = await res.json();          // or .text(), .arrayBuffer()
// On early-return paths: res.body?.cancel()
```
Unread bodies hold connections open (undici keeps sockets until the body is consumed).

### Timeout
```js
// Modern — Node 18+, all evergreen browsers
const res = await fetch(url, { signal: AbortSignal.timeout(5000) });

// Composed with your own cancellation
const controller = new AbortController();
const res = await fetch(url, {
  signal: AbortSignal.any([controller.signal, AbortSignal.timeout(5000)]),
});
```

### Retry with backoff (idempotent requests only)
```js
async function fetchRetry(url, retries = 3, signal) {
  for (let attempt = 0; ; attempt++) {
    try {
      const res = await fetch(url, { signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res;
    } catch (err) {
      if (attempt >= retries || signal?.aborted) throw err;
      await new Promise(r => setTimeout(r, 2 ** attempt * 250)); // 250ms, 500ms, 1000ms
    }
  }
}
```
Never blindly retry non-idempotent POST requests.

### Sync throws inside an async chain
```js
// ✗ If buildRequest() throws synchronously, this throws instead of rejecting
const p = fetchWith(buildRequest(input));

// ✓ Promise.try (ES2025) normalizes sync throws into rejections
const p = Promise.try(() => fetchWith(buildRequest(input)));
```
Inside an `async function` this is already handled — the issue only arises in
synchronous code that builds promises. → [modern-features.md](./modern-features.md)

## Concurrency

### Parallel — independent operations
```js
const [users, posts] = await Promise.all([fetchUsers(), fetchPosts()]);
```

### First success wins — `Promise.any`
```js
// Resolves with the FIRST fulfilled promise; rejects only if ALL reject
const fastest = await Promise.any([fromCache(), fromReplica(), fromOrigin()]);
```
On total failure it throws an `AggregateError` — the individual reasons live on
`.errors`, not `.message`:
```js
try {
  await Promise.any(sources);
} catch (err) {
  if (err instanceof AggregateError) logger.error({ reasons: err.errors }, 'all sources failed');
  throw err;
}
```

| Combinator | Settles when | Rejects when |
|---|---|---|
| `Promise.all` | all fulfil | **any** rejects (fast-fail) |
| `Promise.allSettled` | all settle | never |
| `Promise.any` | **first** fulfils | all reject → `AggregateError` |
| `Promise.race` | first **settles** | first settles as a rejection |

**Gotcha:** `Promise.race` rejects on the first rejection even if a later promise would
have succeeded — reach for `Promise.any` when you want "whichever works", and `race`
only for timeouts. Neither cancels the losers; pass an `AbortSignal` if that matters.

### Partial failure tolerance
```js
const results = await Promise.allSettled([fetchA(), fetchB(), fetchC()]);
const successes = results.filter(r => r.status === 'fulfilled').map(r => r.value);
const failures = results.filter(r => r.status === 'rejected').map(r => r.reason);
```

### Sequential — order matters
```js
for (const id of ids) {
  await processItem(id);
}
```

### Bounded concurrency
```js
async function mapBounded(items, fn, limit = 5) {
  const results = [];
  const executing = new Set();
  for (const item of items) {
    const p = fn(item).then(r => { executing.delete(p); return r; });
    executing.add(p);
    results.push(p);
    if (executing.size >= limit) await Promise.race(executing);
  }
  return Promise.all(results);
}
```

## Cancellation

### AbortController — the standard
```js
const controller = new AbortController();
const { signal } = controller;

// With fetch
const res = await fetch(url, { signal });

// In your own async functions
async function myOperation(signal) {
  signal?.throwIfAborted();
  return new Promise((resolve, reject) => {
    const onAbort = () => reject(signal.reason);
    signal?.addEventListener('abort', onAbort, { once: true });
    // ... work ...
    signal?.removeEventListener('abort', onAbort);
    resolve(result);
  });
}

// Cancel
controller.abort();
```
**Rule:** Any async function doing significant work should accept an `AbortSignal`.

## Promise creation

### `Promise.withResolvers()` (ES2024)
```js
const { promise, resolve, reject } = Promise.withResolvers();
// resolve/reject callable from outside the executor
```
**When:** Event-driven APIs where resolve/reject need escaping the executor scope.
**Caveat:** Requires Node 22+ / Chrome 119+ / Safari 17.4+. Below that, use the traditional constructor. → [modern-features.md](./modern-features.md)

### Don't wrap already-async functions
```js
// ✗ Anti-pattern: unnecessary promise wrapper
return new Promise((resolve, reject) => {
  someAsyncFn().then(resolve).catch(reject);
});

// ✓
return someAsyncFn();
```

## Async iterators

```js
// Consuming
for await (const chunk of readableStream) {
  process(chunk);
}

// Creating
async function* paginate(url) {
  let cursor = null;
  do {
    const { data, nextCursor } = await fetchPage(url, cursor);
    yield* data;
    cursor = nextCursor;
  } while (cursor);
}

// Materializing (ES2026 edition; shipped since Node 22 / Chrome 121)
const arr = await Array.fromAsync(asyncIterable);
```
