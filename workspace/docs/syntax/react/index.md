# React Knowledge Base

Quick reference map. Each file is self-contained and cross-referenced.
Current target: **react/react-dom 19.3.0** (checked 2026-09-26). Historical execution baseline below: 19.2.8 with React Compiler v1.0. Verified September
2026 by execution, not by reading release notes: React exports and `defaultProps`
behaviour, JSX escaping, `<Activity>`, `use()`, callback-ref cleanup, StrictMode, key
reconciliation and context re-renders were run under react@19.2.8 + jsdom; hydration
mismatches were measured through `onRecoverableError`; the React Compiler modes were run
through babel-plugin-react-compiler@1.0.0; and Cache Components, Turbopack and the Server
Action endpoint exposure were verified against a real Next.js 16.3.4 production build.

| File | Scope |
|------|-------|
| [components-and-jsx.md](./components-and-jsx.md) | Function components, JSX, XSS, portals, DOM refs |
| [a11y.md](./a11y.md) | Accessibility: semantics, forms, focus, jsx-a11y lint |
| [props-and-state.md](./props-and-state.md) | Props rules, state primitives |
| [rendering-model.md](./rendering-model.md) | Render/reconcile/commit, keys, concurrent features |
| [hooks-reference.md](./hooks-reference.md) | Hook API table (16.8 → 19.2) |
| [hook-patterns.md](./hook-patterns.md) | Custom hooks, effect patterns, useEffectEvent |
| [error-handling.md](./error-handling.md) | Error boundaries, hydration mismatches |
| [rsc-architecture.md](./rsc-architecture.md) | Server Components boundaries, 'use client'/'use server' |
| [rendering-strategies.md](./rendering-strategies.md) | CSR/SSR/SSG/ISR/streaming/PPR, `<Activity />` |
| [state-strategy.md](./state-strategy.md) | Decision tree: local/server/global/form state |
| [url-state.md](./url-state.md) | Search params as state |
| [data-fetching.md](./data-fetching.md) | TanStack Query vs RSC decision matrix |
| [react-compiler.md](./react-compiler.md) | Compiler setup, capabilities, migration |
| [optimization.md](./optimization.md) | Profiling, virtualization, code splitting |
| [styling.md](./styling.md) | Tailwind v4, CSS Modules, what's dead |
| [forms.md](./forms.md) | useActionState + Zod, Server Actions; React Hook Form for complex forms |
| [routing-and-frameworks.md](./routing-and-frameworks.md) | Next.js / React Router / TanStack Start |
| [testing.md](./testing.md) | Vitest + RTL, Playwright |
| [typescript.md](./typescript.md) | TS setup, React 19 type changes |
| [build-and-tooling.md](./build-and-tooling.md) | Vite, linting, env vars, monorepos |
| [deprecations.md](./deprecations.md) | Removed/deprecated APIs + codemods |

## Core principles

1. **Function components only; classes are legacy** → [components-and-jsx.md](./components-and-jsx.md)
2. **Never fetch in `useEffect`** → [data-fetching.md](./data-fetching.md)
3. **Context is dependency injection, not state management** → [state-strategy.md](./state-strategy.md)
4. **Prefer the React Compiler over manual memoization** → [react-compiler.md](./react-compiler.md)
5. **Server Components by default in RSC frameworks**; `'use client'` only at leaves → [rsc-architecture.md](./rsc-architecture.md)
6. **`dangerouslySetInnerHTML` = XSS until proven otherwise** → [components-and-jsx.md](./components-and-jsx.md)
7. **Every `'use server'` function is a public endpoint — authorize inside it** → [forms.md](./forms.md)

## Out of scope

Class component patterns, HOCs and render-props composition (superseded by hooks),
Create React App (sunset Feb 2025), styled-components (maintenance mode),
Formik/Final Form (unmaintained), Enzyme (dead), legacy context, string refs,
Fiber internals, legacy lifecycle methods.
