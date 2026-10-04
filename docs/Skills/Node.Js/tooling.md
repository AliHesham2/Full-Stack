# Tooling & Ecosystem (2026 state)

Package managers, TS execution, lint/format, dev-time flags. Verified against official docs + current ecosystem surveys (see sources inline).

## Package managers

**R1 — Default npm; pnpm for monorepos/disk-sensitive setups; bun only as runtime choice. [Context]**
| Manager | State 2026 | Choose when |
|---|---|---|
| npm 11 (bundled with Node 24) | default, zero setup | single-package apps, lowest friction |
| pnpm 10 | fastest disk-lean installer (symlink store), first-class workspaces | monorepos, CI caching, big teams |
| Yarn 4 (Berry) | mature PnP/node_modules modes | existing Yarn codebases |
| bun | a runtime *and* PM | only if adopting Bun as runtime (not as add-on to Node) |

**R2 — Corepack is no longer distributed with Node (from Node 25). [Stable — the fact]**
TSC voted 2025-03 to stop shipping Corepack; Node 25+ does not include it ([socket.dev summary](https://socket.dev/blog/node-js-tsc-votes-to-stop-distributing-corepack)). Corepack continues as a standalone package (`npm i -g corepack`) while Node 24 is supported.
Do: keep the `packageManager` field for tooling that reads it (pnpm, standalone corepack, CI setups); don't assume `corepack enable` exists on future runtimes. Install the manager explicitly in Docker/CI.

**R3 — Commands map (npm ↔ pnpm). [Stable]**
| npm | pnpm |
|---|---|
| `npm ci` | `pnpm install --frozen-lockfile` |
| `npm run x` / `node --run x` | `pnpm x` |
| `npx pkg` | `pnpm dlx pkg` |
| `overrides` field | `pnpm.overrides` |

`node --run <script>` runs package.json scripts without npm's shell overhead — good for Docker ENTRYPOINT ([deployment.md](./deployment.md)).

> **`node --run` is not a drop-in `npm run`.** Verified on Node 22: with both `prehello`
> and `hello` defined, `node --run hello` executes **only** `hello` — pre/post scripts are
> deliberately not supported, and neither are npm lifecycle hooks or `npm_*` env vars
> beyond a minimal set. A `prebuild` step that silently stops running inside a container
> is a nasty way to find this out. Use `node --run` for leaf scripts; keep `npm run` where
> a pre/post chain exists.

## TypeScript execution (see [typescript.md](./typescript.md) for full rules)

- **Native**: `node file.ts` — erasable syntax only, no config. Default choice on ≥ 22.18.
- **`tsx`** (`node --import=tsx file.ts`): enums/decorators/paths-aliases support, on-the-fly transform. The community standard for full-feature TS.
- **ts-node**: legacy/maintenance — migrate off it in new work. **[Avoid]**

## Lint & format

**R4 — ESLint 10 flat config is the supported target (10.11.0 checked 2026-09-26). ESLint 9 reached EOL 2026-08-06. Verify plugin compatibility before upgrading. [Stable]**
```js
// eslint.config.js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  { rules: { 'no-console': 'error' } },
);
```
`.eslintrc` is legacy — new projects must use flat config. `typescript-eslint` for TS. `eslint-plugin-security` for [security.md](./security.md) R9.

**R5 — Formatter: Prettier (default) or Biome (speed-seeking single-tool). [Context]**
Biome replaces ESLint+Prettier in one Rust binary (10–25× faster) but has a smaller rule/plugin ecosystem — viable for greenfield apps; keep ESLint+Prettier where plugin needs dominate (e.g. eslint-plugin-import, custom rules). Pick one per repo; formatting debates are bikeshedding.

## Built-in dev flags replacing old dependencies

`--env-file` caveats (verified on Node 22 — it is a small parser, not dotenv):
- **Existing environment variables win.** `FOO=from_shell node --env-file=.env` keeps
  `from_shell`. There is no `override` option.
- `node --watch` does **not** watch the env file (nodejs/node#54001) — edit `.env`, restart
  manually.
- Quoting works (`A="a b"`), `\n` inside double quotes becomes a real newline, and an
  unquoted `#` starts a comment mid-line (`B=raw#c` → `raw`).
- No variable expansion (`${OTHER}`), no `export` prefix handling. If you depend on those,
  keep `dotenv-expand`.
- Missing file throws; use `--env-file-if-exists` for optional local overrides.


| Old habit | Modern built-in | Notes |
|---|---|---|
| `nodemon` | `node --watch app.js` | restarts on imported-module changes; `--watch-path=src` to scope |
| `npm run` (leaf scripts) | `node --run` | **no pre/post scripts** — see above |
| `dotenv` (dev) | `node --env-file=.env` / `--env-file-if-exists` | see caveats below |
| `npm run` in containers | `node --run` | no npm binary/shell needed |
| `rimraf` | `node -e "fs.rmSync(p,{recursive:true,force:true})"` or `fs/promises` | [runtime-apis.md](./runtime-apis.md) R4 |
| `node-fetch`/`axios` (basic) | global `fetch` | [runtime-apis.md](./runtime-apis.md) R1 |
| `uuid` | `crypto.randomUUID()` | |
| `minimist` (simple CLIs) | `node:util parseArgs` | [runtime-apis.md](./runtime-apis.md) R5 |

**R6 — Version managers: nvm for humans, mise/fnm/volta for speed, corepack-agnostic. [Context]**
All honor `.nvmrc`/`.node-version`. In CI, use `actions/setup-node` `node-version-file`.

## Repo baseline for new Node 24+ projects

```
.nvmrc                          # 24
package.json                    # "type": "module", engines, packageManager
eslint.config.js                # flat config
tsconfig.json                   # per typescript.md R4 (TS projects)
.env.example                    # documented, gitignored real .env
.github/workflows/ci.yml        # lint -> tsc --noEmit -> node --test / vitest
Dockerfile                      # per deployment.md
```
