# Testing (2026)

## Stack (versions as of Sept 2026)
- **Unit/Integration**: Vitest 4 + `@testing-library/react` 16.3
- **E2E**: in this project the user's, by hand (frontend-guide/04-quality/testing.md § E2E
  Testing). Playwright only if the user asks for automated E2E
- **Visual**: Playwright screenshots or Chromatic, only with automated E2E

> RTL 16 requires `@testing-library/dom` as an explicit peer dependency — it is no longer
> bundled. Install both, or `render` fails to resolve.

## Vitest + RTL setup
```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect } from 'vitest';
import { Counter } from './Counter';

describe('Counter', () => {
  it('increments on click', async () => {
    const user = userEvent.setup();
    render(<Counter />);

    const button = screen.getByRole('button', { name: /increment/i });
    await user.click(button);

    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
});
```

## Philosophy
- Test what the user sees/does, not implementation details
- Query by **role/label/text** first; `data-testid` only when no accessible query works
- `<Activity mode="hidden">` subtrees stay in the DOM, but getByRole excludes hidden
  elements by default. Use { hidden: true } only to inspect hidden DOM; assert visibility
  separately. within(container) scopes the query but does not change hidden filtering
  → [rendering-strategies.md](./rendering-strategies.md)
- Prefer integration tests over isolated unit tests for components

```tsx
// ✓ RTL — accessible queries
await user.type(screen.getByLabelText('Email'), 'user@example.com');

// ✓ Playwright — same principle
await page.getByLabel('Email').fill('user@example.com');
await page.getByRole('button', { name: 'Log in' }).click();
await expect(page).toHaveURL('/dashboard');
```

## What to test in React apps
| Layer | Test |
|---|---|
| Pure logic/hooks | Unit tests (Vitest) |
| Components | Integration via user interactions (RTL) |
| Flows | The user's E2E check, by hand (Playwright only if the user asks) |
| Server Functions/Actions | Backend-style tests against the handler |

## Deprecated
- Enzyme: dead, no React 17+ support
- react-test-renderer: deprecated in React 19 → use `@testing-library/react`
- `react-shallow-renderer`: **do not adopt** — last published **April 2022** (16.15.0), no
  React 18/19 support. Shallow rendering tests implementation details anyway; if you reach
  for it, the component under test usually needs decomposition instead.
- Jest for new React web projects → use Vitest; Jest remains standard for React Native

→ Sources: [vitest.dev](https://vitest.dev/guide/), [testing-library.com](https://testing-library.com/docs/react-testing-library/intro/), [playwright.dev](https://playwright.dev/)
