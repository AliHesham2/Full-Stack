Accessibility is not a feature — it is an architectural constraint with legal weight. The European Accessibility Act (EAA) is enforced since June 2025. ADA Title III lawsuits in the US continue to climb year over year. Every decision in this guide assumes WCAG 2.2 Level AA compliance as a baseline requirement, not a nice-to-have.

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

WCAG 3.0 is a working draft. Do not target it. WCAG 2.2 was ISO-approved in October 2025 and is the stable, legally recognized standard.

## Semantic HTML First

Use native elements before reaching for ARIA. A `<button>` already has the right role, keyboard behavior, and implicit accessible name. A `<div onClick>` has none of these.

```tsx
// Correct
<button onClick={handleClick}>Submit</button>

// Wrong — no keyboard support, no role, no semantics
<div className="btn" onClick={handleClick}>Submit</div>
```

Native elements to use by default: `<button>`, `<nav>`, `<main>`, `<header>`, `<footer>`, `<dialog>`, `<search>`, `<details>`/`<summary>`, `<fieldset>`/`<legend>`, `<time>`. Each provides semantics, keyboard behavior, and screen reader support for free.

## The Five Rules of ARIA

From the W3C's First Rule of ARIA and its extensions:

1. **Use native HTML first.** If a `<button>` works, do not use `<div role="button">`.
2. **Do not change native semantics.** Avoid redundant roles on native elements by default. role="list" on ul is not inherently harmful: it can restore list semantics in Safari/VoiceOver when list-style:none removes them. Apply such a workaround only after checking the actual browser/screen-reader behavior.
3. **ARIA roles need keyboard handlers.** If you set `role="tab"`, you must implement Arrow key navigation, Home/End, and activation on Enter/Space per the APG pattern. ARIA communicates semantics; it does not provide keyboard behavior.
4. **Never put `role="presentation"` or `aria-hidden="true"` on a focusable element.**
   `aria-hidden` removes the element from the accessibility tree while leaving it in the tab
   order, so a keyboard user lands on something a screen reader cannot announce — focus
   apparently vanishes. `role="presentation"` strips the element's semantics, leaving a
   focusable thing with no role. If an element should not be reachable, use `inert` or
   `tabindex="-1"` — not an ARIA attribute. (`aria-hidden` on focusable content is the more
   common of the two in practice.)
5. **All interactive elements need accessible names.** Every `<button>`, `<a>`, `<input>`, and custom widget must have a visible label or an `aria-label`/`aria-labelledby`.

## Keyboard Interaction

Follow APG (ARIA Authoring Practices Guide) patterns exactly for any custom widget. The APG defines expected key bindings for tabs, menus, comboboxes, trees, and other patterns. Deviating from APG creates inconsistency that screen reader users must learn for your site alone.

**Decision:** If you are building a custom widget, look up the APG pattern first. If the widget is complex (combobox, menu, tree), use a headless UI library instead of building it yourself. See [components](./components.md) for library choices.

## Focus Management

Use `:focus-visible` for keyboard-only focus indicators. This is supported in all modern browsers and prevents visible focus rings on mouse clicks while preserving them for keyboard navigation.

```css
/* Baseline — do this */
:focus-visible {
  outline: 2px solid var(--ring);
  outline-offset: 2px;
}

/* Never do this without a replacement */
*:focus {
  outline: none;
}
```

WCAG 2.2 added **Focus Not Obscured (Minimum)**. Sticky headers, cookie banners, or fixed footers must not cover the element receiving focus. Test by Tab-ing through your page and verifying the focused element is fully visible. If you have a sticky header, add `scroll-padding-top` to compensate.

```css
html {
  scroll-padding-top: 64px; /* match your sticky header height */
}
```

## Modals and Dialogs

Use native `<dialog>` with `.showModal()` for modals. This provides built-in focus trapping,
`aria-modal` semantics, top-layer rendering, Escape-to-close, and **background inertness** —
all of it, without any extra work:

```tsx
function Modal({ open, onClose, children }) {
  const ref = useRef<HTMLDialogElement>(null);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    // Guard both calls: showModal() on an already-modal dialog is a no-op; calling it on a non-modal open dialog throws InvalidStateError.
    if (open && !el.open) el.showModal();
    if (!open && el.open) el.close();
  }, [open]);

  return (
    <dialog ref={ref} onClose={onClose} aria-labelledby="modal-title">
      {children}
    </dialog>
  );
}
```

```css
/* showModal() does NOT lock body scroll — this is the one piece left to you. */
html:has(dialog[open]) { overflow: hidden; }
```

> ⚠️ **Two errors an earlier revision of this file contained.**
>
> 1. It set a class and commented `CSS: .inert-bg > * { inert: true; }`. **`inert` is an HTML
>    attribute, not a CSS property** — there is no CSS declaration that makes an element inert,
>    so that rule is silently discarded and the background stays interactive.
> 2. Even a correct `inert` implementation would be redundant: `showModal()` **already**
>    renders the dialog in the top layer and makes everything outside it inert. Adding your own
>    inertness on top fights the browser's.
>
> Use `inert` for the cases `showModal()` does not cover — a non-modal drawer, or an
> off-canvas panel — where you genuinely need to disable a region yourself:
> `<div inert={!open}>`.

**Do not omit the accessible name.** A `<dialog>` with no `aria-labelledby`/`aria-label`
announces as an unnamed dialog. Point it at the dialog's heading.

For complex dialogs with animations, Radix Dialog handles animation orchestration and scroll locking. See the dialog guidance in [components](./components.md).

## Forms

Accessible form patterns are covered in [forms](./forms.md). Key rules: every `<input>` gets a `<label htmlFor>`, hints use `aria-describedby`, and invalid fields set `aria-invalid="true"`.

## Images and Media

Every `<img>` needs `alt` text. Describe the content or purpose. Empty `alt=""` is correct for decorative images — do not omit the attribute. For decorative SVGs, add `aria-hidden="true"` and ensure they have no interactive descendants.

```tsx
// Informative — describe what the image conveys
<img src="/chart.png" alt="Revenue increased 40% from Q1 to Q4 2025" />

// Decorative — empty alt
<img src="/divider.png" alt="" aria-hidden="true" />
```

## Color and Motion

Do not rely on color alone to convey information. Pair color with text, icons, or patterns.

**Contrast ratios — and the unit trap.** 4.5:1 for normal text, 3:1 for *large* text. WCAG
defines large text in **points, not pixels**:

| WCAG wording | In CSS pixels | Threshold |
|---|---|---|
| 18pt | **24px** | 3:1 |
| 14pt bold | **18.66px** | 3:1 |
| anything smaller | — | 4.5:1 |

An earlier revision of this file wrote "18px+ or 14px+ bold". That is the right numbers with the
wrong unit, and it fails in the direction that matters: **18px text at 3:1 is a WCAG failure**,
because 18px is well under the 24px threshold. When in doubt, hold everything to 4.5:1 — the
large-text exemption buys less than people expect.

Respect `prefers-reduced-motion: reduce`. Replace animations with instant state changes for users who prefer reduced motion:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;   /* required — see below */
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**`animation-iteration-count: 1` is not optional.** Without it, an
`animation: spin 1s infinite` keeps its infinite count and simply runs at 0.01ms per cycle —
an unbounded loop burning CPU and battery for a user who asked for *less* motion.

This is a safety net for pages you have not audited, not the implementation. It also kills
animations that carry meaning — a loading spinner, a progress bar. Prefer **targeted
overrides** that remove *movement* while keeping feedback: swap a slide for a fade, a rotation
for an opacity pulse. "Reduce motion" means reduce movement, not remove all feedback. See [design-tokens](./design-tokens.md) for integrating with your design system.

## Testing

Automated tools catch only a minority of accessibility issues — commonly cited as 30–40%. They miss keyboard traps, illogical tab order, missing focus management, and incorrect ARIA usage that technically passes but confuses users.

**Automated (THE CHECK):** a current compatible stable `axe-core` release through `jest-axe` in component tests
(`@axe-core/playwright` only if the user asks for automated E2E). Fail the check on any Level AA
violation:

```ts
// in a tester's own helper, in its folder (jest-axe: an approved package decision in Planner Output/software TODO)
import { expect } from 'vitest';
import { toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

// in a component test
import { axe } from 'jest-axe';

test('button has no a11y violations', async () => {
  const { container } = render(<Button>Click</Button>);
  expect(await axe(container)).toHaveNoViolations();
});
```

> **Not `vitest-axe`**, despite the name fitting better: its last release is **0.1.0 (October
> 2022)**, which fails this guide's own "no release in 12+ months" rule
> ([dependencies](../04-quality/dependencies.md)). `jest-axe` works with Vitest and is actively
> maintained (11.0.0, July 2026). See [testing](../04-quality/testing.md).

**Manual (mandatory for critical flows):** NVDA+Chrome on Windows and VoiceOver+Safari on macOS. Test login, checkout, form submission, navigation, and any modal or custom widget. There is no substitute.

## Headless UI Libraries

Radix UI and React Aria provide the strongest APG compliance. Both handle keyboard interaction, focus management, and ARIA attributes correctly. Use them for custom widgets — see the comparison in [components](./components.md).

## Common AI-Generated Accessibility Mistakes

These patterns appear frequently in AI-generated code. Catch and fix them:

- **`<div>` instead of `<button>`** for clickable elements. No keyboard support, wrong semantics.
- **Missing labels** on icons-only buttons. `aria-label` is required when there is no visible text.
- **No keyboard handlers on custom widgets.** Adding `role="tab"` without implementing Arrow key navigation.
- **`aria-hidden` on focusable elements.** Removes semantics but the element remains in the tab order.
- **Unnecessary ARIA on native elements.** Preserve native semantics; a tested role="list" workaround for Safari/VoiceOver with list-style:none is valid, not a blanket defect.
- **`aria-label` with a placeholder string.** "button" or "icon" is not a useful accessible name.
