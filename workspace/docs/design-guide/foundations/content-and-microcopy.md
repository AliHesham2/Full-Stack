# Content and Microcopy

Words are the highest-density UI element there is. A button is a rectangle plus a word, and the
word carries nearly all the meaning. Interfaces are read before they are used.

**The failure this file prevents:** filler copy. Generated interfaces default to "Lorem ipsum",
"Welcome back!", "Oops! Something went wrong", "Get Started", and "Learn More" — text that
occupies the correct space while communicating nothing. Placeholder copy shipped as real copy is
the most common way a competent-looking design turns out to be unusable.

## Rule 1: Write the Copy Before the Layout

Layout designed around fake text breaks when real text arrives — real labels are longer, real
error messages wrap, real product names are not "Acme". Designing with real content exposes
these problems while they are still cheap to fix.

If the real content is unknown, use **realistic worst-case** content: the longest plausible
label, a two-line error, an empty list, a user with one very long name. Never Lorem ipsum.

## Rule 2: Buttons Say What They Do

A button label is a **verb phrase describing the outcome**, understandable with no surrounding
text — because screen-reader users often hear the labels in isolation, and everyone else clicks
without reading the paragraph.

| Weak | Strong | Why |
|---|---|---|
| OK | Delete project | Names the action and object |
| Submit | Send invoice | "Submit" describes the form's job, not the user's |
| Yes / No | Discard changes / Keep editing | Both options readable alone |
| Learn More | See pricing | Says where it goes |
| Get Started | Create your first project | Says what happens |
| Click here | View the report | Never describe the mechanism |

- **The button must echo the title.** Title "Delete project", button "Delete". If the title asks
  a question and the button says "OK", the user has to reconstruct the meaning.
- **Never make the destructive option the neutral-sounding one.** "OK" to delete is a dark
  pattern by ambiguity → [../anti-patterns/dark-patterns.md](../anti-patterns/dark-patterns.md).
- **First person vs second person:** pick one and never mix. "Update my settings" and "Update
  your settings" in the same product is a tell that no one is in charge of the voice.

## Rule 3: Error Messages Have Three Jobs

**What happened → why → how to fix it.** Missing the third is what makes an error message
useless, and it's the one that gets omitted.

| Bad | Good |
|---|---|
| "Oops! Something went wrong." | "Couldn't save your changes — the connection dropped. Your work is safe. Retry." |
| "Invalid input." | "Card numbers are 16 digits. This one has 15." |
| "Error 422." | "This email is already registered. Sign in instead, or reset your password." |
| "Password does not meet requirements." | "Passwords need at least 12 characters. This one has 8." |

**Rules:**

- **Never blame the user.** "You entered an invalid date" → "Enter the date as DD/MM/YYYY".
  Passive framing of the user's action, active framing of the fix.
- **Never expose internals.** Stack traces, error codes, and table names are not messages. If a
  support code is genuinely needed, show it *in addition to* a human sentence.
- **Say what was preserved.** "Your draft is saved" turns a failure into an inconvenience. This
  matters more than the error text itself → [../interaction-design/user-flows.md](../interaction-design/user-flows.md).
- **Be specific about the constraint** — the actual limit, the actual format, the actual field.
- **Drop the exclamation marks and the humour.** Nobody has ever been cheered up by "Oops!"
  while losing work. Tone falls as stakes rise (see Rule 6).
- **Put the message next to the field it concerns**, and announce it to assistive tech
  → [../interaction-design/feedback-systems.md](../interaction-design/feedback-systems.md).

## Rule 4: Empty States Explain and Offer

Three parts, in order: **why it's empty → what belongs here → the action that fills it.**

> **No invoices yet**
> Invoices appear here once you bill a client.
> **[Create your first invoice]**

Distinguish the three kinds — they need different copy:

| Kind | Copy focus |
|---|---|
| **First use** (never had data) | Explain the concept and offer the primary action |
| **Cleared** (had data, now none) | Confirm this is expected — "All caught up" — not an error |
| **No results** (filter/search) | State what was searched, and offer to clear the filter |

A "no results" state that looks identical to a "nothing here yet" state makes users think the
product is broken.

## Rule 5: Labels, Placeholders, and Helper Text Are Not Interchangeable

- **Label** — always visible, always present. Never replace it with a placeholder: the
  placeholder disappears on focus, taking the question with it, and fails WCAG 3.3.2.
- **Placeholder** — an *example* of the format, never an instruction. "e.g. +966 5X XXX XXXX",
  not "Enter your phone number".
- **Helper text** — persistent guidance below the field, shown *before* the error occurs. Format
  requirements belong here, not in the error.
- **Error text** — replaces or accompanies helper text after failure, and is announced.

→ Field mechanics: [../interaction-design/forms-input.md](../interaction-design/forms-input.md).

## Rule 6: One Voice, Variable Tone

**Voice** is constant — it's the product's personality. **Tone** shifts with the user's
situation. The single most common copy failure is holding tone constant.

| Situation | Tone |
|---|---|
| Onboarding, empty states, success | Warm, encouraging |
| Routine interaction | Neutral, brief, invisible |
| Errors, warnings | Plain, calm, specific — no jokes, no exclamation marks |
| Data loss, security, money, deletion | Serious, unambiguous, zero personality |

Playfulness is a feature in an empty state and a liability in a payment failure.

## Rule 7: Write for Translation Even If You Aren't Translating

These cost nothing now and are expensive to retrofit:

- **Never concatenate sentence fragments** in code. `"Delete " + count + " items"` cannot be
  translated correctly — word order and plural rules differ. Use complete, parameterised
  strings.
- **Never assume two plural forms.** English has 2; Arabic has 6. Use a plural-aware formatter,
  not `count === 1 ? "item" : "items"`.
- **Allow ~30% text expansion** from English. A button sized exactly to "Save" breaks at
  "Guardar"/"Speichern". Design buttons to hug content with real padding, never fixed widths.
- **Never build meaning from word order in the UI.** "Sort by: [dropdown] ascending" won't
  reorder cleanly.
- **Format numbers, dates, and currency with the locale's formatter**, never by hand.
  `DD/MM` vs `MM/DD` is a correctness bug, not a preference.
- **Don't put text in images**, and don't rely on an icon alone to carry a word's meaning.

→ Arabic/RTL specifics, including text expansion and mixed-direction text:
[../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md).

## Rule 8: Mechanical Conventions

- **Sentence case for everything** — headings, buttons, labels, menu items. It reads faster than
  Title Case and avoids per-word capitalisation decisions. Pick one convention and hold it: a UI
  mixing "Create New Project" and "Delete project" looks unfinished.
- **No ALL CAPS for anything longer than a short label.** It reduces reading speed and is read
  letter-by-letter by some screen readers.
- **Numerals, not words**, for anything scannable: "3 items", not "three items".
- **Cut ruthlessly.** "In order to" → "to". "Please note that" → delete. "You can use this page
  to" → delete. Front-load the meaningful word: users scan the first two words of each line.
- **No jargon the user didn't teach you.** If it must appear, define it on first use.
- **Truncate with intent.** Middle-truncate filenames (`report_2026_fin….pdf` hides the
  extension; `report_2026…_final.pdf` doesn't). Always expose the full string on hover *and*
  focus — hover-only fails keyboard and touch.

## Content Anti-Patterns

| Anti-pattern | Why it fails |
|---|---|
| Lorem ipsum shipped as content | The design was never tested against real meaning |
| "Oops! Something went wrong" | Communicates nothing; the three jobs all unfilled |
| "Click here" / "Learn more" | Meaningless out of context; fails link-purpose expectations |
| Placeholder used as the label | Vanishes on focus; WCAG 3.3.2 failure |
| Exclamation marks in errors | Cheerfulness at the user's worst moment |
| Same jokey tone everywhere | Trivialises the moments that matter |
| Vague CTA ("Get Started") | Says nothing about what actually happens |
| Error text that only says what's wrong | The fix is the part the user needs |
| Emoji in headings and CTAs | Generated-copy tell → [../anti-patterns/ai-design-cliches.md](../anti-patterns/ai-design-cliches.md) |
| Fixed-width buttons sized to English | Breaks on the first translation |

## Cross-References

- Naming things in the structure → [information-architecture.md](information-architecture.md)
- Announcing messages to assistive tech → [../interaction-design/feedback-systems.md](../interaction-design/feedback-systems.md)
- Empty/error state design → [../interaction-design/state-patterns.md](../interaction-design/state-patterns.md)
- Button hierarchy and styling → [../components/buttons-ctas.md](../components/buttons-ctas.md)
- Manipulative wording → [../anti-patterns/dark-patterns.md](../anti-patterns/dark-patterns.md)
- Arabic, RTL, expansion → [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md)
