# Tailwind CSS v4 — Breaking Changes (v3 → v4)

**Automated migration**: Run `npx @tailwindcss/upgrade` (requires Node 20+). Creates a git diff you can review. Handles most changes automatically.

Source: https://tailwindcss.com/docs/upgrade-guide

---

## Browser Requirements

v4 targets **Safari 16.4+, Chrome 111+, Firefox 128+**. It uses `@property`, `color-mix()`, and cascade layers internally. There is no compatibility shim — if you need older browsers, stay on v3.4.

## Installation Changes

| v3 | v4 | Notes |
|----|----|-------|
| `npm i tailwindcss` | `npm i tailwindcss @tailwindcss/postcss` | PostCSS plugin moved to separate package |
| `tailwindcss` in postcss.config | `@tailwindcss/postcss` in postcss.config | |
| `postcss-import` plugin | **Removed** (built-in) | |
| `autoprefixer` plugin | **Removed** (Lightning CSS handles it) | |
| `npx tailwindcss -i ... -o ...` | `npx @tailwindcss/cli -i ... -o ...` | CLI moved to `@tailwindcss/cli` |

## Import Syntax

```css
/* v3 — REMOVED */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* v4 */
@import "tailwindcss";
```

## Configuration

`tailwind.config.js` is replaced by `@theme` and other CSS directives. See [configuration.md](./configuration.md).

For incremental migration, `@config "./tailwind.config.js"` still works as a bridge.

## Removed vs. merely renamed — they fail very differently

Compiled against **tailwindcss 4.3.3** to see which old class names still emit CSS. The
distinction decides how urgent a migration is, and the two are usually listed together.

### Genuinely removed — emit **nothing**, silently

| Removed (v3) | Replacement (v4) |
|---------------|------------------|
| `bg-opacity-*` | `bg-{color}/{opacity}` (e.g. `bg-blue-500/50`) |
| `text-opacity-*` | `text-{color}/{opacity}` |
| `border-opacity-*` | `border-{color}/{opacity}` |
| `divide-opacity-*` | `divide-{color}/{opacity}` |
| `ring-opacity-*` | `ring-{color}/{opacity}` |
| `placeholder-opacity-*` | `placeholder-{color}/{opacity}` |

Verified: `bg-opacity-50`, `text-opacity-50` and `border-opacity-50` produce **no rule at
all** in the output. No error, no warning — the element just loses its transparency. These
are the ones to grep for first.

### Still compile in 4.3.3 — legacy names kept as aliases

| Old name | Preferred name | Status in 4.3.3 |
|---|---|---|
| `flex-shrink-*` | `shrink-*` | still emits `.flex-shrink-0` |
| `flex-grow-*` | `grow-*` | still emits `.flex-grow` |
| `overflow-ellipsis` | `text-ellipsis` | still emits `.overflow-ellipsis` |
| `decoration-slice` / `-clone` | `box-decoration-*` | still emits |
| `bg-gradient-to-*` | `bg-linear-to-*` | both emit |
| `!flex` (leading `!`) | `flex!` (trailing) | both emit |
| `bg-[--var]` | `bg-(--var)` | both emit |
| `start-*` / `end-*` | `inset-s-*` / `inset-e-*` | **byte-identical output** — `.start-4` and `.inset-s-4` both compile to `inset-inline-start: calc(var(--spacing) * 4)` |

Renaming these is a consistency cleanup, not a bug fix. Do it with the codemod, not by hand,
and never at the same time as the group below.

## The shifted scales — the most dangerous change in the whole migration

The `sm` step moved down one notch across four scales. **Every old class name still
compiles**, so nothing breaks — it just renders one size larger than it used to, across the
entire app, with no error anywhere.

Measured values from a 4.3.3 build:

| Class | Value in 4.3.3 | Was in v3 |
|---|---|---|
| `shadow-xs` | `0 1px 2px 0 rgb(0 0 0 / 0.05)` | this was `shadow-sm` |
| `shadow-sm` | `0 1px 3px 0 …, 0 1px 2px -1px …` | this was `shadow` |
| `blur-xs` | `4px` | this was `blur-sm` |
| `blur-sm` | `8px` | this was `blur` |
| `rounded-xs` | `0.125rem` | this was `rounded-sm` |
| `rounded-sm` | `0.25rem` | this was `rounded` |

So a v3 template that says `shadow-sm` gets **v3's `shadow`** after upgrading — a bigger
shadow, silently. Same for `blur-sm` (4px → 8px) and `rounded-sm` (0.125rem → 0.25rem).

| v3 name | rename to | Also applies to |
|---|---|---|
| `shadow-sm` | `shadow-xs` | `drop-shadow-sm` → `drop-shadow-xs` |
| `shadow` | `shadow-sm` | `drop-shadow` → `drop-shadow-sm` |
| `blur-sm` | `blur-xs` | `backdrop-blur-sm` → `backdrop-blur-xs` |
| `blur` | `blur-sm` | `backdrop-blur` → `backdrop-blur-sm` |
| `rounded-sm` | `rounded-xs` | |
| `rounded` | `rounded-sm` | |
| `outline-none` (a11y version) | `outline-hidden` | see below |

The bare `shadow`, `blur` and `rounded` also still compile in 4.3.3 and now carry the new
`-sm` values — so leaving them alone is *visually* correct but keeps a deprecated name.

**Rule**: run `npx @tailwindcss/upgrade` on a clean tree and review its diff. Because none of
this throws, a hand migration that "looks done" is exactly the case where these slip through
— compare screenshots, not the build log.

## Outline Changes

- `outline` now sets `outline-width: 1px` by default (consistent with `border` and `ring`)
- All `outline-*` utilities default `outline-style` to `solid` (no need to combine with `outline`)
- `outline-none` in v3 set an invisible outline for a11y (forced colors). Renamed to **`outline-hidden`**.
- New `outline-none` in v4 actually sets `outline-style: none`

```html
<!-- v3 -->
<input class="outline-none focus:ring-2" />

<!-- v4 -->
<input class="outline-hidden focus:ring-2" />
```

## Ring Default Changes

- **Width**: `ring` was 3px → now 1px. Use `ring-3` for old behavior.
- **Color**: `ring` was `blue-500` → now `currentColor`. Add `ring-blue-500` explicitly.

```html
<!-- v3 -->
<button class="ring ring-blue-500">

<!-- v4 (same visual) -->
<button class="ring-3 ring-blue-500">
```

**Compat bridge** (not recommended for new projects):
```css
@theme {
  --default-ring-width: 3px;
  --default-ring-color: var(--color-blue-500);
}
```

## Default Border Color

v3: `border-*` and `divide-*` defaulted to `gray-200`.
v4: Defaults to `currentColor`.

**Fix**: Always specify a color with border utilities:
```html
<div class="border border-gray-200">
```

**Compat bridge**:
```css
@layer base {
  *, ::after, ::before, ::backdrop, ::file-selector-button {
    border-color: var(--color-gray-200, currentColor);
  }
}
```

## Gradient Rename: `bg-gradient-*` → `bg-linear-*`

```html
<!-- v3 -->
<div class="bg-gradient-to-r from-red-500 to-blue-500">

<!-- v4 -->
<div class="bg-linear-to-r from-red-500 to-blue-500">
```

## Gradient Variant Preservation

In v3, overriding part of a gradient with a variant "reset" the entire gradient. In v4, values are preserved (consistent with other utilities).

```html
<!-- v3: dark mode lost the 'to-*' color -->
<div class="bg-gradient-to-r from-red-500 to-yellow-400 dark:from-blue-500">

<!-- v4: to-yellow-400 preserved. Use via-none to unset it -->
<div class="bg-linear-to-r from-red-500 via-orange-400 to-yellow-400 dark:via-none dark:from-blue-500 dark:to-teal-400">
```

## `space-*` and `divide-*` Selector Changes

Two things changed, not one: the combinator **and** the properties.

- Selector: `~` (general sibling) → `:not(:last-child)`, for performance on large pages.
- Properties: physical `margin-top` → **logical `margin-block-start`/`-end`**, driven by a
  `--tw-space-y-reverse` flag so `space-y-reverse` works without a second rule.

The logical part matters: unlike `ml-*`/`mr-*` (still physical → architecture.md), `space-*`
does respect writing mode.

```css
/* v3 */
.space-y-4 > :not([hidden]) ~ :not([hidden]) { margin-top: 1rem; }

/* v4 — actual output from 4.3.3 */
.space-y-4 > :not(:last-child) {
  --tw-space-y-reverse: 0;
  margin-block-start: calc(calc(var(--spacing) * 4) * var(--tw-space-y-reverse));
  margin-block-end:   calc(calc(var(--spacing) * 4) * calc(1 - var(--tw-space-y-reverse)));
}
```

**Impact**: May cause visual differences with inline elements or when adding custom margins to children. **Recommended fix**: migrate to `flex flex-col gap-4`.

## Preflight Changes

- Placeholder color: was `gray-400` → now `currentColor` at 50% opacity
- **Buttons use `cursor: default`**, not `cursor: pointer` (restore via `@layer base { button:not(:disabled) { cursor: pointer } }`)
- `<dialog>` margins are reset (restore centering with `dialog { margin: auto }`)
- The `hidden` attribute now beats display classes (`hidden` + `flex` = hidden; remove the attribute to show)
- Some minor border and form element resets updated

## Prefix Syntax Change

Prefixes look like variants and go at the start; configured via the import, not a config file:

```html
<div class="tw:flex tw:hover:bg-red-600">
```
```css
@import "tailwindcss" prefix(tw);
```

Generated CSS variables include the prefix (`--tw-font-display`).

## Important Modifier Moved to the End

```html
<!-- v3 — deprecated but still works -->
<div class="!flex">

<!-- v4 -->
<div class="flex!">
```

## Variant Stacking Order Flipped

v3 applied stacked variants right-to-left; v4 applies left-to-right (reads like CSS). Reverse order-sensitive stacks, e.g. `first:*:pt-0` → `*:first:pt-0`.

## CSS Variables in Arbitrary Values

```html
<!-- v3 -->
<div class="bg-[--brand-color]">

<!-- v4 — parentheses instead of brackets -->
<div class="bg-(--brand-color)">
```

## Commas in Arbitrary Grid/Object Values

`grid-cols-*`, `grid-rows-*`, `object-*` no longer convert commas to spaces. Use underscores: `grid-cols-[max-content_auto]`.

## `hover:` Only Applies on Hover-Capable Devices

The `hover` variant is wrapped in `@media (hover: hover)` in v4. Touch devices no longer trigger hover styles on tap.

**Escape hatch** if you depended on tap-hover:
```css
@custom-variant hover (&:hover);
```

## Individual Transform Properties

`rotate-*`, `scale-*`, `translate-*` now use CSS individual transform properties:

- `transform-none` does **not** reset them anymore — use `scale-none`, `rotate-none`, etc.
- Transitioning `transform` no longer transitions these — use e.g. `transition-[opacity,scale]`

## `theme()` Function Deprecated

Prefer `var(--color-red-500)` over `theme(colors.red.500)`. Where variables don't work (media queries), use variable-name syntax: `theme(--breakpoint-xl)`, not `theme(screens.xl)`.

## `resolveConfig()` Removed

Use generated CSS variables directly (e.g., pass `var(--color-blue-500)` to JS animation libs), or `getComputedStyle(document.documentElement).getPropertyValue('--shadow-xl')`.

## `@apply` in SFCs / CSS Modules Requires `@reference`

Stylesheets bundled separately from your main CSS file (Vue/Svelte/Astro `<style>` blocks, CSS modules) have no access to theme variables or custom utilities. Import without duplicating CSS:

```vue
<style>
  @reference "../../app.css";
  h1 { @apply text-2xl font-bold; }
</style>
```

Or skip `@apply` and use raw variables: `h1 { color: var(--color-red-500); }`.

## Sass / Less / Stylus Not Supported

Tailwind v4 is not designed to run through preprocessors — treat Tailwind as your preprocessor. Do not use Sass/Less/Stylus for stylesheets or `<style>` blocks processed by Tailwind.

## Other Removals

- `corePlugins` config option — gone
- JS config's `safelist`, `separator` options — unsupported (`safelist` → `@source inline()`)
- `transition` / `transition-colors` now include `outline-color` — set outline colors unconditionally if you transition focus outlines

## Container Configuration

The `container` utility no longer has `center` or `padding` config options. Extend it with `@utility`:

```css
@utility container {
  margin-inline: auto;
  padding-inline: 2rem;
}
```

## Dark Mode

v3 class-based dark mode (`darkMode: 'class'` in config) → v4 uses `@custom-variant`:

```css
@custom-variant dark (&:where(.dark, .dark *));
```

See [dark-mode.md](./dark-mode.md) for full details.

## Content Configuration

The `content` array in `tailwind.config.js` is removed. v4 auto-detects source files. Use `@source` to add/override. See [architecture.md](./architecture.md#content-detection).

## Plugin API

The JavaScript plugin API (`addUtilities`, `addComponents`, `plugin()`) is deprecated. Load v3-era plugins with `@plugin "plugin-name"`. See [ecosystem.md](./ecosystem.md).
