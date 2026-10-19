# Motion Sensitivity and Accessibility

**35.4% of US adults aged 40 and over have measurable vestibular dysfunction** — roughly 69
million people. For these users, screen motion can cause dizziness, nausea, and headaches.

> **Cite this one accurately.** Source: Agrawal Y, Carey JP, Della Santina CC, Schubert MC,
> Minor LB. *Disorders of balance and vestibular function in US adults: data from the
> National Health and Nutrition Examination Survey, 2001–2004.* Arch Intern Med.
> 2009;169(10):938–44. Two caveats that get dropped when the number is repeated: it measures
> **dysfunction** via the modified Romberg balance test, not a **diagnosed disorder**, and
> the sample is **US** adults. Only the symptomatic subset (those reporting dizziness) showed
> the dramatic 12-fold fall risk. The design conclusion is unaffected — the population is
> large either way — but do not restate it as "35% have a vestibular disorder."

Vestibular sensitivity is also not the only reason to respect the setting: users turn on
Reduce Motion for migraine, concussion recovery, attention, and simple preference. Treat the
media query as an instruction, not as a disability disclosure.

## `prefers-reduced-motion`

### What It Detects
Users who have enabled the "Reduce Motion" setting in their OS (macOS: System Preferences > Accessibility > Display; Windows: Settings > Ease of Access > Display; iOS: Settings > Accessibility > Motion).

### What It Means for Your Design
When `prefers-reduced-motion: reduce` is active:

1. **Remove non-essential animations** — entrance animations, parallax, background animations, decorative transitions
2. **Keep essential state-change feedback** — but make it instant (0.01ms duration) rather than animated
3. **Use opacity and color changes** instead of movement for state feedback
4. **Don't auto-play video or carousels**

### Implementation

```css
/* Base: all animations defined normally */
.card {
  transition: transform 0.2s ease-out, box-shadow 0.2s ease-out;
}
.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.1);
}

/* Reduced motion: disable animations */
@media (prefers-reduced-motion: reduce) {
  .card {
    transition: none;
  }
  .card:hover {
    transform: none;
    /* Use a subtle color change instead */
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  }
}
```

### Global Reset (Nuclear Option)

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Use this as a safety net, but prefer targeted overrides that preserve some visual feedback.

## Animations That Commonly Cause Problems

| Animation Type | Problem | Alternative |
|---|---|---|
| Parallax scrolling | Disorienting | Static positioning or subtle opacity change |
| Auto-scrolling carousels | Hard to stop, disorienting | User-controlled carousel or static content |
| Background motion (particles, video) | Distracting, nausea-inducing | Static background or very subtle gradient animation |
| Zoom on scroll | Vertigo-inducing | Fade-in on scroll, no zoom |
| Page transition animations | Disorienting | Instant cross-fade or no animation |
| Bouncing/shaking elements | Can trigger nausea | Brief color flash |

## Motion for Neurodivergent Users

Beyond vestibular disorders, some neurodivergent users (autism, ADHD) are sensitive to:
- **Blinking/flashing content** (can trigger seizures in photosensitive epilepsy)
- **Constant motion** (distracting, anxiety-inducing)
- **Unexpected motion** (startling)

**Rules:**
- Nothing should flash more than 3 times per second (WCAG 2.3.1)
- Provide a way to pause/stop any moving content
- Don't use motion to draw attention to content (use color, size, position instead)

## References
- WCAG 2.3.1 (Three Flashes or Below Threshold)
- `prefers-reduced-motion` MDN: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- WebAIM: Animation and Accessibility
