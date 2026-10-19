## Route Organization

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

Use `src/app/` as the filesystem root. Group routes with `(name)/` route groups — they create logical folders without affecting the URL. Use route groups for layout-scoped concerns like `(dashboard)/` or `(auth)/` rather than nesting shared logic in middleware.

```
src/app/
  (auth)/
    login/page.tsx
    register/page.tsx
    layout.tsx          # auth-specific layout, no nav
  (dashboard)/
    settings/page.tsx
    team/page.tsx
    layout.tsx          # dashboard layout with sidebar
  layout.tsx            # root layout (html, body, providers)
```

**Decision:** Prefer route groups over deeply nested folders. If a segment only exists to share a layout, it should be a route group. Avoid route groups with a single child — the layout can live in that child's directory instead.

## Route Files and Their Roles

Each route directory can contain these special files:

- **`page.tsx`** — The route's UI. Required for the route to be accessible. Can be async.
- **`layout.tsx`** — Shared UI that persists across navigations within its subtree. Never unmounts during client-side navigation (see [state-and-data](../03-data/state-and-data.md) for state persistence implications).
- **`loading.tsx`** — Automatically wraps `page.tsx` in a Suspense boundary. Shows during navigation and initial load. Do not fetch data here; it's purely a spinner/skeleton.
- **`error.tsx`** — Must be a Client Component. Catches rendering errors from `page.tsx` and below, not from `layout.tsx`.
- **`not-found.tsx`** — Triggered by `notFound()` from `next/navigation`. Unlike `error.tsx`, this renders inside the closest layout.
- **`default.tsx`** — Fallback for parallel routes when no slot matches.

## Async Params in Next.js 16

`params` and `searchParams` are now Promises. Destructure them with `await` in Server Components:

```tsx
export default async function ProjectPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // ...
}
```

This change supports partial rendering — the route can stream while params are resolving. Do not access `params` without awaiting in async components.

## Parallel Routes

Named slots (`@team`, `@analytics`) render independently in the same layout. Each slot gets its own loading, error, and streaming behavior:

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div>
      {children}
      <div className="grid grid-cols-2">
        {team}
        {analytics}
      </div>
    </div>
  );
}
```

**Decision:** Use parallel routes when sections have independent data loading timelines. If sections share data dependencies, a single `page.tsx` with compositional components is simpler. Always provide a `@slot/default.tsx` to avoid unhandled undefined slots.

## Intercepting Routes

Use the `(..)` convention to intercept navigations and render modals instead of full page transitions:

```
app/
  (dashboard)/
    layout.tsx              # must render both {children} and {modal}
    @modal/
      default.tsx           # renders null when no modal is active
      (.)team/
        [id]/page.tsx       # intercepts same-level navigation to /team/[id]
    team/
      [id]/page.tsx         # the full page, used on direct load / refresh
```

**Getting the pieces right matters — each one is a silent failure if omitted:**

- The `@modal` slot is a **sibling** of the route it intercepts, both under the layout that
  renders them. An earlier revision of this file drew two `team/` directories at the same
  level, which is not a valid tree.
- The layout must destructure and render the slot:
  `function Layout({ children, modal }) { return <>{children}{modal}</>; }`
- `@modal/default.tsx` returning `null` is **required** — without it, a hard navigation to
  an unrelated route 404s the slot.
- **Match the marker to the distance:** `(.)` same level, `(..)` one level up,
  `(..)(..)` two levels, `(...)` from the app root. The markers count *route segments*, and
  route groups like `(dashboard)` do not count.

**Decision:** Only use intercepting routes for modal patterns where the user might also deep-link to the full page. The modal intercepts same-app navigation; a direct URL load renders the full page. Close the modal with `router.back()`, not custom state.

## URL State

Use `useSearchParams` for reading and writing search params in Client Components. For complex state (filters, multi-step wizards), use [`nuqs`](https://nuqs.dev/) which provides typed, serializable URL state with debouncing.

```tsx
// nuqs example
import { useQueryStates, parseAsStringLiteral, parseAsInteger } from 'nuqs';

const STATUSES = ['active', 'archived'] as const;

const [filters, setFilters] = useQueryStates({
  status: parseAsStringLiteral(STATUSES).withDefault('active'),
  page: parseAsInteger.withDefault(1),
});
```

> **There is no `parseAsEnum` in nuqs.** Verified against the published package (v2.10.1): the
> exported parsers are `parseAsString`, `parseAsStringLiteral`, `parseAsStringEnum`,
> `parseAsInteger`, `parseAsFloat`, `parseAsHex`, `parseAsIndex`, `parseAsBoolean`,
> `parseAsIsoDate`, `parseAsIsoDateTime`, `parseAsTimestamp`, `parseAsNumberLiteral`,
> `parseAsJson`, `parseAsArrayOf`, `parseAsNativeArrayOf`. For a union of string literals
> use **`parseAsStringLiteral`** with an `as const` array — that is what preserves the
> literal type. `parseAsStringEnum` is for TypeScript `enum` members.

**Decision:** Every meaningful, shareable state belongs in the URL. If a user would want to bookmark it or send it to a colleague, it goes in search params. Transient state (draft form input, dismissed banners) stays in component state or [state-and-data](../03-data/state-and-data.md).

## Navigation

Use `<Link>` from `next/link` for all client-side navigation. Use `router.push()` and `router.replace()` from `next/navigation` for programmatic navigation. Prefer `replace` for state mutations that shouldn't create history entries (e.g., deleting a search param after applying it).

## Loading and Streaming

`loading.tsx` provides automatic Suspense boundaries. Server Components can `await` data fetches directly, and Next.js streams the HTML. Combine with `loading.tsx` for progressive loading:

```tsx
// app/dashboard/page.tsx
export default async function DashboardPage() {
  // Independent fetches must start together, or the second waits for the first.
  const [user, stats] = await Promise.all([getUser(), getStats()]);
  return <Dashboard user={user} stats={stats} />;
}
```

> ⚠️ **Sequential `await`s are a request waterfall.** Writing
> `const user = await getUser(); const stats = await getStats();` makes the page take
> `getUser + getStats` instead of `max(getUser, getStats)` — the exact anti-pattern
> [state-and-data](../03-data/state-and-data.md) warns about for `useEffect`, just moved to the
> server. `loading.tsx` hides the delay from you; it does not remove it.
>
> Only chain `await`s when the second fetch genuinely needs the first one's result.
>
> **Better still for slow sections:** don't block the page at all. Give each slow region its own
> Server Component inside `<Suspense>` so the fast part of the page streams immediately —
> see [resilience](../04-quality/resilience.md).

## Error Boundaries

`error.tsx` catches rendering errors below its layout. It does not catch errors in the layout itself or in sibling parallel route slots — each slot needs its own `error.tsx`. For root layout failures, use `global-error.tsx` (which must include its own `<html>` and `<body>` tags since the root layout is broken).

## Protected Routes

Check authentication in Server Components or `layout.tsx`. Redirect unauthenticated users with `redirect()` from `next/navigation`. `getSession()` is a `lib/session.ts` helper — not a Supabase call. It calls external API through the same `apiFetch` used everywhere else (see [backend-integration](../03-data/backend-integration.md)); Next.js still never talks to Supabase directly, even to check who is logged in:

```tsx
// lib/session.ts
import 'server-only';
import { cache } from 'react';
import { z } from 'zod';
import { apiFetch, ApiError } from '@/lib/http';
const UserEnvelope = z.object({ data: z.object({
  id: z.uuid(), email: z.email(), role: z.enum(['admin', 'member']),
}) }); // match the generated external API /auth/me schema

// cache(): the layout and the page can both call this; external API is hit once per request.
export const getSession = cache(async () => {
  try {
    return UserEnvelope.parse(await apiFetch('/auth/me')).data; // external API validates identity
  } catch (err) {
    if (err instanceof ApiError && err.status === 401) return null;
    throw err;   // external API down ≠ logged out — show error.tsx, don't bounce everyone to /login
  }
});
```

```tsx
import { redirect } from 'next/navigation';
import { getSession } from '@/lib/session';

export default async function SettingsLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await getSession();
  if (!session) redirect('/login');
  return <>{children}</>;
}
```

**Decision:** Always protect at the Server Component level. Never use client-side route guards that flash unauthenticated content before redirecting. The server redirect happens before any HTML is sent, so there is no flash. For role-based access, check in the layout and redirect to a 403 page or show a restricted view. See [backend-integration](../03-data/backend-integration.md) for auth token handling.
