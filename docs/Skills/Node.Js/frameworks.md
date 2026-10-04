# HTTP Frameworks (decision guide)

Which server framework to pick. [Context] throughout — all options are actively maintained; choice depends on team and constraints. Sources: framework docs, 2026 comparisons (pkgpulse, ortemtech).

## Decision table

| Framework | Pick when | Watch out for |
|---|---|---|
| **Fastify 5** | throughput matters, want built-in JSON-schema validation, plugin architecture | schema-first style; plugin typing has a learning curve |
| **Express 5** | ecosystem/middleware needs, team familiarity, boring stability | slowest of the three; Express 4→5 broke some middleware & path patterns |
| **Hono** | runtime portability (Node/Bun/Deno/edge), tiny footprint, TS-first | younger ecosystem; ultra-thin by design |
| **NestJS** | enterprise structure: DI, modules, guards, opinionated layers | heavy abstraction; boot time; overkill for small services |
| raw `node:http` | health checks, proxies, protocol experiments | you own routing/parsing/security headers |

Rough 2026 consensus: Fastify ≈ 2–3× Express throughput (validated JSON responses); Hono competitive and portable; Express remains the largest middleware ecosystem.

## Rules

**R1 — Validate at the boundary with schemas, not hand-rolled checks. [Stable]**
Fastify: declare JSON-schema on routes (validation + serialization + docs from one source). Elsewhere: `zod`/`valibot` in a middleware or per-handler parse. Rationale and fail-fast env parsing: [security.md](./security.md) R5.

**R2 — Express 5 migration: use the dedicated guide, not a checklist. [Context]**
Express 4 is in maintenance with a TC target EOL no sooner than 2026-10-01. The migration
has several changes that **fail silently** rather than throwing (reversed
`res.redirect(url, status)` argument order, `req.body` becoming `undefined`, `req.host`
gaining the port), so a three-bullet summary is actively dangerous here.

Full removed-API table, silent-behaviour table, path-syntax rewrites and codemods:
`../Express JS/migration-v4-to-v5.md`.

**R3 — Register error-handling middleware once, at the end. [Stable]**
```js
// Express — ONE central handler, registered after every route and middleware.
// Its code lives in ONE place, never copied: ../Express JS/error-handling.md
app.use(errorHandler);
// Fastify
app.setErrorHandler((err, req, reply) => { ... });
```
Let unknown errors crash-and-be-restarted rather than converting 500s into limp-along behavior ([error-handling.md](./error-handling.md) R1–R2).

**R4 — Timeouts on every outbound call. [Stable]**
```js
const res = await fetch(url, { signal: AbortSignal.timeout(2_000) });
```
A missing outbound timeout lets one slow dependency cascade to whole-pod exhaustion. Framework request timeouts (server.requestTimeout/headersTimeout) have sane Node defaults since 18 — keep them, don't zero them out.

**R5 — Keep framework code thin; push logic into testable plain modules. [Stable]**
Handlers parse input → call domain functions (framework-free, directly unit-testable — [testing.md](./testing.md) R5) → serialize output. This keeps framework migration cheap and tests fast.
