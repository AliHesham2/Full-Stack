# When to Use What: Pattern Selection Guide

Decision trees for common design choices. Use this when you're unsure which pattern to apply.

## Navigation Decision

```
How many top-level sections?
├── 2–4 → Tab bar
├── 3–7 → Horizontal top nav (default for web apps)
├── 5–15 → Sidebar nav
└── 15+ → Sidebar with collapsible groups

Is this mobile?
├── Yes → Bottom tab bar (3–5 items) + hamburger for rest
└── No → See above
```
→ See `components/navigation.md`

## Content Container Decision

```
Does the content have images or rich media?
├── Yes → Cards
└── No →
    Are items independent (browsed) or sequential (scanned)?
    ├── Independent → Cards (if multiple attributes) or List (if 1–2 attributes)
    └── Sequential → List

Do users need to compare specific attributes?
├── Yes → Table
└── No → Cards or List
```
→ See `components/cards-lists.md`, `components/data-display.md`

## Feedback/Overlay Decision

```
Is the user's action complete?
├── Yes → Toast (non-critical) or inline confirmation (critical)
└── No →
    Does the user need to confirm before proceeding?
    ├── Yes, and it's destructive → Modal with confirm/cancel
    ├── Yes, and it's not destructive → Inline confirmation or modal
    └── No, but there's a sub-task → Modal (brief) or Drawer (complex)

Is there an error?
├── Field-level → Inline error message below the field
├── Page-level, non-blocking → Toast or inline banner
├── Page-level, blocking → Centered error state with retry
└── Requires user decision → Modal
```
→ See `interaction-design/feedback-systems.md`, `components/modals-dialogs.md`

## Friction Decision (Confirm vs. Undo vs. Type-to-Confirm)

The default is **undo**, not confirmation. A dialog taxes every user to prevent a rare mistake;
undo costs nothing until the mistake happens.

```
Can the action be reversed after the fact?
├── Yes, cheaply → Do it immediately + undo affordance (toast with Undo)
│                  NO dialog. This is the default.
└── No →
    How bad is the consequence?
    ├── Recoverable from a backup / soft-deleted → Undo, or a light confirm
    ├── Real loss, single object → Modal naming the object and the consequence
    ├── Real loss, many objects or another user's data → Modal + item count + scope
    └── Catastrophic and permanent (delete account, wipe workspace)
                     → Type-to-confirm (type the object's name)

Is the action merely unusual, not dangerous?
└── No dialog. Unusual is not the same as destructive.
```

Friction must be **proportional**. Type-to-confirm on a trivial action is condescending and
trains users to type past warnings; a bare dialog on account deletion is negligent. A dialog
that says only "Are you sure?" is worse than none — it confirms nothing and trains reflexive
clicking.

→ See `interaction-design/user-flows.md` (Destructive Flows), `components/modals-dialogs.md`,
`foundations/content-and-microcopy.md` (button wording)

## Onboarding Decision

```
Can a new user reach value without explanation?
├── Yes → Ship nothing. Design good empty states instead.
└── No →
    Is the missing knowledge needed everywhere, or at one moment?
    ├── One moment → Contextual hint at that moment (dismissible forever)
    └── Everywhere →
        Does the product literally not function unconfigured?
        ├── Yes → Setup wizard, minimum possible steps, resumable
        └── No → Empty states + progressive hints. Not a tour.

Considering an upfront tour or carousel?
└── Almost always the wrong answer. It is skipped, unread, and forgotten,
    and it delays time-to-first-value — the metric onboarding exists to improve.
```

**Empty states are the default onboarding surface**: they appear exactly where the user is
stuck, need no separate flow, and nothing has to be dismissed.

→ See `interaction-design/user-flows.md` (Onboarding Flows), `interaction-design/state-patterns.md`

## Responsive Axis Decision

```
What is adapting?
├── Page shell, global nav, overall density → Viewport media query
├── A component reused in more than one slot → Container query
│   (a component styled with media queries is only correct in one slot)
└── A single value that should scale smoothly → clamp(), no breakpoint at all
    (include a rem term in the preferred value, or you fail WCAG 1.4.4)
```

→ See `visual-design/responsive-strategy.md`, `visual-design/grid-systems.md`, `visual-design/typography.md`

## Overlay Primitive Decision

```
Must the user deal with this before continuing?
├── Yes → <dialog> + showModal()  (focus trap, Esc, inert background — all free)
└── No →
    Is it a menu, tooltip, combobox popup, or a light panel?
    ├── Yes → popover="auto"  (light-dismiss, Esc, top layer — NO focus trap)
    └── Is it a panel read alongside the page? → non-modal drawer (dialog.show())

Never hand-build a focus trap. Never combine showModal() with your own inert/overlay logic.
```

→ See `components/modals-dialogs.md` (includes the Baseline support table — Invoker Commands
are new and do **not** degrade gracefully)

## Loading Pattern Decision

```
How long will the load take?
├── < 300ms → No indicator (feels instant)
├── 300ms–1s → Spinner (for buttons/controls) or subtle skeleton
├── 1s–5s → Skeleton screen or progress bar
└── > 5s → Progress bar with estimated time + cancel option

Do you know the content shape?
├── Yes → Skeleton screen
└── No → Spinner with descriptive text
```
→ See `interaction-design/state-patterns.md`

## Typography Decision

```
What is the content?
├── Long-form reading (articles, docs) →
│   Base: 16–18px, line-height: 1.6, max-width: 65ch
│   Serif or humanist sans-serif
├── Application UI (dashboards, tools) →
│   Base: 13–14px, line-height: 1.5
│   Grotesque/neo-grotesque sans-serif
├── Marketing/landing page →
│   Base: 16–18px, display sizes: 40–72px+
│   Geometric or distinctive sans-serif
└── Code/data → Monospace for code, tabular nums for data
```
→ See `visual-design/typography.md`

## Density Decision

```
Who is the primary user?
├── Expert, daily use → High density (compact)
├── Mixed expertise → Medium density + density toggle
├── New or casual users → Low density (comfortable)
```
→ See `visual-design/spacing-composition.md`
