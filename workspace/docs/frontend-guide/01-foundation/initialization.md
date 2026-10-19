A strong foundation is one you can build on immediately without revisiting. This phase is not about choosing the perfect stack—it's about establishing the defaults that remove decision fatigue for every subsequent phase.

## Core Principle: Strong Foundation Without Speculative Complexity

Set up only what blocks feature development. Every tool, config, or pattern added during initialization must answer "will the first feature PR fail without this?" If the answer is no, defer it. Initialization is not the time to add a design system, a component library, or an API client abstraction. It *is* the time to ensure TypeScript compiles, tests run, and the build produces correct output.

## Technology Defaults (2026)

These are not suggestions. Use them unless the project brief explicitly overrides them.

**Runtime & Framework:** Next.js 16 (requires Node.js 20.9+, React 19.2) with App Router. Turbopack is the default dev server *and* build tool — do not configure Webpack. Use `proxy.ts` for request interception — it is `middleware.ts` renamed in Next.js 16 (auth redirects, rewrites, header logic; Node.js runtime only). Do not use it for API proxying: data comes from Express through `apiFetch` in Server Components and Server Actions ([backend-integration](../03-data/backend-integration.md)), and cross-origin routing belongs to your reverse proxy or platform config.

**UI:** React 19+. Server Components are the default for every file under `app/`. Only add `'use client'` when the component requires interactivity (event handlers, hooks, browser APIs). React Compiler is stable but opt-in via top-level `reactCompiler: true` — enable it after verifying the build passes and build times are acceptable; it forces Babel into the pipeline. For large apps, prefer `reactCompiler: { compilationMode: 'annotation' }` and mark hot components with `"use memo"` first. Opt a specific component out with the `"use no memo"` directive.

**Styling:** Tailwind CSS v4, CSS-first. Do not create a `tailwind.config.js` or `tailwind.config.ts`. The tokens are split across three files: `src/app/globals.css` (imports), `src/styles/tokens.css` (the kit's token names, never edited) and `src/styles/theme.css` (the brand values, the only file a project edits). Their complete content is in [theme-and-dark-mode](../02-ui/theme-and-dark-mode.md); copy it from there. The minimal shape of a v4 token, for reference only:

```css
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.7 0.15 250);
  --font-sans: "Inter", sans-serif;
}
```

**TypeScript:** Strict mode enabled. No `any` types without an explicit `// eslint-disable-next-line` and a comment explaining why. The `tsconfig.json` should have `"strict": true` and no overrides that weaken it.

**Testing:** Vitest for unit and integration tests. Do not set up Jest—even with `ts-jest`, it's slower and conflicts with Turbopack's module resolution. E2E is the user's, by hand ([testing](../04-quality/testing.md) § E2E Testing): no Playwright unless the user asks for automated E2E.

**Linting & Formatting:** ESLint flat config (`eslint.config.mjs` — what create-next-app generates; `@next/eslint-plugin-next` is flat-config-first now). Run ESLint via its own CLI (`eslint .`) — the `next lint` command was removed in Next.js 16. Use Prettier for formatting ([code-quality](../04-quality/code-quality.md)). Biome is the alternative if you want one unified fast linter+formatter instead of two tools.

## Initialization Checklist by Domain

### Project Scaffolding

**Set up:**
- **npm 11 or newer** (`npm --version`; upgrade with `npm install -g npm@11`). npm 10.9 (bundled
  with Node 22) crashes installing Vitest with `Cannot read properties of null (reading
  'edgesOut')`. npm 11 may hold back install scripts and say so; everything below works without
  them. *(Verified in a real setup, 2026-09-23.)*
- From the project root, non-interactive:
  `npx create-next-app@latest frontend --ts --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm --yes --disable-git`.
  The app lives in `frontend/` (this guide's README and project-structure define its app boundary)
- If scaffolding emits client-entry notes inside the app, inspect their scope before using them. They do not appoint operators or replace the root Roster/role startup route. Tool-generated or installed framework notes do not override Implementer's exclusive local `workspace/docs/syntax/` authority. A required unsupported framework API is a source gap to report; source updates/research need separate authority. This does not require recreating deleted project-level client instructions.
- `.gitignore` verified (node_modules, .next, .env*.local, coverage, test-results)
- Single `.env.example` file documenting required env vars (even if empty)
- `.vscode/settings.json` with recommended extensions and format-on-save

**Do NOT set up:**
- Husky + lint-staged (defer until CI is configured in [project-structure](./project-structure.md))
- Multiple environments (staging/prod) in the repo (defer)
- Monorepo tooling (npm workspaces, Turborepo). The project is two independent apps side by side,
  `frontend/` and `backend/`, and neither imports the other's code

### TypeScript Configuration

**Set up:**
- `"strict": true` in `tsconfig.json`
- Path aliases: `@/*` mapping to `src/*`
- `"noUncheckedIndexedAccess": true`—catches `arr[0]` returning `T | undefined`
- `"incremental": true` for faster rebuilds

**Do NOT set up:**
- Custom type declaration files unless a library lacks types
- `enum` usage guidelines—use `as const` objects instead
- Barrel files (`index.ts` re-exports) during initialization

### Testing Infrastructure

**Set up** (verified end to end in a real setup, 2026-09-23):
- Install the **pinned** versions from the README's Freshness table, not `@latest`. The newest
  Vitest (5.x) refuses `create-next-app`'s `@types/node` 20:
  `npm install -D vitest@4.1.11 jsdom @testing-library/react @testing-library/jest-dom`
- `vitest.config.mts` (`.mts`: a `.ts` config makes Vite warn in this CommonJS package). Tests
  need a DOM (`jsdom`), the `@/` alias, and the `jest-dom` matchers such as `toBeInTheDocument`:

```ts
// vitest.config.mts
import { defineConfig } from 'vitest/config';
import path from 'node:path';

export default defineConfig({
  resolve: { alias: { '@': path.resolve(import.meta.dirname, 'src') } },
  test: { environment: 'jsdom', setupFiles: ['./src/test-utils/setup.ts'] },
});
```

```ts
// src/test-utils/setup.ts
import '@testing-library/jest-dom/vitest';
```

- **The implementer writes no test.** The testers write every test, setup's included, each in its
  own folder, in the test round ([testing](../04-quality/testing.md)). Until the first one
  exists, THE CHECK must still pass,
  so the `test` script runs `vitest run --passWithNoTests` (THE CHECK below). When the testers
  write the first component test: never point it at `app/page.tsx`. Under the server-first
  default ([architecture](./architecture.md)) that file is an async Server Component, and React
  Testing Library cannot render one — `render(<Page />)` receives a Promise and the test fails
  for a reason that has nothing to do with the app. Server Components are covered by the user's
  own E2E check, by hand ([testing](../04-quality/testing.md) § E2E Testing).
- The `npm run test` script (THE CHECK below)

**Do NOT set up:**
- Playwright, `playwright.config.ts` or a `test:e2e` script: E2E is the user's, by hand
  ([testing](../04-quality/testing.md) § E2E Testing). Only if the user asks for automated E2E:
  config at the app root, `testDir: './e2e'`, webServer serving the production build
  (`npm run build` + `npm run start`), since E2E always runs against the built app
- Test utilities, mock factories, or fixtures (create when first test needs them)
- Coverage thresholds (defer until meaningful tests exist)
- Storybook (defer to [project-structure](./project-structure.md))

### Linting & Formatting

**Set up:**
- Flat config ESLint with Next.js and TypeScript plugins
- A `lint` script and a `lint:fix` script
- **THE CHECK** (below): the one command that decides whether work is done

**Do NOT set up:**
- Commitlint, conventional commits enforcement (team decision)
- Complex custom rules. The only custom rules are the architecture rules below; each one
  enforces a decision a written rule alone could not.

### THE CHECK — `npm run check`

A rule written in a guide can be ignored; a check that fails cannot. `npm run check` must pass
before any work counts as done. The orchestrator runs it itself; a worker's "done" never replaces
it.

```json
// package.json
"scripts": {
  "lint": "eslint . --max-warnings 0",
  "typecheck": "tsc --noEmit",
  "test": "vitest run --passWithNoTests",
  "check:kit": "bash scripts/check-kit.sh",
  "check": "npm run lint && npm run typecheck && npm run test && npm run check:kit"
}
```

**Architecture rules as ESLint errors.** This is the complete `eslint.config.mjs`: the file
`create-next-app` generates (a `defineConfig([...])` array), with the two rule blocks added after
the presets. Verified in a real setup (2026-09-23): each violation fails `npm run lint` with the
message below, and clean code passes.

```js
// eslint.config.mjs
import { defineConfig, globalIgnores } from 'eslint/config';
import nextVitals from 'eslint-config-next/core-web-vitals';
import nextTs from 'eslint-config-next/typescript';

const NO_DATABASE = {
  group: ['@supabase/*', 'pg', 'postgres', '@prisma/client', 'drizzle-orm', 'drizzle-orm/*'],
  message: 'Next.js never talks to a database or Supabase. Call Express through apiFetch — see 03-data/backend-integration.md.',
};

export default defineConfig([
  ...nextVitals,
  ...nextTs,
  {
    rules: { 'no-restricted-imports': ['error', { patterns: [NO_DATABASE] }] },
  },
  {
    // A later block REPLACES the rule for these files, so it repeats NO_DATABASE.
    files: ['src/components/ui/**'],
    rules: {
      'no-restricted-imports': ['error', {
        patterns: [
          NO_DATABASE,
          {
            group: ['@/features/*', '@/app/*', '@/actions/*', '@/lib/http', '@/lib/session'],
            message: 'components/ui is the portable kit — see 02-ui/portability.md.',
          },
        ],
      }],
    },
  },
  globalIgnores(['.next/**', 'out/**', 'build/**', 'next-env.d.ts']),
]);
```

`scripts/check-kit.sh` is the kit-purity script in [portability](../02-ui/portability.md). On
Windows it runs through Git Bash.

**Test-discovery gate:** `--passWithNoTests` is temporary for untested bootstrap only.
A zero-test bootstrap check verifies tooling, not product behavior. The Roster/brief identifies the actual Tester operator(s) and their permitted test folders; no fixed clients or mandatory pair is implied. After the assigned initial implementation test phase produces nonempty discovered/executed tests, Orchestrator reviews every result and the required AC coverage.

Orchestrator routes the strict-script change through Planner's approved software TODO/change record and briefs Implementer to remove the temporary flag (`vitest run`). Implementer edits application configuration and reports the new candidate; Orchestrator personally reruns the existing suites and quality gates. Do not automatically send the fix back to Tester. If new/changed tests or unavailable verification are required, keep that scope blocked for the owner's decision under the shared delivery cycle.

Before setup is recommended ready, verify nonempty execution in each assigned test scope, the strict script and actual evidence for applicable automatable ACs. Missing/skipped/unverified required checks block that readiness. Pending owner manual E2E remains explicit and blocks final product acceptance until the actual owner check. A passing script alone is not acceptance.

Run a production build before readiness. In the frontend, the empty kit directory may exist
during bootstrap, but its directory itself must exist for kit checks.

## What the Output Should Look Like

After initialization, the repo should have this structure:

```
src/
  app/
    layout.tsx          # Root layout with html/body, lang attribute
    page.tsx            # Placeholder home page
    globals.css         # imports tailwindcss + styles/tokens.css + styles/theme.css
  styles/
    tokens.css          # the kit's token names (02-ui/theme-and-dark-mode.md)
    theme.css           # the brand values
  test-utils/
    setup.ts            # the test runner's setup file (jest-dom matchers)
scripts/
  check-kit.sh         # kit-purity checks, run by `npm run check`
.env.example
eslint.config.mjs      # flat config + the architecture rules above
next.config.ts         # no Webpack config; add reactCompiler only after verifying the build
tsconfig.json
vitest.config.mts
```

The dev server starts with `npm run dev`. `npm run test` passes. `npm run lint` (ESLint CLI) passes. `npm run build` produces a `.next/` output without errors.

Report the measured baseline with candidate/environment evidence in your role Output and link it from the task control record: the production build's initial-route JS size (gzipped) as the starting performance budget ([performance](../04-quality/performance.md)). A deviation from these defaults is never the implementer's own: it is recorded as an approved DEC-n in Planner Output and reflected in the software TODO first, and building it without one is BLOCKED ([environments-and-config](../05-delivery/environments-and-config.md) covers env var conventions).

## Verification

Run this sequence before declaring initialization complete:

1. `npm run dev` → loads in under 3 seconds, page renders
2. `npm run build` → completes without warnings
3. `npm run check` → passes (lint, types, tests, kit purity)
4. Prove the architecture rule works: add `import { createClient } from '@supabase/supabase-js'`
   to any file → `npm run lint` must FAIL → remove the line
5. Delete `node_modules/`, run `npm install`, repeat steps 1–3 (clean install check)

## Common Mistakes

**Adding abstractions too early.** Creating unused lib/format.ts or hooks/useDebounce.ts before a feature requires them. The empty components/ui directory is a setup prerequisite for kit checks. Create implementations at the point of first use.

**Configuring Webpack.** Turbopack is the default bundler for **all** apps in Next.js 16
(verified against the release notes: *"Turbopack is now the default bundler for all apps; opt
out with `next build --webpack`"*). If you're writing a `next.config.ts` with
`webpack: {}`, you're fighting the framework. The `--webpack` flag exists for apps with
custom webpack setups they cannot yet port — treat it as a migration escape hatch with an exit
date, not a configuration choice, recorded with its reason and actual approval in Planner Output/DEC-n and the software TODO. Keep next.config.ts minimal. Justified additions include reactCompiler, images.remotePatterns (prefer it to deprecated images.domains), serverExternalPackages, compiler.removeConsole, cacheComponents and experimental.serverActions.bodySizeLimit. Record the need and verify the installed framework API; this is not a closed allowlist.

**Using `middleware.ts` for anything new.** It is deprecated in Next.js 16 — the file and exported function are now `proxy.ts`/`proxy` (Node.js runtime only; keep `middleware.ts` only if you specifically need the Edge runtime). Request-level API proxying is not proxy.ts's job: production traffic routing belongs to infrastructure, and interactive reads and streaming use explicit Route Handlers; RSC reads call server queries directly.

**Enabling React Compiler without verifying.** Run `npm run build` after enabling it — it pulls Babel into the pipeline, so build times will rise. If a specific component breaks under the compiler (intentional mutation, referential-identity dependencies), add `"use no memo"` to that function or file rather than disabling globally. For large existing apps, start with `compilationMode: 'annotation'`.

**Installing libraries "you'll need later."** If you're adding `zod`, `date-fns`, or `framer-motion` during initialization without a feature that uses them, remove them. They inflate bundle size and create false confidence that the architecture accounts for them.
