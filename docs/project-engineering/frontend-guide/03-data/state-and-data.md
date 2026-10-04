Every piece of state has exactly one owner. Getting this wrong causes the most persistent bugs in frontend applications: stale data, unnecessary re-renders, state that disappears on navigation. This file defines the five state categories, their owners, and when to use each. This is the core decision framework for data flow — refer back to it every time you add state.

## State Categories

### 1. Server/Remote State

**Owner:** Server Components for reads. TanStack Query v5 for interactive client-side cases. Server Actions for mutations.

The default path for reading data is Server Components fetching during render. The fetch adds no client JavaScript. Loading/error UI and cache/revalidation decisions still matter:

```tsx
// app/projects/page.tsx
import { getProjects } from '@/features/projects/queries';
import { ProjectList } from '@/features/projects/project-list';

export default async function ProjectsPage() {
  const projects = await getProjects();
  return <ProjectList projects={projects} />;
}
```

Use TanStack Query v5 when you need client-side interactivity that Server Components cannot provide: real-time polling, optimistic updates, background refetching after a mutation, or cache invalidation on window focus. Prefetch on the server with `queryClient.prefetchQuery()` and hydrate via `HydrationBoundary`:

```tsx
// app/projects/page.tsx
import { HydrationBoundary, dehydrate } from '@tanstack/react-query';
import { getQueryClient } from '@/lib/query-client';
import { getProjects } from '@/features/projects/queries';
import { ProjectList } from '@/features/projects/project-list';

export default async function ProjectsPage() {
  const queryClient = getQueryClient();
  await queryClient.prefetchQuery({ queryKey: ['projects'], queryFn: () => getProjects() });
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <ProjectList />
    </HydrationBoundary>
  );
}
```

For mutations, use Server Actions. Do not write `fetch` calls in client components for mutations — Server Actions give you progressive enhancement, automatic CSRF protection, and typed input/output. The action itself never touches a database; it calls Express through `apiFetch` (see [backend-integration](./backend-integration.md) for why Next.js is never fullstack):

```tsx
// features/projects/actions.ts
'use server';
import { apiFetch, ApiError } from '@/lib/http';
import { revalidatePath } from 'next/cache';

export async function deleteProject(id: string) {
  try {
    await apiFetch(`/projects/${id}`, { method: 'DELETE' });  // Express enforces ownership/role
  } catch (err) {
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message };   // a refusal the user can act on: shown inline
    }
    throw err;                           // 401/403, an outage or a bug → error.tsx
  }
  revalidatePath('/projects');
  return { success: true };
}
```

### 2. URL State

**Owner:** The URL bar.

Use `useSearchParams` (simple cases) or `nuqs` (complex cases with type parsing, defaults, and history management) for filters, pagination, active tabs, search queries — anything the user might bookmark, share, or navigate back to.

```tsx
// Simple: useSearchParams
function ProjectFilters() {
  const searchParams = useSearchParams();
  const status = searchParams.get('status') ?? 'active';
}

// Complex: nuqs for type-safe URL state — prefer the built-in parsers
import { useQueryState, parseAsStringLiteral } from 'nuqs';

const STATUSES = ['active', 'archived'] as const;

function ProjectFilters() {
  const [status, setStatus] = useQueryState(
    'status',
    parseAsStringLiteral(STATUSES).withDefault('active'),
  );
}
```

URL state replaces `useState` for any value that should survive a page refresh. If you catch yourself thinking "I'll add a share button later," the state should already be in the URL.

### 3. Local/Component State

**Owner:** The component that renders it.

`useState` and `useReducer` for UI state that doesn't need to be shared: modal open/closed, a local form draft, a tooltip visible state. This state dies when the component unmounts, and that's correct behavior.

Do not put server data in local state. A component should not `useState` a list of projects it fetched — the server or TanStack Query owns that data.

### 4. Shared Client State

**Owner:** A Zustand store, Jotai atoms, or `useSyncExternalStore`.

For state shared across unrelated components that don't share a parent: theme preference, locale, sidebar collapsed state, notification count. The tool choices:

- **Zustand v5:** Simplest API. One function call to create a store, no providers needed. Default choice.
- **Jotai:** Atomic state model. Best when state is highly granular and composed from many independent pieces.
- **`useSyncExternalStore`:** For subscribing to non-React sources (broadcast channels, WebSockets). Don't use this for app state — use Zustand.

**Context is only for low-frequency state.** Theme, auth session, locale — these change rarely.
Never put frequently updating state (mouse position, typing input, animation frames) in Context:
it re-renders every consumer on every change. See [architecture](../01-foundation/architecture.md) for
the dependency direction rules.

**Theme is the exception:** never keep it in a client store. It must be known before first
paint, so it is owned by `next-themes`. See [theme-and-dark-mode](../02-ui/theme-and-dark-mode.md).

### 5. Form State

**Owner:** React 19 `useActionState` + Server Actions.

Do not reach for dedicated form libraries (React Hook Form, Formik). React 19's built-in form handling with `useActionState` covers validation, pending states, and error display:

```tsx
'use client';
import { useActionState } from 'react';
import { createProject } from './actions';

function CreateProjectForm() {
  const [state, formAction, isPending] = useActionState(createProject, null);
  return (
    <form action={formAction}>
      {state?.message && <p role="alert">{state.message}</p>}
      <input name="name" defaultValue={state?.values?.name} />
      {state?.errors?.name && <p>{state.errors.name}</p>}
      <button disabled={isPending}>Create</button>
    </form>
  );
}
```

Use React Hook Form only when you need complex field-level validation (cross-field rules, async validation per keystroke) or performance with very large forms (100+ fields). For most forms, `useActionState` is sufficient.

## Data Flow

Unidirectional. Parent passes data down via props or URL. Child sends events up via callbacks or Server Actions. A child never reaches into a parent's state via ref, context as a backchannel, or direct mutation.

## Caching

Next.js 16 (with the `cacheComponents` flag) is dynamic by default: nothing is cached unless you opt in with `'use cache'` on a function or component. This is explicit and auditable — you know exactly what's cached and what isn't:

```tsx
// features/catalog/queries.ts
async function getPublicCategories() {
  'use cache';
  return apiFetch('/categories', { auth: false });   // caches the Express response
}
```

**Only cache public data this way.** A cached function cannot read `cookies()`, so it has no
session. That is why the call passes `auth: false`. It is also a safety feature: user-specific
data (a user's projects, their cart) must never go in a shared cache. Fetch it uncached on each
request instead.

Pair `'use cache'` with `cacheLife` profiles and `cacheTag` for revalidation control. Uncached functions run per-request like classic dynamic rendering.

## Derived State

Compute derived values during render. Never store them in state:

```tsx
// Wrong: storing derived state
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);

// Right: compute during render
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);
```

If the computation is expensive, wrap it in `useMemo`. But profile first — most "expensive" computations on arrays under 10,000 items aren't.

## Decision Table

| You need... | Use | Don't use |
|---|---|---|
| Page data on load | Server Component fetch | `useEffect` + `fetch`, Zustand |
| Data that updates in real-time | TanStack Query | Server Components alone |
| Filter that should be shareable | URL state (`nuqs`) | `useState` |
| Modal open/close | `useState` in the component | Zustand, URL state |
| Theme across the app | `next-themes` (or a cookie read on the server) | `useState` in layout; a client-only store |
| Form submission | `useActionState` + Server Action | `useState` + manual `fetch` |
| Notification badge count | Server data — TanStack Query, or push into a store from one subscription | `useState` duplicated per component |

## What Not to Do

- **Global state for everything.** Zustand is not a replacement for props. If a parent can pass data to a child, it should.
- **Context for high-frequency state.** Every context value change re-renders every consumer. Mouse position, typing, and animation frames do not belong in Context.
- **`useEffect` for data fetching.** This causes waterfalls (render → fetch → re-render). Fetch in Server Components or TanStack Query.
- **Storing server data in client state.** If the data comes from the server, the server or TanStack Query owns it. Copying it into `useState` creates a second source of truth that will go stale.
- **MSW in production code.** MSW (Mock Service Worker) is for tests and development only. Never import it in application code. See [testing](../04-quality/testing.md).