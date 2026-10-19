# Motion Design in UI

Motion in UI serves three functional purposes. If an animation doesn't serve one of these, it's decoration and should be removed or minimized.

## The Three Functions of Motion

### 1. Provide Feedback
Confirm that an action was received.
- Button press: brief scale-down (95–98%) on :active, return on release
- Toggle: smooth transition between states (150–300ms)
- Checkbox/radio: brief fill animation
- Success: subtle checkmark animation

### 2. Maintain Context During Transitions
Help users understand where content came from and where it went.
- **Layout transitions:** Elements animate to their new positions (CSS `layout` animations, FLIP technique)
- **List reordering:** Items smoothly slide to new positions
- **Expand/collapse:** Height animates, content fades in
- **Page transitions:** Content cross-fades or slides in the navigation direction

### 3. Direct Attention
Guide the eye to what matters.
- **Entrance animations:** Elements fade/slide in with staggered delays to create reading order
- **State changes:** A notification slides in from the edge
- **Error highlighting:** A field briefly shakes or pulses red

## Timing Guidelines

| Type | Duration | Easing | When to Use |
|------|----------|--------|-------------|
| Micro-feedback | 100–150ms | ease-out | Button press, toggle, hover |
| State transition | 150–300ms | ease-in-out | Tab switch, accordion, theme change |
| Content entrance | 200–400ms | ease-out | Modal open, page load, list items |
| Complex transition | 300–500ms | custom cubic-bezier | Page transition, major layout shift |

**Rule of thumb:** If you have to wait for it, it's too slow. Animations > 500ms for UI transitions are almost always too long.

## Easing Functions

Easing determines how an animation accelerates and decelerates.

- **`ease-out`** (fast start, slow end): For elements entering the screen. Feels responsive.
- **`ease-in`** (slow start, fast end): For elements leaving the screen. Natural-feeling exit.
- **`ease-in-out`** (slow start and end, fast middle): For state changes where the element stays on screen.
- **`linear`**: Almost never use in UI. Only for continuous motion (progress bars, loading spinners).

## Accessibility: `prefers-reduced-motion`

**Non-negotiable.** Every non-essential animation must be disabled when the user has "Reduce
Motion" enabled. Measurable vestibular dysfunction affects ~35% of US adults over 40 (Agrawal
et al. 2009 — see [../accessibility/motion-sensitivity.md](../accessibility/motion-sensitivity.md)
for the exact claim and its limits), and motion can cause dizziness and nausea. Users also
enable the setting for migraine, concussion recovery, and preference — never gate the
behaviour on whether you think the user "really" needs it.

Minimum safety net:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;   /* ← required, see below */
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**Two things about this snippet that are easy to get wrong:**

1. **`animation-iteration-count: 1` is not optional.** Without it, an
   `animation: spin 1s infinite` keeps its infinite count and merely runs at 0.01ms per
   cycle — an unbounded loop firing continuously, burning CPU and battery for a user who asked
   for *less* activity. Shortening the duration without capping the iterations makes the
   problem worse, not better.

2. **It is a safety net, not the implementation.** This rule disables *every* animation,
   including the ones that carry meaning — a loading spinner, a progress bar, a state
   transition that shows where content went. Reduced-motion users still need to know something
   is loading. The rule above stops harm on a page you haven't audited; the correct approach is
   **targeted overrides**: keep the animation, remove the movement.

   ```css
   /* Preferred: swap motion for a non-moving equivalent */
   @media (prefers-reduced-motion: reduce) {
     .modal   { animation: fade-in 150ms ease-out; }  /* was: slide + scale */
     .spinner { animation: pulse-opacity 1s infinite; } /* was: rotate */
   }
   ```

   "Reduce motion" means **reduce movement**, not "remove all feedback". Opacity and colour
   changes are generally safe; large translations, scaling, parallax, and spinning are not.

Full implementation guidance — targeted overrides, which animations cause harm, flashing limits (WCAG 2.3.1), and neurodivergent considerations — is in [../accessibility/motion-sensitivity.md](../accessibility/motion-sensitivity.md).

## Common AI Motion Failures

1. **Over-animating:** Every element fades, slides, and bounces on page load. Feels like a presentation, not a tool.
2. **Wrong easing:** Using `linear` or `ease-in` for entrances (feels sluggish).
3. **No `prefers-reduced-motion`:** Excludes users with motion sensitivity.
4. **Stagger everything:** Staggered entrance animations on a list of 20+ items means the last item appears 4+ seconds after the first.

## When NOT to Animate

- Loading spinners when the load will take < 300ms (the spinner appears and disappears before the user registers it)
- Scrolling to sections that are already visible
- Tooltips that appear after a long delay (> 500ms)
- Anything that delays the user from completing their task

## References
- NN/g on animation: https://www.nngroup.com/articles/animation-usability/
- `prefers-reduced-motion` MDN: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- Material Design motion guidelines (reference for timing values)
