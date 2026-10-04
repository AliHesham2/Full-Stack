# Error Handling: boundaries and hydration

## Error Boundaries

### Rule: Every route/feature area gets an error boundary
Error boundaries catch render/lifecycle errors in the subtree below them. React ships no component for this — use `react-error-boundary` (or a class implementing `getDerivedStateFromError`).

```tsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary
  fallbackRender={({ error, resetErrorBoundary }) => (
    <div role="alert">
      <p>Something broke: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  )}
  onError={(error) => logger.error(error)}
>
  <Dashboard />
</ErrorBoundary>
```

### What they do NOT catch
- Event handlers (try/catch there)
- Server Components and Server Functions (framework handles — e.g., Next.js `error.tsx`)
- Errors during `setTimeout` callbacks / async callbacks outside React's control

### Rule: Reset state on retry when appropriate
`resetErrorBoundary` remounts children. If the error came from bad local state, clear it via `onReset`.

## Hydration mismatches

### Rule: Render identical output on server and client
Top causes and fixes:

| Cause | Fix |
|---|---|
| `Date.now()` / `Math.random()` / `crypto.randomUUID()` in render | Generate in effects or on the server; pass down |
| Browser-only APIs (`window`, `localStorage`) in render | Read them in effects, or gate with a mounted flag |
| Locale/timezone-dependent formatting (`toLocaleString`) | Format with an explicit fixed locale/timezone, or format after mount |
| Invalid HTML nesting (e.g., `<div>` inside `<p>`) | Fix the markup — browsers auto-correct differently |
| Conditional rendering on `typeof window` | Use `useSyncExternalStore` or mounted flag |

### What React actually does with each mismatch

Measured on react-dom@19.2.8 (server render → `hydrateRoot`), watching both
`onRecoverableError` and `console.error`:

| Mismatch | Reported via | Winning value in the DOM |
|---|---|---|
| none | — | — |
| **text** content | `onRecoverableError` | **client** — subtree regenerated |
| **element structure** (extra/different child) | `onRecoverableError` | **client** — subtree regenerated |
| **attribute** (e.g. `className`) | `console.error` only | **server** — silently kept, never corrected |
| text under `suppressHydrationWarning` | nothing | **server** — kept on purpose |
| child *element* mismatch under `suppressHydrationWarning` | `onRecoverableError` | client |

Three consequences that change how you debug this:

1. **Hydration errors do not reach `console.error`.** They arrive at the root's
   `onRecoverableError`. Error monitoring that only patches `console` sees nothing:
   ```tsx
   hydrateRoot(el, <App />, {
     onRecoverableError: (error, info) => reportToSentry(error, info.componentStack),
   });
   ```
2. **Attribute mismatches are not repaired.** A `className` that differs between server and
   client keeps the *server* value forever — no visible error, wrong styling, and nothing in
   the recoverable-error stream. This is the hydration bug class that survives to production.
3. **`suppressHydrationWarning` does more than silence.** It tells React to keep the server
   value, so a suppressed timestamp shows the server's render until something else triggers
   a re-render. That is usually what you want for `<time>`; it is wrong for anything the
   user is meant to interact with.

```tsx
<time suppressHydrationWarning>{new Date(createdAt).toISOString()}</time>
```
It applies to **that element's own text and attributes only** — verified: a differing child
*element* under a suppressed parent still raises a recoverable error. Never a blanket fix.

**Debugging**: the recoverable-error message names the failure kind ("server rendered text
didn't match" vs "server rendered HTML didn't match"); in dev React also prints the diff.
