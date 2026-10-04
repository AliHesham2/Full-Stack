# React Rendering Model

## Mental model
1. **Render**: React calls your component function to get JSX
2. **Reconcile**: React diffs the new tree against the old
3. **Commit**: React applies DOM mutations

Rendering must be pure: same props/state → same output, no side effects. The React Compiler enforces this.

## When re-renders happen
- State change in the component
- Context value change in a consuming component
- Parent re-renders (children re-render too unless memoized — or compiled)

## Keys

### Rule: Keys identify list items across renders — use stable IDs
```tsx
// ✗ Index keys: reorder/insert/delete corrupts input focus, animation, state
{items.map((item, i) => <Row key={i} item={item} />)}

// ✓ Stable identity from data
{items.map(item => <Row key={item.id} item={item} />)}
```
**Why**: Reconciliation matches old/new children by key. Index keys make React pair the
wrong elements after reordering — DOM state bugs, not just perf issues.

Measured on react@19.2.8. Render `['b','c']` with an uncontrolled `<input>` per row, type
into the first input, then prepend `'a'`:

| Key | Input values after the prepend |
|---|---|
| `key={i}` | `["USER_TYPED", "c", "c"]` — the typed text stayed on the wrong row **and `b` vanished** |
| `key={item.id}` | `["a", "USER_TYPED", "c"]` — the typed row moved with its data |

Note what the broken case looks like from the outside: the *data* is correct (a, b, c) and
only the DOM state is wrong. It reads as a mysterious UI bug, never as a keys bug.
**Exception**: Static, never-reordered, never-filtered lists may use index keys without harm.

## StrictMode (dev-only behavior)
`<React.StrictMode>` intentionally double-invokes render functions, effects, and their
cleanups in development. Measured on react@19.2.8:

| | Renders | Effect log |
|---|---|---|
| inside `<StrictMode>` | **2** | `run → clean → run` |
| without it | 1 | `run` |

That `run → clean → run` sequence is the point: an effect that cannot survive being torn
down and re-run immediately is broken, and StrictMode makes it fail in development instead
of in production during a Fast Refresh or a Suspense retry.
**Rule**: Never "fix" double-invocation by removing StrictMode — fix the effect to be idempotent (proper cleanup) and keep renders pure. Production runs single-pass.

## Concurrent features
- Renders are interruptible since React 18
- Transitions mark updates as non-urgent (`useTransition`, `startTransition`)
- Suspense boundaries show fallbacks during async work

## React 19.2 additions
- `<Activity />` — hides/shows subtree while preserving state → [rendering-strategies.md](./rendering-strategies.md)
- Suspense boundary batching for SSR; Web Streams resume APIs for Partial Pre-rendering

→ Sources: [react.dev/learn/render-and-commit](https://react.dev/learn/render-and-commit), [react.dev/blog/2025/10/01/react-19-2](https://react.dev/blog/2025/10/01/react-19-2)
