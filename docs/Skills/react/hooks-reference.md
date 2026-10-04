# Hooks Reference (React 19.2)

## Core (16.8)
| Hook | Purpose |
|------|---------|
| useState | Local state |
| useReducer | Complex state logic |
| useEffect | Side effects after render |
| useContext | Read context |
| useRef | Mutable ref / DOM access |
| useMemo / useCallback | Manual memoization — rarely needed with the Compiler → [react-compiler.md](./react-compiler.md) |
| useLayoutEffect | Sync DOM measurement before paint |
| useImperativeHandle | Custom imperative API exposed through a ref |
| useInsertionEffect | DOM insertion before other effects — CSS-in-JS library authors only |
| useDebugValue | DevTools label for custom hook state |

## React 18+
| Hook | Purpose |
|------|---------|
| useTransition | Non-blocking state updates |
| useDeferredValue | Deferred re-render of expensive tree |
| useId | SSR-safe unique IDs |
| useSyncExternalStore | Subscribe to external stores |

## React 19+
| Hook | Purpose |
|------|---------|
| use | Read promises/context conditionally (never inside try/catch) |
| useActionState | Form action state |
| useFormStatus | Parent form status — imported from `react-dom`, not `react` |
| useOptimistic | Optimistic UI updates |

## React 19.2+
| Hook | Purpose |
|------|---------|
| useEffectEvent | Non-reactive logic extracted from effects (stable identity) |

**Not hooks, but new in 19.2** — plain functions, no Rules-of-Hooks constraints:

| Export | Purpose |
|--------|---------|
| `cacheSignal()` | `AbortSignal` tied to the `cache()` lifetime; **react-server (RSC) only** |
| `captureOwnerStack()` | Dev-only owner stack for custom error overlays |
| `<Activity>` | Component, not a hook → [rendering-strategies.md](./rendering-strategies.md) |

Verified present in react@19.2.8: `use`, `useActionState`, `useOptimistic`, `useEffectEvent`, `cache`, `cacheSignal`, `captureOwnerStack`, `Activity`. `useFormStatus` ships from **`react-dom`**, not `react`.

## Rules of Hooks
1. Only call at top level — never in loops, conditions, or nested functions
2. Only call from React functions (components or custom hooks)
3. Effect cleanup is mandatory for subscriptions/timers/listeners

**Enforcement**: eslint-plugin-react-hooks v6+ (current major: v7, April 2026) enforces all rules; flat config is the default `recommended` preset since 6.x. It also flags `use()` inside try/catch and `useEffectEvent` called from arbitrary closures.

→ Sources: [react.dev/reference/react/hooks](https://react.dev/reference/react/hooks), [React 19.2 release notes](https://github.com/facebook/react/releases/tag/v19.2.0)
