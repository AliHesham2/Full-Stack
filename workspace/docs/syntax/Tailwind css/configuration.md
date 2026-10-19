# Tailwind CSS v4 — Configuration (CSS-First)

## Core Concept

v4 replaces `tailwind.config.js` with CSS-based configuration using directives. All customization lives in your CSS file alongside `@import "tailwindcss"`.

## `@theme` — Design Tokens

The `@theme` block defines design tokens as CSS custom properties. These become available both as utility classes and as native CSS variables at runtime.

```css
@import "tailwindcss";

@theme {
  /* Fonts */
  --font-display: "Satoshi", sans-serif;
  --font-mono: "JetBrains Mono", monospace;

  /* Breakpoints */
  --breakpoint-3xl: 1920px;

  /* Colors — OKLCH recommended for P3 gamut */
  --color-brand-500: oklch(0.84 0.18 117.33);
  --color-brand-600: oklch(0.53 0.12 118.34);

  /* Spacing multiplier (default 0.25rem) */
  --spacing: 0.375rem;

  /* Easing */
  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
  --ease-snappy: cubic-bezier(0.2, 0, 0, 1);

  /* Animation */
  --animate-fade-in: fade-in 0.3s ease-out;

  /* Shadows */
  --shadow-glow: 0 0 20px oklch(0.84 0.18 117.33 / 0.4);
}
```

### Generated Output — not every token becomes a `:root` variable

Measured on a 4.3.3 build with all of these declared in one `@theme` block:

| Namespace | Reaches `:root`? |
|---|---|
| `--color-*`, `--font-*`, `--ease-*`, `--animate-*`, `--radius-*`, `--spacing` | **yes** |
| `--shadow-*`, `--text-shadow-*`, `--inset-shadow-*` | **no** — inlined into the utility rule |
| `--breakpoint-*` | **no** — build-time only; it generates the `@media` variant |

```css
:root {
  --font-display: "Satoshi", sans-serif;
  --color-brand-500: oklch(0.84 0.18 117.33);
  /* --shadow-glow and --breakpoint-3xl are NOT here */
}
```

**And the default palette is tree-shaken.** In a build whose markup referenced only two
colors, `:root` contained exactly those two plus the custom one — `--color-lime-400` and the
rest of the default palette were absent entirely.

Two consequences for hand-written CSS:
- `box-shadow: var(--shadow-glow)` resolves to nothing. Use the `shadow-glow` **utility**, or
  declare the variable yourself outside `@theme`.
- `color: var(--color-lime-400)` resolves to nothing unless some class in your source used
  `lime-400`. Reference theme colors through utilities, or add `@theme static { … }` to force
  emission.

What *is* emitted is genuinely usable — passing `--color-brand-500` or `--ease-snappy` to a
JS animation library works.

### Namespace Reference

The prefix of each variable determines its type and which utilities it generates:

| Namespace | Generates | Example |
|-----------|-----------|--------|
| `--color-*` | Color utilities (`bg-*`, `text-*`, `border-*`, etc.) | `--color-brand-500` → `bg-brand-500` |
| `--font-*` | Font family utilities | `--font-display` → `font-display` |
| `--spacing` | Multiplier for `p-*`, `m-*`, `w-*`, `h-*`, `gap-*`, etc. | `--spacing: 0.25rem` → `p-4` = 1rem |
| `--breakpoint-*` | Responsive variants (`sm:`, `md:`, etc.) | `--breakpoint-3xl: 1920px` → `3xl:` — verified: emits `@media (width >= 1920px)`, the modern range syntax, not `min-width` |
| `--ease-*` | Transition timing function utilities | `--ease-snappy` → `ease-snappy` |
| `--animate-*` | Animation utilities | `--animate-fade-in` → `animate-fade-in` |
| `--shadow-*` | Box shadow utilities | `--shadow-glow` → `shadow-glow` |
| `--inset-shadow-*` | Inset shadow utilities (v4) | `--inset-shadow-inner` → `inset-shadow-inner` |
| `--text-shadow-*` | Text shadow utilities (v4.1) | `--text-shadow-glow` → `text-shadow-glow` |
| `--radius-*` | Border radius utilities | `--radius-pill: 9999px` → `rounded-pill` |

### Color Tokens in Detail

v4 uses OKLCH by default for the built-in palette. When defining custom colors, OKLCH is recommended for P3 display support:

```css
@theme {
  --color-primary-50: oklch(0.97 0.01 250);
  --color-primary-500: oklch(0.55 0.2 250);
  --color-primary-900: oklch(0.25 0.06 250);
}
```

You can use any valid CSS color format (`oklch()`, `hsl()`, `rgb()`, `hex`, `color-mix()`, etc.).

## `@utility` — Custom Utilities

Define custom utility classes directly in CSS:

```css
@utility container {
  margin-inline: auto;
  padding-inline: 2rem;
  max-width: var(--breakpoint-xl);
}

@utility tap-highlight-transparent {
  -webkit-tap-highlight-color: transparent;
}
```

Custom utilities are emitted into `@layer utilities` and follow the same specificity rules as built-in utilities. Don't name them after existing utilities (`text-balance`, `shadow-sm`, ...) — that shadows a built-in.

**Rule**: Use `@utility` for reusable, composable single-property or small multi-property patterns. Use `@layer components` for complex multi-property styles.

## `@custom-variant` — Custom Variants

Create custom variant selectors:

```css
/* Class-based dark mode */
@custom-variant dark (&:where(.dark, .dark *));

/* Data attribute dark mode */
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));

/* High-contrast mode */
@custom-variant contrast (@media (prefers-contrast: high));

/* Reduced motion */
@custom-variant motion (@media (prefers-reduced-motion: reduce));

/* Interaction state */
@custom-variant hover-only (&:hover:not(:focus-visible));
```

See [dark-mode.md](./dark-mode.md) for detailed dark mode configuration.

## `@source` — Content Detection

```css
@import "tailwindcss";

/* Add a source that's excluded by default */
@source "../node_modules/@my-company/ui-lib";

/* Ignore a large directory */
@source not "./src/generated/**";
/* ⚠ only meaningful if automatic detection would have picked it up.
   To take full manual control, disable auto-detection first:
     @import "tailwindcss" source(none);
   Verified on 4.3.3: with source(none) + @source "./srcdir" + @source not "./gen/**",
   a class used only in ./gen was correctly excluded from the output. */

/* Safelist specific classes not in source files */
@source inline("my-dynamic-class another-class");
```

## `@plugin` — Plugin Loading

Load v3-compatible JavaScript plugins via CSS:

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "daisyui";
```

Verified on 4.3.3: `@plugin "@tailwindcss/typography"` emits `.prose` / `.prose-lg`, and
`@plugin "@tailwindcss/forms"` emits `.form-input` — the bridge genuinely works, no shim
needed.

The `@config` bridge works too. A plain v3 `tailwind.config.js` with
`theme.extend.colors.legacy` and `theme.extend.spacing.huge` produced `.bg-legacy` and
`.p-huge` when loaded with `@config "./tailwind.config.js"`. That makes an incremental
migration realistic: keep the JS config working while you move tokens into `@theme`.

**Caveat**: both are **compatibility bridges** for the v3 era. Native v4 plugins use CSS-only
APIs, and the JS plugin API (`addUtilities`, `addComponents`, `theme()`) is deprecated —
don't write new plugins against it.

## `@layer` in User CSS

You can add your own styles to any Tailwind layer:

```css
@import "tailwindcss";

@layer base {
  h1 { @apply text-2xl font-bold; }
}
```

**In this project only `@layer base` is used** (in `globals.css`). There are no component classes
(`.btn`, `.card`): a repeated style becomes a component in `components/ui/`
(frontend-guide/02-ui/components.md).

## `@apply` in v4

`@apply` still works but is **not recommended for most cases** in v4. Prefer:
- **Extracting components** (React/Vue/Svelte components with utility classes in templates)
- **`@utility`** for truly reusable single-concern utilities (this project defines its named
  z-index layers this way: frontend-guide/02-ui/theme-and-dark-mode.md)

When `@apply` is appropriate:
- Migrating legacy code incrementally
- Styles shared across many independent templates where component extraction isn't possible
- Third-party CSS that needs to use Tailwind values

In Vue/Svelte/Astro `<style>` blocks or CSS modules, `@apply` needs `@reference` first. If
your project uses only the default theme, the shorthand is `@reference "tailwindcss";`.

Without it the build **fails hard** — verified on 4.3.3, this is the exact message:

```
Error: Cannot apply unknown utility class `px-4`.
Are you using CSS modules or similar and missing `@reference`?
```

So this failure is loud, not silent — if a build is green, `@apply` resolved.

`@reference` does **not** inline the framework. A file containing only
`@reference "tailwindcss";` plus one `.btn { @apply … }` compiled to **296 bytes**, and the
theme values came through as `var()` with literal fallbacks:

```css
.btn {
  border-radius: var(--radius-md, 0.375rem);
  background-color: var(--color-blue-600, oklch(54.6% 0.245 262.881));
  padding-inline: calc(var(--spacing, 0.25rem) * 4);
}
```

The fallbacks mean the rule still renders correctly even where the `:root` variables are not
present — useful for shadow DOM and isolated component styles.

## `@variant` — Applying Variants in CSS

Apply a variant to a block of declarations **inside a rule** — `@variant` nests within the selector, it does not wrap selectors:

```css
.custom-card {
  background: white;

  @variant dark {
    background: var(--color-surface-dark);
  }
}
```

Stacked (`@variant hover:focus { ... }`) and compound (`@variant hover, focus { ... }`) forms work since v4.3.

## Build-Time Functions

Two documented functions simplify working with tokens in custom CSS:

```css
.my-element {
  color: --alpha(var(--color-lime-300) / 50%);  /* → color-mix(in oklab, ... 50%, transparent) */
  margin: --spacing(4);                          /* → calc(var(--spacing) * 4) */
}
```

`--spacing()` also works inside arbitrary values: `py-[calc(--spacing(4)-1px)]`.

Source: https://tailwindcss.com/docs/functions-and-directives

## `@config` — JS Config Bridge (Compat)

For incremental migration, you can still load a JS config:

```css
@import "tailwindcss";
@config "./tailwind.config.js";
```

This is a compatibility feature, not the recommended path for new projects.