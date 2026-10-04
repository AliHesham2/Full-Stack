## Architecture Anti-Patterns

**Premature abstraction.** Do not extract a shared utility or component before 3+ call sites exist. Two copies of similar code are cheaper to maintain than one wrong abstraction. See [decision-framework.md](../00-navigation/decision-framework.md) for the extraction criteria.

**Giant components.** A file over 200 lines prompts a cohesion review; length alone is not a defect. Split distinct responsibilities and keep cohesive code together. See [components.md](../02-ui/components.md) for composition patterns.

**Global state overuse.** Not every piece of state belongs in a global store. Form input values, toggle states, and modal open flags should be local. Reserve global state for data shared across unrelated component trees. See [state-and-data.md](../03-data/state-and-data.md) for the state placement decision tree.

**Utility dumping grounds.** `lib/utils.ts` should not become a catch-all for unrelated functions. Group utilities by domain: `lib/format.ts`, `lib/validation.ts`, `lib/math.ts`. A 500-line utils file is unmaintainable. See [project-structure.md](../01-foundation/project-structure.md) for file organization rules.

**Speculative architecture.** Do not build plugin systems, abstraction layers, or configuration schemas for hypothetical future requirements. Build what is needed now. Refactor when a real need arises. YAGNI applies to frontend architecture decisions.

**Circular feature-folder imports.** Feature folders must not import from each other. If feature A needs feature B's component, lift the shared piece to `components/ui/` (if presentational) or to `lib/`/`types/` (if it's logic or types). Circular imports cause silent failures in bundlers and make the dependency graph unreadable.

## React Anti-Patterns

**`useEffect` for data fetching.** Use Server Components for initial data or TanStack Query for client-side fetching. `useEffect` fetches race, do not deduplicate, and lack caching, background refetch, and error handling. See [state-and-data.md](../03-data/state-and-data.md) for the data fetching strategy.

**Context for high-frequency state.** React Context causes every consumer to re-render when the value changes. Never use it for input values, mouse positions, animation frames, or any state updating faster than ~100ms. Use `useSyncExternalStore`, Zustand with selectors, or Jotai atoms. See [state-and-data.md](../03-data/state-and-data.md) for the full comparison.

**Wrapping too much in `'use client'`.** Every `'use client'` directive negates RSC benefits: no streaming, no reduced bundle size, no direct backend access. Mark only the leaf components that need interactivity as client components. Pass Server Components as children to Client Components. See [components.md](../02-ui/components.md) for the Server/Client boundary strategy.

**Importing Server Components into Client Components.** This is architecturally impossible. Server Components cannot be passed as imports to Client Components. Pass them as `children` props instead: `<ClientWrapper>{serverContent}</ClientWrapper>`.

**Manual memoization everywhere.** When the React Compiler is enabled, avoid `useMemo`, `useCallback`, and `React.memo` unless profiling proves they are necessary. The compiler handles memoization automatically. Premature manual memoization adds noise without guaranteed benefit.

**Storing server data in client state.** Data fetched from the server should not be copied into `useState`. This creates a second source of truth and leads to stale data. Use TanStack Query's cache or keep data in Server Components. See [state-and-data.md](../03-data/state-and-data.md).

**HOCs and render props in new code.** Hooks replaced both patterns. Do not introduce new HOCs or render props. Existing ones should be migrated to hooks during natural refactoring cycles.

## Security Anti-Patterns

**JWT in localStorage.** `localStorage` is accessible to any JavaScript on the page, making it vulnerable to XSS attacks. Use HttpOnly cookies set by the server. See [security.md](./security.md) for the authentication security model.

**`dangerouslySetInnerHTML` without sanitization.** Never render raw HTML from user input or external sources without running it through DOMPurify first. Even trusted CMS content can be compromised. See [security.md](./security.md).

**`Access-Control-Allow-Origin: *`** in production. Wildcard ACAO permits cross-origin reading of non-credentialed responses; browsers reject wildcard ACAO with credentials. It is not authentication or a firewall. In this project the browser never calls Express, so Express allows no browser origin at all and needs no CORS ([backend-integration.md](../03-data/backend-integration.md) § Authentication).

**Missing security headers.** Every production deployment must set `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, and `Permissions-Policy`. See [security.md](./security.md) for the full header list.

**Client-side authorization as the real security boundary.** Hiding a button or route client-side is UX, not security. Authorization must be enforced server-side in API routes, middleware, and database queries. See [security.md](./security.md) and the Authentication guide.

**Hardcoded API keys or secrets.** API keys, database URLs, and secrets must never appear in client-side code. Use environment variables with `NEXT_PUBLIC_` prefix only for non-sensitive values. Server-side secrets stay server-side. See [security.md](./security.md).

## Accessibility Anti-Patterns

**`<div>` with `onClick` instead of `<button>`.** A `<div>` is not focusable by default, lacks button semantics, and does not respond to Enter/Space. Always use `<button>` for clickable elements. See [accessibility.md](../02-ui/accessibility.md) and the HTML guide.

**Missing `<label>` for form inputs.** Every `<input>`, `<select>`, and `<textarea>` must have an associated `<label>` via `htmlFor`/`id` or wrapping. Placeholder text is not a label. See [accessibility.md](../02-ui/accessibility.md).

**Unnecessary ARIA.** Prefer native semantics; redundant roles are usually unnecessary, not inherently harmful. role="list" can restore Safari/VoiceOver list semantics after list-style:none; verify the target combination before adding or removing it. See [accessibility.md](../02-ui/accessibility.md).

**`aria-hidden="true"` on focusable elements.** An element cannot be both hidden from assistive technology and focusable. This creates a trap where keyboard users can focus an element that screen readers cannot announce. See [accessibility.md](../02-ui/accessibility.md).

**`:focus { outline: none }` without a replacement.** Removing the focus indicator without providing a custom one makes the page unusable for keyboard navigation. Always provide a visible `:focus-visible` style. See [accessibility.md](../02-ui/accessibility.md) and the CSS guide.

**Relying only on automated a11y testing.** axe-core catches ~30-40% of accessibility issues. Manual testing with a screen reader and keyboard is required for real coverage. Automated tools cannot test logical reading order, sensible alt text content, or cognitive accessibility. See [accessibility.md](../02-ui/accessibility.md) and [testing.md](./testing.md).

## Performance Anti-Patterns

**JavaScript scroll listeners.** Attaching `scroll` event listeners for animations, parallax, or lazy loading wastes main-thread time. Use CSS scroll-driven animations for scroll-linked effects and `IntersectionObserver` for visibility-based logic. See [performance.md](./performance.md).

**WOFF font fallback.** In 2026, all modern browsers support WOFF2. Do not ship WOFF files. The size difference (30-50% larger) directly impacts page load. See [performance.md](./performance.md).

**Domain sharding.** Splitting assets across multiple domains was a HTTP/1.1 optimization that hurts performance under HTTP/2 and HTTP/3. Use a single origin with multiplexing. See [performance.md](./performance.md).

**Excessive preconnects.** Each `<link rel="preconnect">` consumes a TCP connection and browser budget. Preconnect only to domains used within the first 10 seconds of page load. Two to three is the maximum. See [performance.md](./performance.md).

**Ignoring INP.** Largest Contentful Paint (LCP) and Cumulative Layout Shift (CLS) are necessary but insufficient. Interaction to Next Paint (INP) is the current critical responsiveness metric. A page with good LCP but bad INP feels sluggish. See [performance.md](./performance.md).

**Large unsplit client bundles.** A single client bundle over 100 kB gzipped signals missing code splitting. Use dynamic imports for below-the-fold components, route-based splitting, and analyze with `@next/bundle-analyzer`. See [performance.md](./performance.md).

## Testing Anti-Patterns

**Testing implementation details.** Testing internal state, component methods, or which functions were called couples tests to refactoring. Test behavior: what the user sees and does, not how the component works internally. See [testing.md](./testing.md).

**Snapshot testing for UI components.** Snapshots break on every style change and become noise that developers accept without reading. Use targeted assertions on specific attributes and text content instead. See [testing.md](./testing.md).

**100% coverage targets.** Pursuing 100% coverage forces useless tests on trivial code while draining engineering time. Target coverage on business logic, edge cases, and error paths. See [testing.md](./testing.md).

**`jest.mock()` for API calls.** Mocking at the module level creates fragile tests that do not verify real network behavior. Use MSW (Mock Service Worker) to intercept at the network level. See [testing.md](./testing.md).

**Testing trivial prop forwarding.** A component that only renders its props does not need a test. Test the behavior that depends on those props, not the forwarding itself. See [testing.md](./testing.md).

## AI-Generated Code Anti-Patterns

**Generic AI UI patterns.** Be alert for overly centered layouts with excessive vertical spacing, generic gradient backgrounds, and uniform card grids with no information hierarchy. These patterns signal AI-generated filler. Apply deliberate visual hierarchy and the project's design system instead. See [design-tokens.md](../02-ui/design-tokens.md) and the UI/UX guide.

**Hallucinated npm packages.** AI agents sometimes reference packages that do not exist or have different APIs than described. Always verify package existence and version before installing. See [dependencies.md](./dependencies.md).

**Repetitive card layouts.** A page of identical cards with icon + title + description is the default AI layout for any content. Break the pattern: use varied layouts, prioritized content sizing, and progressive disclosure. See [design-tokens.md](../02-ui/design-tokens.md).

**Overly complex solutions.** AI tends to add unnecessary abstraction layers, custom hooks for single-use logic, and config-driven architectures for three options. Simplify. See [decision-framework.md](../00-navigation/decision-framework.md) for when abstractions are warranted.

**Missing error, loading, and empty states.** AI-generated code often shows only the happy path. Every data-fetching component must handle loading, error, and empty states. See [resilience.md](./resilience.md).