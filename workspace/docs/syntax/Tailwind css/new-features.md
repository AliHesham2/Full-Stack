# Tailwind CSS v4 — New Features

## Container Queries (v4.0)

Built into core — no plugin needed. Uses `@container` class on the parent, then `@sm:`, `@md:`, etc. for container query variants.

```html
<div class="@container">
  <!-- Stack on small containers, 3 cols on medium, 4 on large -->
  <div class="grid grid-cols-1 @sm:grid-cols-3 @lg:grid-cols-4">
  </div>

  <!-- Max-width container queries -->
  <div class="flex @max-md:grid-cols-1">
  </div>

  <!-- Range queries -->
  <div class="flex @min-md:@max-xl:hidden">
  </div>
</div>
```

**Note**: The `@` prefix distinguishes container query variants from media query variants (`sm:`, `md:` use viewport width; `@sm:`, `@md:` use container width).

Source: https://tailwindcss.com/blog/tailwindcss-v4

## 3D Transforms (v4.0)

New utilities for 3D transformations:

```html
<div class="perspective-distant">
  <article class="rotate-x-12 rotate-y-5 translate-z-10 transform-3d">
  </article>
</div>
```

Key utilities: `perspective-*`, `perspective-origin-*`, `rotate-x-*`, `rotate-y-*`, `rotate-z-*`, `translate-z-*`, `scale-z-*`, `backface-*`, and **`transform-3d` / `transform-flat`** for `transform-style`. There is no `transform-style-*` utility — verified against 4.3.3, it emits nothing.

## Expanded Gradients (v4.0)

### Angle-based linear gradients

```html
<div class="bg-linear-45 from-indigo-500 via-purple-500 to-pink-500">
```

### Rename: `bg-gradient-*` → `bg-linear-*`

### Interpolation modifiers

```html
<!-- Default: OKLAB interpolation (vivid) -->
<div class="bg-linear-to-r from-indigo-500 to-teal-400">

<!-- Force sRGB interpolation -->
<div class="bg-linear-to-r/srgb from-indigo-500 to-teal-400">

<!-- Force OKLCH interpolation -->
<div class="bg-linear-to-r/oklch from-indigo-500 to-teal-400">
```

Polar color spaces (OKLCH, HSL) produce more vivid gradients when start/end colors are far apart on the color wheel.

### Radial and conic gradients

```html
<!-- Radial gradient with position -->
<div class="bg-radial-[at_25%_25%] from-white to-zinc-900 to-75%">

<!-- Conic gradient with hue interpolation -->
<div class="bg-conic/[in_hsl_longer_hue] from-red-600 to-red-600">
```

## Opacity Modifiers via `color-mix()` (v4.0)

All color utilities now support the `/opacity` syntax natively (not just predefined values):

```html
<div class="bg-blue-500/30 text-red-600/70 border-green-400/50">
```

This uses `color-mix(in oklab, color 30%, transparent)` internally — works with any color including CSS variables and `currentColor`.

## Dynamic Utility Values (v4.0)

Many utilities accept any numeric value without configuration:

```html
<div class="grid grid-cols-15 w-64 h-17 mt-[3.5rem]">
```

Spacing utilities (`p-*`, `m-*`, `w-*`, `h-*`, `gap-*`, etc.) are derived from `--spacing` multiplier.

## `@starting-style` Support (v4.0)

Animate elements appearing on the page without JavaScript:

```html
<button popovertarget="my-popover">Check for updates</button>
<div popover id="my-popover" class="transition-discrete starting:open:opacity-0">
</div>
```

**Support**: `@starting-style` has been **Baseline newly available since 2024-08-06**
(Chrome 117+, Firefox 129+, Safari 17.5+) — it is not "still maturing". Where it is missing
the element simply appears without the entry transition, so no gate is needed.

Verified in 4.3.3: `starting:opacity-0`, `starting:open:opacity-0` and `transition-discrete`
all compile, and the output contains a real `@starting-style` block.

## `not-*` Variant (v4.0)

Negate any variant, selector, or query:

```html
<!-- Not hovered -->
<div class="not-hover:opacity-75">

<!-- Not supporting a feature -->
<div class="not-supports-hanging-punctuation:px-4">
```

## `in-*` Variant (v4.0)

Like `group-*` but without needing a `group` class — targets descendants of the current element:

```html
<div class="in-focus:ring-2">
  <input class="focus:outline-none" />
  <!-- When input is focused, the parent div gets ring-2 -->
</div>
```

## Other New Variants (shipped across v4.0/v4.1)

- **`**:`** — style all descendants (`**:underline`); **`*:`** targets direct children only (`*:font-bold`). There is no `descendant:` variant — verified against 4.3.3, it emits nothing.
- **`nth-*`** — Target nth-child (`nth-of-type-3:`, `nth-last-2:`, etc.) (v4.0)
- **`inert`** — Target elements with `inert` attribute (v4.0)
- **`open`** — Target open popovers (via `:popover-open`) (v4.0)
- **`details-content:*`** — Style the content box of `<details>` elements (v4.1)
- **`noscript`** — Styles apply when JS is disabled (v4.1)
- **`user-valid`** / **`user-invalid`** — Validation styling only after user interaction (v4.1)
- **`inverted-colors`** — Matches OS inverted-colors mode (v4.1)
- **`pointer-*`** / **`any-pointer-*`** — Target input devices (`pointer-coarse`, `pointer-fine`) (v4.1)

## New Utilities (v4.0)

- **`inset-shadow-*`** / **`inset-ring-*`** — Up to 4 shadow layers per element
- **`field-sizing-*`** — Auto-resizing textareas without JS
- **`scheme-*`** — sets the CSS `color-scheme` property so scrollbars and native form controls follow your theme (`scheme-light`, `scheme-dark`, `scheme-light-dark`, `scheme-normal`). The utility is **not** called `color-scheme-*`.
- **`font-stretch-*`** — Variable font width control

## Alignment & Wrapping (v4.1)

```html
<!-- Safe alignment: falls back to start when content would overflow -->
<ul class="flex justify-center-safe gap-2">

<!-- Align to last text line baseline -->
<div class="grid grid-cols-[1fr_auto] items-baseline-last">
  <p class="self-baseline-last">...</p>
</div>

<!-- Overflow wrap: class names are wrap-*, not overflow-wrap-* -->
<p class="wrap-break-word">Long words break instead of overflowing</p>
<div class="flex"><p class="wrap-anywhere">Also breaks during intrinsic sizing — replaces min-width:0 hacks in flex children</p></div>
```

Source: https://tailwindcss.com/blog/tailwindcss-v4-1

## Text Shadows (v4.1)

```html
<p class="text-shadow-sm">Default shadow</p>
<p class="text-shadow-lg text-shadow-blue-500">Blue shadow</p>
<p class="text-shadow-lg/30">30% opacity shadow</p>
```

Five built-in sizes: `text-shadow-2xs`, `text-shadow-xs`, `text-shadow-sm`, `text-shadow-md`, `text-shadow-lg`.

## Mask Utilities (v4.1)

Composable, purpose-built for masking — per-side (`mask-t-from-*`, `mask-b-from-*`, `mask-l-*`, `mask-r-*`), axis (`mask-x-*`, `mask-y-*`), or shape (`mask-radial-*`, `mask-conic-*`, `mask-linear-*`):

```html
<!-- Radial mask with from/to stops -->
<img class="mask-radial-from-transparent mask-radial-to-black" src="...">

<!-- Side masks compose with shape masks -->
<div class="mask-b-from-50% mask-radial-[50%_90%] mask-radial-from-80% bg-cover"></div>
```

**Note**: There is no bare `mask-from-*` — masks are always scoped to a side, axis, or shape.

## Colored Drop Shadows (v4.1)

```html
<div class="drop-shadow-blue-500">
<div class="drop-shadow-lg drop-shadow-red-500">
```

## `@source` Enhancements (v4.1)

```css
@source not "./src/generated/**";    /* Ignore paths */
@source inline("safelist-class");      /* Safelist classes */
```

## P3 Color Palette

The entire default color palette now uses OKLCH, producing more vivid colors on P3-capable displays while maintaining similar appearance on sRGB displays. The balance between colors was preserved from v3.

## Automatic `@import` Bundling

No need for `postcss-import` — v4 handles CSS `@import` rules natively and with tighter integration to the engine for better performance.

---

# v4.2 (2026-02-18)

## Four New Neutral Palettes

`mauve`, `olive`, `mist`, `taupe` — neutral-adjacent palettes alongside slate/gray/zinc/neutral/stone:

```html
<div class="bg-mauve-950 text-mauve-100">
<div class="border border-mist-200 shadow-taupe-950/10">
```

Source: https://tailwindcss.com/blog/tailwindcss-v4-3 (v4.2 section)

## First-Class Webpack Plugin

New `@tailwindcss/webpack` loader — skips the PostCSS detour entirely, measured 2x+ faster in large webpack/Turbopack projects (e.g., 932ms → 429ms on tailwindcss.com docs):

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/i, use: [MiniCssExtractPlugin.loader, "css-loader", "@tailwindcss/webpack"] },
    ],
  },
};
```

Works through Turbopack's webpack-loader compatibility layer too — significant for Next.js.

## Logical Property Utilities (Block Direction + Sizing + Insets)

Writing-mode/direction aware utilities for international layouts:

```html
<!-- Block-start/end: margin, padding, scroll-padding, scroll-margin, borders -->
<div class="mbs-6 mbe-2 pbs-4 pbe-8 scroll-pbs-12 border-bs border-be-2">

<!-- Logical sizing: inline-size/block-size + min/max variants -->
<div class="inline-full block-64 min-inline-0 max-block-screen">

<!-- Logical insets -->
<div class="absolute inset-s-0 inset-e-4 inset-bs-2 inset-be-8">
```

**Deprecated in v4.2**: `start-*` / `end-*` → use `inset-s-*` / `inset-e-*`.

## `font-features-*`

Escape hatch for OpenType features without dedicated utilities (prefer `tabular-nums` etc. when they exist):

```html
<div class='font-features-["tnum"]'>
```

---

# v4.3 (2026-05-08)

## Scrollbar Utilities

First-party styling of the standard CSS scrollbar APIs:

```html
<!-- scrollbar-width + scrollbar-color -->
<div class="scrollbar-thin scrollbar-thumb-sky-700 scrollbar-track-sky-100 overflow-auto">

<!-- Prevent layout shift when scrollbars appear -->
<div class="scrollbar-gutter-stable overflow-auto">
```

Widths: `scrollbar-auto`, `scrollbar-thin`, `scrollbar-none`. Gutter: `scrollbar-gutter-auto|stable|both`. Colors accept opacity modifiers (`scrollbar-thumb-slate-900/60`).

Source: https://tailwindcss.com/blog/tailwindcss-v4-3

## `@container-size`

v4.0's `@container` creates an *inline-size* container, so height-based units (`cqb`, `cqh`) have nothing to measure. `@container-size` creates a full size container:

```html
<div class="@container-size">
  <div class="h-[50cqb]"></div>
</div>
```

Nameable via `@container-size/{name}`, same as `@container/{name}`.

## `zoom-*` and `tab-*`

```html
<div class="zoom-125">          <!-- CSS zoom property, all browsers since 2024 -->
<pre class="tab-2">             <!-- tab-size, defaults sensible, arbitrary values work -->
```

## Stacked + Compound `@variant` in CSS

```css
.button {
  background: var(--color-sky-500);
  @variant hover:focus {        /* stacked */
    background: var(--color-sky-600);
  }
  @variant hover, focus {       /* compound: shares one block */
    background: var(--color-sky-600);
  }
}
```

## Default Values for Functional Utilities

Custom utilities can now fall back when no value is given:

```css
@utility tab-* {
  tab-size: --value(integer, --default(4));
}
```
```html
<pre class="tab">...</pre>    <!-- tab-size: 4 -->
<pre class="tab-2">...</pre>  <!-- tab-size: 2 -->
```
