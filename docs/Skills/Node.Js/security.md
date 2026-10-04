# Security

Production security rules. Sources: [nodejs.org security best practices](https://nodejs.org/learn/getting-started/security-best-practices), [goldbergyoni/nodebestpractices](https://github.com/goldbergyoni/nodebestpractices) (2026-07 edition), official [permissions docs](https://nodejs.org/api/permissions.html).

## Runtime & permission model

**R1 — Permission model: real, but opt-in. [Context — production tooling, sandboxing]**
```bash
node --permission \
  --allow-fs-read=/app,/tmp \
  --allow-fs-write=/app/data \
  --allow-child-process \
  app.js
```
- Flag renamed `--experimental-permission` → `--permission` in Node 24 (increasing stability).
- Deny-by-default for fs read/write, child processes, workers, native addons, WASI. Native addons are blocked under any permission set unless `--allow-addons` (`ERR_DLOPEN_DISABLED`).
- Use it for: CLI tools running untrusted config, CI scripts, dev-dependency hardening, blast-radius reduction in plugins.
- Do NOT treat it as a full container/sandbox substitute: V8 escapes and process-level attacks remain out of scope. Layer with containers/VMs for untrusted code.

**R2 — Stay on supported LTS; patch promptly. [Stable]**
Node 20/25 = EOL = no CVE fixes ([versions-and-releases.md](./versions-and-releases.md)). Subscribe to [nodejs.org/security](https://nodejs.org/en/security) releases; pin digests and rebuild base images weekly-ish.

## Dependencies (the dominant attack surface)

**R3 — Lockfiles committed; installs via `npm ci` (or `pnpm install --frozen-lockfile`). [Stable]**
Why: `npm ci` installs exactly the lockfile and fails on drift — reproducible builds, no surprise supply-chain updates. Plain `npm install` in CI/prod images can resolve new (possibly malicious) versions mid-build.
- Enable provenance checks where available (npm provenance/signatures).
- `npm audit` in CI as a gate for high/critical; `npm audit fix` for safe updates; `overrides` field to force-fix a transitive dependency that maintainers won't patch:
```json
{ "overrides": { "lodash.template": "^4.5.0" } }
```
- Block `preinstall`/`postinstall` execution unless needed. Set it **per project**, not
  globally — `npm config set ignore-scripts true` writes to `~/.npmrc` and will silently
  break every package that needs a native build, everywhere on the machine:
  ```
  # .npmrc (committed, project-scoped)
  ignore-scripts=true
  ```
  Opt back in for a specific package with `npm rebuild <pkg>`. pnpm blocks lifecycle
  scripts by default and prompts for an allowlist — the better default if you can switch.

**R4 — Minimal dependency budget. [Stable]**
Every transitive dep is attack surface + install weight. Prefer built-ins ([runtime-apis.md](./runtime-apis.md) R1: fetch/WebSocket/crypto/parseArgs/sqlite cover a lot). Before adding a package: check last-publish date, maintainers, install scripts, download trend.

## Application rules

**R5 — Validate every input at the boundary with a schema library. [Stable]**
`zod` / `valibot` / ajv (Fastify has built-in JSON-schema validation — [frameworks.md](./frameworks.md)). Never trust `req.body`/`req.query`/headers/env vars; parse env once at startup and fail fast:
```ts
const env = EnvSchema.parse(process.env);  // crash at boot, not at 3am
```

**R6 — Secrets: env vars via `--env-file`, never in code or logs. [Stable]**
```bash
node --env-file=.env.local app.js   # .env* is gitignored
```
Keep `--env-file` for local dev; inject real secrets via orchestrator secret stores in prod. Redact before logging (`pino` redact option).

**R7 — Web hygiene (non-negotiable baseline). [Stable]**
- `helmet`-style security headers (or framework defaults).
- Parameterized queries always (SQLite/Postgres drivers support placeholders — never string-concat SQL).
- `crypto.timingSafeEqual` for token/secret comparisons.
- Rate-limit auth endpoints; constant-time error messages (no user enumeration).
- TLS verification ON by default — never `rejectUnauthorized: false` ([error-handling.md](./error-handling.md) for debugging cert issues instead).

**R8 — Fail fast on unhandled rejections; exit codes matter. [Stable]**
Crash-on-rejection (default since Node 15) is a security/stability feature — an app in unknown state should die and let the supervisor restart it. Details: [error-handling.md](./error-handling.md).

**R9 — Lint for security, not just style. [Context]**
`eslint-plugin-security`, `npm audit signatures`, Socket/Dependabot/Renovate in the loop. Add secrets scanning (`gitleaks`) pre-commit.
