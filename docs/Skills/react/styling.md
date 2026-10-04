# Styling (2026)

## This project's choice
**Tailwind CSS v4 only, through the project's token system:** semantic tokens and scales
(frontend-guide/02-ui/design-tokens.md) wired by three CSS files
(frontend-guide/02-ui/theme-and-dark-mode.md). No CSS Modules, no CSS-in-JS. Inline styles only
for a truly dynamic value (e.g. a computed position).

## Deprecated / avoid for new work
- styled-components: maintenance mode since March 2025 — though still shipping (v6.5–6.6 with `createTheme`, RSC improvements; v7 prereleases), it's a single-maintainer project and the ecosystem has moved to zero-runtime approaches
- Emotion: sustained mainly by MUI's needs
- Runtime CSS-in-JS generally: conflicts with RSC/streaming, adds bundle + runtime cost

## Tailwind v4 changes
- Config lives in CSS via `@theme` (no more `tailwind.config.js`)
- Oxide engine — dramatically faster builds
- Automatic content detection

In this project the tokens are NOT written straight into `@theme`: brand values live in
`src/styles/theme.css` and `src/styles/tokens.css` maps them with `@theme inline`, so light and
dark swap by changing variables (frontend-guide/02-ui/theme-and-dark-mode.md).

```tsx
// Every fill uses its -foreground pair — never text-white, which breaks in the other theme.
// Real components build variants with cva: frontend-guide/02-ui/components.md
function Badge({ tone }: { tone: 'primary' | 'muted' }) {
  return (
    <span className={tone === 'primary'
      ? 'bg-primary text-primary-foreground'
      : 'bg-muted text-muted-foreground'}>
      New
    </span>
  );
}
```

## React 19 native `<style>` / `<link>` support

React deduplicates and hoists style resources, and `precedence` controls insertion order
independently of render order — so a late-rendered component cannot accidentally outrank a
base layer.

```tsx
<style href="button" precedence="medium">{`
  .btn { padding: 8px 16px; }
`}</style>

<link rel="stylesheet" href="/theme.css" precedence="default" />
```

Verified on react@19.2.8: the `<link>` is hoisted out of its parent and emitted as
`<link rel="stylesheet" href="/theme.css" data-precedence="default"/>`. React blocks paint
until a `precedence`-carrying stylesheet loads, which prevents FOUC but also means a slow
stylesheet blocks the render — keep them small and few.

`href` is the dedupe key: two components rendering the same `href` emit one tag.
Document metadata (`<title>`, `<meta>`) hoists the same way → [components-and-jsx.md](./components-and-jsx.md).

**Rule**: Pick ONE primary approach per project. Mixing utility classes with runtime CSS-in-JS in one codebase is a maintenance trap.

→ Sources: [tailwindcss.com/blog/tailwindcss-v4](https://tailwindcss.com/blog/tailwindcss-v4)
