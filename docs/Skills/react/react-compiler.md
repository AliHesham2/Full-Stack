# React Compiler (v1.0, October 2025)

## What it does
Automatically memoizes components and hooks at build time. Eliminates most manual `useMemo`, `useCallback`, and `React.memo`.

## Status
- Stable since v1.0 (October 2025)
- Supported by Vite (`@vitejs/plugin-react`), Next.js, Expo
- eslint-plugin-react-hooks ships Compiler-powered lint rules; Oxlint adds compatible ones

## Setup (Next.js — this project's frontend)
```bash
npm install -D babel-plugin-react-compiler
```
```ts
// frontend/next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  reactCompiler: true,   // a stable top-level option in Next.js 16, off by default
};

export default nextConfig;
```
Turn it on only after the build passes without it and build times stay acceptable: it puts Babel
back into the pipeline. For a large app, start with `reactCompiler: { compilationMode: 'annotation' }`
and mark hot components with `"use memo"` (frontend-guide/01-foundation/initialization.md).

## What it memoizes
- Component return values
- Hook return values
- JSX elements passed as props

## What it does NOT fix
- Side effects
- External mutable references read during render
- Violations of the Rules of React — the compiler **skips** components that break purity
  rules. Verified: a component that mutates a prop is left uncompiled, with no error. So
  "the compiler is on" does not mean "this component is memoized" — check the lint output,
  or the React DevTools ✨ badge, before concluding a component is covered.

## Directives and modes — verified against babel-plugin-react-compiler@1.0.0

| Setup | Result |
|---|---|
| `{}` (default) | every eligible component compiled |
| `{ compilationMode: 'annotation' }` | **nothing** compiled unless annotated |
| `'annotation'` + `'use memo'` in the function | that function compiled |
| default + `'use no memo'` in the function | that function **skipped** |
| component that mutates a prop | skipped automatically (impure) |

```tsx
function Chart({ data }) {
  'use no memo';       // ← the escape hatch you will actually need
  // ...compiler leaves this component completely alone
}
```

**`'use no memo'` is the important one.** When the compiler changes behaviour in one
component — usually because that component was quietly breaking the Rules of React and
relying on the old re-render timing — this unblocks you without turning the compiler off
repo-wide. Treat each one as a bug to fix later, not a permanent annotation.

## Migration
Start with `compilationMode: 'annotation'`, add `'use memo'` file by file, then flip to
the default mode once the lint plugin is clean. Keep the lint plugin on throughout — it
flags the impurity patterns the compiler would silently skip.

## Runtime requirement

Compiled output imports from `react/compiler-runtime`:

```js
import { c as _c } from "react/compiler-runtime";
function Counter(t0) { const $ = _c(5); /* ... */ }
```

That entry point exists in **React 19+** (confirmed in react@19.2.8's `exports`). On React
17/18 you must also install `react-compiler-runtime` (1.0.0) or the build fails to resolve
at runtime — a failure that shows up only when the component first renders.

→ Sources: [react.dev/learn/react-compiler](https://react.dev/learn/react-compiler), [github.com/facebook/react/tree/main/compiler](https://github.com/facebook/react/tree/main/compiler)
