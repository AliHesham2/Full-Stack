# Data Fetching (2026)

## Decision matrix (this project: Next.js frontend → Express backend)
| Context | Solution |
|---------|----------|
| Page data on load | Fetch in a Server Component, through server-only `features/<feature>/queries.ts` (`apiFetch` → Express) |
| Client interactivity (polling, optimistic updates, refetch) | TanStack Query whose queryFn fetches a same-origin Next.js GET Route Handler → Express — never an Express URL from the browser |

The browser never calls Express, and Next.js never touches a database
(frontend-guide/03-data/backend-integration.md).

## TanStack Query (client-side)
```tsx
'use client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { fetchTodos } from '@/features/todos/client-queries'; // fetch('/api/todos') on Next.js
import { addTodo } from '@/features/todos/actions'; // mutation Server Action

function Todos() {
  const { data, isPending, error } = useQuery({
    queryKey: ['todos'],
    queryFn: ({ signal }) => fetchTodos(signal),
  });

  const client = useQueryClient();
  const mutation = useMutation({
    mutationFn: addTodo,
    retry: false,
    onSuccess: () => client.invalidateQueries({ queryKey: ['todos'] }),
  });

  if (isPending) return <Spinner />;
  if (error) return <ErrorState error={error} />;
  return <TodoList todos={data} onAdd={mutation.mutate} />;
}
```

## RSC (server-side)
```tsx
import { getProduct } from '@/features/products/queries'; // server-only apiFetch → Express

async function ProductPage({ id }: { id: string }) {
  const product = await getProduct(id);
  return <ProductDetail product={product} />;
}
```
No client JS for the server fetch; still provide loading/error/empty states and Suspense for slow regions.

## use(promise) and Suspense

use() can read a stable promise and may be called conditionally, but never inside try/catch.
Do not put an authenticated fetch promise in a module-global variable: it can survive logout
or, on the server, cross request/user boundaries. Prefer TanStack Query useSuspenseQuery with
identity-scoped keys, or a request-scoped server promise passed to the client. Clear private
query caches when identity changes. Streaming still requires explicit loading/error UI.

## Anti-patterns
- ✗ `useEffect` + `useState` for fetching (waterfalls, no cache, race conditions, manual loading/error states)
- ✗ Global loading/error state in Redux/Zustand — query libraries own this
- ✗ Client-side fetching of data the Server Component could load directly

**Rule**: Server data never lives in global client state. Cache invalidation is a solved problem in TanStack Query / RSC revalidation.

→ Sources: [tanstack.com/query](https://tanstack.com/query/latest/docs/framework/react/overview), [react.dev/reference/rsc/server-components](https://react.dev/reference/rsc/server-components)
