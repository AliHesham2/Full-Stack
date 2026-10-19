# WCAG Essentials

The non-negotiable accessibility baseline. As of 2026, WCAG 2.2 Level AA is the legal requirement in many jurisdictions (EU Accessibility Act, US DOJ web rule for state/local governments).

## The Four Principles (POUR)

| Principle | Meaning | Key Questions |
|-----------|---------|---------------|
| **Perceivable** | Users can perceive the content | Can I see/hear it? Is the text readable? |
| **Operable** | Users can operate the interface | Can I navigate? Can I use a keyboard? |
| **Understandable** | Users can understand the content | Is the language clear? Are errors explained? |
| **Robust** | Content works with assistive technologies | Does it work with screen readers? With voice control? |

## Critical WCAG 2.2 AA Requirements

### Perceivable
- **1.1.1 Non-text Content:** All images have alt text. Decorative images use `alt=""`.
- **1.3.1 Info and Relationships:** Use semantic HTML (`<button>`, `<nav>`, `<main>`, `<h1>`-`<h6>`, `<table>`). Don't use `<div>` for everything.
- **1.3.2 Meaningful Sequence:** DOM order matches visual order.
- **1.4.3 Contrast (Minimum):** 4.5:1 for normal text, 3:1 for large text and UI components. → See `contrast-readable.md`
- **1.4.11 Non-text Contrast:** 3:1 for UI components (borders, icons) against adjacent colors.
- **1.4.13 Content on Hover/Focus:** Hover/focus content is dismissible, hoverable, and persistent.

### Operable
- **2.1.1 Keyboard:** All functionality available via keyboard. No keyboard traps.
- **2.1.2 No Keyboard Trap:** Users can navigate away from any component using keyboard.
- **2.4.3 Focus Order:** Logical tab order that follows the visual layout.
- **2.4.7 Focus Visible:** A visible focus indicator on all focusable elements. → See `keyboard-focus.md`
- **2.4.11 Focus Not Obscured (Minimum) (WCAG 2.2 new):** Sticky headers/floating bars must not fully hide the focused element.
- **2.5.7 Dragging Movements (WCAG 2.2 new):** Any drag action needs a single-pointer alternative (buttons for reorder, click-to-move).
- **2.5.8 Target Size (Minimum) (WCAG 2.2 new):** Minimum 24×24 CSS px target size for pointer inputs (with spacing/inline exceptions). Platform HIGs recommend more: 44×44pt iOS / 48dp Android.

### Understandable
- **3.1.1 Language of Page:** `<html lang="en">` is set.
- **3.2.1 On Focus:** No context change when an element receives focus.
- **3.2.2 On Input:** No context change when the user inputs data (no auto-submit on typing).
- **3.3.1 Error Identification:** Error messages identify the field and describe the error.
- **3.3.2 Labels or Instructions:** All form inputs have visible labels.
- **3.3.3 Error Suggestion:** When an error is detected, suggestions for fixing it are provided (when possible).
- **3.3.7 Redundant Entry (WCAG 2.2 new):** Don't make users re-enter information already provided in the same session.

### Robust
- **4.1.2 Name, Role, Value:** All custom UI components have appropriate ARIA attributes.
- **4.1.3 Status Messages:** Dynamic content changes (toasts, live regions) are announced to screen readers.

## Semantic HTML: The Foundation

Many accessibility requirements are met simply by using the right HTML elements:

| Use | Don't Use | Why |
|-----|-----------|-----|
| `<button>` | `<div onclick="...">` | Built-in keyboard support, role, state |
| `<nav>` | `<div class="nav">` | Landmark for screen reader navigation |
| `<main>` | `<div class="main">` | Skips to main content |
| `<h1>`-`<h6>` | `<div class="heading">` | Document outline for screen readers |
| `<a href="...">` | `<span onclick="navigate()">` | Link behavior: open in new tab, copy link |
| `<input type="checkbox">` | Custom div checkbox | Built-in state management |

## References
- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- W3C WAI: https://www.w3.org/WAI/
- ADA web rule (2024): https://www.ada.gov/resources/2024-03-08-web-rule
- **EU Accessibility Act** — Directive (EU) 2019/882. Applies to products placed on the
  market and services provided to consumers **after 28 June 2025** (Arts. 2(1)–(2), verified
  against the EUR-Lex text of CELEX:32019L0882). Article 32 adds a transitional period to
  **28 June 2030** during which providers may keep serving with products lawfully in use
  before that date — so "already shipping" is not a permanent exemption, it is a five-year
  runway that has already started. New work gets no grace period at all.
