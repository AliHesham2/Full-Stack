# State Management Strategy (2026)

## Decision tree
1. **Local UI state** → `useState` / `useReducer`
2. **Server state** → TanStack Query (client) or RSC (server) → [data-fetching.md](./data-fetching.md)
3. **URL state** (filters, tabs, pagination) → `nuqs` or framework search params → [url-state.md](./url-state.md)
4. **Global client state** → Zustand (default) or Redux Toolkit (complex/legacy teams)
5. **Form state** → React Hook Form → [forms.md](./forms.md)

## Zustand (default recommendation)
```tsx
import { create } from 'zustand';

interface Store {
  count: number;
  increment: () => void;
}

export const useStore = create<Store>((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
}));

// ✓ Select the exact slice — re-renders only when it changes
function Counter() {
  const count = useStore((s) => s.count);
  const increment = useStore((s) => s.increment);
  return <button onClick={increment}>{count}</button>;
}
```
**Rule**: Always select minimal slices. Selecting the whole store re-renders on every change.

## Redux Toolkit (when needed)
Complex state logic with many cross-dependencies, time-travel debugging requirements, existing team knowledge, or RTK Query adoption.

## XState
For complex multi-step flows needing explicit states/transitions (state machines).

## Context: React 19 provider syntax

```tsx
const Theme = createContext('light');

<Theme value="dark"><App /></Theme>          {/* React 19 — the context IS the provider */}
<Theme.Provider value="dark"><App /></Theme.Provider>   {/* legacy — still works */}
```
Verified on react@19.2.8: both render identically. `Context.Provider` is deprecated, not
removed; `Context.Consumer` is also deprecated — use `useContext` / `use`.

## Context is NOT for state
Context re-renders ALL consumers on every value change — no selector mechanism.
Use Context for: theme, locale, dependency injection of rarely-changing values.
Don't use it for: shared mutable/frequently-changing application state.

→ Sources: [zustand docs](https://zustand.docs.pmnd.rs/), [redux-toolkit.js.org](https://redux-toolkit.js.org/)
