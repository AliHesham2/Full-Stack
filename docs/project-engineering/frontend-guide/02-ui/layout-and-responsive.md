# Layout & Responsive

**Philosophy: mobile first, fluid layouts, one component per purpose.** There are no separate
mobile and desktop versions: the same components adapt to the space they have.

## Breakpoints

Tailwind's defaults, unchanged:

| Prefix | From | Typical device |
|---|---|---|
| *(none)* | 0 | phone (this is the base, so there is no `xs`) |
| `sm:` | 640px | large phone |
| `md:` | 768px | tablet |
| `lg:` | 1024px | small laptop |
| `xl:` | 1280px | desktop |
| `2xl:` | 1536px | large screen |

Write the phone layout first, then add prefixes as the screen grows: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`.

## Two Mechanisms

- **Viewport breakpoints** (`md:`, `lg:`) for page-level layout, meaning anything that spans the
  page.
- **Container queries** for components that appear in spaces of different widths (a card in a
  sidebar vs in the main column). Kit components prefer them, because they cannot know where
  they will be placed. When in doubt, container queries are the safer choice.

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">…</div>   // page: viewport

<div className="@container">                                                     // component: container
  <div className="grid grid-cols-1 @min-[400px]:grid-cols-2">…</div>
</div>
```

> **v4 syntax:** container queries are built in. The arbitrary form is `@min-[400px]:` /
> `@max-[960px]:`, plus named steps `@sm:` `@md:` `@lg:`. The old `@[400px]:` is v3-plugin syntax:
> in v4 it generates nothing, silently, and the layout just never adapts.

**No device detection.** No `if (isMobile)` and no user-agent checks. CSS decides; when two
arrangements are needed, both are in the markup and CSS shows the right one.

## Layout System

| Piece | Built as | Rule |
|---|---|---|
| **Container** | `components/ui/container.tsx`: `mx-auto w-full max-w-7xl px-4 md:px-6 lg:px-8` | every page's content sits in it; pages never invent their own side padding |
| **Section** | vertical rhythm `py-12 md:py-16 lg:py-24` | spacing grows with the screen |
| **Stack / Grid** | plain Tailwind `flex` / `grid` + `gap-*` | not components; `grid` for layouts and card sets, `flex` for alignment inside a component |
| **App shell** | `app/(dashboard)/layout.tsx` composing Header + Sidebar + main | the layout arranges; it does not restyle components |

Responsive spacing steps: page side padding `px-4` → `md:px-6` → `lg:px-8`; gaps `gap-4` →
`md:gap-6`.

## Rules by Pattern

| Pattern | Phone | Tablet | Desktop |
|---|---|---|---|
| **Sidebar** | hidden; a menu button opens it in a **Sheet** | collapsed to icons | fixed, full width |
| **Header** | logo + menu button | logo + menu button or short nav | logo + nav + actions + profile |
| **Tables** | rows shown as **cards** | compact table | full table |
| **Forms** | 1 column | 2 columns for wide forms | 2 columns |
| **Dialogs** | bottom sheet: same Dialog, positioned at the bottom by `max-sm:` classes | centered | centered, `max-w-lg` (or `max-w-2xl` for big content) |
| **Buttons** | full width in forms and page actions | normal | normal |
| **Card grids** | 1 column | 2 | 3–4 |

How the table / card switch works: render the table from `md:` up (`hidden md:table`) and the
card list below it (`md:hidden`), from the same data. Horizontal scroll is the last resort, only
for genuinely wide numeric tables.

## Always

- **No fixed widths.** `w-full` + `max-w-*`, never `w-[500px]`.
- **Typography scales:** headings grow with the screen (`text-2xl md:text-3xl lg:text-4xl`).
- **Cards don't change shape.** What changes is the column count, padding and text size.
- **Touch targets: at least 44×44px** for anything clickable ([components](./components.md)).
- **Images:** `next/image` with a correct `sizes`, `object-cover`, a fixed aspect ratio (no layout
  shift), and lazy loading (the default) except the main above-the-fold image. See
  [performance](../04-quality/performance.md).
- **Every state is responsive:** loading, empty, error and success, not just the happy path.
- **Hide only with a replacement.** A nav hidden on phones is replaced by the Sheet, never just
  removed.
- **Lighter on phones:** fewer and cheaper animations, and heavy charts or media loaded later.
- **RTL:** logical utilities (`ps-*`, `me-*`, `inset-s-*`, `text-start`) so every layout mirrors
  for Arabic automatically. The full list is in [components](./components.md). In Tailwind 4.3,
  `start-*` / `end-*` were renamed `inset-s-*` / `inset-e-*`: use the new names.

## Decision Order

```
Viewport → choose the page layout (header / sidebar / content)
         → choose the grid (columns per breakpoint)
         → place sections
         → components adapt to their container (container queries)
```

The page decides **where** components go; each component decides **how it looks** in the space
it gets. A second version of a component (mobile vs desktop) exists only when the experience is
entirely different, as with the Sheet vs the fixed sidebar.
