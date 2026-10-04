Performance is a feature, but optimizing the wrong thing wastes engineering time. This guide covers what matters in 2026 and what to ignore.

## Core Web Vitals

Three metrics, unchanged since INP replaced FID:

- **LCP (Largest Contentful Paint):** 2.5s threshold. Measures perceived load speed.
- **INP (Interaction to Next Paint):** 200ms threshold. Replaced FID in March 2024. Measures responsiveness to all interactions, not just the first.
- **CLS (Cumulative Layout Shift):** 0.1 threshold. Measures visual stability.

These are the current Core Web Vitals; verify future changes against official Web Vitals documentation. Measure user flows as well as aggregate scores.

## Server Components: The Biggest Win

Server Components ship zero JavaScript to the client. A component that fetches data and renders HTML on the server costs the user nothing at download time. Default to Server Components — only add `'use client'` when the component needs interactivity, browser APIs, or React hooks.

This is not a micro-optimization. It is the single largest performance lever available in a Next.js application. See [architecture](../01-foundation/architecture.md) for the server-first architecture and [components](../02-ui/components.md) for where to place the `'use client'` boundary.

## Code Splitting

Next.js handles route-based splitting automatically — each route is a separate chunk. For large client components within a route, use dynamic imports:

```tsx
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('./heavy-chart'), {
  loading: () => <Skeleton className="h-64" />,
  ssr: false, // if it uses browser APIs like Canvas
});
```

**Decision:** Split at the component level when a client component exceeds ~50 KB or uses a heavy dependency (D3, a rich text editor, a charting library). Do not split every component — each chunk has overhead.

## Images

AVIF is Baseline-supported in all modern browsers; WebP is the universal fallback. Use `next/image` for automatic format negotiation, responsive sizing, and lazy loading.

```tsx
<Image
  src="/hero.jpg"
  alt="Team working on project"
  width={1200}
  height={600}
  priority // for LCP images only
  sizes="(max-width: 768px) 100vw, 1200px"
/>
```

For the LCP image, set `priority` which adds `fetchpriority="high"` and preloads the image. All other images are lazy-loaded by default. Specify `sizes` to prevent the browser from downloading an image larger than needed.

## Fonts

WOFF2 is universally supported. There is no need for WOFF fallbacks — remove them. Use `next/font` which handles `font-display: swap`, preloading, and CSS variable injection:

```tsx
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'], display: 'swap' });
```

For CLS prevention, `next/font` applies font metrics overrides (`ascent-override`, `descent-override`, `line-gap-override`) automatically. This eliminates layout shift when custom fonts load. If you are using self-hosted fonts outside `next/font`, you must apply these overrides manually.

## JavaScript Budget

Performance-conscious sites target under 200 KB of initial JavaScript (gzipped). The median site ships ~620 KB. Measure your budget with the Network tab in DevTools — filter by JS and check the transferred size after compression.

**Decision:** Set a project budget at initialization. Track it in CI with Lighthouse or a bundle analyzer. See [initialization](../01-foundation/initialization.md) for setting budgets at project start.

## React Compiler

Stable since React Compiler 1.0 (October 2025) and a top-level `reactCompiler` flag in Next.js 16, the compiler eliminates the need for manual `useMemo`, `useCallback`, and `React.memo`. It analyzes component renders and automatically memoizes values and callbacks that would cause unnecessary re-renders.

**Decision:** It is stable but not enabled by default — enabling it pulls Babel into the build pipeline and raises build times. For new projects: enable via top-level `reactCompiler: true`, then verify the build passes and build time is acceptable. For large apps: start with `{ compilationMode: 'annotation' }` and mark hot components with `"use memo"`. Opt individual components out with `"use no memo"`.

```ts
const nextConfig: NextConfig = {
  reactCompiler: true, // stable top-level flag in Next.js 16
};
```

The compiler reduces manual memoization, but do not blindly remove existing useMemo/useCallback calls: consumers/effects may rely on their stability. Change them with behavior checks and profiling.

## `scheduler.yield()` — The #1 INP Optimization

`scheduler.yield()` (available in Chrome and Firefox) yields to the browser's event loop, allowing pending user interactions to be processed before long tasks continue. This is the single most effective INP optimization for sites with heavy client-side work:

```ts
async function processLargeDataset(items) {
  for (const [i, item] of items.entries()) {   // need the index to throttle yields
    processItem(item);
    if (i % 100 === 0) {
      await yieldToMain();   // let the browser handle clicks
    }
  }
}

// scheduler.yield() is NOT Baseline — Chrome and Firefox have it, Safari does not.
// Always feature-detect; the fallback keeps the same yielding behaviour everywhere.
function yieldToMain(): Promise<void> {
  if ('scheduler' in globalThis && 'yield' in (globalThis as any).scheduler) {
    return (globalThis as any).scheduler.yield();
  }
  return new Promise((resolve) => setTimeout(resolve, 0));
}
```

> The earlier version of this snippet iterated with `for (const item of items)` and then
> tested `i % 100` — **`i` was never defined**, so the code throws `ReferenceError` on the
> first iteration. Copy-pasting it would break the page it was meant to speed up.
>
> Note also that `scheduler.yield()` is **not Baseline** (`web-features` v3.36.0 lists it as
> not interoperable). The `setTimeout(0)` fallback is coarser — it goes to the back of the
> task queue rather than keeping priority — but it yields, which is the point.

**Decision:** Use `scheduler.yield()` in any loop or recursive operation that runs on the client and processes more than ~50 items. Do not use it for server-side operations.

## CSS Performance

Tailwind v4 builds are approximately 5× faster than v3. Native CSS nesting, container queries, and `:has()` are all Baseline features — use them without concern. See [design-tokens](../02-ui/design-tokens.md) for the Tailwind setup.

## View Transitions API

The View Transitions API reached Baseline in October 2025. Use it for page transitions instead of JavaScript animation libraries:

```tsx
// In a layout or page transition
document.startViewTransition(() => {
  // update the DOM
});
```

**Decision:** Use View Transitions for cross-page animations in Next.js (via the experimental `viewTransition` flag). Keep it simple — fade and slide transitions. Avoid complex morphing animations that require careful snapshot management.

## Caching and Prefetching

HTTP `stale-while-revalidate` is the 2026 default caching strategy. Set it on your CDN/edge for static assets:

```
Cache-Control: public, max-age=31536000, immutable
```

For HTML pages, use `stale-while-revalidate` to serve cached content instantly while fetching updates in the background.

Speculation Rules (`<script type="speculationrules">`) are used by Shopify and WordPress for navigation prefetching. Next.js prefetches route chunks on link hover by default — you likely do not need Speculation Rules unless you have a static site with predictable navigation paths.

**Decision:** Service Workers are declining for non-PWA apps. Do not add a Service Worker unless you need offline support or background sync. The performance gains from HTTP caching and Next.js prefetching cover most use cases.

## What NOT to Do

These are common performance anti-patterns that waste time or actively hurt performance:

- **Premature micro-optimization.** Optimize based on measurements, not guesses. Profile with DevTools Performance tab and Lighthouse first.
- **JavaScript scroll listeners.** Use CSS `scroll-timeline` or Intersection Observer instead. JS scroll handlers block the main thread.
- **Domain sharding.** HTTP/2 and HTTP/3 multiplex requests over a single connection. Sharding adds DNS lookups and connection overhead.
- **WOFF fallbacks.** WOFF2 is universally supported. Extra `@font-face` declarations add parsing cost and bandwidth.
- **Excessive preconnects.** Only `preconnect` to domains used in the critical path (fonts, API). More than 2-3 preconnects wastes browser connection slots.

## Monitoring

Use the `web-vitals` library to report Core Web Vitals to your analytics provider. This gives you real-user measurement (RUM) for LCP, INP, and CLS from actual users, not lab scores.

```ts
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
```

**Decision:** Add `web-vitals` reporting from day one. Lab tools (Lighthouse) catch regressions in CI. RUM catches real-world problems that lab tests miss (slow networks, low-end devices, geographic latency). See [routing](../01-foundation/routing.md) for integrating with your monitoring setup.