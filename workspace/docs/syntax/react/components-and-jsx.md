# Components and JSX

## Rule: Use function components exclusively
Class components are legacy and receive no new features.

**Why**: React 19's ref-as-prop, Actions, `use()`, and the React Compiler all target function components. The React team has not added class-component features since hooks shipped.

```tsx
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}</h1>;
}

// children is a regular prop
function Card({ children, title }: { children: React.ReactNode; title: string }) {
  return (
    <section>
      <h2>{title}</h2>
      {children}
    </section>
  );
}
```

## JSX requirements
- New JSX transform (automatic runtime) is required — no `import React` for JSX
- Enable via `jsx: "react-jsx"` in tsconfig (default in Vite/Next.js)

→ Source: [react.dev/learn/your-first-component](https://react.dev/learn/your-first-component)

## Security

### Rule: `dangerouslySetInnerHTML` is an XSS sink
```tsx
// ✗ Only if html is sanitized with DOMPurify — never raw user/API input
<article dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />

// ✓ Default: JSX escapes everything automatically
<p>{userComment}</p>
```
**Why**: JSX escapes by default — the ONLY injection vector in React markup is this escape hatch. If content comes from users, a CMS, or any API, sanitize server-side or treat as plain text.
→ Background: [MDN — XSS](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/XSS), [DOMPurify](https://github.com/cure53/DOMPurify)

## Document metadata and resources (React 19)

React hoists `<title>`, `<meta>` and `<link>` out of wherever you render them and into
`<head>` — so a component can own its own metadata with no framework helper and no
`react-helmet`. Verified on react@19.2.8: rendering them inside a `<div>` produced

```html
<title>My Page Title</title><meta name="description" content="desc"/>
<link rel="canonical" href="https://x.test/"/><div><p>body</p></div>
```

```tsx
function ProductPage({ product }) {
  return (
    <article>
      <title>{product.name} — Shop</title>
      <meta name="description" content={product.summary} />
      <link rel="canonical" href={`https://shop.test/p/${product.id}`} />
      <h1>{product.name}</h1>
    </article>
  );
}
```

**Caveats**:
- **Deduplication is your job** for `<title>`: two mounted components each rendering one
  produce two tags. Render metadata at one level per route.
- In Next.js the `metadata` export is still the right API — it runs at build/request time
  and feeds streaming and social crawlers properly. Use React's version in Vite/SPA apps
  and libraries, not to fight the framework.
- Stylesheets and scripts get `precedence` / async loading instead → [styling.md](./styling.md).

### Preloading resources
```tsx
import { preload, preinit } from 'react-dom';

preload('/fonts/inter.woff2', { as: 'font', type: 'font/woff2', crossOrigin: 'anonymous' });
preinit('/analytics.js', { as: 'script' });   // fetch AND execute early
```
Call these during render or in an event handler on the path to a known-needed resource —
React dedupes and emits the right `<link rel="preload">`. → [optimization.md](./optimization.md)

## Portals — rendering outside the parent tree

```tsx
import { createPortal } from 'react-dom';

function Modal({ children }: { children: React.ReactNode }) {
  return createPortal(children, document.body);
}
```
**When**: Modals, tooltips, toasts that must escape `overflow: hidden` / z-index contexts. Event bubbling still works through the React tree, not the DOM tree.
**Prefer native first**: the HTML `<dialog>` element and popover attribute often remove the need entirely → [MDN — dialog](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog)

## Refs and DOM access

### Rule: `ref` is a regular prop now (React 19)
```tsx
// No forwardRef needed anymore
function SearchInput({ ref }: { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} />;
}
```

### Rule: Use callback refs when something must run on attach/detach

React 19 adds cleanup functions to callback refs:
```tsx
<input ref={(node) => {
  node.focus();
  return () => console.log('detached');   // runs on detach (React 19+)
}} />
```

**Returning a cleanup changes the old contract.** Measured on react@19.2.8: when the ref
callback returns a function, React calls it on detach and **no longer calls the ref with
`null`**. A ref that mixes both styles silently stops tearing down:

```tsx
// ✗ Broken in React 19 — the null branch never runs once you return a cleanup
<div ref={(node) => {
  if (node === null) { teardown(); return; }   // dead code now
  setup(node);
  return () => {};
}} />

// ✓ Pick one style
<div ref={(node) => { setup(node); return () => teardown(); }} />
```

React 19 also **requires the return value to be a cleanup function or nothing** — an arrow
body that implicitly returns something else (`ref={(n) => (this.n = n)}`) is a type error
under the React 19 types and will be treated as a cleanup at runtime. Use a block body.
**Rule**: Never read/write refs during render. Refs are for effects, event handlers, and imperative APIs only.
