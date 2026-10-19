# Tailwind CSS v4 — Architecture

## Oxide Engine

Tailwind CSS v4 is a ground-up rewrite built in Rust, codenamed **Oxide**. It replaces the v3 JavaScript-based compiler.

**Why it matters**: The Rust engine enables microsecond-class incremental builds and dramatically faster full builds.

### Performance Benchmarks (Tailwind Labs, Catalyst project)

| Metric | v3.4 | v4.0 | Improvement |
|--------|------|------|-------------|
| Full build | 378ms | 100ms | 3.78x |
| Incremental (new CSS) | 44ms | 5ms | 8.8x |
| Incremental (no new CSS) | 35ms | 192μs | 182x |

Source: https://tailwindcss.com/blog/tailwindcss-v4

The "no new CSS" case is the most impactful in practice — most HMR triggers during development hit this path because you're reusing existing utility classes.

### Build Pipeline

v4 uses **Lightning CSS** internally for:
- Vendor prefixing (no separate `autoprefixer` needed)
- Modern CSS syntax transforms (for broader browser compat)
- CSS bundling/nesting resolution

This means `postcss-import` and `autoprefixer` are no longer needed as separate dependencies.

## Modern CSS Foundations

v4 is built on cutting-edge CSS features. This is why browser requirements are stricter than v3.

### Browser Requirements

- Safari 16.4+
- Chrome 111+
- Firefox 128+

If you need older browser support, stay on v3.4. Since v4.1, output degrades more gracefully in old browsers (OKLAB colors, `@property`-dependent features, and opacity modifiers get fallbacks), but the framework is still *designed* for modern browsers and some things won't render perfectly. There is no full legacy compatibility mode.

### Key CSS Features Used

**Cascade Layers** (`@layer`)

Tailwind emits styles into three layers with defined precedence:

```css
@layer theme, base, components, utilities;
@layer utilities {
  .mx-6 { margin-inline: calc(var(--spacing) * 6); }
}
```

This gives deterministic specificity ordering without relying on source order hacks.

**Registered Custom Properties** (`@property`)

```css
@property --tw-gradient-from {
  syntax: "<color>";
  inherits: false;
  initial-value: #0000;
}
```

Enables animating gradients and improves paint performance on large pages by telling the browser the type of each custom property.

**`color-mix()`**

```css
.bg-blue-500/50 {
  background-color: color-mix(in oklab, var(--color-blue-500) 50%, transparent);
}
```

Replaces the old `bg-opacity-*` pattern. Works with any color value, including CSS variables and `currentColor`. This is why `bg-blue-500/50` works in v4 — it's not opacity on the element, it's color mixing.

**Logical Properties — only the axis shorthands, not everything**

This is widely overstated, including in earlier drafts of this file. Compiled from 4.3.3:

| Class | Emits | Flips in RTL? |
|---|---|---|
| `px-4` | `padding-inline` | ✅ |
| `py-2` | `padding-block` | ✅ |
| `ms-4` / `me-4` | `margin-inline-start` / `-end` | ✅ |
| `mt-8` / `mb-4` | `margin-top` / `margin-bottom` | — (no horizontal axis) |
| **`ml-2` / `mr-2`** | **`margin-left` / `margin-right`** | ❌ **stays physical** |
| `w-17` / `h-8` | `width` / `height` | ❌ physical |

So "v4 uses logical properties, RTL just works" is **false**. The axis shorthands
(`px`/`py`/`mx`/`my`) and the dedicated logical families (`ms`/`me`/`ps`/`pe`,
`inset-s`/`inset-e`, `border-s`/`border-e`, `text-start`/`text-end`) are logical. Every
single-side `ml`/`mr`/`pl`/`pr` and all sizing utilities are still physical.

**For an RTL-capable UI, ban `ml-*`/`mr-*`/`pl-*`/`pr-*` in review** and use `ms-*`/`me-*`
instead — that substitution is the whole RTL story in Tailwind, and nothing in the build
warns you when you get it wrong.

## Content Detection

v4 automatically discovers template files — no `content` array configuration needed.

### How It Works

1. Scans all files in the project root
2. **Respects `.gitignore`** — anything git-ignored is excluded automatically
3. **Filters binary extensions** — images, videos, `.zip`, etc. are skipped
4. Heuristics identify template-like files worth scanning

### Overriding Detection

```css
@import "tailwindcss";

/* Add a source excluded by default (e.g., a node_module) */
@source "../node_modules/@my-company/ui-lib";

/* Explicitly ignore a path */
@source not "./src/legacy";

/* Safelist classes not found in source */
@source inline("my-dynamic-class another-class");
```

The `@source` directive uses the same heuristics — binary files are still filtered even when explicitly adding a path.

## CSS Output Structure

v4 generates CSS organized into cascade layers:

1. **`@layer theme`** — CSS custom properties for all design tokens (`:root` block)
2. **`@layer base`** — Preflight (reset/normalize), plus any `@layer base` blocks in user CSS
3. **`@layer components`** — User-defined component styles (via `@layer components` in user CSS)
4. **`@layer utilities`** — All utility classes

Verified against a 4.3.3 build — the emitted declaration is exactly:

```css
@layer properties;
@layer theme, base, components, utilities;
```

Note the extra, undocumented **`properties`** layer declared *first*. Tailwind uses it for
`@property` registrations and their fallbacks; because it is declared before the others it
loses to all of them, so it never interferes with your styles — but it will show up in
DevTools and in any tooling that asserts on layer order.

This layering means:
- Utilities always win over components and base styles
- Components win over base styles
- Theme variables are always available but don't create specificity conflicts
- Users can interleave their own `@layer` rules naturally

## Dynamic Utility Generation

v4 derives many utilities dynamically from theme variables rather than generating every possible permutation at build time.

### Spacing Example

The `--spacing` variable (default `0.25rem`) is a multiplier:

```css
@layer theme {
  :root { --spacing: 0.25rem; }
}
@layer utilities {
  .mt-8 { margin-top: calc(var(--spacing) * 8); }
  .w-17 { width: calc(var(--spacing) * 17); }  /* No config needed */
}
```

This means `mt-15`, `w-100`, `px-3` etc. all work out of the box without extending the config.

### Grid Columns Example

```html
<div class="grid grid-cols-15">  <!-- Works, no config -->
```

### Data Attribute Variants

```html
<div data-current class="opacity-75 data-current:opacity-100">
```

Boolean data attributes work as variants without any configuration.
