## Test Runner: Vitest

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

Vitest is the default choice for new projects. It shares your Vite config, supports native ESM, and delivers a significantly faster cold start than Jest. Configuration is minimal: this project's `vitest.config.mts` is in [initialization](../01-foundation/initialization.md) (environment, alias, setup file; no globals). Jest 30 exists but sees mostly maintenance; only choose it for existing large Jest test suites
where migration cost is prohibitive. **Enzyme is dead** — it never supported React 18+, has no
React 19 adapter, and any code still using it should migrate to React Testing Library.

## Component Testing: React Testing Library

Test behavior, not implementation. Query elements by role, text, or label — never by test ID, CSS class, or component state. If a query requires a `testId`, question whether the element is actually user-visible.

Never test internal state (e.g., `useState` values, reducer actions). Test what the user sees and interacts with. If you cannot assert behavior through the DOM, the component may have a design issue — extract the testable logic into a pure function and unit test that separately.

Avoid testing library internals (wrapper components, prop drilling paths, hook mechanics). See [components.md](../02-ui/components.md) for component design patterns that facilitate testing.

## E2E Testing: Playwright

**In this project E2E is the user's:** the user checks every finished feature end to end, by hand,
in the browser ([delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md)). No worker writes E2E tests, and setup does not
install Playwright. The rest of this section applies only if the user asks for automated E2E.

Playwright is the clear winner over Cypress in 2026. It provides multi-browser coverage (Chromium, Firefox, WebKit), auto-waiting (no manual `waitFor` calls), a built-in trace viewer for debugging failures, and native visual comparisons.

Structure E2E tests around critical user flows, not page coverage. A checkout flow, an authentication flow, and a data entry form are worth testing end-to-end. A static marketing page is not.

Use Playwright's page object pattern for reusable page interactions. Keep tests isolated — each test should set up its own state via API calls, not depend on prior test execution order.

## Accessibility Testing

Integrate the current compatible stable `axe-core` release in two places (verify versions when initializing a real app):

1. **Component tests** — assert on rendered output, and put the check in each tester's own
   render helper (in its folder) so it runs by default.
2. **E2E tests** — only if the user asks for automated E2E: a compatible stable
   `@axe-core/playwright` release to audit full pages during critical flows.

**On the component-test matcher — check the publish date before you install.** `vitest-axe`
is the obvious-sounding choice and its last release was **0.1.0, October 2022**: nearly four
years stale, which fails this guide's own dependency policy ("no release in 12+ months" —
[dependencies.md](./dependencies.md)). Recommending it here contradicted that rule.

Use `jest-axe` instead — the name is historical, it works with Vitest, and it is actively
maintained (11.0.0, July 2026):

```ts
// in a tester's own helper, in its folder (jest-axe: an approved package decision in Planner Output/software TODO)
import { expect } from 'vitest';
import { toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

// in a test
import { axe } from 'jest-axe';
const { container } = render(<Button>Save</Button>);
expect(await axe(container)).toHaveNoViolations();
```

Or call `axe-core` directly and assert on `results.violations` — one dependency fewer, and
it is the package both wrappers are wrapping anyway.

Automated tools catch a minority of accessibility issues — the commonly cited range is
30–40%, and Deque's own testing of axe-core claims a higher figure for the subset it targets.
Treat any such number as an order of magnitude, not a measurement: what matters is the shape of
what automation *cannot* see. Screen reader testing (VoiceOver, NVDA) and keyboard-only navigation testing are mandatory for any project with accessibility requirements. Automated checks are a floor, not a ceiling.

## API Mocking: MSW

Use Mock Service Worker v2 for all API mocking. MSW intercepts at the network level, meaning the same mock handlers work across Vitest, Playwright, and Storybook without modification. Organize handlers around your real API surface — one handler file per resource or endpoint group.

Using `jest.mock()` or `vi.mock()` to intercept API call functions is a code smell. It couples tests to implementation details (which function is called) rather than behavior (what data is displayed). MSW decouples tests from the data-fetching layer entirely.

For MSW handler organization, each tester creates a `mocks/` directory inside its own folder, with handlers mirroring your API routes. Use request URL matching, not import path interception. This keeps mocks stable when internal code organization changes.

## Visual Regression Testing

Only if the user asks for automated E2E (E2E is the user's, by hand): start with Playwright's built-in `toHaveScreenshot()`. It handles screenshot capture, pixel comparison, and diff output natively. For most teams, this is sufficient.

Add Chromatic (Storybook-native) when you need review workflows — visual diffs posted as pull request comments with before/after comparisons. This adds process overhead but catches unintended visual changes that unit tests miss.

Do not start with a visual regression suite covering every component. Begin with the most visually critical components (brand elements, complex layouts, data-dense tables) and expand based on where regressions actually occur.

## Server Component Testing

Vitest does not support React Server Components. The user's E2E check, by hand, is the primary check of RSC rendering, data fetching, and streaming behavior: the full server-rendered output as the user experiences it.

For RSCs with complex data transformations or conditional rendering logic, extract the testable logic into separate pure functions and unit test those with Vitest. The component itself becomes a thin rendering layer, checked in the user's E2E.

## Test Organization

- **Unit and component tests belong to the assigned Tester operator(s), one authorized folder each:** `tests/<assigned-tester>/` inside the relevant app. Roster and the brief identify actual operators, count and write scopes; examples do not appoint clients or require two testers. Each folder mirrors the source path, such as `tests/<assigned-tester>/features/auth/actions.test.ts`.
- **Every test's name starts with the acceptance check it proves:** `AC-3: rejects a wrong password`. Compare actual evidence AC by AC; two viewpoints from one operator are not independent suites.
- **Nobody else writes in a Tester's folder.** Implementer runs existing suites through `npm run check`, fixes product code and reports test defects without editing assertions/fixtures. Orchestrator reviews the initial Tester report individually, routes corrections and personally executes existing correction/regression checks; fixes do not automatically return to Tester.
- **E2E is the owner's, by hand** (§ E2E Testing above): no `e2e/` folder unless explicitly requested.
- **Test helpers live in each assigned Tester's own folder** — render helpers, fixtures, factories and mock handlers; operators do not overwrite/share mutable test records. `src/test-utils/` holds only the runner's setup file. Multiple suites remain separately owned when actually assigned.


## Coverage Policy

Do not enforce percentage-based project-wide coverage gates. They incentivize useless tests (testing constants, prop forwarding, CSS classes) and penalize honest work on untestable legacy code.

Instead, use **diff coverage**: new or modified code should have meaningful test coverage. Most CI platforms support this. A PR that adds business logic without tests should fail review, but a PR that modifies a config file should not.

## What to Test

- Business logic and data transformations (pure functions)
- State management behavior (what happens when actions are dispatched)
- Critical user flows (authentication, checkout, data entry)
- Form validation rules and error states
- Edge cases in conditional rendering
- Error boundary behavior

## What NOT to Test

- **Hook internals** — test the component behavior that hooks produce, not the hook mechanics
- **Snapshot testing for UI components** — brittle, low signal, high noise. Snapshots for serializable data (ASTs, config objects) are acceptable
- **100% coverage targets** — always wrong. 80% with meaningful tests beats 95% with test pollution
- **Third-party library behavior** — trust that React, your router, and your state library work. Test your code, not theirs
- **Trivial prop forwarding** — `return <Child {...props} />` needs no test
- **CSS class names** — they are implementation details subject to frequent change
