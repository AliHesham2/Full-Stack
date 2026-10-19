# Theme & Dark Mode

Tailwind v4 is configured **in CSS**: no `tailwind.config.js` and no `@tailwind base;` (those
are v3 and do nothing in v4). The token *names and scales* are in
[design-tokens](./design-tokens.md); this file is the wiring.

## Three Files, Three Jobs

```
src/
  app/globals.css      # entry: imports + base styles. Kit: same in every project
  styles/tokens.css    # semantic variables → Tailwind utilities. Kit: never edited per project
  styles/theme.css     # brand VALUES for light and dark. THE ONLY FILE A PROJECT EDITS
```

This split is what makes the UI kit portable: a new project replaces `theme.css` and its fonts,
nothing else ([portability](./portability.md)).

## `src/app/globals.css`

```css
@import "tailwindcss";
@import "tw-animate-css";            /* animations used by shadcn/ui components */
@import "../styles/tokens.css";
@import "../styles/theme.css";

@layer base {
  * { @apply border-border outline-ring/50; }
  body { @apply bg-background text-foreground font-sans antialiased; }
  html:has(dialog[open]) { overflow: hidden; }   /* native <dialog> doesn't lock scroll */
}
```

Only base rules go here. A repeated style becomes a component in `components/ui/`, never a CSS
class like `.btn-base`.

## `src/styles/tokens.css` (the kit)

```css
@custom-variant dark (&:where(.dark, .dark *));   /* next-themes puts .dark on <html> */

/* `inline` is required: these point at variables whose values change per theme. */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-success: var(--success);
  --color-success-foreground: var(--success-foreground);
  --color-warning: var(--warning);
  --color-warning-foreground: var(--warning-foreground);
  --color-info: var(--info);
  --color-info-foreground: var(--info-foreground);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);

  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);

  /* next/font sets --font-body / --font-display / --font-code in app/layout.tsx */
  --font-sans: var(--font-body, ui-sans-serif), system-ui, sans-serif;
  --font-heading: var(--font-display, var(--font-body, ui-sans-serif)), system-ui, sans-serif;
  --font-mono: var(--font-code, ui-monospace), monospace;
}

/* No z-index namespace in v4, so named layers are custom utilities. */
:root {
  --z-base: 0; --z-dropdown: 10; --z-sticky: 20; --z-fixed: 30; --z-overlay: 40;
  --z-modal: 50; --z-popover: 60; --z-tooltip: 70; --z-toast: 80;
}
@utility z-base     { z-index: var(--z-base); }
@utility z-dropdown { z-index: var(--z-dropdown); }
@utility z-sticky   { z-index: var(--z-sticky); }
@utility z-fixed    { z-index: var(--z-fixed); }
@utility z-overlay  { z-index: var(--z-overlay); }
@utility z-modal    { z-index: var(--z-modal); }
@utility z-popover  { z-index: var(--z-popover); }
@utility z-tooltip  { z-index: var(--z-tooltip); }
@utility z-toast    { z-index: var(--z-toast); }
```

## `src/styles/theme.css` (the brand, the one file a project edits)

A neutral default, so the kit works out of the box. A project changes **values** and never
renames or removes a variable. Every pair must pass 4.5:1 in **both** blocks. The status colors
are starting values; check them against the real brand.

```css
:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);                  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);                        --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);                     --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);                 --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);                --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);                    --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);                   --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);    --destructive-foreground: oklch(0.985 0 0);
  --success: oklch(0.527 0.154 150.069);       --success-foreground: oklch(0.985 0 0);
  --warning: oklch(0.769 0.188 70.08);         --warning-foreground: oklch(0.205 0 0);
  --info: oklch(0.546 0.245 262.881);          --info-foreground: oklch(0.985 0 0);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);              --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);                    --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.205 0 0);                 --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);                 --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);               --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);                   --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);                  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);    --destructive-foreground: oklch(0.985 0 0);
  --success: oklch(0.696 0.17 162.48);         --success-foreground: oklch(0.145 0 0);
  --warning: oklch(0.828 0.189 84.429);        --warning-foreground: oklch(0.145 0 0);
  --info: oklch(0.707 0.165 254.624);          --info-foreground: oklch(0.145 0 0);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.556 0 0);
}
```

## Fonts

Loaded with `next/font` in `src/app/layout.tsx` and exposed as variables that `tokens.css`
already reads. Add `--font-display` for a heading font or `--font-code` for code the same way.
For Arabic, load a font with the `arabic` subset and set `dir="rtl"`; the kit's logical utilities
mirror the layout automatically.

```tsx
// src/app/layout.tsx
import { Inter } from 'next/font/google';
import { ThemeProvider } from '@/components/theme-provider';

const body = Inter({ subsets: ['latin'], variable: '--font-body', display: 'swap' });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  // suppressHydrationWarning: next-themes edits <html> before React hydrates
  return (
    <html lang="en" className={body.variable} suppressHydrationWarning>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

## Dark Mode

**Modes:** `light`, `dark`, `system` (follows the OS live). The default is `system`.

**Use `next-themes`; never hand-build a Context + `localStorage` provider.** The server cannot
read `localStorage`, so a hand-built provider paints the default theme first and switches after
JavaScript runs: a wrong-theme flash on every load, plus a hydration mismatch. `next-themes` sets
the class with a tiny blocking script before first paint, and remembers the choice (saved choice
→ system → light) with no extra code.

```tsx
// src/components/theme-provider.tsx
'use client';
import { ThemeProvider as NextThemesProvider } from 'next-themes';

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  return (
    <NextThemesProvider attribute="class" defaultTheme="system" enableSystem disableTransitionOnChange>
      {children}
    </NextThemesProvider>
  );
}
```

**The toggle** (`components/ui/theme-toggle.tsx`) uses `useTheme()` and offers Light / Dark /
System with icons *and* an accessible name. Until it mounts, it renders a same-size placeholder,
because the server can't know the theme.

**Rules:**

- **No `dark:` color classes in components.** `bg-background` already changes. `dark:` is allowed
  only to swap something genuinely different per theme, such as a logo with two versions
  (`dark:hidden` / `hidden dark:block`).
- **The switch is instant** (`disableTransitionOnChange`). Animating every color on switch makes
  elements change at different speeds. Hover and focus transitions stay.
- **Icons use `currentColor`**; images prefer SVG or transparent PNG.
- **Another theme later** (for example high contrast) is a new variable block in `theme.css`
  (`.high-contrast { … }`) plus its name in `next-themes` (`themes={[…]}`). No component changes;
  that is the test that the token system works.

## Plugins

Declared in CSS with `@plugin`, only when used. `@tailwindcss/typography` is only for long-form
rich text (`prose`). Container queries are built into v4. `@tailwindcss/forms` is **not** used:
its global resets fight the kit's styled inputs.
