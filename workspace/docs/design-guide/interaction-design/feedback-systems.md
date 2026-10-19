# Feedback Systems

Users need to know: "Did it work? Is it still working? What went wrong?" Every action should produce a perceptible response.

## The Feedback Hierarchy

| Urgency | Feedback Type | Timing | Example |
|---------|--------------|--------|----------|
| Immediate | **Micro-feedback** | 0–100ms | Button press, toggle, checkbox |
| Short | **Status feedback** | 100ms–2s | Loading indicator, progress bar, spinner |
| Medium | **Result feedback** | 2s–5s | Success toast, error message, content update |
| Long | **Confirmation feedback** | On completion | Email receipt, confirmation screen |

## Micro-Feedback (0–100ms)

The most important and most often omitted by AI designs. Users should feel that the interface is responding to them at every interaction point.

**Required micro-feedback:**
- **Button press:** Visual change on :active (scale 95–98%, slight color shift)
- **Hover on interactive elements:** Color change, underline, shadow increase, cursor change
- **Toggle/checkbox:** Immediate visual state change with brief animation
- **Text input:** Focus ring, cursor appearance
- **Drag start:** Visual pickup (shadow, slight scale, opacity change)

**Missing micro-feedback is a signifier failure** → See `../foundations/signifiers-affordances.md`

## Loading & Progress Feedback

See `state-patterns.md` for the full loading pattern taxonomy. Key rules:

- **< 300ms:** No indicator needed. The response feels instant.
- **300ms–1s:** Show a subtle indicator (spinner, skeleton).
- **1s–5s:** Show a progress bar or skeleton screen.
- **> 5s:** Show progress with estimated time, or provide a way to cancel.
- **Indeterminate:** Never show a progress bar without progress information if you have it. If you don't know the progress, use a skeleton or spinner with descriptive text.

## Success Feedback

- **Small actions** (save, toggle, favorite): Brief visual change + optional subtle toast
- **Medium actions** (submit form, upload file): Toast notification + state change in UI
- **Large actions** (purchase, publish, delete): Confirmation screen or prominent inline confirmation

**Rule:** Match the feedback intensity to the action's irreversibility and importance.

## Error Feedback

### Where to Show Errors
- **Inline** (next to the field): For form validation errors. Always preferred.
- **Toast/notification:** For system-level errors (network failure, server error).
- **Modal/dialog:** Only for errors that require the user to make a decision (e.g., "You have unsaved changes.").
- **Inline banner:** For page-level errors that affect the current view but don't block all interaction.

### Error Message Quality

| Bad | Good |
|-----|------|
| "Error" | "Could not save changes. Please check your connection and try again." |
| "Invalid input" | "Email addresses need an @ — for example, name@example.com" |
| "Something went wrong" | "Your session expired. Please sign in again." |
| "Failed" | "Upload failed. The file 'report.pdf' is too large (max 10MB)." |

**Rules for error messages:**
1. Say what went wrong in plain language
2. Say why it went wrong (if known)
3. Say what the user should do about it
4. Never blame the user ("You entered an invalid...")

## Notification/Toast Guidelines

- **Position:** Pick one position per product and stay consistent — bottom-right (most common on web) or top-right/top-center. On mobile, top of screen is conventional.
- **Duration:** 3–5 seconds for informational toasts. Persistent for errors requiring action.
- **Stacking:** New toasts push old ones up. Limit visible toasts to 3.
- **Content:** What happened + undo/dismiss option.
- **Don't use toasts for:** Critical errors, content that requires reading more than one line, anything the user needs to remember.

### The auto-dismiss trap

Two of the rules above conflict, and the conflict is the single most common toast bug:

> "Duration: 3–5 seconds" + "Content: what happened **+ undo**"
> = an Undo button that deletes itself before it can be used.

Five seconds is not enough time for a screen-reader user to hear the announcement, find the
toast, and reach the button — and a keyboard user has to tab to it at all. If the toast is the
**only** route to Undo, a timed dismissal puts a time limit on the recovery path, which is what
**SC 2.2.1 Timing Adjustable** exists to prevent.

**Resolve it like this:**

| Toast contains | Duration |
|---|---|
| Information only ("Saved") | 3–5s auto-dismiss is fine |
| An action (Undo, Retry, View) | **Do not auto-dismiss**, or pause the timer on hover **and** on focus, and keep the same action reachable elsewhere |
| An error needing a decision | Persistent, with an explicit dismiss — and consider that a toast is the wrong surface |

- **Pause on hover and on focus**, and resume on leave. Hover-only pausing helps mouse users
  and abandons everyone else.
- **Never make a toast the only path to an action.** Undo must also exist in the UI (a menu, the
  item's row, a history panel). The toast is a shortcut, not the mechanism.

### Toasts must be announced

A toast that appears silently does not exist for a screen-reader user — this is
**SC 4.1.3 Status Messages**.

- Render toasts into a container that is **already in the DOM** with `role="status"`
  (polite) — injecting the live region and its text at the same moment often announces nothing.
- Use `role="alert"` (assertive) **only** for errors that interrupt. Assertive announcements
  cut off whatever the user was hearing; overusing them makes the product hostile.
- **Never move focus to a toast.** It's non-modal and unrequested; stealing focus loses the
  user's place. Make it reachable, don't force it.
- Don't announce every toast in a burst — collapse repeats ("3 items saved") rather than firing
  three announcements.

## References
- NN/g on microinteractions: https://www.nngroup.com/articles/microinteractions
- IxDF on micro-interactions (Dec 2025): https://ixdf.org/literature/article/micro-interactions-ux
