# Keyboard Focus Management

Many users navigate entirely by keyboard (tab, shift+tab, enter, space, arrows, escape). Focus management is how they know where they are and what they can do.

## Focus Visibility

Every focusable element must have a **visible** focus indicator.

### The Browser Default
Browsers provide a default focus ring (usually a blue outline). **Don't remove it without replacing it with something better.**

```css
/* BAD: removes focus visibility */
*:focus { outline: none; }

/* GOOD: replaces with custom focus style */
*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
  border-radius: 4px;
}
```

### `:focus` vs `:focus-visible`
- `:focus` shows for ALL focus (including mouse clicks)
- `:focus-visible` shows only for keyboard focus (what you want for custom styles)

**Use `:focus-visible` for custom focus styles.** This prevents the focus ring from appearing on mouse click while keeping it for keyboard navigation.

## Focus Order

Tab order must follow the **visual layout** (left to right, top to bottom in LTR). This means:

1. DOM order matches visual order
2. CSS `order` property doesn't disrupt tab order (or if it does, use `tabindex` to correct)
3. Modal focus traps cycle through interactive elements in logical order
4. Skip links appear first in tab order

## Skip Links

A skip link allows keyboard users to jump past repetitive navigation:

```html
<a href="#main-content" class="skip-link">Skip to main content</a>
<nav>...</nav>
<main id="main-content">...</main>
```

```css
.skip-link {
  position: absolute;
  top: -100%;
  left: 50%;
  transform: translateX(-50%);
}
.skip-link:focus {
  top: 8px;
}
```

**Why `:focus` here and `:focus-visible` everywhere else.** This is deliberate, not an
inconsistency with the rule above. A skip link is only ever reachable by keyboard, and it must
appear whenever it receives focus — including focus moved programmatically, where
`:focus-visible` may not match. The `:focus-visible` rule exists to stop focus rings
appearing on *mouse* clicks; a skip link has no mouse interaction to suppress.

**Never hide a skip link with `display: none` or `visibility: hidden`** — both remove it
from the tab order entirely, which defeats the purpose. Move it off-screen, as above, or use a
clip-based visually-hidden utility.

## Focus Management in Interactive Components

### When a Modal Opens
1. Move focus to the modal
2. Trap focus within the modal (tab wraps around)
3. On close, return focus to the element that triggered the modal

### When Content Changes Dynamically
1. If the change is important (error, success, new content), use `aria-live` to announce it
2. Don't move focus unless the user's context has fundamentally changed

### When Navigating Between Pages/Views
1. Move focus to the new page's heading or main content area
2. Don't leave focus on a navigation link after navigation

## Roaming Focus vs. Virtual Focus

- **Roaming focus:** Actual DOM focus moves between elements (tabs, buttons). Standard for most UIs.
- **Virtual focus:** A container has DOM focus, and JavaScript manages an internal "virtual" focus for child elements (combo boxes, listboxes, trees). Used when standard focus behavior isn't sufficient.

**For AI agents:** Use roaming focus (standard tabindex) unless you're building a complex widget (combobox, tree view). For those, follow WAI-ARIA Authoring Practices patterns.

## References
- WAI-ARIA Authoring Practices: https://www.w3.org/WAI/ARIA/apg/
- MDN: `:focus-visible` — https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible
- Related: focus is itself a signifier of current location → [../foundations/signifiers-affordances.md](../foundations/signifiers-affordances.md)
