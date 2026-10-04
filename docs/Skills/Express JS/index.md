# Express.js Knowledge Base

Quick reference map. Each file is self-contained and cross-referenced.
Scope: building and operating HTTP services on Express. Target:
**Express 5.2.x on Node 24 (Active LTS)**. Express 4 appears only in the
migration file. Verified September 2026 — routing, `req.*` semantics, body-parser,
static-file and async-error behaviour were checked by running express 5.2.1, not
read from docs alone.

| File | Scope |
|------|-------|
| [overview.md](./overview.md) | Version/support landscape, real dependency tree, Node compatibility, when *not* to use Express |
| [migration-v4-to-v5.md](./migration-v4-to-v5.md) | Removed APIs, silent behaviour changes, path-syntax rewrites, codemods |
| [routing.md](./routing.md) | path-to-regexp v8 syntax, `req.params` semantics, Router, request/response API |
| [middleware.md](./middleware.md) | Pipeline order, body/query parsing (no cookies), static files, pino logging, multer uploads |
| [error-handling.md](./error-handling.md) | Async forwarding, error classes, centralized handler, crash semantics |
| [security.md](./security.md) | trust proxy, helmet, CORS, rate limiting, zod validation, CSRF, JWT, CVE table |
| [performance.md](./performance.md) | NODE_ENV, workers, keep-alive timeouts, ETag/caching, reverse proxy, observability |
| [streaming.md](./streaming.md) | Large responses, `pipeline`, SSE, downloads, backpressure |
| [health-shutdown.md](./health-shutdown.md) | Liveness/readiness probes, SIGTERM handling, Kubernetes wiring |
| [testing.md](./testing.md) | Vitest 4 + supertest, app-factory pattern, middleware isolation |
| [project-structure.md](./project-structure.md) | Folder layout, app/index split, env validation, npm scripts |

## Core principles

1. **Express 5 only; 4.x is EOL-bound (target 2026-10-01)** → [migration-v4-to-v5.md](./migration-v4-to-v5.md)
2. **Export an app factory; never listen inside the app module** → [project-structure.md](./project-structure.md)
3. **Let async errors propagate — v5 forwards rejections for you** → [error-handling.md](./error-handling.md)
4. **Set `trust proxy` correctly before any IP-based logic** → [security.md](./security.md)
5. **Validate every boundary with zod 4 top-level formats** → [security.md](./security.md)
6. **Name every wildcard; no regex in route strings** → [routing.md](./routing.md)
7. **Rate-limit before body-parsing; error handler last** → [middleware.md](./middleware.md)
8. **Offload TLS, compression and static files to the proxy/CDN** → [performance.md](./performance.md)
9. **`Router({ mergeParams: true })` under any parameterized mount** → [routing.md](./routing.md)
10. **Stream with `pipeline`; clean up on `res.on('close')`** → [streaming.md](./streaming.md)

## Out of scope

Node runtime internals, module system, and process management → `../Node.Js/`.
Database/ORM selection, GraphQL, WebSockets (SSE is covered in
[streaming.md](./streaming.md)), and framework alternatives
(Fastify, NestJS, Hono) beyond the decision table in
[overview.md](./overview.md) → `../Node.Js/frameworks.md`.

## Authoritative sources

- API reference: https://expressjs.com/en/5x/api.html
- Migration guide: https://expressjs.com/en/guide/migrating-5.html
- Security releases: https://expressjs.com/en/blog/
- Repository: https://github.com/expressjs/express
