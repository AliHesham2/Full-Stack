# Tailwind CSS v4 — Best Practices

## Component Extraction Over `@apply`

**Prefer extracting components in your framework** (React/Vue/Svelte) over `@apply`:

```tsx
// ✅ Always — a framework component, styled with the project's semantic tokens
function Button({ children }) {
  return (
    <button className="px-4 py-2 bg-primary text-primary-foreground rounded-md hover:bg-primary/90 transition-colors">
      {children}
    </button>
  );
}
```

**Never** a component class (`.btn { @apply … }` in `@layer components`): in this project a
repeated style becomes a component in `components/ui/`, with variants built by `cva`
(frontend-guide/02-ui/components.md).

**Why**: Framework components carry semantic meaning, support props, types, and composition. `@apply` creates invisible CSS dependencies.

## Use `gap` Over `space-*`

```html
<!-- ⚠️ Works, but it sets margins on children: v4 compiles space-y-4 to
     `.space-y-4 > :not(:last-child) { margin-block-start: … }` (verified on 4.3.3),
     so any child with its own margin fights it, and it breaks with flex-wrap. -->
<div class="space-y-4">
  <div>Item</div>
  <div>Item</div>
</div>

<!-- ✅ Preferred — gap is more predictable and composable -->
<div class="flex flex-col gap-4">
  <div>Item</div>
  <div>Item</div>
</div>
```

Source: Evil Martians — https://evilmartians.com/chronicles/5-best-practices-for-preventing-chaos-in-tailwind-css

## Name Design Tokens, Not Colors

```css
@theme {
  /* ❌ Naming by color */
  --color-blue-500: oklch(0.55 0.2 250);

  /* ✅ Naming by role */
  --color-primary-500: oklch(0.55 0.2 250);
  --color-danger-500: oklch(0.6 0.22 25);
  --color-surface: oklch(1 0 0);
  --color-on-surface: oklch(0.15 0.01 250);
}
```

**Why**: When you change your brand color, you change one `--color-primary-500` value. With `--color-blue-500`, the name lies when the color changes.

## Always Specify Border Colors

v4's preflight sets `border: 0 solid` on every element and **never sets a color**, so
`border-color` falls back to the CSS initial value, `currentColor`. v3's preflight set it to
`gray-200`. Verified on 4.3.3 — the universal reset is:

```css
*, ::after, ::before, ::backdrop, ::file-selector-button {
  box-sizing: border-box; margin: 0; padding: 0; border: 0 solid;
}
```

and `.border` emits only `border-style` + `border-width: 1px`. So an unstyled `border` takes
the element's text color — usually near-black on light backgrounds, which reads as "the
border is too dark" rather than as a missing class. Be explicit:

```html
<!-- ✅ Explicit color -->
<div class="border border-gray-200">

<!-- ❌ Relies on currentColor — may look wrong -->
<div class="border">
```

## Avoid Extracting Utility Combinations Prematurely

```html
<!-- ❌ Don't create a component for this — it's one instance -->
<div class="flex items-center gap-3 p-4 rounded-lg bg-white shadow-sm border border-gray-200">

<!-- ✅ Component extraction makes sense when reused 3+ times with shared behavior -->
```

**Guideline**: If a pattern appears once, inline the utilities. If it appears twice, notice it. If it appears three or more times with the same behavior, extract a component.

## Use Arbitrary Values Sparingly

```html
<!-- ❌ Overuse of arbitrary values → inconsistent design system -->
<div class="w-[347px] h-[183px] rounded-[13px] pl-[22px]">

<!-- ✅ Theme values only — no brackets at all -->
<div class="w-88 h-44 rounded-lg p-6">
```

`w-88` is not a special case: v4 derives spacing utilities from the `--spacing` multiplier,
so `w-88` compiles to `width: calc(var(--spacing) * 88)` = **22rem** with the default
`0.25rem`. Verified on 4.3.3. Before reaching for `w-[22rem]`, divide by `--spacing` and
check whether the plain number already exists — it usually does.

Arbitrary values (`[...]`) are for one-off cases. If a value appears more than twice, add it
to `@theme`.

## Responsive Design: Mobile First

```html
<!-- ✅ Default = mobile, md: = tablet+ -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4">
```

Use `max-*` container query variants for component-level responsiveness:

```html
<div class="@container">
  <div class="grid grid-cols-1 @sm:grid-cols-3 @max-lg:grid-cols-2">
  </div>
</div>
```

## Performance: Large Pages

### `space-*` / `divide-*` Selector Impact
The v4 selector (`> :not(:last-child)`) is faster than v3's (`> :not([hidden]) ~ :not([hidden])`), but on very large lists (1000+ items), prefer `flex gap-*` or `grid gap-*`.

### Minimize Utility Count Per Element
There's no hard limit, but >15 utilities on a single element is a smell. Extract a component.

### Build Performance
- In Next.js, Tailwind runs through `@tailwindcss/postcss` (`create-next-app` sets it up); no Vite plugin
- Use `@source not` to exclude large generated directories
- Incremental builds are in the microsecond range — HMR should feel instant

## Anti-Patterns

### 1. Using the Default Palette Without Customization

```css
/* ❌ Using raw Tailwind blue everywhere */
<div class="bg-blue-500">

/* ✅ Semantic roles, no shade numbers: the value lives in theme.css, the name in tokens.css */
<div class="bg-primary text-primary-foreground">
```
The project's token names and scales: frontend-guide/02-ui/design-tokens.md.

### 2. `@apply` for Everything

Leads to CSS that's hard to understand without reading the Tailwind docs. In this project
`@apply` appears only in the base layer of `globals.css` (e.g. `@apply border-border`), never to
build component classes.

### 3. Ignoring RTL

"v4 uses logical properties internally" is only half true — see
[architecture.md](./architecture.md) for the compiled proof. Only the **axis shorthands**
(`px`/`py`/`mx`/`my`), `space-*`/`divide-*`, and the dedicated logical families are
logical. Every single-side utility stays physical:

| Physical (does NOT flip) | Logical replacement |
|---|---|
| `ml-*` / `mr-*` | `ms-*` / `me-*` |
| `pl-*` / `pr-*` | `ps-*` / `pe-*` |
| `left-*` / `right-*` | `inset-s-*` / `inset-e-*` |
| `border-l-*` / `border-r-*` | `border-s-*` / `border-e-*` |
| `text-left` / `text-right` | `text-start` / `text-end` |
| `rounded-l-*` / `rounded-r-*` | `rounded-s-*` / `rounded-e-*` |

The physical ones compile and look correct in LTR, so **nothing in the build or the browser
warns you** — the bug only appears when someone loads the RTL locale. If the product ships
Arabic or Hebrew, ban the left column in code review; that substitution is the entire RTL
story in Tailwind.

### 4. Not Using Container Queries Where Appropriate

If a card/sidebar/widget changes layout based on its own width (not the viewport), use container queries:

```html
<aside class="@container w-full">
  <div class="flex flex-col @md:flex-row">
  </div>
</aside>
```

### 5. Over-Relying on `!important`

```html
<!-- ❌ v4 puts ! at the END of the utility; leading ! is deprecated v3 syntax -->
<div class="!flex !w-full">

<!-- ✅ Fix specificity issues at the source -->
```

If you need `!important`, there's likely a specificity or cascade layers issue in your CSS. When truly unavoidable, v4 syntax appends it: `flex!`.

## DX Tooling

- **`prettier-plugin-tailwindcss`** — official Prettier plugin; sorts classes in a consistent canonical order and removes duplicates/extra whitespace. Eliminates class-order diffs in code review.
- **Tailwind CSS IntelliSense (VS Code)** — autocomplete, linting for unknown/conflicting classes, hover previews, syntax highlighting inside class strings.
- Both are first-party and maintained by Tailwind Labs. Configure once per repo (`.prettierrc` with `"plugins": ["prettier-plugin-tailwindcss"]`).
