# Routing in Express 5

## Path Syntax (path-to-regexp v8)

Express 5 uses [path-to-regexp v8](https://github.com/pillarjs/path-to-regexp), which removed regex sub-expressions to prevent ReDoS attacks.

### Named Parameters
```js
app.get('/users/:id', (req, res) => {
  // req.params.id = '123'
});
```

### Optional Segments (braces)
```js
// Braces mark optional parts
app.get('/:file{.:ext}', (req, res) => {
  // GET /image       → { file: 'image' }          (ext omitted, not undefined)
  // GET /image.png   → { file: 'image', ext: 'png' }
});

// Complex optional: /base{/:optional}/:required
app.get('/base{/:category}/:id', (req, res) => {
  // GET /base/123         → { id: '123' }
  // GET /base/books/123   → { category: 'books', id: '123' }
});
```

### Wildcards (named, array result)
```js
app.get('/*splat', (req, res) => {
  // GET /foo/bar → req.params.splat = ['foo', 'bar']
});

// Match root too: wrap in braces
app.get('/{*splat}', (req, res) => {
  // GET /       → splat key OMITTED (params = {}) — not []
  // GET /a/b    → req.params.splat = ['a', 'b']
});
```

### Repetition (v8 has NO `+` modifier — `*name` only, position matters)

`+` is a reserved character in v8; `:name+` and `:name*` throw `PathError`. Behavior of `*name` depends on placement:

```js
// Mid-path wildcard: requires AT LEAST ONE segment
app.get('/assets/*parts', (req, res) => {
  // GET /assets      → 404
  // GET /assets/js   → { parts: ['js'] }        (array)
  // GET /assets/a/b  → { parts: ['a', 'b'] }
});

// Root-level zero-or-more: braces make the leading segment optional too
app.get('/files{/*rest}', (req, res) => {
  // GET /files       → rest key omitted
  // GET /files/x.txt → { rest: ['x.txt'] }
});
```

> **Need one-or-more that also accepts the bare prefix?** Register two routes (`'/assets'` + `'/assets/*parts'`) or check `req.params.parts.length` in the handler.

Verified against express 5.2.1 / path-to-regexp 8.x.

### Multiple Routes for Regex-like Patterns
```js
// Alternation → register an array of paths (regex alternation is not supported)
app.get(['/user', '/users'], handler);

// Numeric constraint → validate in the handler, not the path
app.get('/:id', (req, res, next) => {
  if (!/^\d+$/.test(req.params.id)) return res.status(400).json({ error: { code: 'INVALID_ID', message: 'id must be numeric' } });
  next();
}, handler);
```

### Reserved Characters
`( ) [ ] ? + !` are reserved in v5 paths. Escape with `\` to match them literally:
```js
// Matches literal /wiki/page(1)
app.get('/wiki/page\\(1\\)', (req, res) => { /* ... */ });
```

## req.params Behavior

- Wildcard captures are **arrays**: `/*splat` on `/a/b` → `{ splat: ['a','b'] }`
- **Unmatched params are omitted** from `req.params` entirely (never empty strings / undefined)
- `req.params` has a **null prototype** for string paths — use `Object.hasOwn(params, key)`, never `params.hasOwnProperty(key)`
- Regex-defined paths keep a normal prototype (avoid regex paths anyway)

> **Implication**: No inherited properties means `params.toString` etc. are absent — safe against prototype-pollution lookups, but destructure carefully.

## Router

```js
import { Router } from 'express';

const router = Router();

router.get('/users', listUsers);
router.get('/users/:id', getUser);
router.post('/users', createUser);

// Mount at prefix
app.use('/api/v1', router);
```

### `mergeParams` — nested routers lose the parent's params

A `Router` gets **only its own** path parameters unless you opt in. Verified on 5.2.1:

```js
const child = Router();                        // default
child.get('/items/:itemId', (req, res) => res.json(req.params));
app.use('/orgs/:orgId', child);
// GET /orgs/o1/items/i1  →  { "itemId": "i1" }        ← orgId is GONE

const child = Router({ mergeParams: true });
// GET /orgs/o1/items/i1  →  { "orgId": "o1", "itemId": "i1" }
```

**Rule:** every router mounted under a parameterized prefix needs `{ mergeParams: true }`.
The failure is silent — `req.params.orgId` is `undefined`, so a tenant check reads as
"no tenant" rather than throwing. This is an authorization bug, not a routing bug.

```js
// Same applies to error/param middleware mounted on the child
const tenant = Router({ mergeParams: true });
tenant.use((req, res, next) => {
  if (!req.params.orgId) return next(new AppError('Missing tenant', 400));
  next();
});
```

> On name collision the **child wins** — a `:id` in both parent and child yields the
> child's value. Name parent params distinctly (`:orgId`, not `:id`).

### router.param() — name must be a string
```js
// Arrays are not accepted here → TypeError: argument name must be a string
router.param('id', handler);
router.param('page', handler);
// Note: app.param() DOES accept an array of names
```

### app.router

`app.router` is a reference to the base Express router. It exists for framework-internal use — application code should use `Router()` instances and `app.use()` instead.

## Route Methods

```js
app.get('/', handler);
app.post('/', handler);
app.put('/', handler);
app.patch('/', handler);
app.delete('/', handler);
app.all('/', handler);        // all HTTP methods
app.use('/path', middleware);  // mount middleware
```

## next('route')
Skip to the next route handler for the same path:
```js
app.get('/users/:id',
  (req, res, next) => {
    if (!req.user.isAdmin) return next('route'); // skip to next matching route
    next();
  },
  adminHandler
);
app.get('/users/:id', publicHandler); // fallback
```

> **Verified on express 5.2.1**: `Host: api.v1.example.com:3000` gives `req.host = 'api.v1.example.com:3000'`, `req.hostname = 'api.v1.example.com'`, `req.subdomains = ['v1', 'api']`. The reversal trips people up — `subdomains[0]` is the segment *closest to the domain*, not the leftmost label.

## Common Anti-patterns

1. **Don't use regex routes** — move validation into handlers
2. **Don't rely on `req.params` having a prototype** — it's null-prototype in v5
3. **Don't use ordered numeric params** — `req.params[0]` is gone; name everything
4. **Don't forget wildcard arrays** — `*splat` returns an array, not a string

## Key Request Properties

| Property | Returns | Notes |
|----------|---------|-------|
| `req.ip` | Client IP | Requires `trust proxy` behind reverse proxy |
| `req.host` | Host **with port** | `localhost:3000` — use `req.hostname` when you need host only |
| `req.ips` | IP chain | Only the **trusted** portion of `X-Forwarded-For`, left-to-right. Empty when `trust proxy` is unset |
| `req.hostname` | Hostname only | From `Host` header or `X-Forwarded-Host` with `trust proxy` |
| `req.protocol` | `http` or `https` | From `X-Forwarded-Proto` with `trust proxy` |
| `req.secure` | Boolean | `req.protocol === 'https'` |
| `req.subdomains` | Array | **Reverse** order, from `Host`: `api.v1.example.com` → `['v1', 'api']`. Cut point is `app.set('subdomain offset', 2)` |
| `req.fresh` | Boolean | Cache validation — `true` if response is fresh (ETag/Last-Modified match) |
| `req.stale` | Boolean | Opposite of `req.fresh` |
| `req.xhr` | Boolean | `true` if `X-Requested-With: XMLHttpRequest` (deprecated — use `Accept` header instead) |

## Key Response Methods

```js
// Set status + headers
res.status(201);
res.set('X-Custom-Header', 'value');
res.type('application/json');           // sets Content-Type

// No cookies in this project: a login returns the session id in the JSON body, and Next.js
// sets its own cookie (backend-guide/03-auth/authentication.md §0)
res.status(200).json({ token: sessionId });

// Misc contracts
res.vary('Accept');         // throws if the field argument is missing
res.status(200);            // integers 100–999 only; res.status(1000) and res.status('200') throw RangeError

// Redirects (status first in v5)
res.redirect(302, '/login');
res.redirect('/login');                // defaults to 302

// File serving
res.sendFile('/path/to/file', { root: './public', dotfiles: 'deny' });
res.download('/path/to/file.pdf');     // triggers download dialog
res.attachment('report.pdf');          // sets Content-Disposition

// Content negotiation
res.format({
  'text/html': () => res.send('<h1>Hello</h1>'),
  'application/json': () => res.json({ message: 'Hello' }),
  default: () => res.status(406).json({ error: { code: 'NOT_ACCEPTABLE', message: 'Not Acceptable' } }),
});

// Link headers (v5.1+ supports multiple same-rel links)
res.links({ next: '/api/users?page=2', prev: '/api/users?page=1' });

// Append to existing header (e.g. Set-Cookie)
res.append('Set-Cookie', 'foo=bar; Path=/');
res.append('Set-Cookie', 'baz=qux; Path=/');
```

> **Caveat**: `req.xhr` is unreliable — modern SPAs don't set `X-Requested-With`. Check `Accept` header instead: `req.accepts('html')`.
