# Design Anti-Averaging: How to Escape the AI Design Mean

A practical methodology for AI agents to produce designs that feel intentional and distinctive rather than averaged and generic.

## The Problem

LLMs produce the weighted average of their training data. In design, this manifests as:
- Layouts that look like every other SaaS page
- Color choices that are "pleasantly inoffensive" but unmemorable
- Structure that follows the hero → features → testimonials → CTA template
- Visual weight distributed evenly (no clear hierarchy)

## Anti-Averaging Methodology

### Step 1: Identify the Averaged Default

Before designing, ask: "What would the generic version of this look like?" Write it down. If you can't identify it, you'll accidentally produce it.

Common averaged defaults:
- Landing page: centered hero, gradient background, 3-column features, testimonial carousel, bottom CTA
- Dashboard: sidebar nav, top bar with search, 4 metric cards, line chart, recent activity table
- Settings page: left nav categories, form fields on the right, save button at bottom

### Step 2: Make One Structural Decision Differently

Change ONE structural choice from the averaged default. Not visual decoration — structure.

| Averaged Default | Structural Alternative |
|---|---|
| Hero section at top | Start with content, social proof, or a demo |
| 3-column feature grid | Alternating left-right layout, or a single-column feature list with illustrations |
| Centered text | Left-aligned text with asymmetric image placement |
| Sidebar nav | Top nav, or no nav (single-page tool) |
| Card grid for content | Mixed layout: one large featured item + smaller list |

### Step 3: Make One Visual Decision Differently

Change ONE visual treatment from the safe default.

| Averaged Default | Visual Alternative |
|---|---|
| Blue/purple gradient hero | Solid color, photography, illustration, or no hero background |
| White background with gray-50 sections | Warm off-white, very light tint, or dark theme |
| Rounded corners (8–16px) | Sharp corners (0–4px) for a more editorial feel |
| Soft shadows | Flat with borders, or bold elevation |
| Generic sans-serif (Inter, system) | A distinctive typeface with strong personality |

### Step 4: Apply Constraints Deliberately

Constraints force creativity. Choose one:
- "No rounded corners anywhere"
- "Only two colors: black and one accent"
- "Maximum 3 font sizes"
- "No images, only typography and color"
- "Asymmetric layout only"

### Step 5: Verify Against the Cliché Catalog

Check your design against `ai-design-cliches.md`. If you check 3+ boxes, go back to Step 2.

## The Non-Negotiables

While breaking from the average, these must remain:
- Clear visual hierarchy → `../foundations/visual-hierarchy.md`
- Proper signifiers for interactivity → `../foundations/signifiers-affordances.md`
- All states accounted for → `../interaction-design/state-patterns.md`
- WCAG 2.2 AA compliance → `../accessibility/wcag-essentials.md`
- No dark patterns → `dark-patterns.md`

Being distinctive does not mean being unusable. Be unconventional in expression, not in function.

## Quick Anti-Averaging Checklist

Every line is phrased so that **checking the box is the good outcome**. An earlier revision
mixed the two polarities — some items were good when true, others when false — which makes a
checklist unusable as a mechanical pass.

- [ ] I wrote down the averaged default in Step 1 *before* designing, and this design differs
      from it **structurally**, not just decoratively
- [ ] Spacing is consistent but varied in scale (tight within groups, loose between groups)
- [ ] There is one clear primary action, and it is visually dominant
- [ ] At least one choice here is deliberate enough that I can state the reason for it in a
      sentence — "sharp corners because the product is editorial", not "sharp corners because
      variety"
- [ ] Placed beside five other generated designs, **this one would be identifiable** as its own
      thing
- [ ] Nothing in [The Non-Negotiables](#the-non-negotiables) was traded away to achieve any of
      the above

**On Step 1's "what would the generic version look like":** you do not need to search for this,
and if you have no browsing you cannot. **The averaged default is your own first instinct** —
whatever you would have produced without this file. Write that down first. That is precisely
what the training-data average looks like, which is what makes it the right thing to deviate
from.

## References
- NN/g: first impressions and convention research (nngroup.com)
- `ai-design-cliches.md` — the specific cliché catalog
