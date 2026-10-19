Every user-facing operation that can fail must have three states: loading, success, and error. There are no exceptions. A blank screen, a frozen UI, or a silent failure is a bug, not a missing feature. This file covers the patterns for handling each state and recovering from failures gracefully.

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

## Loading States

Route-level loading uses `loading.tsx`, which Next.js wraps in `<Suspense>` automatically. Place a skeleton that matches the page's layout structure so the user sees the shape of the content before it arrives. Use `Skeleton` from your component library (see [design-tokens.md](../02-ui/design-tokens.md)) — never a full-page spinner for route transitions.

For component-level data fetching within a page, use skeleton UI for content areas. Skeletons are preferable to spinners because they reduce perceived latency and prevent layout shift. Use a spinner only for indeterminate operations where the outcome shape is unknown (e.g., file uploads, AI generation tasks).

Never block the entire page for a single component's data. Independent sections stream independently: give each slow section its own Server Component wrapped in `<Suspense>` with a skeleton fallback, so a fast main column renders while a slow widget resolves. A parent `await` that serializes all sections is the wrong default — see [routing.md](../01-foundation/routing.md) for streaming and [state-and-data.md](../03-data/state-and-data.md) for where fetching lives.

## Error States

Route-level errors use `error.tsx`. This file catches any rendering error that occurs below the nearest `layout.tsx`. It **must** be a Client Component (it uses `useEffect` to log errors). Provide a clear message, a retry button that calls `reset()`, and a link back to a safe page. Do not show raw error details to users — log them server-side.

Component-level errors split by rendering context:

- **Server Components**: Catch recognized dependency/domain errors when a defined degraded UI is appropriate. Let unexpected failures reach error.tsx. Do not swallow redirect/notFound control-flow exceptions in a blanket catch; perform redirects outside the data-fetch catch.
- **Client Components**: Wrap unreliable sections in React error boundaries. Create a reusable `<ErrorBoundary>` component that accepts a `fallback` prop.

For mutation errors the user can act on (validation, a refusal), in form submissions and delete actions alike, the Server Action returns them and the UI shows an inline error message near the action; login 401 is expected form feedback. Session expiry/403 uses the agreed authentication/permission flow; operational or unexpected failures use `error.tsx` ([backend-integration](../03-data/backend-integration.md) § Mutations and Error States). Toast notifications are acceptable for secondary feedback but should not be the only error indicator.

## Empty States

When data exists but has no items (e.g., an empty search result, a new dashboard with no data), show a helpful message with a primary action. An empty state is not an error — it's a valid state. "No projects yet. Create your first project." is correct. A blank white panel is not.

## Retry Logic

TanStack Query retries reads three times by default on the client (zero on the server). Override retry to reject 400/401/403/404 and retry only transient failures within the total budget; respect Retry-After for 429. For Server Components that throw, the user can refresh the page. For critical operations like payments or destructive mutations, offer an explicit retry button rather than auto-retrying silently.

Custom retry behavior: adjust `retry` and `retryDelay` in the query config. For mutations, always set `retry: false` — mutations should fail fast and let the user decide.

## Network Failure

Detect offline status with `navigator.onLine` and the `online`/`offline` window events. Show a persistent but non-blocking indicator (a thin banner at the top or a status dot in the nav) when the user is offline.

For mutations while offline, show "you are offline, retry when connected." Never queue,
resume or replay writes automatically when connectivity returns. Guard submission while
offline and use mutation `networkMode: 'always'` with `retry: false` so TanStack Query does
not pause a write for later automatic execution; the request must fail through the normal
error path. Do not restore persisted mutations or call `resumePausedMutations`.
After a timeout, reconcile server state before an explicit user retry because the original
write may already have succeeded. See [backend-integration.md](../03-data/backend-integration.md).


Source: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode

## Optimistic Update Failure

React useOptimistic derives its optimistic view from authoritative state. TanStack Query does NOT automatically restore a cache value you wrote in onMutate: snapshot the previous value, restore it in onError, and invalidate/reconcile in onSettled. However, the user needs to know what happened. Show a toast notification: "Failed to update. Your change was reverted." Include a retry button in the toast if the action is retriable.

## Partial Failures

When part of a page fails but the rest works, the page should still be useful. Wrap independent sections in error boundaries so one failing component doesn't take down the entire page. For example, if a weather widget fails to load, the rest of the dashboard should render normally with the widget showing its error state.

## Graceful Degradation

Non-critical features should fail without affecting core functionality. If a recommendation engine is down, show default content. If real-time updates fail, fall back to polling or manual refresh. If an image CDN is slow, show placeholder images or alt text. The page must remain functional and useful even when optional features are unavailable.

## Timeouts

TanStack Query has **no built-in request timeout**. For reads, fetchProjects is the client helper from backend-integration.md: it fetches the same-origin Next.js GET route, which forwards to external API. Never pass AbortSignal through a Server Action. Use AbortSignal.timeout() on the browser fetch and a separate bounded server deadline:

```ts
useQuery({
  queryKey: ['projects'],
  // TanStack Query passes its own `signal` for cancellation (unmount, key change,
  // refetch). Combine the two rather than replacing one with the other.
  queryFn: ({ signal }) =>
    fetchProjects(AbortSignal.any([signal, AbortSignal.timeout(30_000)])),
});
```

> Passing only `AbortSignal.timeout()` throws away Query's cancellation signal: the request
> keeps running after the component unmounts or the query key changes.
> `AbortSignal.any([...])` aborts on whichever fires first, which is what you want.
> `AbortSignal.any` is widely available; if you must support an engine without it, wire an
> `AbortController` that listens to both.

Reasonable defaults:

- Standard data fetching: 30 seconds
- File uploads or AI operations: 60 seconds
- Real-time subscriptions: handled by the subscription layer, not query timeout

A request that exceeds its timeout should surface as an error state, not spin forever.

## Decision Summary

| Concern | Pattern |
|---|---|
| Route loading | `loading.tsx` with skeleton UI |
| Route error | `error.tsx` (Client Component) with `reset()` |
| Component data loading | Skeleton components |
| Component error | try/catch (Server) or error boundary (Client) |
| Empty data | Helpful message + primary action |
| Mutation error | The user can act on it: inline message near the action + optional toast · otherwise: `error.tsx` |
| Offline | Banner indicator + mutation queueing |
| Optimistic failure | TanStack: restore snapshot in onError; React useOptimistic: reconcile with base state; feedback + explicit retry |
| Partial failure | Error boundaries per section |
| Non-critical failure | Graceful degradation with fallback |
