# Progressive Disclosure

Managing complexity by showing only what's relevant at each level. This is the primary tool for implementing Tesler's Law.

## The Principle

Progressive disclosure means revealing information and functionality gradually, as the user needs it. Show the essentials first; reveal details and advanced options on demand.

## The Five Mechanisms

### 1. Accordion / Collapsible Sections
Content sections that expand on click. Each section heading is visible; the content within is hidden until opened.

**When to use:** FAQ pages, settings panels, long-form content with distinct sections, sidebar navigation.

**When not to use:** When all sections are equally important and users need to see multiple at once (a table, a dashboard).

**Rules:**
- Show section headings at all times
- Allow multiple sections to be open simultaneously (don't force close-others unless there's a reason)
- Don't nest accordions more than 2 levels deep
- Animate height transitions (200–300ms, ease-in-out)

### 2. "Show More" / Expandable Content
A truncated preview with a link to reveal the full content.

**When to use:** Long text blocks, descriptions, reviews, comments.

**Rules:**
- Show enough content to be meaningful (3–5 lines, not one line)
- The "show more" control should be obviously different from the content (a link, not inline text)
- Consider whether the content should collapse again or stay open once expanded

### 3. Tooltips and Hover Reveal
Additional information appears on hover or focus.

**When to use:** Icon-only buttons, truncated text, form field help.

**Critical rules:**
- **Must work on focus too** (keyboard users can't hover)
- **Never put essential information only in a tooltip** — it's unreachable on touch devices and invisible to screen readers without interaction
- Keep tooltip content short (1–2 sentences)
- Appear after a 200–500ms delay

### Tooltips must satisfy WCAG 2.2 SC 1.4.13 (Level AA)

The criterion has three named requirements, and the obvious implementation fails one of them:

| Requirement | What it means | The common failure |
|---|---|---|
| **Dismissible** | Escape closes it **without** moving the pointer or focus | No Escape handler |
| **Hoverable** | The pointer can move **onto** the tooltip without it vanishing | **"Hide on mouseleave" of the trigger** |
| **Persistent** | Stays until the trigger is left, the user dismisses it, or the info goes stale | Auto-hiding on a timer |

> ⚠️ **Do not hide a tooltip the instant the cursor leaves the trigger.** That is the textbook
> *Hoverable* failure: a user who needs to move onto the tooltip — to read it slowly, to zoom,
> to select text — cannot, because it disappears in the gap between trigger and tooltip. Add a
> short close delay (~100–300ms) and keep it open while the pointer is over the tooltip itself.
> An earlier revision of this file said "disappear immediately when the cursor leaves"; that is
> a Level AA failure, and it contradicted
> [../accessibility/wcag-essentials.md](../accessibility/wcag-essentials.md), which states the
> rule correctly.

- **Never auto-hide on a timer.** A timeout is a *Persistent* failure and gives slow readers no
  way to finish.
- **The native `title` attribute is exempt** (the criterion excludes user-agent-rendered
  content) — but it is not a usable tooltip either: no touch support, inconsistent timing, and
  no styling. Use it for nothing you care about.
- If the tooltip contains anything interactive, it is not a tooltip — it is a popover
  → [../components/modals-dialogs.md](../components/modals-dialogs.md).

### 4. Modals and Dialogs
A separate layer that appears over the main content to handle a sub-task.

See `../components/modals-dialogs.md` for full guidelines.

### 5. Advanced / Optional Sections
A clearly labeled section that users can choose to open, usually indicated by "Advanced options", "More settings", or a chevron.

**When to use:** Form fields that most users don't need, configuration options, secondary filters.

**Rules:**
- The trigger must clearly communicate what will be revealed ("Advanced settings" > "More")
- Default to collapsed unless user research shows most users need the advanced options
- Remember the user's preference when possible (don't re-collapse on page reload if they opened it)

## Anti-Pattern: Hidden Critical Path

**Never use progressive disclosure to hide steps the user must complete.** If the user must fill in a field to submit a form, that field must be visible by default. Progressive disclosure is for *optional* or *advanced* content.

## Anti-Pattern: Endless Disclosure

Avoid 4+ levels of progressive disclosure. If you're nesting "show more" inside an accordion inside a modal, the information architecture needs restructuring.

## References
- NN/g: multiple articles on progressive disclosure and complexity management
- NN/g: progressive disclosure and cognitive load articles (nngroup.com)