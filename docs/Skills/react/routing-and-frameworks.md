# Routing and Frameworks (2026)

**This project's choice is fixed: Next.js (App Router), used as the FRONTEND ONLY.** Its data comes
from Express through `apiFetch`; it never talks to a database. Routing rules are in
frontend-guide/01-foundation/routing.md. The table below is background on the alternatives, not
an option for this project.

## Framework decision
| Need | Framework | Status |
|------|-----------|--------|
| Full-stack, RSC, mature ecosystem | Next.js | Production default |
| Type-safe server functions, TanStack DX | TanStack Start | Still **v1 RC** — announced March 2026, not yet cut to stable as of Sept 2026. API is frozen; treat the label, not the API, as the risk |
| SPA / client-side only | Vite + React Router | Stable (v8 current; v7 maintenance-only) |
| Remix users | React Router framework mode | Stable (Remix merged into RR) |
| Static content + islands | Astro | Stable |
| React Native | Expo | Standard |

> **Security floor**: RSC-mode apps must run `react-router@7.18.2+` or `@8.3.0+`.
> [GHSA-qwww-vcr4-c8h2](https://github.com/advisories/GHSA-qwww-vcr4-c8h2) (High, published
> 2026-07-24) is an RSC-mode CSRF bypass letting an action execute before the 400 response.
> Affected: `>=7.12.0 <7.18.2` and `>=8.0.0 <8.3.0`. It follows CVE-2026-22030 and only
> bites apps using the `unstable_` RSC APIs — but "we don't use RSC mode" is worth
> verifying, not assuming.

## React Router (library mode)
```tsx
import { createBrowserRouter, RouterProvider } from 'react-router';

const router = createBrowserRouter([
  { path: '/', element: <Layout />, children: [
    { index: true, element: <Home /> },
    { path: 'about', element: <About /> },
  ]},
]);

function App() {
  return <RouterProvider router={router} />;
}
```
Same API in v8; v7→v8 migration is future-flag driven (`npx react-router upgrade` handles it).

> **Import from `react-router`, not `react-router-dom`.** Since v7 the DOM bindings were
> folded into the core package. `react-router` is on **8.3.1**; `react-router-dom` is
> frozen at **7.18.3** and exists only as a compatibility shim. A project still importing
> from `react-router-dom` cannot reach 8.x — and therefore cannot reach the CSRF fix below.

## Next.js App Router
```tsx
// app/users/[id]/page.tsx
export default async function UserPage(
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;          // params is a Promise since Next.js 15
  const user = await getUser(id);
  return <UserProfile user={user} />;
}
```

## CRA is dead
Create React App was sunset February 2025. Migrate to: **Vite** (SPA) or **Next.js / React Router framework mode** (full-stack).
→ Migration steps: [deprecations.md](./deprecations.md)

→ Sources: [reactrouter.com](https://reactrouter.com/), [nextjs.org/docs](https://nextjs.org/docs/), [Sunsetting Create React App](https://react.dev/blog/2025/02/14/sunsetting-create-react-app)
