# Performance Optimization

## With React Compiler (default)
Automatic memoization eliminates most `useMemo`/`useCallback`. Manual memoization remains only for: non-React interop, compiler-opted-out files, or identity-stability across intentional boundaries.
→ [react-compiler.md](./react-compiler.md)

## Without Compiler (or manual cases)
```tsx
const memoizedValue = useMemo(() => computeExpensive(a, b), [a, b]);
const memoizedFn = useCallback(handleClick, [dep]);
const MemoChild = React.memo(Child);
```
**Rule**: Profile first. Memoization has its own cost — wrong deps make it useless.

## Identifying bottlenecks
1. React DevTools Profiler (highlight re-renders)
2. React 19.2 Performance Tracks (Chrome DevTools custom tracks)
3. Why-did-you-render (local dev only)

## Key techniques
| Problem | Solution |
|---------|----------|
| Slow initial load | Code-split with `React.lazy` + Suspense; route-level splitting; `preload`/`preinit` for known-next resources → [components-and-jsx.md](./components-and-jsx.md) |
| Expensive long list | Virtualize (`@tanstack/react-virtual`) |
| Wide re-renders | Move state down to leaves; lift content up via children props |
| Blocking main thread | `useTransition` / `useDeferredValue` for non-urgent updates |

```tsx
// useTransition — deprioritize YOUR state update
const [isPending, startTransition] = useTransition();

function onTabClick(id: string) {
  startTransition(() => setActiveTab(id));   // input stays responsive
}
// isPending → show inline spinner on the slow tab

// useDeferredValue — deprioritize rendering a VALUE's downstream tree
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => filter(hugeList, deferredQuery), [deferredQuery]);
```
**Rule**: `useTransition` wraps the update you control; `useDeferredValue` wraps the value when the update happens elsewhere (e.g., controlled inputs).

## Anti-patterns
- Premature memoization before profiling
- Context holding frequently-changing values → [state-strategy.md](./state-strategy.md)
- Deriving state with `useEffect` instead of during render → [props-and-state.md](./props-and-state.md)

→ Sources: [react.dev/learn/render-and-commit](https://react.dev/learn/render-and-commit), [Performance Tracks](https://react.dev/reference/dev-tools/react-performance-tracks)
