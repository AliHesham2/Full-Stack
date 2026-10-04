# Error Handling

Express 5's biggest error handling improvement: **rejected promises from async handlers are automatically forwarded to the error handler.** This eliminates the need for try/catch wrappers in most cases.

## Express 5 Async Handling

```js
// async handlers forward errors automatically
app.get('/users/:id', async (req, res) => {
  const user = await db.getUser(req.params.id);
  res.json(user);
  // if db.getUser rejects, Express calls next(err) automatically
});
```

**Best practice**: Still catch errors close to the source when you can provide better context:

```js
app.get('/users/:id', async (req, res) => {
  const user = await db.getUser(req.params.id).catch(err => {
    throw new AppError('User lookup failed', 503, { code: 'DEPENDENCY_UNAVAILABLE', cause: err });
  });
  if (!user) throw new NotFoundError('User'); // successful lookup with no row
  res.json({ data: user });
});
```

Source: [Express 5 Error Handling Docs](https://expressjs.com/en/5x/guide/error-handling)

## Custom Error Class

```js
// Every error carries a machine code: the response envelope is { error: { code, message, details? } }
// (details only for a 400 validation error), and the frontend's ApiError reads exactly those
// fields (backend-guide/03-auth §0).
export class AppError extends Error {
  constructor(message, statusCode = 500, options = {}) {
    super(message, options);
    this.name = 'AppError';
    this.statusCode = statusCode;
    this.code = options.code ?? 'ERROR';
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, { code: 'NOT_FOUND' });
    this.name = 'NotFoundError';
  }
}

export class ValidationError extends AppError {
  constructor(details) {
    super('Validation failed', 400, { code: 'VALIDATION_ERROR' }); // shape errors are 400 (backend-guide validation.md)
    this.name = 'ValidationError';
    this.details = details;
  }
}
```

## Centralized Error Handler

```js
// Must be 4 arguments: (err, req, res, next)
app.use((err, req, res, next) => {
  // If headers already sent (streaming error), delegate to default
  if (res.headersSent) return next(err);

  const statusCode = err.statusCode ?? 500;
  // body-parser rejections (bad JSON → 400, body too large → 413) are client errors, not bugs
  const rejected = !err.isOperational && err.expose === true && statusCode < 500;
  const message = err.isOperational ? err.message
    : rejected ? 'The request was rejected' : 'Internal Server Error';

  // Log full error internally (req.log exists only if pino-http is mounted — see middleware.md)
  (req.log ?? console).error({ err, path: req.path }, 'Request error');

  res.status(statusCode).json({
    error: {
      code: err.isOperational ? err.code : rejected ? 'REQUEST_REJECTED' : 'INTERNAL',
      message,
      ...(err.isOperational && err.details && { details: err.details }), // never a library's own details
    },
  });
});
```

**Critical rules**:
- Define error handlers **after** all routes and middleware
- Always 4 parameters — Express identifies error handlers by arity
- Always check `res.headersSent` before writing
- Never put a stack trace in a response body, in any environment: it is logged above, and the
  body keeps only `code`, `message` and `details` (backend-guide/02-domain-and-api/api-design.md §4)

## Promise Chain Pattern

If you can't use async/await, return the promise:

```js
// Express catches rejection because promise is returned
app.get('/data', (req, res) => {
  return fetchData().then(data => res.json(data));
});

// If you DON'T return, you must catch manually:
app.get('/data', (req, res, next) => {
  fetchData().then(data => res.json(data)).catch(next);
});
```

## Callback APIs

```js
import { readFile } from 'node:fs';

// Pass error to next
app.get('/file', (req, res, next) => {
  readFile('/path', (err, data) => {
    if (err) return next(err);
    res.send(data);
  });
});

// Or use next as callback directly
app.get('/file', (req, res, next) => {
  readFile('/path', next);  // fs passes (err, data), Express uses err
});
```

## What NOT to Do

- **Never *recover* from `uncaughtException`** — process state is unreliable afterwards. Registering a handler purely to flush logs and then `process.exit(1)` is fine; continuing to serve traffic is not. Same for `unhandledRejection`, which terminates the process by default since Node 15.
- **Never use `domains`** — deprecated Node module.
- **Don't swallow errors** — `catch(() => {})` without logging/re-throwing hides bugs.
- **Don't send errors after `res.end()`** — check `res.headersSent`.