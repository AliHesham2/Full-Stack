# Node.js Knowledge Base

Quick reference map. Each file is self-contained and cross-referenced.
Scope: server-side/runtime concerns for engineers building and operating
Node.js services. Target: **Node 24 (Active LTS until 2026-10-20)**; Node 26 notes
included where behavior differs. Verified September 2026 against nodejs/Release
`schedule.json`, the official API docs and the v26 release notes. Shutdown, threadpool,
AsyncLocalStorage and `node --run` behaviours were **measured**, not read.

| File | Scope |
|------|-------|
| [versions-and-releases.md](./versions-and-releases.md) | Release table, LTS policy, annual-cadence change, upgrade rules |
| [module-system.md](./module-system.md) | ESM/CJS, require(esm), exports maps, interop diagnostics |
| [runtime-apis.md](./runtime-apis.md) | Web-platform globals, node:sqlite, streams, fs/promises, ALS |
| [typescript.md](./typescript.md) | Native type stripping, erasable-syntax limits, tsconfig |
| [concurrency.md](./concurrency.md) | Event loop, worker_threads, child_process, cluster |
| [error-handling.md](./error-handling.md) | Crash semantics, exit codes, error causes, debugging flags |
| [security.md](./security.md) | Permission model, supply chain, input validation, secrets |
| [testing.md](./testing.md) | node:test vs Vitest, mocking, deterministic tests, CI order |
| [frameworks.md](./frameworks.md) | Fastify / Express 5 / Hono / NestJS decision table |
| [tooling.md](./tooling.md) | npm/pnpm/Yarn, ESLint 10 flat config, built-in dev replacements |
| [deployment.md](./deployment.md) | Docker, graceful shutdown, health endpoints, SEA |
| [deprecations.md](./deprecations.md) | Removed/runtime-deprecated APIs, dead packages, audit checklist |

## Core principles

1. **Target Active LTS (24); test the lowest version you claim** → [versions-and-releases.md](./versions-and-releases.md)
2. **ESM by default; extensions mandatory; `node:` prefix always** → [module-system.md](./module-system.md)
3. **Never block the event loop with CPU work** → [concurrency.md](./concurrency.md)
4. **Fail fast on unhandled rejections; log-and-exit on uncaught exceptions** → [error-handling.md](./error-handling.md)
5. **Await `server.close()`; reap idle sockets on an interval** → [deployment.md](./deployment.md)
6. **Validate every boundary with a schema; secrets only via env/orchestrator** → [security.md](./security.md)
7. **Prefer built-in web APIs over userland packages** → [runtime-apis.md](./runtime-apis.md)
8. **Run TS natively via type stripping; type-check separately with tsc** → [typescript.md](./typescript.md)

## Out of scope

Frontend bundler internals,
database engine administration beyond driver choice, Kubernetes cluster
operations beyond app-level requirements, native addon authoring (N-API),
PM2-style process managers (superseded by orchestrators). Express specifics —
routing, middleware, migration from 4 — live in ../Express JS/; language-level
JavaScript semantics live in ../javascript/.

## Authoritative sources

- API docs: https://nodejs.org/api/
- Release schedule: https://nodejs.org/en/about/previous-releases
- Deprecations: https://nodejs.org/api/deprecations.html
- Security releases: https://nodejs.org/en/security
