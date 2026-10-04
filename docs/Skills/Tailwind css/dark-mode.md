# Tailwind CSS v4 — Dark Mode

**In this project:** the theme is class-based and set by `next-themes`; light and dark swap the
token variables in `src/styles/theme.css`, so components use `bg-background`, `text-foreground`
and never a `dark:` color class. The `dark:bg-…` examples below only show how the variant works;
`dark:` is used in components only for something genuinely different per theme (e.g. a logo).
Full setup: frontend-guide/02-ui/theme-and-dark-mode.md.

## Default: `prefers-color-scheme`

Out of the box, the `dark:` variant responds to the OS/browser preference:

```html
<div class="bg-white dark:bg-gray-900">
  <p class="text-gray-900 dark:text-white">Hello</p>
</div>
```

## Class-Based Toggle (Most Common)

Override the `dark` variant to use a class selector:

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

```html
<html class="dark">
  <body>
    <div class="bg-white dark:bg-black">  <!-- Applies when .dark is on html -->
    </div>
  </body>
</html>
```

**Why `:where()`**: Using `:where()` keeps specificity at 0, so dark mode utilities don't fight with other utilities. The `*` combinator ensures nested elements inherit the dark context.

**Toggle implementation:** in Next.js, never a hand-made toggle that reads `localStorage` in
React: the server cannot read it, so the page paints the wrong theme first and then flips.
Use `next-themes` (§ Next.js + next-themes Pattern below), which sets the class before first
paint and remembers the choice.

Source: https://tailwindcss.com/docs/dark-mode

## Data Attribute Toggle

```css
@import "tailwindcss";
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

```html
<html data-theme="dark">
```

Useful when you need multiple themes or want to avoid class name collisions.

## Multi-Theme Systems

For systems with more than light/dark, define multiple custom variants:

```css
@import "tailwindcss";

@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
@custom-variant theme-blue (&:where([data-theme=blue], [data-theme=blue] *));
@custom-variant theme-green (&:where([data-theme=green], [data-theme=green] *));
```

Or override theme variables at runtime — theme variables are plain CSS custom properties, so utilities like `bg-surface` (which reference `var(--color-surface)`) re-resolve per scope:

```css
@import "tailwindcss";

@theme {
  --color-surface: var(--surface-light);
  --color-on-surface: var(--on-surface-light);
}

.dark {
  --color-surface: var(--surface-dark);
  --color-on-surface: var(--on-surface-dark);
}
```

Then use `bg-surface text-on-surface` everywhere — colors swap based on the `.dark` scope.

**Notes**:
- You cannot nest `@theme` inside `@custom-variant`; overrides must be plain CSS scoped to
  the class/attribute selector.
- **With `@theme inline`, override the variable it POINTS TO, never the `--color-*` token.**
  `inline` makes the utility read the referenced variable directly. Compiled on 4.3.3:

  ```css
  /* @theme inline { --color-surface: var(--surface-light) } */
  .bg-surface { background-color: var(--surface-light); }
  ```

  So a `.dark { --color-surface: … }` override changes nothing (the utility never reads
  `--color-surface`) — the trap. The working pattern, and **this project's**, keeps one
  variable per role and overrides THAT variable:

  ```css
  @theme inline { --color-background: var(--background); }   /* tokens.css */
  :root { --background: oklch(1 0 0); }                       /* theme.css  */
  .dark { --background: oklch(0.145 0 0); }                   /* theme.css  */
  /* → .bg-background { background-color: var(--background) } switches with .dark */
  ```

  Verified in a real Next.js build (2026-09-23): `.bg-primary{background-color:var(--primary)}`.
  This is also what `create-next-app` generates. Never "fix" it by removing `inline`.

## Preventing Flash of Wrong Theme (FOUC)

`next-themes` injects its own blocking script that sets the class before first paint. Do not
write another one.

## `color-scheme` Utility

v4 adds **`scheme-*`** utilities to fix native UI elements (scrollbars, form controls) in dark mode. Verified in 4.3.3: `scheme-light`, `scheme-dark`, `scheme-light-dark` and `scheme-normal` all compile — **`color-scheme-dark` does not exist and emits nothing**:

```html
<html class="dark">
  <head>
    <meta name="color-scheme" content="dark light">
  </head>
</html>
```

The meta tag declares which schemes the page *supports*; the `scheme-*` utility (or the CSS
`color-scheme` property) declares which one is *active*. Set the meta once with both values
so the browser paints the correct default background before CSS arrives, then let the
utility follow your theme state.

Or via Tailwind on the root element:

```html
<html class="dark scheme-dark">
```

## Next.js + next-themes Pattern

```css
/* globals.css */
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

```tsx
// layout.tsx
import { ThemeProvider } from 'next-themes';

export default function RootLayout({ children }) {
  return (
    <html suppressHydrationWarning>
      <body>
        <ThemeProvider attribute="class">{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

The `attribute="class"` tells `next-themes` to add/remove the `dark` class, which matches our `@custom-variant`.