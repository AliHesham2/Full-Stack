# Middleware

## Standard Pipeline Order

```js
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import pinoHttp from 'pino-http';
import { pino } from 'pino';

const logger = pino();

app.disable('x-powered-by');

// 1. Security headers (first)
app.use(helmet());

// 2. No browser CORS headers: Next.js is the intended caller.
//    Simple requests can still arrive; enforce private networking or service authentication.
//    CORS only controls cross-origin response access (backend auth model).

// 3. Rate limiting
app.use(rateLimit({ windowMs: 15*60*1000, limit: 100, standardHeaders: 'draft-8' }));

// 4. Request logging (after security, before handlers)
app.use(pinoHttp({ logger }));

// 5. Body parsing
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: false }));

// 6. Routes
app.use('/api', apiRouter);

// 7. 404 handler
app.use((req, res) => res.status(404).json({ error: { code: 'NOT_FOUND', message: 'Not found' } }));

// 8. Error handler (last)
app.use(errorHandler);
```

## Body Parser

```js
// JSON parsing (Brotli, gzip, deflate decompression built in)
app.use(express.json({
  limit: '10kb',           // cap body size
  strict: true,            // default — accepts only objects AND arrays; rejects bare "str", 12, true, null
}));

// URL-encoded (extended defaults to false)
app.use(express.urlencoded({ extended: false }));
// Use extended: true only if you need nested objects from form data
```

> **`strict` is commonly misread.** `strict: true` does *not* mean "objects only" — `[1,2]` parses fine. It rejects top-level JSON **primitives**, which `JSON.parse` would otherwise accept. Set `strict: false` only if a client genuinely posts a bare scalar.

### req.body is undefined until parsed

`req.body` is `undefined` (not `{}`) when no body parser has run. Access safely:
```js
const data = req.body ?? {};
```

## Query Parsing (`req.query`)

Default parser is `simple` — flat string keys, no nesting:

```js
// GET /search?filter[status]=open&page=2
req.query;  // { 'filter[status]': 'open', page: '2' }
```

Opt in to nested parsing only if clients rely on bracket syntax:

```js
app.set('query parser', 'extended');  // ?filter[status]=open → { filter: { status: 'open' } }
```

**Rules**:
- `req.query` is a read-only getter — clone it instead of mutating
- Prefer flat keys + zod validation over enabling `extended`

## Cookies — none in this project

This Express neither sets nor reads cookies. The session id arrives as
`Authorization: Bearer <session id>`, and the login returns `{ token }` in JSON; Next.js owns the
browser cookie (backend-guide/03-auth/authentication.md §0). No `cookie-parser`, and ESLint fails
the build on `res.cookie` / `clearCookie`.

## Static Files (production options)

```js
app.use(express.static('public', {
  maxAge: '1y',        // long cache is only safe for fingerprinted assets
  immutable: true,     // browsers skip revalidation entirely
  dotfiles: 'ignore',  // v5 default
}));
```

**Rule**: Fingerprint assets (`app.3f9d1c.js`) before using `maxAge: '1y'` + `immutable`. Non-hashed files get stale silently. For high-traffic static, offload to Nginx/CDN — see [performance.md](./performance.md).

The `dotfiles: 'ignore'` default also blocks hidden *directories* inside served paths. Serve dot-directories explicitly when needed (ACME challenges, app links):

```js
app.use('/.well-known', express.static('public/.well-known', { dotfiles: 'allow' }));
app.use(express.static('public'));
```

## Validation Middleware

See [security.md](./security.md) for zod validation pattern.

## Logging: Pino (not Morgan)

Pino is the fastest Node.js logger. Morgan is synchronous by default and not suitable for production.

```js
import pinoHttp from 'pino-http';
import { pino } from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  redact: ['req.headers.authorization', 'req.body.password'],
});

app.use(pinoHttp({ logger }));
```

Access logger in handlers via `req.log`:
```js
app.get('/users/:id', (req, res) => {
  req.log.info({ userId: req.params.id }, 'Fetching user');
  // ...
});
```

## Request context with AsyncLocalStorage

Threading a request ID through every function signature does not scale. `node:async_hooks`
carries it implicitly across async boundaries — verified working through `setTimeout`,
promises and awaits on Node 22+:

```js
// context.js
import { AsyncLocalStorage } from 'node:async_hooks';
export const requestContext = new AsyncLocalStorage();
export const getRequestId = () => requestContext.getStore()?.requestId;
```

```js
import { randomUUID } from 'node:crypto';
import { requestContext } from './context.js';

app.use((req, res, next) => {
  const requestId = req.get('x-request-id') ?? randomUUID();   // honour an upstream ID
  res.setHeader('x-request-id', requestId);
  requestContext.run({ requestId, userId: null }, next);       // next() INSIDE run()
});
```

Now any module — a repository, a retry helper, a logger — reads it without a parameter:

```js
logger.info({ requestId: getRequestId() }, 'charging card');
```

**Rules**:
- Call `next()` **inside** `.run()`. Calling it after leaves the store undefined for the
  rest of the pipeline.
- Mount it **first**, before logging and routing, or early middleware runs without context.
- Store identifiers and scalars only. It is not a request-scoped DI container; putting a
  DB connection in there hides its lifetime.
- `pino-http` can emit the ID for you via `genReqId`; use ALS for everything that is not
  the HTTP logger.

> Trace-ID propagation is the same problem solved at a different layer. If you run
> OpenTelemetry ([performance.md](./performance.md)), read the trace ID from the active
> span instead of minting a second identifier.

## Common Middleware

| Package | Purpose | Notes |
|---------|---------|-------|
| `helmet` | Security headers | Use default config, tune CSP for your app |
| `cors` | Cross-origin | **Not used**: no browser calls this API |
| `express-rate-limit` | Rate limiting | v8: use `limit` (not `max`); Redis store for multi-instance |
| `pino-http` | Request logging | Replaces morgan |
| `express.json()` | JSON body parsing | Built-in, wraps body-parser |
| `express.urlencoded()` | Form body parsing | Built-in, `extended: false` default |
| `cookie-parser` | Parse cookies | **Not used**: no cookies (sessions are Bearer ids) |
| `express-session` | Server-side sessions | **Not used**: sessions are Express's own records (security.md § Sessions) |
| `multer` | File uploads | Multipart form data |
| `compression` | Gzip response | Prefer at reverse proxy level |
| `csrf-csrf` / `csrf-sync` | CSRF tokens | **Not used**: no browser, no cookies — see [security.md](./security.md) |
| `morgan` | HTTP logging | Dev only — use Pino in production |

## File Uploads (multer v2)

Multer parses `multipart/form-data`. It does NOT validate file contents — only metadata.

```js
import multer from 'multer';
import crypto from 'node:crypto';
import path from 'node:path';
import { AppError } from './errors.js'; // the error classes from error-handling.md

const ALLOWED = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];

const storage = multer.diskStorage({
  destination: '/tmp/uploads',
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname).toLowerCase();
    cb(null, `${crypto.randomUUID()}${ext}`);  // never use original filename
  },
});

const upload = multer({
  storage,
  limits: { fileSize: 10 * 1024 * 1024, files: 5 },  // 10MB, max 5 files
  fileFilter: (req, file, cb) => {
    if (ALLOWED.includes(file.mimetype)) return cb(null, true);
    // an AppError, so the central handler sends 415 with its own code, not INTERNAL
    cb(new AppError(`Unsupported file type: ${file.mimetype}`, 415, { code: 'UNSUPPORTED_FILE_TYPE' }));
    // a real error → your error handler; cb(null, false) silently skips the file
  },
});

app.post('/upload', upload.single('asset'), (req, res) => {
  res.json({ filename: req.file.filename, size: req.file.size });
});

// Handle multer errors
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') return res.status(413).json({ error: { code: 'FILE_TOO_LARGE', message: 'File too large' } });
    return res.status(400).json({ error: { code: 'UPLOAD_REJECTED', message: 'The upload was rejected' } });
  }
  next(err);
});
```

**Rules**:
- **Never use original filenames** — path traversal and injection attacks. Generate UUIDs.
- **Always set `limits`** — UI checks are convenience; server checks are protection.
- **Whitelist MIME types** — `file.mimetype` comes from client (can be forged), but still blocks lazy misuse.
- **Don't add multer globally** — only on routes that handle uploads.
- **Use the current multer 2.x** (2.3.0 as of Sept 2026). 1.x is unmaintained and carries CVE-2025-47935, -47944, -48997, -7338.
- **Verify content, not just MIME** — for images, re-encode with `sharp`; for anything executable, store outside the web root and never serve it from the upload directory.

Source: [multer GitHub](https://github.com/expressjs/multer)

## Writing Custom Middleware

```js
// Simple
function addTimestamp(req, res, next) {
  req.startTime = Date.now();
  next();
}

// Async (v5 — errors auto-forward)
async function loadUser(req, res, next) {
  req.user = await db.findUser(req.headers.authorization);
  next();
}

// With error
function validateAuth(req, res, next) {
  if (!req.headers.authorization) {
    return next(new AppError('Missing authorization', 401));
  }
  next();
}

// Configurable factory
function requireRole(role) {
  return (req, res, next) => {
    if (req.user?.role !== role) {
      return next(new AppError('Forbidden', 403));
    }
    next();
  };
}

app.get('/admin', requireRole('admin'), adminHandler);
```

## Anti-patterns

1. **Calling `next()` after `res.send()`** — headers already sent, causes warnings
2. **Forgetting `return` before `next(err)` or `res.send()`** — execution continues, may send twice
3. **Using `morgan` in production** — synchronous I/O; use pino-http
4. **Body parser before rate limiter** — lets attackers send large payloads before being throttled. Parse after rate limiting.
5. **Not setting `limit` on `express.json()`** — default is 100kb but still good to reduce

## Debug Logging

Express uses the `debug` package. Enable internal logs:

```bash
# All Express + router logs
DEBUG=express:*,router,router:* node index.js

# Just router
DEBUG=router,router:* node index.js

# Your app's logs
DEBUG=myapp:* node index.js
```

```powershell
# Windows PowerShell
$env:DEBUG = "express:*,router,router:*"; node index.js
```

In v5, router logs use the `router` namespace (not `express:router`). Combine with your app namespace:

```bash
DEBUG=myapp:*,router node index.js
```

Source: [Debugging Express](https://expressjs.com/en/guide/debugging/)