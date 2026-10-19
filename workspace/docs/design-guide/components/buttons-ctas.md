# Buttons and CTAs

Buttons are the primary interactive element in most UIs. Small decisions about buttons have outsized impact on conversion and usability.

## Button Hierarchy

### Primary Button
The single most important action on the screen.
- **Visual:** Filled with brand/primary color, white text
- **One per view** (or per section in complex layouts)
- **Placement:** Bottom-right of forms, top-right of toolbars, end of the reading flow

### Secondary Button
Alternative or supporting actions.
- **Visual:** Outlined (border only) or subdued fill (light tint of primary)
- **Multiple allowed per view** but don't overuse
- **Placement:** Next to primary button

### Tertiary / Text Button
Low-emphasis actions.
- **Visual:** Text only, no border, subtle hover effect
- **Use for:** "Cancel", "Learn more", "Reset", "Skip"
- **Placement:** Inline or near the element they relate to

### Ghost Button
Actions on dark or colored backgrounds.
- **Visual:** Transparent background, light border, light text
- **Use for:** Actions within a hero section, on colored cards

### Danger Button
Destructive actions.
- **Visual:** Red fill or red outline
- **Use ONLY for:** Delete, remove, destroy — actions that are hard or impossible to undo
- **Never use for:** Cancel, close, dismiss, decline (these are neutral actions)

## Button Anatomy

```
[  Icon?  ] [     Label Text     ] [  Badge?  ]
  8-12px        16-24px horiz        0-8px
               8-12px vert
                8-12px icon gap
```

- **Label:** Verb + object ("Save changes", not "Save") or clear intent ("Next", "Sign up")
- **Icon (optional):** Left side for action, right side for navigation ("Next →")
- **Loading state:** Replace icon with spinner, disable clicks, keep the button visible

## Touch Targets

- **Minimum:** 44×44pt (iOS HIG), 48×48dp (Material Design)
- **Recommended:** 48×48px for primary actions on touch devices
- **The visual button can be smaller than the touch target** — use padding to extend the tap area

## Common AI Failures

1. **Multiple primary buttons** competing for attention — user doesn't know what to click
2. **Same visual weight for all actions** — "Save" and "Delete" look equally important
3. **Vague labels:** "Submit" instead of "Create account", "OK" instead of "Confirm deletion"
4. **Missing disabled state:** Button looks the same whether an action is possible or not.
   **But a disabled button is usually the wrong fix.** `disabled` removes the button from the
   tab order, so keyboard and screen-reader users cannot reach it to discover *why* it's
   unavailable. A greyed label is also frequently too faint to read — WCAG exempts inactive
   components from the contrast minimum, so this is legal, which makes it a usability problem
   nobody's tooling will flag.
   Prefer: keep the button reachable with `aria-disabled="true"` (blocking the action in the
   handler) and explain what's missing, inline or on activation. Reserve real `disabled` for
   states whose cause is already visible — a submit button during an in-flight request, where
   the spinner is the explanation.
   → [../interaction-design/state-patterns.md](../interaction-design/state-patterns.md)
5. **Icon-only buttons without labels or tooltips** — users must guess what the icon means
6. **Button that looks like a link** or vice versa — see `../foundations/signifiers-affordances.md`

## When to Use a Button vs. a Link

| Use a Button | Use a Link |
|---|---|
| Submits data, changes state, triggers an action | Navigates to another page | 
| Represents the primary action in a flow | Provides supplementary navigation |
| Has immediate, local effect | Takes user away from current context |