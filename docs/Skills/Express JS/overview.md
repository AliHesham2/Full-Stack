# Express.js Overview & Current State

## Version Landscape (September 2026)

| Version | Status | Notes |
|---------|--------|-------|
| 5.2.x | **ACTIVE** (`latest` on npm) | v5.2.1 — current production release (shipped Dec 2025, still `latest`) |
| 4.x | MAINTENANCE | Security patches only; TC target EOL **no sooner than 2026-10-01** — migrate now |
| 3.x | EOL | No updates of any kind |

**Rule**: All new work targets Express 5.2+. This guide documents Express 5 only — upgrading an existing 4.x codebase is covered in [migration-v4-to-v5.md](./migration-v4-to-v5.md).

## Release Lifecycle (Express TC Policy)

Express majors move through phases per the [TC LTS policy](https://expressjs.com/en/blog/2025-03-31-v5-1-latest-release/):

- **CURRENT** → new major, not yet `latest` on npm
- **ACTIVE** → tagged `latest`; fully supported; the line this guide covers
- **MAINTENANCE** → previous major after a new ACTIVE exists; security fixes only
- **EOL** → no updates of any kind

> **Express 6**: Not released as of Sept 2026. When it ships, 5.x enters MAINTENANCE — re-check this folder then.

## Express 5 Fundamentals

Present-tense facts about how Express 5 works:

- `engines.node` is `>= 18`, but see [Node.js Compatibility](#nodejs-compatibility) — 18 and 20 are both EOL
- Routes use [path-to-regexp v8](https://github.com/pillarjs/path-to-regexp): named params, brace optionals `{...}`, wildcard arrays (`*name`) — no regex sub-expressions (ReDoS-proof)
- Async handlers/middleware forward rejected promises to the error handler automatically
- Body parsers (`express.json`, `express.urlencoded`, `.text`, `.raw`) decompress Brotli, gzip, deflate
- `res.send()` accepts `Uint8Array` for binary responses; `res.sendFile()` accepts an `ETag` option
- Default query parser is `simple` (flat keys); opt into nesting with `app.set('query parser', 'extended')`

Source: [Official v5 release blog](https://expressjs.com/en/blog/2024-10-15-v5-release), [v5.2.0 release](https://github.com/expressjs/express/releases/tag/v5.2.0)

## When to Use Express

**Good fit:**
- REST/JSON APIs where you want minimal framework overhead
- Teams that value simplicity and a large ecosystem over structure
- Projects needing maximum middleware compatibility
- Rapid prototyping or small-to-medium services
- When you need fine-grained control over the request pipeline

**Consider alternatives when:**
- You want built-in TypeScript-first support and JSON schema validation → **Fastify**
- You need opinionated architecture (DI, modules, decorators) → **NestJS**
- You're building a full-stack app with SSR → **Next.js**
- You need maximum raw throughput → **Fastify** (2-3x Express on benchmarks)

> Express remains the most widely-deployed Node.js framework with the largest middleware ecosystem. Performance differences rarely matter for I/O-bound apps behind a reverse proxy.

## Key Dependencies (v5)

Actual `dependencies` of express@5.2.1 (verify with `npm ls`):

| Package | Range | Role |
|---------|-------|------|
| `router` | `^2.2.0` | Routing engine, extracted from core; pulls in `path-to-regexp` v8 |
| `body-parser` | `^2.2.1` | Backs `express.json()` / `.urlencoded()` / `.text()` / `.raw()` |
| `serve-static` | `^2.2.0` | Backs `express.static()`; `dotfiles` default `"ignore"` |
| `qs` | `^6.14.0` | Only used when `query parser` is `extended` |
| `mime-types` | `^3.0.0` | Content-Type lookup for `res.type()` / `res.sendFile()` |
| `accepts` | `^2.0.0` | `req.accepts()`, `res.format()` |
| `cookie` | `^0.7.1` | `res.cookie()` serialization (reading cookies still needs `cookie-parser`) |
| `finalhandler` | `^2.1.0` | Default error handler / 404 |
| `http-errors` | `^2.0.0` | Typed HTTP errors thrown internally |
| `debug` | `^4.4.0` | `DEBUG=` namespaces |

> `path-to-regexp` is **not** a direct express dependency — it comes in through `router`. Don't add it to `package.json` yourself.

## TypeScript Support

Express does **not** ship first-party types. Use `@types/express` from DefinitelyTyped:

```bash
npm install -D @types/express   # 5.0.6 as of Sept 2026 — the 5.x line matches express 5
```

**Rule**: `@types/express@4.x` does not describe Express 5 (null-prototype `req.params`, v8 path syntax, async error forwarding). Pin the major to 5.

The Express TC has a [TypeScript Working Group](https://github.com/expressjs/typescript-wg) exploring shipping types in-repo for Express 6. As of Sept 2026, no timeline confirmed.

> **Gotcha**: The unofficial `@types/express` types occasionally lag behind API changes. If you hit type errors on valid v5 code, check the DT issue tracker.

## Node.js Compatibility

Express declares `engines.node: ">= 18"`, but that floor is **not** a deployment recommendation — Node 18 and 20 are past end-of-life and receive no security patches.

| Node | Status (Sept 2026) | Use for new work? |
|------|--------------------|-------------------|
| 24.x | **Active LTS** (maintenance starts 2026-10-20, EOL 2028-04-30) | **Yes — default target** |
| 22.x | Maintenance LTS (EOL 2027-04-30) | Only for existing deployments |
| 20.x | **EOL** 2026-04-30 | No |
| 18.x | **EOL** 2025-04-30 | No |
| 26.x | Current line; becomes LTS 2026-10-28 | Not yet in production |

```json
{ "engines": { "node": ">=22" } }
```

Pin CI and the runtime image to the same major (`node:24-alpine`, `.nvmrc`). Check your running version against the security DB with `npx is-my-node-vulnerable`.

Source: [Node.js Release schedule](https://github.com/nodejs/Release)

## Installation

```bash
npm install express    # installs latest (5.x)
```

## Minimal V5 App

```js
import express from 'express';

const app = express();
app.disable('x-powered-by');
app.use(express.json());

app.get('/', (req, res) => {
  res.json({ ok: true });
});

app.listen(3000, () => console.log('Listening'));
```

## Express vs Fastify (2026)

| Dimension | Express | Fastify |
|-----------|---------|--------|
| Throughput | Baseline | 2-3x higher |
| Plugin system | Middleware (linear) | Hooks + plugins (encapsulated) |
| Validation | External (zod/joi) | Built-in JSON Schema |
| TypeScript | `@types/express` (community) | Native first-class |
| Ecosystem | Largest | Growing, smaller |
| Learning curve | Lowest | Low-medium |
| Stability | Extremely mature | Mature, faster-moving |

**Rule**: Don't switch from Express to Fastify unless you have a measured performance problem or specifically want schema-first validation.
