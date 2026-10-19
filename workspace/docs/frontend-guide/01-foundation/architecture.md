Architecture is the set of decisions that make the codebase predictable to navigate and safe to modify. Every architectural decision should be made at the point it's needed, not predicted. This guide defines the default patterns and the decision framework for when to deviate from them.

## Server-First Rendering

Every component under `app/` is a Server Component by default. Do not add `'use client'` to a file unless it contains at least one of: an event handler (`onClick`, `onChange`), a `useState`/`useReducer` call, a `useEffect` call, or a browser API (`window`, `document`, `localStorage`).

Data fetching belongs in Server Components. A page component fetches data and passes it as props to children:

```tsx
// app/dashboard/page.tsx — Server Component, no directive needed
import { DashboardShell } from '@/features/dashboard/shell';
import { getProjects } from '@/features/dashboard/queries';

export default async function DashboardPage() {
  const projects = await getProjects();
  return <DashboardShell projects={projects} />;
}
```

The child component is a Client Component only if it's interactive. If `DashboardShell` just renders a static layout with the projects prop, it stays a Server Component. This is the server component boundary: the line where data (server) meets interactivity (client). Design the boundary to be as deep as possible—push `'use client'` down to the leaf component that actually needs it.

## Dependency Direction

Imports may only point *down* this layering:

```
app/ (routes)                    — may import: features/, components/ui/, lib/, types/, actions/
features/                        — may import: components/ui/, lib/, types/, hooks/, actions/
components/ui/                   — may import: lib/cn.ts, types/ (never lib/http.ts or lib/session.ts — see project-structure)
hooks/                           — may import: lib/, types/
actions/                         — may import: lib/, types/
lib/                             — shared utilities/adapters: types/, platform/library APIs; no upward imports
types/                           — imports nothing project-internal
```

- **`components/ui/`**: Presentational components with zero feature logic. No imports from `features/`.
- **`features/`**: Feature modules that compose `ui/` components and call `lib/` utilities. No imports from other features directly—use shared types from `types/` or shared fetch functions in `lib/`.
- **`lib/`, `hooks/`, `actions/`, `types/`**: Shared layers. Nothing in them may import from `features/` or `components/ui/`.

> **Why `app/` may not import `hooks/`.** Not a style preference — a hook implies client
> state, and files under `app/` are Server Components. A route file reaching for a hook is
> almost always a `'use client'` boundary in the wrong place: push the interactivity into a
> feature component and let *that* use the hook. Keep page files thin
> ([project-structure](./project-structure.md)).

A file may never import from a layer above its own, and no rule here is satisfied by an import that skips layers downward when a nearer layer owns the concern. A feature importing from another feature is a circular dependency risk. If `features/dashboard` needs data from `features/projects`, the shared types live in `types/` and server-only query functions in `lib/projects.ts` (shared mutations in `actions/`), and both features import from there. See [project-structure](./project-structure.md) for folder layout.

## Component Architecture

### Composition Over Inheritance

Never use class inheritance for components or hooks. Use composition: pass components as props (the children pattern), use render props, or extract logic into hooks.

### Hooks-First Logic

Any reusable logic that involves React state, effects, or context goes in a hook. Name hooks by the consumer's goal, not the mechanism inside: `useSearchQuery` describes what the caller gets; `useDebounceEffect` describes an implementation detail. The exception: tiny generic utilities whose behavior *is* their name (`useMediaQuery`, `useLocalStorage`) are fine. Hooks that don't use React APIs (no state, no effects) should be plain functions in `lib/`.

## Feature Organization

A feature folder contains everything needed for one product capability: the page route, components, hooks, server actions, and types specific to that feature.

```
features/dashboard/
  page.tsx
  dashboard-shell.tsx
  project-card.tsx
  actions.ts
  use-project-filters.ts
```

Feature UI becomes reusable after a third real call site needs the same cohesive behavior; design-system primitives are reusable from the start. Shared actions, queries, hooks and types move on a second feature consumer to avoid sibling-feature imports. UI abstraction and shared-layer placement are different rules.

## State Ownership

Every piece of state has a single owner. Match the state to its source of truth:

| State Type | Owner | Tool | Example |
|---|---|---|---|
| Local UI state | The component that renders it | `useState` | Modal open/close, selected tab |
| Server data | The server / cache layer | RSC props or TanStack Query | User profile, project list |
| URL state | The URL | `searchParams` in RSC, `useSearchParams` in client | Filters, pagination, active tab |
| Shared client state | A Zustand/Jotai store | Zustand or Jotai | Theme preference, sidebar collapsed, locale |
| Form state | The form action | React 19 `useActionState` or server actions | Login form, settings edit |

**Decision rule:** If the data exists on the server, fetch it in a Server Component and pass it down. Only reach for TanStack Query when you need client-side cache invalidation, polling, or optimistic updates. Do not use Zustand for data that lives on the server—that's what Server Components and server actions are for.

URL state is underused. If toggling a filter should update the page content and be shareable via URL, it belongs in `searchParams`, not in a `useState` call that vanishes on refresh.

## Data Flow

Data flows down, events flow up. A child component receives data via props and communicates back via callbacks or server actions. Never reach into a parent's state from a child (no ref-based state access, no context as a backchannel).

Server actions replace most `fetch` calls from client components:

```tsx
// features/dashboard/actions.ts
'use server';
import { z } from 'zod';
import { apiFetch, ApiError } from '@/lib/http';
import { revalidatePath } from 'next/cache';

const DeleteInput = z.object({ id: z.string().uuid() });

export async function deleteProject(raw: unknown) {
  // 1. Validate the input. It arrives from the network, not from your component.
  const { id } = DeleteInput.parse(raw);

  // 2. Forward to Express. Express is what authenticates the session, authorizes
  //    the specific row, and touches the database — Next.js does none of that
  //    itself (see 03-data/backend-integration.md). A refusal the user can act on
  //    comes back as state; a 401/403, an outage or a bug is rethrown to error.tsx.
  try {
    await apiFetch(`/projects/${id}`, { method: 'DELETE' });
  } catch (err) {
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message };
    }
    throw err;
  }

  revalidatePath('/dashboard');
  return { success: true };
}

// features/dashboard/project-card.tsx
'use client';
import { useState, useTransition } from 'react';
import { deleteProject } from './actions';

export function ProjectCard({ project }) {
  const [isPending, startTransition] = useTransition();
  const [message, setMessage] = useState<string | null>(null);
  return (
    <>
      <button
        disabled={isPending}
        onClick={() => startTransition(async () => {
          const result = await deleteProject({ id: project.id });
          setMessage(result.message ?? null);   // a refusal: shown inline, near the action
        })}
      >
        {isPending ? 'Deleting…' : 'Delete'}
      </button>
      {message && <p role="alert">{message}</p>}
    </>
  );
}
```

> ⚠️ **A Server Action is a public HTTP endpoint, not a private function call.** Next.js
> compiles it to a POST route with a generated id. Anyone can invoke it with any arguments —
> the fact that your UI only renders the button for the right user is irrelevant. Every action
> must independently **authenticate, validate its input, and authorize the specific resource**.
> This is Key Rule 5 in [agent-workflow](../00-navigation/agent-workflow.md) applied concretely, and
> [security](../04-quality/security.md) is the full treatment.
>
> Scope the mutation to the caller (`where: { id, ownerId: session.user.id }`) rather than
> checking ownership in a separate query — a check-then-act pair is a race condition.
>
> **On the client side:** calling an action bare from `onClick` gives you no pending state and
> no error handling, and the UI will not reflect the result. Wrap it in `useTransition` (as
> above) or drive it from a `<form action={…}>` with `useActionState`
> ([forms](../02-ui/forms.md)).

## Routing as Architecture

Routes are not just URL mappings—they define the feature boundaries. Each route segment in `app/` maps to a feature. Nested layouts create shared UI shells without prop drilling:

```
app/
  (auth)/
    login/page.tsx
    register/page.tsx
  (dashboard)/
    layout.tsx          # Sidebar + header shell
    projects/page.tsx
    settings/page.tsx
```

Route groups `(auth)` and `(dashboard)` don't affect the URL but create layout boundaries. This is where you place shared providers, data loading, and error boundaries.

## When to Abstract

The rule is **extend before abstracting**. When you see duplicated code, first ask: can I extend the existing implementation to cover both cases? Add a parameter, a prop, or a conditional branch. Only extract an abstraction when the third use case appears and the duplication is structural, not coincidental.

**Create an abstraction when:**
- Three or more call sites share identical logic with identical shape
- The abstraction has a clear, singular responsibility
- Removing the abstraction would require changing three files in the same way

**Keep code inline when:**
- Two call sites share similar but not identical logic
- The "abstraction" would need configuration flags to distinguish its behaviors
- The extraction would make the code harder to follow for someone seeing it for the first time

A `Button` component is worth extracting on day one because every page uses it. A `useFormValidation` hook is not worth extracting until you have two forms with validation that share the same rules and error-display pattern.

## Module Boundaries and Cohesion

A module (feature folder or `lib/` file) should have high cohesion: everything inside it relates to one concern. A sign of low cohesion is a file that imports from many unrelated modules. A sign of a missing module boundary is code in a feature folder that's used by three other features—move it to `lib/`.

## How Architecture Evolves

Architecture is not designed upfront—it emerges from refactoring. Start with a flat structure. When a file exceeds 200 lines, review cohesion; split distinct responsibilities, not a line count. When a feature folder exceeds 10 files, consider sub-folders. When two features share queries/actions/hooks/types, extract the shared contract; feature UI waits for a third matching call site. When a `lib/` utility is only used by one feature, move it into that feature.

The goal is a codebase where adding a feature requires touching files in at most two locations: the feature folder and potentially one shared utility. If adding a feature requires changes in five unrelated files, the architecture needs refactoring—not more abstractions layered on top.
