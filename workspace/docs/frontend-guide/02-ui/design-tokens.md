# Design Tokens

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

**Goal:** no component knows a raw color or size. Components use semantic names (`bg-primary`,
`rounded-lg`, `p-4`). Values live in one file, so a rebrand or a new theme never touches a
component. The CSS wiring is in [theme-and-dark-mode](./theme-and-dark-mode.md).

```tsx
// ❌ knows a raw color and invented sizes
<div className="bg-blue-600 p-[13px] rounded-[10px]" />
// ✅ semantic tokens and scale steps only
<div className="bg-primary p-3 rounded-lg" />
```

**The rule:** if a value isn't a token, don't use it. No arbitrary values (`p-[13px]`,
`text-[#3b59f1]`, `leading-[1.8]`) in application code. The only exception is a one-off
positioning detail no token could express. If the design needs a new value, add the token first:
a value in `theme.css`, or a new name in `tokens.css`. If a design spec is missing a value, ask;
don't invent one.

## 1. Color: Raw Values vs Semantic Names

- **Raw values** (OKLCH) exist **only** in `src/styles/theme.css`, the per-project file.
- **Semantic names** are what components use. They describe a *role*, never a hue.

| Role | Tokens | Used for |
|---|---|---|
| Page | `background`, `foreground` | page background and default text |
| Surfaces | `card`, `card-foreground`, `popover`, `popover-foreground` | cards, menus, popovers |
| Brand | `primary`, `primary-foreground`, `secondary`, `secondary-foreground` | main and secondary actions |
| Quiet | `muted`, `muted-foreground`, `accent`, `accent-foreground` | subtle backgrounds, hints, hover fills |
| Status | `destructive`, `success`, `warning`, `info` (each with `-foreground`) | errors, confirmations, alerts |
| Lines | `border`, `input`, `ring` | borders, input outlines, focus rings |

- **Every fill has a `-foreground` pair.** Text on `bg-primary` is `text-primary-foreground`;
  never `text-white` on a colored fill, because it breaks in the other theme.
- **Names are shadcn/ui's** (plus `success` / `warning` / `info`), so shadcn components copy in
  with no renaming. `destructive` is the error / danger color.
- **Forbidden in components:** hue names (`blue`, `gray-900`), hex / rgb / hsl, arbitrary colors,
  and `dark:` color overrides ([theme-and-dark-mode](./theme-and-dark-mode.md)).
- **Transparency** is the opacity modifier on a semantic token, in steps of 10 (plus `/5`), for
  example `bg-primary/10`. This works because the values are OKLCH. Never hand-compute a lighter
  color.
- **Which color means what** (for example `destructive` vs `primary` for a button) comes from
  the design spec, not the implementer's taste.
- **Contrast:** every pair meets 4.5:1 for text in **both** themes ([accessibility](./accessibility.md)).

## 2. Typography

| Token | Utilities |
|---|---|
| Family | `font-sans` (body), `font-heading` (headings), `font-mono` (code). Nothing else |
| Size | `text-xs` `text-sm` `text-base` `text-lg` `text-xl` `text-2xl` `text-3xl` `text-4xl` |
| Weight | `font-normal` `font-medium` `font-semibold` `font-bold` |
| Line height | `leading-tight` `leading-snug` `leading-normal` `leading-relaxed` |
| Letter spacing | `tracking-tight` `tracking-normal` `tracking-wide` |

Headings scale with the screen (`text-2xl md:text-4xl`). A missing step becomes a `--text-*`
token. Note: the font-size namespace is `--text-*`; `--font-size-*` is silently ignored by
Tailwind v4.

## 3. Spacing

Tailwind's numeric scale on a **4px grid**, from one base (`--spacing: 0.25rem`). Allowed steps:
`0` `0.5` `1` `1.5` `2` `3` `4` `5` `6` `8` `10` `12` `16` `20` `24` `32` (`4` = 16px).

**No named spacing tokens** (`--spacing-md`): `sm`/`md`/`lg` are already size keys for `max-w-*`
and other utilities, and named spacing collides with them. If the design asks for a value off the
scale, first check whether a nearby step looks the same.

## 4. Radius, Shadows, Borders

- **Radius:** one variable per project (`--radius`, default `0.625rem`). The steps derive from it:
  `rounded-none` · `rounded-sm` · `rounded-md` · `rounded-lg` (= `--radius`) · `rounded-xl` ·
  `rounded-full`.
- **Shadows:** `shadow-xs` `shadow-sm` `shadow-md` `shadow-lg` `shadow-xl` only. A brand that needs
  different shadows overrides the values in `theme.css`.
- **Borders:** `border` (1px, the default) and `border-2`; `border-4` / `border-8` only for
  deliberate emphasis. The color is set globally to `border-border`.

## 5. Z-Index Layers

Named layers only; a raw number (`z-50`) is forbidden. Tailwind v4 has no z-index namespace, so
these are `@utility` classes in `tokens.css`.

| Utility | Value | For |
|---|---|---|
| `z-base` | 0 | normal flow |
| `z-dropdown` | 10 | dropdowns, select lists |
| `z-sticky` | 20 | sticky headers |
| `z-fixed` | 30 | fixed sidebars and bars |
| `z-overlay` | 40 | backdrops |
| `z-modal` | 50 | dialogs, drawers, sheets |
| `z-popover` | 60 | popovers and menus opened inside a modal |
| `z-tooltip` | 70 | tooltips |
| `z-toast` | 80 | toasts, always on top |

## 6. Motion

| Name | Utility | Use |
|---|---|---|
| Fast | `duration-150` | hover, focus, color changes |
| Normal | `duration-200` | menus, popovers, toggles |
| Slow | `duration-300` | dialogs, drawers |

`ease-out` for entering, `ease-in` for leaving, `ease-in-out` for movement. Every animation has
`motion-reduce:transition-none` ([accessibility](./accessibility.md)).

## 7. Breakpoints, Containers, Icons

- **Breakpoints:** Tailwind defaults (`sm` 640 · `md` 768 · `lg` 1024 · `xl` 1280 · `2xl` 1536).
  There is no `xs`, because unprefixed classes *are* mobile. Rules are in
  [layout-and-responsive](./layout-and-responsive.md).
- **Max widths:** `max-w-sm` … `max-w-7xl`, `max-w-full`. The page container is `max-w-7xl`.
- **Icons:** `size-3` (12px) · `size-4` (16px, default) · `size-5` (20px) · `size-6` (24px) ·
  `size-8` (32px). Always colored with `currentColor`.

## Applying the Design

- **Consistency:** two elements with the same purpose use the same tokens. A new card-like
  surface reuses `bg-card rounded-lg border p-4`, not similar-but-different values. Check whether
  a pattern already exists before styling anything.
- **Follow the spec literally.** Avoid generic AI-looking UI the design did not ask for:
  decorative gradients, emoji in headings, dividers everywhere, identical padding on every
  surface, everything centered. If the spec says the top padding is 8 and the bottom is 16, build
  that asymmetry.
- **Deviate only for:** (1) an accessibility failure such as insufficient contrast, (2) something
  that cannot be built performantly, (3) a state the design doesn't cover (empty, error, long
  text). Use existing tokens. The deviation is reviewed through Orchestrator and recorded in the approved design TODO/UI-n and affected Planner Output/software task: what was specified, what changes, why and the actual authority. It is never introduced silently in code.
- **Naming:** by role, never by value or hue: `primary`, `muted`, never `blue`, `gray`. That is
  what lets a project change its brand from blue to purple by editing one file.
