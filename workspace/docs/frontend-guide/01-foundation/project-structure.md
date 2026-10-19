Every file has one correct location. The structure implements the dependency layering defined in [architecture](./architecture.md): `app/` routes import from `features/`; `features/` import from `components/ui/`, `hooks/`, `actions/`, `lib/`, `types/`; those shared layers never import upward. Files may only import from their own directory or lower layers — never upward, never sideways into a sibling feature.

## Canonical Structure

```
src/
  app/                    # Next.js App Router — routing, layouts, pages
    (auth)/
      login/page.tsx
      register/page.tsx
      layout.tsx
    (dashboard)/
      layout.tsx
      dashboard/
        page.tsx
        loading.tsx
        error.tsx
      projects/
        page.tsx
        [id]/page.tsx
    not-found.tsx
    layout.tsx
    globals.css           # imports tailwindcss + styles/*, base styles only
  styles/                 # design tokens (02-ui/theme-and-dark-mode.md)
    tokens.css            # kit: semantic names → Tailwind; never edited per project
    theme.css             # brand values; the ONLY styling file a project edits
  components/
    theme-provider.tsx    # next-themes wrapper (client)
    ui/                   # Reusable primitives: the portable kit (02-ui/portability.md)
      button.tsx
      dialog.tsx
      input.tsx
  features/               # Feature modules (see 01-foundation/architecture)
    dashboard/
      dashboard-shell.tsx
      project-card.tsx
      actions.ts
      use-project-filters.ts
      types.ts
    auth/
      login-form.tsx
      actions.ts
  lib/                    # Shared utilities + explicitly documented server/framework adapters
    format-currency.ts
    cn.ts
    http.ts               # apiFetch: the ONLY file that knows Express exists (03-data/backend-integration)
    session.ts            # getSession() via Express /auth/me — never Supabase
  hooks/                  # Shared custom hooks only
    use-debounce.ts
    use-media-query.ts
  actions/                # Cross-feature Server Actions
    create-invite.ts
  types/                  # Shared TypeScript types/interfaces
    user.ts
    api-responses.ts
  test-utils/             # only the test runner's setup file
    setup.ts
tests/                    # unit + component tests, one folder per tester (04-quality/testing)
  kilo/                   #   mirrors src/: kilo/features/dashboard/project-card.test.tsx
  opencode/               #   the same paths, an independent suite
public/
  favicon.ico
  images/
  fonts/
```

## Directory Responsibilities

### `src/app/`
**Responsibility:** Routing and route-level concerns only.

**Belongs here:** `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `default.tsx`, `template.tsx`, route groups `(name)`, and same-origin `route.ts` transport handlers. `proxy.ts` lives at src/proxy.ts alongside app/, not inside app/.

**Does NOT belong here:** Components, hooks, utilities, business logic, data transformation. A `page.tsx` should import a feature component and render it—nothing more. Keep page files thin.

**Dependency direction:** May import from `features/`, `components/ui/`, `lib/`, `types/`, `actions/`. **Never imports from `hooks/`** — a hook implies client state, and files here are Server Components; a route file reaching for a hook means the `'use client'` boundary is in the wrong place. Push the interactivity into a feature component and let that use the hook. This matches the layering table in [architecture](./architecture.md).

**Relationship to other directories:** Route groups `(name)/` create layout boundaries without URL segments—use them to group routes that share a layout, auth requirements, or data loading context.

### `src/components/ui/`
**Responsibility:** Reusable, styled, accessible UI primitives. These are the building blocks features compose.

**Belongs here:** Button, Input, Select, Dialog, Tooltip, Tabs, Badge, Avatar—anything that is (a) used by multiple features, (b) has no feature-specific logic, and (c) is backed by a headless library (Radix or React Aria) when interactive.

**Does NOT belong here:** Feature-specific components (e.g., `ProjectCard`, `UserMenu`), components that import from `features/`, components that contain business logic or make data-fetching decisions.

**Dependency direction:** May import `lib/cn.ts`, `types/`, and other `ui/` components. Must NOT import from `features/`, `actions/`, `hooks/`, or the server-only `lib/http.ts` / `lib/session.ts`, and never reads env variables. This is what keeps `ui/` copyable into another project unchanged. The full contract and the checks that enforce it are in [portability](../02-ui/portability.md).

**Relationship to other directories:** `ui/` is the lowest component layer. Features compose `ui/` primitives. See [components.md](../02-ui/components.md) for API design and [design-tokens.md](../02-ui/design-tokens.md) for token consumption.

### `src/features/`
**Responsibility:** Self-contained feature modules. Each feature owns everything specific to that capability.

**Belongs here:** Feature components, feature-specific hooks, **feature Server Actions** (`features/<feature>/actions.ts` — this is the default home for an action; promote to `src/actions/` only when a second feature calls it), feature types. (Tests live in each tester's folder, `tests/<tester>/`, not here.)

**Does NOT belong here:** Imports from sibling features. For two feature-UI call sites, keep local copies until the third establishes a shared primitive; design-system primitives start in components/ui. Shared queries/actions/hooks/types move on a second feature consumer. Utilities without feature coupling move to lib/.

**Dependency direction:** May import from `components/ui/`, `lib/`, `types/`, `hooks/`. Must NOT import from another feature directory. If feature A needs data from feature B, the shared types belong in types/ and server-only queries in lib/<domain>.ts; shared mutation actions belong in actions/.

**Relationship to other directories:** Most implementation lives here. A feature maps to a route segment in `app/` but the folder is independent of routing.

### `src/lib/`
**Responsibility:** Shared utilities plus explicitly documented server/framework adapters. Pure utilities remain framework-agnostic.

**Belongs here:** String formatting, date helpers, constants, the `cn()` classname merge utility.

**Documented server adapters:** `lib/http.ts` and `lib/session.ts` are server-only (they read cookies through `next/headers`) and own all communication with Express. Feature-specific endpoint calls live in features/<feature>/queries.ts (server-only reads) or actions.ts (mutations), built on apiFetch ([backend-integration.md](../03-data/backend-integration.md)).

**Does NOT belong here:** Feature business rules or upward imports. React/Next/TanStack imports are allowed only in documented framework adapters (for example session.ts and query-client.ts), not in pure utilities. Avoid ambient mutable request state.

**Dependency direction:** Imports shared types and platform/library APIs; pure utilities remain framework-independent. Documented adapters such as http.ts, session.ts and query-client.ts may use Next.js/React/TanStack APIs. Never import upward from components/, features/, hooks/ or actions/.

### `src/hooks/`
**Responsibility:** Shared custom hooks used by multiple features.

**Belongs here:** `useDebounce`, `useMediaQuery`, `useLocalStorage`—hooks with zero feature-specific logic that at least two features consume.

**Does NOT belong here:** Feature-specific hooks (those live in the feature folder), plain functions without React APIs (those live in `lib/`).

**Dependency direction:** May import from `lib/`. Must NOT import from `features/`.

### `src/actions/`
**Responsibility:** Cross-feature Server Actions—mutations or form submissions that multiple features trigger.

**Belongs here:** `createInvite`, `updateUserPreferences`—actions whose scope spans features.

**Does NOT belong here:** Feature-scoped actions (those live in the feature folder alongside the feature they serve).

**Dependency direction:** May import from `lib/`, `types/`. Must NOT import from `features/` or `components/`.

### `src/types/`
**Responsibility:** Shared TypeScript types, interfaces, and type guards used across multiple features.

**Belongs here:** `User`, `ApiResponse<T>`, `PaginationParams`—types that two or more features reference.

**Does NOT belong here:** Feature-specific types (e.g., `DashboardStats` used only in the dashboard feature—keep it in `features/dashboard/types.ts`).

### `src/test-utils/`
**Responsibility:** Only the test runner's setup file (`setup.ts`, created at setup: [initialization](./initialization.md)).

**Does NOT belong here:** Test helpers of any kind — render wrappers, factories, fixtures, mock handlers — and test files. Each tester keeps its own helpers in its own `tests/<tester>/` folder: the two suites are independent and share nothing.

### `e2e/`
**Only if the user asks for automated E2E.** In this project E2E is the user's, by hand ([testing](../04-quality/testing.md) § E2E Testing), so there is no `e2e/` folder and no `playwright.config.ts`. If the user asks: Playwright tests here, separate from `src/`, never importing from it; the config at the app root with `testDir: './e2e'`.

### `public/`
**Responsibility:** Static assets served at the root URL path.

**Belongs here:** `favicon.ico`, images, fonts, `robots.txt`. Files here are served as-is with cache headers.

**Does NOT belong here:** Anything that needs transformation at build time (those go in `src/` and are imported).

## File Placement Decision Tree

```
Is it a route/page/layout?
  → src/app/[route]/

Is it a reusable UI primitive with no feature logic?
  → src/components/ui/

Is it specific to one feature?
  → src/features/[feature]/

Is it a pure utility with no React dependency?
  → src/lib/

Is it a shared hook (no feature logic, 2+ consumers)?
  → src/hooks/

Is it a Server Action or server query used by multiple features?
  → src/actions/ for mutations; src/lib/<domain>.ts for server-only queries

Is it a type shared across 2+ features?
  → src/types/

Is it a test helper (render wrappers, factories, mocks)?
  → the tester's own tests/<tester>/ folder (src/test-utils/ holds only the runner's setup file)

Is it an E2E test?
  → none: E2E is the user's, by hand (e2e/ only if the user asks for automated E2E)

Is it a static asset?
  → public/
```

## When to Create a New Directory

Never create a directory for a single file. A directory exists to group related files and signal a boundary. Create a new directory when:

1. **Three or more related files** share a common concern (e.g., three dashboard components justify `features/dashboard/`).
2. **A clear module boundary emerges**—high cohesion, low coupling with the rest.
3. **The files are imported together** by the same consumers.

Do not create directories for organizational aesthetics, speculative future files, or to "keep things tidy." An extra level of nesting adds navigation cost. Flat is better than nested until the nesting is justified.

## Cross-Feature Import Rules

Features must not import from other features. This is the most important structural rule. When feature A needs something from feature B:

- **Shared types:** Extract to `src/types/`.
- **Shared data fetching:** Extract the fetch function to `src/lib/`.
- **Shared UI:** Design-system primitives live in src/components/ui/ from the start; feature UI waits for a third matching call site. Two local copies avoid forbidden sideways imports.
- **Communication between features:** Use URL state, a shared store (Zustand/Jotai), or callback props passed through a shared layout.

No sibling-feature type-import exception: shared types move to a domain-named types/<domain>.ts on the second feature consumer, keeping ownership clear without a dumping ground.

## Root Configuration Files

Root-level files are project infrastructure, not application code:

- `next.config.ts` — Next.js configuration. Keep minimal: top-level flags like `reactCompiler`, `images.remotePatterns`, `serverExternalPackages`. Request interception logic goes in `proxy.ts` (Next.js 16 convention).
- `tsconfig.json` — TypeScript configuration with path aliases (`@/`).
- No `tailwind.config.ts` — Tailwind v4 is configured in CSS (`@theme {}`, `@plugin`). Only add a JS config file if a legacy plugin requires it, referenced via `@config` in CSS. See [design-tokens](../02-ui/design-tokens.md).
- `vitest.config.mts` — Test runner configuration. See [testing](../04-quality/testing.md).
- `eslint.config.mjs` or `biome.json` — Linting and formatting. Pick one toolchain per project ([code-quality](../04-quality/code-quality.md)).
- `package.json` — Dependencies and scripts. Keep scripts minimal; complex build logic belongs in `next.config.ts`.

Root config files should not contain application logic. They configure tools, not behavior.
