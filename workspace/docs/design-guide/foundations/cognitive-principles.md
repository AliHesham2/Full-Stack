# Cognitive Principles for UI Design

The psychological laws that explain *why* certain designs work and others fail. These are not optional guidelines — they describe how human perception and cognition actually function.

## Why This Matters for AI Agents

AI agents tend to produce designs that *look* reasonable but violate cognitive principles because LLMs optimize for visual plausibility, not perceptual efficiency. Understanding these principles helps you produce designs that actually work better, not just look different.

## Core UX Laws

### Fitts's Law

**Principle:** The time to reach a target is proportional to the distance to it and inversely proportional to its size.

**Implication for design:**
- Make frequently-used targets larger (primary buttons, main nav items)
- Place frequently-used targets closer to where the user's attention already is
- The edges and corners of the screen are effectively "infinitely large" in one dimension — use them for persistent controls (macOS menu bar, Windows taskbar exploit this)
- On touch devices, the minimum comfortable target size is 44×44pt (iOS HIG) or 48×48dp (Material Design)

**Common AI failure:** Placing small, similarly-sized buttons at equal distances from each other, forcing the user to carefully aim every time.

### Hick's Law

**Principle:** Decision time increases logarithmically with the number of choices available.

**Implication for design:**
- Categorize choices into small, clearly-labeled groups — the win is the *grouping*, not a magic count (see Miller's Law below)
- Use progressive disclosure to hide options until they're relevant
- Highlight recommended/default options to reduce effective choices
- In navigation, use clear information architecture to narrow the path

**Common AI failure:** Presenting 10+ equally-weighted options in a flat list or grid, forcing the user to read and evaluate every item.

### Miller's Law (7±2)

**Principle:** Working memory can hold approximately 7 (±2) chunks of information at once.

**What it actually says:** Miller (1956) measured **working memory for recall** — how many
chunks a person can hold *in their head* with nothing in front of them. It is a claim about
memory, not about screens.

**The misapplication to avoid.** "No more than 7 navigation items" is the most widely repeated
false design rule, and it does not follow from Miller. Visible options are **recognition**, not
recall — the user is not memorising your menu, they are reading it. Nothing is being held in
working memory, so the limit does not bind. A well-grouped list of 20 links is easier to use
than 5 vague ones hiding three levels each.

**Implication for design — where it genuinely applies:**
- **When the user must carry information *between* steps.** A confirmation code to retype, a
  value from step 2 needed in step 4, options that vanish when a menu closes. Here the limit is
  real: don't make users hold things in their head — show the information where it's needed.
- **The real principle is *chunking*.** Grouping turns many items into few chunks, which is why
  a grouped list scales and a flat one doesn't. Group, label the groups, and stop counting.
- Comparison across items *is* memory-bound: comparing 9 pricing plans on visual scan alone is
  hard. Use a comparison table so the comparison is on screen instead of in memory.

**Where it does not apply:** the length of a visible navigation menu, a list, or a set of
filters. Optimise those for **scannability** — grouping, labeling, order, and hierarchy — not
for a count → [information-architecture.md](information-architecture.md) (Rule 3: prefer
breadth over depth) and [visual-hierarchy.md](visual-hierarchy.md).

### Jakob's Law

**Principle:** Users spend most of their time on *other* sites, so they expect your site to work like those sites.

**Implication for design:**
- Follow established conventions for common patterns (shopping carts, search bars, login forms)
- Deviate from convention only when you have strong evidence it improves the experience
- Innovation in interaction patterns has a high cost — users must learn your unique approach

**Critical nuance:** This does NOT mean copy other designs. It means respect user expectations for *where things are and how they behave*, not *how they look*. A login form should be where users expect it, but it doesn't need to look like every other login form.

### von Restorff Effect (Isolation Effect)

**Principle:** Items that stand out from their peers are more likely to be remembered.

**Implication for design:**
- Make the primary action visually distinct from secondary actions
- Use color, size, or weight contrast sparingly — if everything stands out, nothing does
- This is the principle behind the "one primary button per view" rule

### Tesler's Law (Law of Conservation of Complexity)

**Principle:** Every system has inherent complexity that cannot be eliminated, only moved between user and system.

**Implication for design:**
- Don't try to simplify complex tasks into a single screen — distribute complexity appropriately
- The system should absorb complexity that is mechanical or repeatable
- The user should handle complexity that requires human judgment
- Progressive disclosure is the primary tool for managing this distribution

### Aesthetic-Usability Effect

**Principle:** Users perceive aesthetically pleasing designs as more usable, even when they're not.

**Implication for design:**
- Visual polish matters for perceived trust and usability
- However, do not trade actual usability for aesthetics — the effect enhances trust but doesn't fix broken flows
- This is why AI-generated designs that look "clean" but have poor information architecture still feel wrong to users

## Gestalt Principles in UI

These describe how the human visual system automatically groups elements. They operate below conscious awareness.

### Proximity
Elements close together are perceived as related. This is the strongest Gestalt principle in UI design.
- **Use:** Group related form fields, navigation items, or content sections closer together than unrelated items
- **AI failure:** Even spacing everywhere, which destroys implicit grouping

### Similarity
Elements that look alike are perceived as related.
- **Use:** Consistent icon styles, consistent card shapes, consistent interactive element styling
- **Trade-off:** Over-consistency makes it hard to distinguish between different types of elements

### Common Region
Elements within the same closed boundary are perceived as a group.
- **Use:** Cards, panels, sections with borders or background fills
- **AI failure:** Creating cards around everything, making the page feel like a grid of equal-weight boxes

### Continuity
The eye follows continuous lines and paths.
- **Use:** Alignment, visual flow from top-left to bottom-right (in LTR languages), list layouts

### Closure
The mind completes incomplete shapes.
- **Use:** Truncating content with "..." to imply more content, cropped images suggesting more
- **Caution:** Don't rely on closure for critical information — not all users will infer what you intended

### Figure-Ground
Elements are perceived as either figure (foreground) or ground (background).
- **Use:** Modals and overlays, elevated surfaces, dropdown menus
- **AI failure:** Too many elevated layers creating confusing depth relationships

## Cognitive Load

Cognitive load has three types. Understanding the difference prevents common design mistakes.

### Intrinsic Load
The inherent difficulty of the task. You cannot reduce this — you can only manage it.
- **Strategy:** Break complex tasks into steps, provide examples, use clear language

### Extraneous Load
Load caused by poor design: confusing layout, unclear labeling, unnecessary steps.
- **Strategy:** This is what you eliminate through good UX. Every unnecessary click, confusing label, or poorly organized page adds extraneous load.
- **This is where AI-generated designs most often fail** — they add decorative elements, redundant information, and unclear visual hierarchy.

### Germane Load
The mental effort required to build understanding and transfer knowledge. This is *beneficial* load.
- **Strategy:** Design patterns that help users build mental models (consistent navigation, clear section headers, meaningful icons)

## References

- Laws of UX: https://lawsofux.com
- NN/g on Fitts's Law: https://www.nngroup.com/articles/fitts-law
- Gestalt principles applied to UI: multiple IxDF articles (ixdf.org)
- Cognitive Load Theory (Sweller, 1988)
