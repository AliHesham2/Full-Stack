# Rendering Strategies

## Options (2026)
| Strategy | When | Frameworks |
|----------|------|------------|
| CSR | Pure SPA, no SEO needs | Vite + React |
| SSR | SEO + dynamic data | Next.js, React Router, TanStack Start |
| SSG | Static content | Next.js, Astro |
| ISR | Static + periodic revalidation | Next.js |
| Streaming SSR | Progressive page load | Next.js, React Router |
| Partial Prerender (PPR) | Static shell + dynamic holes | Next.js 16+ via Cache Components — see below |

## Streaming
- Suspense boundaries stream independently — shell renders immediately, holes fill in
- React 19.2 batches Suspense fallback commits for SSR and adds Web Streams/Node resume APIs for PPR
### Cache Components in Next.js 16 — verified on 16.3.4

The config key is **top-level `cacheComponents`**, not under `experimental`:

```js
// next.config.mjs
export default { cacheComponents: true };
```

- `experimental.ppr: true` is now a **hard build error**:
  *"`experimental.ppr` has been merged into `cacheComponents`."*
- `experimental.cacheComponents` still works but warns:
  *"has been moved to `cacheComponents`."* — much older guidance still shows this form.

**Enabling it changes the prerendering contract.** Any route touching `params`,
`searchParams`, `cookies()`, `headers()`, `connection()`, or an uncached `fetch`
**outside a `<Suspense>` boundary** fails the build:

```
Error: Route "/users/[id]": Next.js encountered uncached or runtime data during prerendering.
```

The fix is to push dynamic access below a boundary so the static shell can be prerendered:

```tsx
// app/users/[id]/page.jsx
import { Suspense } from 'react';

async function User({ params }) {
  const { id } = await params;              // dynamic — must live under Suspense
  return <p>user:{id}</p>;
}

export default function Page({ params }) {
  return <Suspense fallback={<em>loading user…</em>}><User params={params} /></Suspense>;
}
```

That route then builds as **◐ Partial Prerender** — "prerendered as static HTML with dynamic
server-streamed content". A `'use cache'` function reports its cache profile in the build
table (default: revalidate 15m, expire 1y). Escape hatch for a genuinely blocking route:
`export const instant = false`.

## `<Activity />` (React 19.2)
Preserves component state and DOM order while hidden — unmounts effects, defers updates until idle:

```tsx
import { Activity, useState } from 'react';

function Tabs() {
  const [active, setActive] = useState('a');
  return (
    <>
      <Activity mode={active === 'a' ? 'visible' : 'hidden'}>
        <PanelA />   {/* state kept while hidden */}
      </Activity>
      <Activity mode={active === 'b' ? 'visible' : 'hidden'}>
        <PanelB />
      </Activity>
    </>
  );
}
```
**When**: Tabs/wizards where losing state would be wrong but keeping effects running would
be wasteful. Replaces the manual "display: none + keep mounted" hack.

Measured on react@19.2.8, toggling `visible → hidden → visible`:

```
mount    : layout-mount, effect-mount
hidden   : layout-cleanup, effect-cleanup      ← effects torn down
visible  : layout-mount, effect-mount          ← effects re-run
state    : preserved across the whole cycle (count stayed at 1)
```

**Two consequences people miss:**

1. **The DOM stays in the document while hidden** (React applies `display: none`). So
   `document.querySelector` still finds it. Testing Library getByRole excludes hidden DOM by
   default; { hidden: true } opts in. Assert visibility separately. Verify
   `aria-hidden`/focusability for anything a screen reader could reach → [a11y.md](./a11y.md).
2. **Effects re-run on every reveal.** Anything in a mount effect — a fetch, an analytics
   event, a subscription — fires again each time the tab is re-selected. That is correct
   behaviour for subscriptions and wrong for "track once" logic.

On the server, `mode="hidden"` renders **nothing at all** (verified: empty string from
`renderToString`) — hidden content is not in the SSR payload.

→ Sources: [react.dev/reference/react/Activity](https://react.dev/reference/react/Activity), [react.dev/blog/2025/10/01/react-19-2](https://react.dev/blog/2025/10/01/react-19-2)
