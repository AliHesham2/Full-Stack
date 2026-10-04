# Security

## Mandatory Baseline

Every Express app in production should include:

```js
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';

app.set('trust proxy', 1);          // trust first proxy (Nginx/ALB)
app.disable('x-powered-by');
app.use(helmet());
// no cors(): only the Next.js server calls this API (see § CORS)
app.use(rateLimit({ windowMs: 15 * 60 * 1000, limit: 100 }));
app.use(express.json({ limit: '10kb' }));  // cap body size
```

### trust proxy

**Rule**: If you run behind a reverse proxy (Nginx, ALB, Cloudflare), you MUST set `trust proxy`. Without it, `req.ip` returns the proxy's IP, not the client's — breaking rate limiting, logging, and geo-detection.

```js
// Single proxy (most common)
app.set('trust proxy', 1);

// Two proxies (CloudFront → ALB → Express)
app.set('trust proxy', 2);

// Specific proxy IP/subnet
app.set('trust proxy', '10.0.0.0/8');

// Loopback only (Nginx on same host)
app.set('trust proxy', 'loopback');
```

### How the number is counted

`trust proxy: n` means "the **n hops nearest to me** are mine". Express builds the chain `[socket address, ...X-Forwarded-For reversed]` and returns the first entry it does *not* trust — it is **not** "take the leftmost XFF value".

```
socket 127.0.0.1   X-Forwarded-For: 203.0.113.9, 10.0.0.1
chain:  [127.0.0.1, 10.0.0.1, 203.0.113.9]
             ^trusted (n=1)   ^req.ip
```

Verified on express 5.2.1: with `trust proxy: 1` that request yields `req.ip === '10.0.0.1'` and `req.ips === ['10.0.0.1']`. Getting the real client (`203.0.113.9`) needs `trust proxy: 2`, because there are genuinely two proxies in front.

**How to get it right**: count the proxies you actually run (CDN, LB, ingress, sidecar), then log `req.ip` from a known external address and confirm it matches. Guessing is how rate limiters end up keyed on a single proxy IP.

> **Caveat**: `trust proxy: true` trusts ALL proxies — dangerous on the public internet. Always use a specific number or subnet. Too high a number lets attackers prepend a fake `X-Forwarded-For` and bypass per-IP rate limits; too low buckets every client together.

Source: [Express Behind Proxies](https://expressjs.com/en/guide/behind-proxies/)

### Helmet

`helmet()` with no options sets exactly these headers (dumped from helmet 8.3.0):

| Header | Default value |
|---|---|
| `Content-Security-Policy` | `default-src 'self'; base-uri 'self'; font-src 'self' https: data:; form-action 'self'; frame-ancestors 'self'; img-src 'self' data:; object-src 'none'; script-src 'self'; script-src-attr 'none'; style-src 'self' https: 'unsafe-inline'; upgrade-insecure-requests` |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` |
| `Cross-Origin-Opener-Policy` | `same-origin` |
| `Cross-Origin-Resource-Policy` | `same-origin` |
| `Referrer-Policy` | `no-referrer` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `SAMEORIGIN` |
| `X-XSS-Protection` | `0` |
| `Origin-Agent-Cluster` | `?1` |
| `X-DNS-Prefetch-Control` | `off` |
| `X-Download-Options` | `noopen` |
| `X-Permitted-Cross-Domain-Policies` | `none` |

Helmet also removes `X-Powered-By`, but keep `app.disable('x-powered-by')` anyway — it
covers responses that never reach helmet.

**Three defaults that break real apps** — know them before you debug a blank page:

1. **`script-src 'self'`** blocks every CDN script, inline script, and analytics tag.
   A JSON API is unaffected; anything serving HTML needs a real policy.
2. **`Cross-Origin-Resource-Policy: same-origin`** stops other origins from loading your
   images/fonts/files. Serving assets to a separate frontend domain needs
   `crossOriginResourcePolicy: { policy: 'cross-origin' }`.
3. **`X-XSS-Protection: 0` is intentional.** It *disables* the legacy browser XSS auditor,
   which was itself exploitable. Do not "fix" it to `1; mode=block`.

```js
// API-only service: CSP is dead weight, COEP/CORP still matter
app.use(helmet({ contentSecurityPolicy: false }));

// HTML app with a CDN
app.use(helmet({
  contentSecurityPolicy: {
    directives: { 'script-src': ["'self'", 'https://cdn.example.com'] },   // merged with defaults
  },
}));
```

`Cross-Origin-Embedder-Policy` is **not** enabled by default (it breaks most third-party
embeds). Turn it on only if you need `SharedArrayBuffer`.

Source: [Official security best practices](https://expressjs.com/en/advanced/best-practice-security), [helmet docs](https://helmetjs.github.io/)

### CORS — none in this project

Only the Next.js server calls this API; no browser ever does. So **no `cors` middleware at all**:
without CORS headers, browsers cannot expose cross-origin responses to page JavaScript.
Simple requests can still reach the server; CORS is not authentication or a firewall.
Private networking or a verified per-service credential enforces Next.js-only access. Never add
`cors()` "to make it work" — that means a browser is calling Express, which the architecture
forbids (backend-guide/03-auth/authentication.md §0, backend-guide/01-foundation/initialization.md
§ 2.9). Keep Express on a private network where the platform allows it.

### Rate Limiting
```js
import rateLimit from 'express-rate-limit';   // v8.x

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,      // 15 minutes
  limit: 100,                    // per IP  (v7 renamed `max` → `limit`)
  standardHeaders: 'draft-8',    // combined `RateLimit` + `RateLimit-Policy` headers
  legacyHeaders: false,          // drop X-RateLimit-*
  identifier: 'global',          // policy name in draft-8 headers
});
app.use(limiter);

// Stricter for auth endpoints
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, limit: 5, identifier: 'auth' });
app.post('/login', authLimiter, loginHandler);
```

**v8 facts that change behaviour**:

- `max` still works but is deprecated — write `limit`.
- `standardHeaders: true` means **draft-6**, not the newest draft. Pass `'draft-8'` explicitly.
- IPv6 clients are bucketed by a **/56 subnet** by default, so one ISP allocation can't cycle addresses. Tune with `ipv6Subnet`.
- `req.rateLimit.current` was renamed to `req.rateLimit.used`.
- `onLimitReached` and `draft_polli_ratelimit_headers` were **removed** in v8 — use `handler` and `standardHeaders: 'draft-6'`.
- `limit: 0` blocks every request (it no longer disables the limiter). Use `skip: () => true` to disable.

For multi-instance deployments the default in-memory store is wrong — each pod keeps its own counter. Use a `rate-limit-redis` store, and fix `trust proxy` first or every request buckets under the proxy IP.

Source: [express-rate-limit changelog](https://express-rate-limit.mintlify.app/reference/changelog)

## Input Validation

Never trust user input. Validate at the boundary with zod:

```js
import { z } from 'zod';
import { ValidationError } from './errors.js'; // the error classes from error-handling.md

// Zod 4.x — string formats are top-level functions now
const createUserSchema = z.object({
  name: z.string().trim().min(1).max(100),
  email: z.email().max(254),
  age: z.int().min(0).max(150).optional(),
});

// source: 'body' | 'params' | 'query' — req.query is read-only in v5, so results go to req.validated
// A failure goes to the ONE central error handler as a ValidationError (error-handling.md), which
// sends the one envelope { error: { code: 'VALIDATION_ERROR', message, details } }.
function validate(schema, source = 'body') {
  return (req, res, next) => {
    const result = schema.safeParse(req[source]);
    if (!result.success) {
      const details = {};                                  // { "field.path": ["message", …] }
      for (const issue of result.error.issues) {          // zod 4: .issues, not .errors
        const key = issue.path.join('.') || source;
        (details[key] ??= []).push(issue.message);
      }
      return next(new ValidationError(details));
    }
    req.validated = { ...req.validated, [source]: result.data };
    next();
  };
}

app.post('/users', validate(createUserSchema), createUserHandler);
```

**Zod 4 deprecations** — `z.string().email()`, `.url()`, `.uuid()` still run but are deprecated in favour of the tree-shakeable top-level `z.email()`, `z.url()`, `z.uuid()`. Migrate with `npx @zod/codemod --transform v3-to-v4 ./src`. Ordering caveat: `.trim()` is a string method, so trimming stays `z.string().trim()` — there is no `z.email().trim()`.

Source: [Zod v4 changelog](https://zod.dev/v4/changelog)

**Validate every input surface, not just the body** — `req.params`, `req.query` and headers are equally attacker-controlled:

```js
const idParams = z.object({ id: z.uuid() });
app.get('/users/:id', validate(idParams, 'params'), getUser);
```

> `req.query` is a read-only getter in Express 5 — assigning to it throws. That is why the helper writes to `req.validated`, never back onto the request property it read.

## Open Redirects

Validate URLs before redirecting:

```js
app.get('/redirect', (req, res) => {
  const url = req.query.url;
  try {
    const parsed = new URL(url);
    if (parsed.host !== 'example.com') {
      return res.status(400).json({ error: { code: 'UNSUPPORTED_REDIRECT', message: 'Unsupported redirect host' } });
    }
  } catch {
    return res.status(400).json({ error: { code: 'INVALID_URL', message: 'Invalid URL' } });
  }
  res.redirect(url);
});
```

## Sessions and Auth (this project)

The model is fixed in backend-guide/03-auth/authentication.md §0 and §0.1. What it means here:

- **No cookies.** Express never sets or reads one (ESLint blocks `res.cookie` / `clearCookie`).
  No `express-session`, no `cookie-parser`, no `cookie-session`.
- **The session is Express's own record:** a random id of at least 128 bits, stored HASHED in
  the database, mapped to the user and the (encrypted) Supabase tokens, with absolute and idle
  timeouts. It arrives on every request as `Authorization: Bearer <session id>`.
- **No CSRF tokens.** CSRF needs a browser that sends credentials automatically; no browser calls
  this API, and it uses no cookies. CSRF protection lives on the Next.js side (the Server Actions
  origin check and its SameSite cookie).
- **No JWTs of our own.** Express does not sign or verify tokens; Supabase's access token is
  checked by Supabase itself (`auth.getUser(accessToken)`). If a JWT is ever verified here, pin
  `algorithms` (RFC 8725).

```js
import { createHash } from 'node:crypto';
import { AppError } from './errors.js';            // AppError: error-handling.md
import { sessions } from '../modules/auth/auth.repo.js';

// Every protected route: resolve the Bearer session id → req.session, or 401
export async function requireAuth(req, res, next) {
  const id = req.headers.authorization?.replace(/^Bearer\s+/i, '');
  if (!id) return next(new AppError('Not authenticated', 401, { code: 'UNAUTHENTICATED' }));
  const session = await sessions.findByHash(createHash('sha256').update(id).digest('hex'));
  if (!session || session.expiresAt <= new Date()) {
    return next(new AppError('Session expired', 401, { code: 'UNAUTHENTICATED' }));
  }
  req.session = session;
  next();
}
```

A 401 means ONLY "not logged in": Next.js logs the user out on it. Any other failure uses its own
status (backend-guide/03-auth/authentication.md §0).

## Dependency Security

```bash
npm audit                  # check for known vulns
npx is-my-node-vulnerable  # check Node.js version against security DB
```

Run `npm audit` in CI. Fail the build on high/critical vulnerabilities.

### Recent Security Advisories (2025)

| Package | CVE | Severity | Fixed in | Description |
|---------|-----|----------|----------|-------------|
| body-parser | CVE-2025-13466 | Moderate | 2.2.1+ | DoS via URL-encoded bodies with thousands of params |
| multer | CVE-2025-47935 | High | 2.0.0 | Memory leak from improper stream handling |
| multer | CVE-2025-47944 | High | 2.0.0 | DoS via malformed multipart request |
| multer | CVE-2025-48997 | High | 2.0.1 | DoS via empty field name in upload |
| multer | CVE-2025-7338 | High | 2.0.2 | DoS via unhandled exception |
| on-headers | CVE-2025-7339 | Low | 1.1.0 | Response header manipulation via `writeHead()` |
| multiparty | CVE-2026-8159 / 8161 / 8162 | High | 4.3.0 | ReDoS on filename, prototype-pollution crash, malformed `filename*` crash |

> **Action**: multer 1.x is unmaintained and carries all four 2025 CVEs — move to the current 2.x line (**2.3.0** as of Sept 2026), not merely to 2.0.2. The May 2026 `multiparty` advisories affect apps parsing multipart directly; Express core is unaffected.

Express 5.2.1 itself has no open advisories as of Sept 2026. Every fix above arrives through a transitive dependency, so `npm audit` on a fresh lockfile — not the express version number — is the real check.

Source: [Express security releases](https://expressjs.com/en/blog/)

## OWASP Top 10 — Express-specific Checklist

1. **Broken Access Control** — auth middleware per route; never trust an ID from `req.params` without an ownership check
2. **Cryptographic Failures** — TLS everywhere; hash passwords with argon2/bcrypt, never store plaintext
3. **Injection** — parameterized queries (Knex, Prisma, Drizzle, Sequelize all do this); never string-concat SQL
4. **Insecure Design** — validate at the boundary with zod; deny by default
5. **Security Misconfiguration** — helmet, `app.disable('x-powered-by')`, no `cors()` and no cookies at all, correct `trust proxy`
6. **Vulnerable Components** — `npm audit` in CI, committed lockfile, Dependabot/Renovate
7. **Auth Failures** — rate limit login; session ids random (≥128 bits) and stored hashed; a new id on every login (§ Sessions and Auth)
8. **Data Integrity Failures** — verify webhook signatures with `crypto.timingSafeEqual`; install with `npm ci` from a lockfile
9. **Logging & Monitoring Failures** — log auth failures and 5xx with pino; redact tokens/passwords (see [middleware.md](./middleware.md))
10. **SSRF** — if the app fetches user-supplied URLs, allowlist the host, block private/link-local ranges, and disable redirect following

Source: [OWASP Node.js Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html), [Express Security Best Practices](https://expressjs.com/en/advanced/best-practice-security)

## Anti-patterns

- Any `app.use(cors(…))` at all — no browser calls this API, so a CORS rule only means one is about to
- Using `eval()` or `new Function()` with user input
- Storing secrets in code — use environment variables
- Serving `.env` files or `.git` directory — configure static middleware carefully
