## Where Should This Code Live?

This is the most common question. Follow this tree:

- **Reusable UI primitive** (button, dialog, input, toast) → `components/ui/`
  - Must be presentational, accept props, have no feature logic
  - See [components.md](../02-ui/components.md) for component API conventions
- **Feature-specific component** → In its feature folder under `src/features/<feature>/`
  - See [project-structure.md](../01-foundation/project-structure.md) for feature folder structure
- **Pure utility function** (formatting, math, validation) → `lib/<domain>.ts`
  - Group by domain, not a single `utils.ts` dump
- **Shared hook** (used across features) → `hooks/<name>.ts`
  - Feature-specific hooks stay with the feature
- **Server Action** → **default: alongside the feature it serves**, `features/<feature>/actions.ts`.
  Promote to `actions/<domain>.ts` only when **two or more features** call it.
  - Shared actions/queries/hooks/types move on a second feature consumer; feature UI waits for a third matching call site (design-system primitives excepted).
  - See [project-structure.md](../01-foundation/project-structure.md) for the directory contract and
    [backend-integration.md](../03-data/backend-integration.md) for Action conventions.
- **Cross-feature type** → `types/<domain>.ts`
  - Locally scoped types stay in the file that uses them
- **Test helper** (custom render, factories) → the tester's own `tests/<tester>/` folder
  (`test-utils/` holds only the test runner's setup file)
- **E2E test** → none: E2E is the user's, by hand ([testing](../04-quality/testing.md) § E2E Testing)

## Should This Be Reusable?

- **3+ features will use it** → Yes, make it reusable in `components/ui/`
- **It is a design system component** → Yes, always reusable. These are the project's primitives.
- **Tightly coupled to one feature's domain logic** → No, keep it feature-specific
- **Only 2 call sites exist** → Defer. Two copies are fine. Extract when a third appears.

The cost of a bad abstraction far exceeds the cost of duplicated code. See [anti-patterns.md](../04-quality/anti-patterns.md) on premature abstraction.

## Where Should This State Live?

This is the second most common question. Follow this tree:

- **Server data** (user profile, API results, database records) → Server Component fetch or TanStack Query
  - Do not copy into `useState`. The fetch result IS the state.
  - See [state-and-data.md](../03-data/state-and-data.md) for the full strategy
- **Should be in the URL** (filters, pagination, selected tab, search query) → URL state via `useSearchParams` or `nuqs`
  - If a user would want to bookmark, share, or navigate back to this state, it belongs in the URL.
  - See [routing.md](../01-foundation/routing.md) for URL state patterns
- **UI state for one component** (modal open, dropdown expanded, tooltip visible) → Local `useState` or `useReducer`
  - Keep this as close to the component as possible. Never lift to global unless needed.
- **Shared across unrelated component trees** (theme, auth status, sidebar collapsed) → Zustand, Jotai, or `useSyncExternalStore`
  - Use selectors to prevent unnecessary re-renders.
  - See [state-and-data.md](../03-data/state-and-data.md) for the store comparison
- **Form state** → `useActionState` + Server Action
  - Progressive enhancement: the form works without JS, and JS enhances with optimistic updates.
  - See [forms.md](../02-ui/forms.md) for the full form strategy
- **Derived from other state** (full name from first + last, filtered list from search + items) → Compute during render, do not store
  - Storing derived state creates sync bugs. `useMemo` is acceptable for expensive computations.

## Should This Be a Server Component or Client Component?

- **Uses `useState`, `useEffect`, browser APIs, or event handlers** → Client Component (`'use client'`)
- **Only renders HTML based on props and server data** → Server Component (default)
- **When in doubt, start as Server Component** and add `'use client'` when a browser API or hook is needed.

Key principle: keep the Client Component boundary as deep in the tree as possible. A page should be a Server Component that passes data and Server Component children down to a small number of Client Component leaves. See [components.md](../02-ui/components.md) for composition patterns and the React guide for RSC details.

## Should I Create a New Abstraction?

- **Duplication across 3+ call sites** → Consider extracting. Check that the extracted abstraction has a clear name and single responsibility.
- **Clear, single responsibility** → Yes, extract. A utility called `formatCurrency` is good. One called `handleThing` is not.
- **Only 2 places, and they might diverge** → Keep separate. The apparent duplication may reflect legitimate differences.
- **Building for a hypothetical future** → No, defer. This is speculative architecture. See [anti-patterns.md](../04-quality/anti-patterns.md).
- **The abstraction would need configuration to handle all cases** → No. A heavily parameterized abstraction is worse than moderate duplication.

## Should I Add This Dependency?

- **Can be done with built-in platform APIs** → No dependency needed. Use `structuredClone`, `AbortController`, `URL`, `crypto`, etc.
- **Well-maintained, actively developed, small bundle** → Consider adding. Check: last release
  within 6 months, no open critical issues. On size, [dependencies.md](../04-quality/dependencies.md)
  is authoritative: **anything over ~10 kB gzipped must justify itself explicitly**, and 50 kB
  is an upper bound you should rarely approach for a single utility.
- **Brings many transitive dependencies** → Evaluate carefully. `npm ls <package>` shows the
  actual tree, and bundlephobia/pkg-size.dev show installed + gzipped weight *including*
  dependencies. (`npm pack --dry-run` only reports the package's own tarball — it says nothing
  about what it drags in, which is the thing you are trying to measure.)
- **Can be implemented in < 50 lines** → Implement it yourself. A small inline function is better than a dependency for simple needs.
- **Is a framework or infra tool** (Next.js, TanStack Query, Zod) → Add it. These provide value far beyond what you would build. **Exception:** never a database client or ORM (Prisma, Drizzle, Supabase) — those belong to Express only, see [security](../04-quality/security.md).

Before adding any dependency, check [dependencies.md](../04-quality/dependencies.md) for the full evaluation checklist.

## What Should I Test?

Not everything needs a test. Use this filter:

- **Business logic** (validation rules, calculations, data transformations) → Unit test with Vitest
  - These are high-value, low-maintenance tests.
- **Component behavior** (user interactions, conditional rendering, error display) → Component test with React Testing Library
  - Test what the user sees and does, not internal state. See [testing.md](../04-quality/testing.md).
- **Critical user flow** (checkout, signup, data entry) → the user's E2E check, by hand
  (automated E2E with Playwright only if the user asks)
- **Accessibility** → `jest-axe` in component tests (THE CHECK), manual keyboard/screen reader testing
  - Automated a11y catches ~30-40% of issues. Manual testing is required.
  - See [accessibility.md](../02-ui/accessibility.md).
- **Trivial prop forwarding, CSS classes, static markup** → Do not test
  - These tests add maintenance cost with no regression detection value.

## How Do I Handle Errors?

- **Data fetching failure** → TanStack Query's `error` state + error boundary component
- **Server Action failure** → an error the user can act on (validation, a refusal) is returned as
  state and rendered inline near the form or the action (including login 401). Handle session expiry/403 with the defined authentication/permission flow; operational or unexpected failures are rethrown to
  `error.tsx` ([backend-integration](../03-data/backend-integration.md) § Mutations and Error States)
- **Uncaught render errors** → React error boundaries wrapping route segments
- **Network offline** → **persistent** indicator (a thin banner or a status dot in the nav),
  not a toast. A toast auto-dismisses; being offline does not. See
  [resilience.md](../04-quality/resilience.md). Use a toast for the *transition* back online if you
  want, never for the state itself.

See [resilience.md](../04-quality/resilience.md) for the complete error handling strategy.
