# React Server Components (RSC)

## Status (Sept 2026)
Production-default in the Next.js 16 App Router (16.3.x current). Unstable/experimental elsewhere: React Router (framework mode, `unstable_`-prefixed RSC APIs) and TanStack Start. React 19 exports the `react-server` condition.

## Mental model
- **Server Components**: Run only on the server; ship zero JS; can be async; read DB/filesystem directly
- **Client Components**: Run in the browser; interactive; hooks/state/effects
- **Server Functions**: `'use server'` functions callable from the client (Actions)

## Directory conventions
```
app/
├── layout.tsx      # Server Component (root)
├── page.tsx        # Server Component (route)
├── components/
│   ├── List.tsx    # Server Component (no directive)
│   └── Filter.tsx  # Client Component ('use client' at top)
└── actions.ts      # Server Functions ('use server')
```

## Rules
1. `'use client'` marks a boundary — everything imported by that module becomes client code
2. Server Components can import/render Client Components; never vice versa (pass children/slots instead)
3. Props crossing the boundary must be serializable (no functions, class instances, Symbols)
4. Client Components can't be async — use Suspense, `use()`, or effects

## Example
```tsx
// app/page.tsx — Server Component
import { getProducts } from '@/features/products/actions';
import { Filter } from './Filter';

export default async function Page() {
  const products = await getProducts(); // → apiFetch → Express. Next.js never touches a database
  return (
    <>
      <Filter />   {/* Client island */}
      <ul>
        {products.map(p => <li key={p.id}>{p.name}</li>)}
      </ul>
    </>
  );
}
```

### Rule: Push `'use client'` as deep as possible
The boundary converts every import into client JS. A leaf button needs `'use client'`; its page doesn't.

## `cache()` — request-level deduplication

`cache()` memoizes a function per server request (RSC context only):

```tsx
import { cache } from 'react';
import { apiFetch } from '@/lib/http';

export const getUser = cache((id: string) =>
  apiFetch(`/users/${id}`)
);

// Same request: layout + page + sidebar all call getUser(id)
// → ONE call to Express
```
- Cache lifetime = one render pass; fresh on every request (no stale data risk)
- Works across the whole Server Component tree for identical arguments
- Related: `cacheSignal()` (19.2) aborts in-flight work when the request's cache expires → [hooks-reference.md](./hooks-reference.md)

## Server Functions are an attack surface

`'use server'` publishes an endpoint. Authentication, argument validation, and per-resource
authorization all belong **inside each action** — page-level or middleware checks do not
cover it. Full rules and an annotated example: [forms.md](./forms.md).

## Secret-leakage guard

```ts
// lib/http.ts
import 'server-only';
```
The `server-only` package (built into Next.js) throws at build time if any Client Component imports this module. Add it to every module holding the backend URL, API keys, or secrets (in this project: `lib/http.ts` and `lib/session.ts`) — accidental imports become compile errors instead of leaked credentials. There is no database client to guard: Next.js never has one.

→ Related: public env vars are still exposed to the client → [build-and-tooling.md](./build-and-tooling.md)

→ Sources: [react.dev/reference/rsc/server-components](https://react.dev/reference/rsc/server-components), [react.dev/reference/react/cache](https://react.dev/reference/react/cache)
