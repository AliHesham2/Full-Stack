# Modals and Dialogs

Modals interrupt the user's flow. Use them sparingly and correctly.

## When to Use a Modal

- **Confirmation of a destructive action** ("Are you sure you want to delete this?")
- **Focused sub-task** that doesn't warrant a new page (image crop, quick edit)
- **Alert** that requires the user to acknowledge ("Your session will expire in 5 minutes")

## When NOT to Use a Modal

- **Multi-step forms** (use a dedicated page)
- **Complex content** that requires reading and reference to the page behind (users can't see the page behind a modal)
- **Frequently needed information** (use a side panel or drawer)
- **Progress indication** (use inline feedback)
- **Navigation** (modals that navigate break the back button)

## Modal Anatomy

```
┌─────────────────────────────────┐
│  Title                    [X]  │  ← Header: title + close
├─────────────────────────────────┤
│                                 │
│  Content area                   │  ← Body: scrollable if needed
│  (scrollable)                   │
│                                 │
├─────────────────────────────────┤
│        [Cancel]  [Confirm]      │  ← Footer: actions
└─────────────────────────────────┘
```

## Modal Rules

1. **Close mechanisms:** Provide at least 2: X button in corner, Cancel button, clicking the overlay (for non-destructive modals). For destructive confirmations, remove the overlay-click-to-close to prevent accidental dismissal.
2. **Focus trap:** Tab key must cycle within the modal. Focus must move to the modal when it opens and return to the triggering element when closed. (Accessibility requirement.)
3. **Scroll lock:** Body scroll must be disabled when a modal is open.
4. **Size:** Default max-width ~480px. Don't make modals full-width on desktop unless the content requires it (image viewing, complex forms).
5. **Title:** Must clearly state the purpose. "Delete project" not "Are you sure?"
6. **Primary action button:** Matches the intent (red for delete, primary color for confirm).
7. **Escape key:** Closes the modal (except for destructive confirmations where it should be explicitly confirmed).

## Build on `<dialog>`, Not on JavaScript

The rules above describe *behaviour*. Do not implement that behaviour by hand — the platform
implements it, correctly, and a hand-rolled focus trap is one of the most common sources of
real accessibility bugs.

```html
<!-- Invoker Commands: see the support note below before using this form -->
<button commandfor="confirm" command="show-modal">Delete project</button>

<dialog id="confirm" aria-labelledby="confirm-title">
  <h2 id="confirm-title">Delete project</h2>
  <p>This cannot be undone.</p>
  <form method="dialog">
    <button value="cancel" autofocus>Cancel</button>
    <button value="delete" class="danger">Delete</button>
  </form>
</dialog>
```

`dialog.showModal()` (or the declarative `command="show-modal"` above) gives you, for free:

| Rule above | What `showModal()` does |
|---|---|
| Focus trap | Tab is confined to the dialog automatically |
| Focus return | Focus returns to the invoker on close |
| Escape closes | Built in, fires a `cancel` event you can `preventDefault()` |
| Inert background | Everything outside becomes inert — not just visually, but for AT |
| Overlay | `::backdrop` pseudo-element, styleable |
| Role/semantics | Implicit `dialog` role and modal semantics |

**What is still yours to do:**

- **Scroll lock.** `showModal()` does *not* lock body scroll. Use
  `html:has(dialog[open]) { overflow: hidden; }`.
- **The accessible name.** Point `aria-labelledby` at the heading, as above.
- **Destructive confirmations.** Rule 7 says Escape should not silently dismiss these —
  handle the `cancel` event and call `preventDefault()`.
- **Initial focus.** The browser picks the first focusable element; for a destructive dialog
  put `autofocus` on **Cancel**, never on the destructive button.

### Modal vs. popover — pick the right primitive

Not everything that floats is a modal. Choosing wrong is why menus trap keyboard users.

| Need | Primitive | Blocks the page? |
|---|---|---|
| Must decide before continuing | `<dialog>` + `showModal()` | Yes |
| Non-blocking panel, dismiss on outside click / Esc | `popover` attribute (`popover="auto"`) | No |
| Menu, tooltip, combobox listbox | `popover="auto"` + the matching ARIA pattern | No |
| Side panel the user reads *alongside* the page | Drawer, non-modal — see below | No |

The Popover API gives light-dismiss, Escape, and top-layer stacking without a focus trap —
which is exactly right for a menu, and exactly wrong for a confirmation.

> **Do not stack mechanisms.** `showModal()` already manages inertness; adding your own
> `inert` toggling or a hand-built overlay on top of it makes the two fight. Pick one.

### Support status — read before choosing the declarative form

Checked against the `web-features` Baseline dataset (v3.36.0):

| Feature | Baseline | Since |
|---|---|---|
| `<dialog>` / `showModal()` | **Widely available** | 2024-09-14 |
| `popover` attribute | Newly available | 2025-01-27 |
| **Invoker Commands** (`command` / `commandfor`) | **Newly available** | **2025-12-12** |
| `:has()` (for the scroll-lock rule) | Widely available | 2026-06-19 |
| `dialog closedby` attribute | **Not Baseline** | — |

`<dialog>` itself is the safe, boring choice — use it without hesitation.

**Invoker Commands are the exception that needs a decision.** They became Baseline only in
December 2025, and unlike most modern CSS they **do not degrade gracefully**: on an engine
without support, `<button commandfor=…>` is an inert button and the dialog simply never
opens. That is a broken feature, not a lesser experience. So:

- Broad/unknown audience → open the dialog from JS (`el.showModal()`), which has been widely
  available since 2024.
- Controlled or modern-only audience → the declarative form is genuinely better (no JS, no
  wiring) and is the direction the platform is going.

Do not use the `closedby` attribute yet — it is not Baseline.

> **Cross-reference:** the HTML-level details — `::backdrop`, Invoker Commands
> (`command`/`commandfor`), `popover` states, and browser support — are in the sibling
> `html/` knowledge folder, `overlays.md`. This file covers *when and why*; that one covers
> *exactly how*.

## Drawer / Side Panel (Alternative to Modal)

When you need more space or the user needs to reference the page behind:

- Slides in from the right (LTR)
- Width: 320–480px on desktop, full-width on mobile
- Can coexist with the page content (the page narrows)
- Better for: detail views, edit forms, context panels, filters
- **A drawer is usually non-modal.** If the user can still read and interact with the page
  behind it, do **not** use `showModal()` — use `dialog.show()` or `popover`. Using
  `showModal()` makes the page behind inert, which contradicts the whole reason to choose a
  drawer. If the drawer *does* block the page, it is a modal wearing a different animation,
  and every modal rule applies.

## Toast vs. Modal vs. Inline

| Need | Use |
|------|-----|
| Acknowledge a completed action | Toast |
| Confirm a destructive action | Modal |
| Show an error that blocks progress | Modal or inline banner |
| Collect information | Modal (brief) or page (complex) |
| Show details about an item | Drawer/side panel |