# Deprecations and Migration Reference

## Removed in React 19
| API | Replacement |
|-----|-------------|
| `propTypes` | TypeScript |
| `defaultProps` (function components) | ES6 default parameters — **see the silent-failure note below** |
| Legacy Context | `createContext` / `useContext` |
| String refs | `ref` objects/callbacks |
| `createFactory` | JSX |
| `ReactDOM.render` | `createRoot().render()` |
| `ReactDOM.hydrate` | `hydrateRoot()` |
| `unmountComponentAtNode` | `root.unmount()` |
| `findDOMNode` | DOM refs |
| `react-dom/test-utils` | `act` from `react` |
| UMD builds | ESM/CJS builds via bundler |

### `defaultProps` removal fails **silently**, and only under the automatic JSX runtime

Measured on react@19.2.8 — the two element-creation paths disagree:

```js
function Fn({ label }) { return <span>{label}</span>; }
Fn.defaultProps = { label: 'DEFAULT' };

createElement(Fn)      // → <span>DEFAULT</span>   legacy path still honours defaultProps
jsx(Fn, {})            // → <span>undefined</span> ← what modern JSX compiles to
```

No warning is logged, in dev **or** production. Consequences for a migration:

- Code compiled with the **automatic** runtime (`jsx: "react-jsx"` — Vite and Next.js
  defaults) silently renders `undefined` wherever a default was relied on.
- A legacy codebase still on the **classic** runtime (`jsx: "react"`, or hand-written
  `React.createElement`) keeps working — so "we upgraded to 19 and nothing broke" is not
  evidence that the defaults are gone. They break later, when someone flips the runtime.
- `defaultProps` on **class** components still works and is not removed.

**Audit step:** `grep -rn "\.defaultProps" src/` before and after a React 19 upgrade, and
convert every hit on a function component to a default parameter. The codemod below covers
it, but only for files it can parse.

## Deprecated in React 19 (still work, don't adopt)
| API | Replacement |
|-----|-------------|
| `forwardRef` | `ref` as a regular prop → [components-and-jsx.md](./components-and-jsx.md) |
| `element.ref` | `element.props.ref` |
| `Context.Provider` | render `<Context value={…}>` directly → [state-strategy.md](./state-strategy.md) |
| `Context.Consumer` | `useContext(Ctx)` or `use(Ctx)` |
| `react-test-renderer` | `@testing-library/react` — **not** `react-shallow-renderer` ([testing.md](./testing.md)) |

## Run the official codemods
```bash
npx codemod@latest react/19/migration-recipe     # full recipe
npx types-react-codemod@latest preset-19 ./src   # TypeScript types
```

## CRA migration (sunset Feb 2025)
1. `npm install -D vite @vitejs/plugin-react`
2. Move `public/index.html` → project root as `index.html`; replace `%PUBLIC_URL%`
3. Rename `src/index.js` entry usage: Vite uses `<script type="module" src="/src/main.tsx">`
4. Update scripts: `"dev": "vite"`, `"build": "tsc && vite build"`
5. Remove `react-scripts`; move env vars to `VITE_` prefix
6. Replace CRA-only ESLint config with flat config + `eslint-plugin-react-hooks`
→ Destination guide: [build-and-tooling.md](./build-and-tooling.md)

## Class components → function components
No codemod covers this fully. Convert leaf components first, keep class error boundaries only if not adopting `react-error-boundary`.

→ Sources: [React 19 Upgrade Guide](https://react.dev/blog/2024/04/25/react-19-upgrade-guide), [Sunsetting Create React App](https://react.dev/blog/2025/02/14/sunsetting-create-react-app)
