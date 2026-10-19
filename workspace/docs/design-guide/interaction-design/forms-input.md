# Forms and Input Patterns

Forms are where users convert intent into action. Poor form design is the #1 cause of abandonment in sign-ups, checkouts, and data entry.

## Layout

### Top-Aligned Labels (Recommended Default)
- Labels above inputs
- Fastest completion time (eye moves down, then down again)
- Works well for long labels
- Requires more vertical space

### Left-Aligned Labels
- Labels to the left of inputs
- Good for short labels and forms where vertical space is limited
- Slower scanning (eye zigzags)
- Requires consistent label width to align inputs

### Floating/Inline Labels
- Labels start inside the input and move up on focus
- Saves space but has usability issues:
  - Users can't see all labels at once before starting
  - Can look like pre-filled values
  - Accessibility challenges with screen readers
- **Use only when space is extremely constrained and labels are short.** The screen-reader
  concern is overstated *if* you use a real `<label>` — the genuine, unavoidable problems are
  visual: the shrunken label often fails contrast at its small size, it collides with browser
  autofill styling, and it can overlap or clip when text is translated or the user zooms.

## Field Ordering

1. **Logical flow:** Group related fields together. Name, then email, then phone (not interleaved).
2. **Required fields first:** Don't make users scroll past optional fields to reach required ones.
3. **Progressive complexity:** Start with easy, familiar fields. Put complex or unfamiliar fields last.
4. **One field per row for most fields.** Exception: very short, related fields (City + State, Month + Day + Year).

## Validation

### When to Validate

| Timing | Use for |
|--------|--------|
| **On blur** (field loses focus) | Format validation (email, phone, URL) |
| **On input** (keystroke) | Character limits, format masks, search filtering |
| **On submit** | Cross-field validation (password match, date ranges) |

**Rule:** Don't validate on input for format checks. Showing "invalid email" while the user is still typing "john@" is frustrating.

### Inline Validation Messages

- Position directly below the field being validated
- Use color + icon: green check for valid, red X for invalid — never colour alone
- Be specific: "Must be at least 8 characters" not "Invalid password"
- Clear the error message when the user starts correcting the field
- **Connect the message to the field programmatically.** A message that only sits *visually*
  below the input is not associated with it: point `aria-describedby` at the message's `id`
  and set `aria-invalid="true"`. Otherwise a screen-reader user hears the field name and no
  reason for the failure.
- **On submit failure, move focus to the first invalid field** (or to a summary listing the
  errors as links). Leaving focus on the submit button strands keyboard users below every error
  they need to fix.
- Wording rules for the message itself → [../foundations/content-and-microcopy.md](../foundations/content-and-microcopy.md)

### Required Field Indication

- Mark required fields with an asterisk (*) next to the label
- Include a note at the top: "* Required" (not at the bottom where it won't be seen)
- Alternatively: only mark optional fields as "(optional)" — this is often clearer because most fields are required
- **The asterisk is decoration; the `required` attribute is the requirement.** Screen readers
  may skip a bare `*` or read it as "star" with no meaning. Always set `required` (or
  `aria-required="true"`) on the input as well, and hide the visual asterisk from AT
  (`<span aria-hidden="true">*</span>`) so it isn't announced twice.
- Whichever convention you pick, **apply it to every field in the product.** A form marking
  required fields next to one that marks optional ones makes both unreadable.

## Input Types

Always use the correct HTML `type` attribute — it affects mobile keyboards, validation, and accessibility:

- `type="email"` → email keyboard with @
- `type="tel"` → phone keyboard
- `type="url"` → URL keyboard with /.com
- `type="number"` → numeric keyboard
- `type="search"` → search keyboard (may show clear button)
- `type="password"` → hides characters (provide show/hide toggle)

### `autocomplete` Is Required, Not Optional

**WCAG 2.1/2.2 SC 1.3.5 "Identify Input Purpose" is Level AA.** Any field collecting
information *about the user* must carry the correct `autocomplete` token. This is the
highest-value, most-often-omitted line in a form: a legal requirement and the biggest
usability win available, for the cost of one attribute.

```html
<input name="fname"  autocomplete="given-name">
<input name="email"  type="email" autocomplete="email">
<input name="tel"    type="tel"   autocomplete="tel">
<input name="addr"   autocomplete="street-address">
<input name="pc"     autocomplete="postal-code">
<input name="ccnum"  autocomplete="cc-number" inputmode="numeric">

<!-- sign-in and sign-up are DIFFERENT tokens; getting this wrong breaks password managers -->
<input type="password" autocomplete="current-password">   <!-- sign in -->
<input type="password" autocomplete="new-password">       <!-- sign up / change -->

<!-- lets the OS offer the SMS code -->
<input autocomplete="one-time-code" inputmode="numeric">
```

**Rules:**

- Use tokens from the HTML spec's fixed list. Invented values like
  `autocomplete="username-field"` do nothing at all.
- **Never `autocomplete="off"`** on user-information fields. It is a 1.3.5 failure, browsers
  increasingly ignore it, and on password fields it actively fights password managers. The
  legitimate uses are rare — a one-off code the browser should not remember.
- Pair with `inputmode` for numeric-but-not-`number` fields (card numbers, OTPs, postal
  codes). `type="number"` on a card number is wrong: it strips leading zeros and adds
  spinners.
- Each part of a name or address needs **its own** token. A single `autocomplete` on the
  `<form>` does not distribute to the fields.

## Multi-Step Forms

Use when:
- The form has 6+ fields that don't all fit on one screen
- The form has logical sections (Shipping → Payment → Review)
- You want to increase perceived progress

**Guidelines:**
- Show a progress indicator (step number + label, not just dots)
- Allow backward navigation (don't force users to complete steps in order if they want to go back)
- Save data between steps (don't lose work if they navigate away)
- Show a review/summary step before final submission
- Don't use more than 5–7 steps

## Submit and Actions

- **Primary action (Submit/Save):** Prominent, right-aligned (in LTR)
- **Secondary action (Cancel):** Less prominent, left-aligned or next to submit
- **Tertiary actions ("Save as draft"):** Text button, not outlined
- **Never use a destructive-looking button for submit.** Red button = delete, not "Place order."

## References
- NN/g: Form Design articles (multiple) — foundational research on form usability
- Luke Wroblewski, *Web Form Design* (2008, principles still valid)