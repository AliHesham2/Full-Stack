# State Patterns

Every UI element exists in multiple states. AI-generated designs almost always show only the "ideal" state. A production-quality design accounts for all states.

## The State Checklist

Two groups. **Element states** apply to a single interactive control; **view states** apply
to any region that displays data. INDEX's "design all states" rule means both lists.

### Element states (6)

| State | Description | Common Omission |
|-------|-------------|----------------|
| **Default** | Normal resting state | Rarely omitted |
| **Hover** | Cursor is over the element | Often missing in AI designs |
| **Active/Pressed** | Element is being activated | Almost always missing in AI designs |
| **Focus** | Element has keyboard focus | Critical for accessibility |
| **Disabled** | Element cannot be interacted with | Often poorly differentiated |
| **Loading** | Element is processing an action | Commonly omitted |

### View states (3)

| State | Description | Covered in |
|---|---|---|
| **Loading** | Data is being fetched | below, "Loading States" |
| **Empty** | Query succeeded, no data | below, "Empty States" |
| **Error** | Request failed | below, "Error States" |

Loading appears in both lists because a button and a list both have one, and they look
different: a button gets an inline spinner, a region gets a skeleton.

### Additional States by Element Type

| Element | Additional States |
|---------|-----------------|
| Text input | Empty, filled, error, success, read-only |
| Toggle/checkbox | Checked, unchecked, indeterminate |
| Select/dropdown | Closed, open, selected, no-selection |
| Data list | Loading, populated, empty, error, filtered-empty |
| Page/section | Loading skeleton, content, error, no-permission |

## Loading States

### Spinner vs. Skeleton vs. Progress Bar

| Pattern | When to Use | When NOT to Use |
|---------|-------------|------------------|
| **Spinner** | Small areas, brief loads (< 1s), action buttons during processing | As the only feedback for loads > 1s |
| **Skeleton screen** | Content areas, cards, lists — when you know the shape of the incoming content | When content shape is unpredictable |
| **Progress bar** | File uploads, multi-step processes, known-duration tasks | When progress is unknown (use spinner or skeleton instead) |
| **Pulsing/blank** | Avoid. Users don't know if the page is broken or loading. | — |

### Skeleton Screen Guidelines

- Match the skeleton shape to the actual content layout (not a generic placeholder)
- Use a subtle pulse animation (opacity 0.3–0.7, 1.5–2s cycle)
- Don't skeleton-ize the entire page at once — load above-the-fold content first
- Skeletons should transition smoothly to real content (cross-fade, not abrupt swap)

## Empty States

An empty state is shown when there's no data to display. This is a design opportunity, not an afterthought.

### Components of a Good Empty State
1. **Illustration or icon** (optional, but helps)
2. **Clear explanation** of why it's empty ("You haven't added any items yet")
3. **Action** to resolve the empty state ("Add your first item" button)

### Common Empty State Scenarios

| Context | Message Tone | Example |
|---------|-------------|----------|
| New user, no data | Welcoming, guiding | "Your dashboard is empty. Let's add your first project." |
| Filtered results, no matches | Neutral, helpful | "No results for 'xyz'. Try adjusting your filters." |
| Error causing empty state | Apologetic, actionable | "Couldn't load data. [Retry]" |
| User cleared all items | Neutral | "All clear! Add items when you're ready." |

**AI failure:** Showing "No data" or a blank screen with no explanation or action.

## Error States

See `feedback-systems.md` for error message guidelines. For visual treatment:

- **Error on a field:** Red border, red error text below the field, error icon
- **Error on a section:** Red-tinted banner at the top of the section with description and retry
- **Error on a page:** Centered error illustration, explanation, and action button
- **Error on a button:** Button shows error state briefly (red flash), then the error appears near the relevant field

## Disabled States

Disabled elements must be clearly distinguishable from enabled elements:
- Reduced opacity (0.4–0.5)
- `cursor: not-allowed` on hover
- No hover/active state changes

### `disabled` and `aria-disabled` are not interchangeable

An earlier revision listed them as alternatives. They behave differently, and the difference is
the whole decision:

| | `disabled` | `aria-disabled="true"` |
|---|---|---|
| Keyboard focusable | **No** — removed from tab order | **Yes** |
| Discoverable by screen reader users browsing the page | Often skipped | Announced as "dimmed/unavailable" |
| Blocks the click | Browser blocks it | **You must block it yourself** in the handler |
| User can find out *why* it's unavailable | No — they can't reach it | Yes — they can focus it and read the reason |

- **`aria-disabled` is the better default** for a button the user might reasonably try to press
  (an incomplete form's submit). It stays reachable, so it can carry an explanation. You must
  prevent the action in JS — ARIA changes semantics, never behaviour.
- **`disabled` is right** when the cause is obvious and momentary — a submit button during an
  in-flight request, where the spinner is the explanation.

### On the opacity value

Text in an **inactive** UI component is explicitly exempt from the WCAG contrast minimum
(1.4.3, "Incidental"), so a faded disabled button is not a violation. That is a compliance
fact, not a design permission: at opacity 0.4 the label is often genuinely unreadable, and a
user who cannot read it cannot work out what they're missing. Fade enough to read as inactive,
not enough to read as absent — and never rely on opacity alone, since it reads as "unavailable"
only in contrast to nearby enabled controls.

**Avoid disabling when possible.** If a user can't do something, explain why rather than graying
out the button with no explanation → [../components/buttons-ctas.md](../components/buttons-ctas.md).

## References
- NN/g on skeleton screens: https://www.nngroup.com/articles/skeleton-screens
- Live design-system references for state coverage: USWDS (https://designsystem.digital.gov/)
  and Carbon (https://carbondesignsystem.com/) — both confirmed serving. GOV.UK
  (https://design-system.service.gov.uk/) is also current, though it blocks automated
  fetches, so it was not machine-verified here.
  *The Australian Government Design System (designsystem.gov.au) is cited in a lot of older
  material but its domain no longer resolves — don't send agents there.*
