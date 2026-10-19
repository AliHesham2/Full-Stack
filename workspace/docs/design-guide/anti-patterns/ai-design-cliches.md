# AI Design Clichés: Why AI Designs Look the Same

LLMs produce designs that converge on the "average" of all designs in their training data. This file documents the specific clichés and how to avoid them.

## The Root Cause

LLMs don't design — they compute the weighted average of every interface they've seen. This means:
- The output is the statistical center of the training distribution
- Unconventional but effective choices are averaged out
- The result is "professionally adequate" but character-free
- Every design from every AI looks like every other AI design

## The Cliché Catalog

### Visual Clichés

| Cliché | Why AI Produces It | How to Break It |
|--------|-------------------|----------------|
| **Gradient hero sections** with blue-to-purple or blue-to-teal gradients | Extremely common in training data | Use a single color, a photo, an illustration, or a typographic hero. Or no hero at all. |
| **Perfectly centered everything** | Symmetry is safe | Use asymmetric layouts. Left-align hero text. Offset images. |
| **Rounded cards with soft shadows everywhere** | Card-based layouts are 60%+ of modern web | Vary the layout. Use full-bleed sections, inline content, or mixed layouts. |
| **"Modern minimalist" gray-on-white** | The safest aesthetic in training data | Choose a deliberate aesthetic direction (editorial, bold, warm, technical) and commit. |
| **The same two fonts — Inter and Plus Jakarta Sans** | These are the most common web fonts in training data | Alternatives with more character: DM Sans, Satoshi, General Sans, Outfit, Geist, Instrument Sans. Do **not** substitute Plus Jakarta Sans — it is half the cliché, not the escape from it. For Arabic, see [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md). |
| **Emoji in headings and CTAs** | AI models associate enthusiasm with emoji | Use emoji only if the brand is casual/playful. Never in headings. |
| **Overuse of icons** (an icon next to every feature, every heading) | Icons are perceived as "professional" | Icons should be selective. If everything has an icon, nothing benefits from one. |
| **Generic stock illustrations** (people pointing at screens, abstract shapes) | These are the most common illustrations in training data | Use no illustration, custom illustration, or photography. A bad illustration is worse than none. |

### Structural Clichés

| Cliché | Why AI Produces It | How to Break It |
|--------|-------------------|----------------|
| **Hero → 3-column features → Testimonials → CTA** | This is the most common SaaS landing page structure in training data | Rearrange sections. Lead with social proof. Embed testimonials contextually. Start with a demo. |
| **Exactly 3 feature columns** | Three is the most common number in feature sections | Use 2, 4, or a varied layout. Or present features as a flowing list with illustrations. |
| **"Trusted by" logo bar** immediately after the hero | Extremely common pattern | Place it somewhere less expected, or remove it entirely if the logos aren't recognizable. |
| **Generic "Get Started" / "Learn More" CTAs** | These are the most common button labels in training data | Use specific, action-oriented labels: "Start free trial", "See the demo", "Create your first project". |

### Interaction Clichés

| Cliché | Why AI Produces It | How to Break It |
|--------|-------------------|----------------|
| **No hover states, no active states** | AI generates static mockups, not interactive specs | Always design hover, active, and focus states. See `../interaction-design/state-patterns.md`. |
| **No loading, error, or empty states** | AI shows the ideal case | Always account for all states. See `../interaction-design/state-patterns.md`. |
| **Identical spacing everywhere** | Uniform spacing is the "average" | Use spacing intentionally: tighter within groups, looser between groups. |

## Breaking the Average: Decision Framework

When you're about to place an element, ask:

1. **Is this the most common position for this element?** If yes, consider an alternative.
2. **Is this the most common visual treatment?** If yes, what's a less common but still effective treatment?
3. **Does this layout look like the last 5 AI-generated pages I've seen?** If yes, change something structural.
4. **Am I using this element because it's needed, or because it's expected?** If expected but not needed, remove it.

## What NOT to Change

Breaking clichés doesn't mean breaking usability. Navigation conventions, form patterns, button signifiers, error message patterns, and accessibility requirements must stay conventional (Jakob's Law). The full non-negotiables list → [design-anti-averaging.md](design-anti-averaging.md).

Be unconventional in *expression*, not in *function*.

## References
- NN/g: "First Impressions Matter" / first-click and convention research (nngroup.com)
- Jakob's Law → [../foundations/cognitive-principles.md](../foundations/cognitive-principles.md)
