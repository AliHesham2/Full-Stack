# User Flows

A flow is a **sequence of states over time** that accomplishes a user goal. Every other file in
this folder describes a moment — a screen, a component, a state. This one describes what
happens between them.

**The failure this file prevents:** designs where every individual screen is well-made and the
sequence is broken — no way back, work lost on error, a dead end after success, or a
"confirmation" that confirms nothing. Screen-by-screen generation produces exactly this,
because each screen is locally plausible and nothing checks the seam between them.

**The rule that governs the rest:** *design the unhappy paths first*. The happy path is the one
everyone designs and the one users least need help with.

## Anatomy of a Flow

Every flow has five parts. A flow missing any of them is incomplete, not minimal.

| Part | Question it answers | Common omission |
|---|---|---|
| **Entry points** | How did the user get here, and from where? | Assuming a single entry |
| **Happy path** | The shortest route when everything works | — (always designed) |
| **Alternate paths** | Valid choices that aren't the default | Guest checkout, SSO, "I don't have one" |
| **Error & recovery** | What happens when a step fails | The whole category |
| **Exit** | What the user sees when done, and where they go next | Dead-end success screens |

### Entry points are plural

The same flow is reached from a nav item, a deep link, an email, a shared URL, an empty state's
CTA, and a returning session. Each arrives with **different context**:

- A deep link may arrive **unauthenticated** — after login, return the user to where they were
  going, not to the dashboard.
- A deep link may arrive **mid-flow** — the design must either resume correctly or explain why
  it cannot. It must never render a broken partial state.
- An entry from an empty state already carries intent — don't re-ask what the user just said.

## Rule 1: One Primary Decision Per Step

A step that asks for three unrelated decisions gets abandoned. A step that asks for one gets
completed. This is why multi-step forms outperform one long form for complex tasks — and why
splitting a *simple* form into steps makes it worse. Split by **decision**, not by field count.

→ Step mechanics, validation, and progress indicators live in [forms-input.md](forms-input.md).

## Rule 2: Every Step Must Be Reversible, and Reversal Must Be Free

- **Back must work** — the browser back button included. If back breaks the flow or resubmits,
  the flow is broken, not the user's expectation.
- **Going back must not destroy input.** Returning to step 2 from step 3 shows what was entered.
  Losing it is the single most damaging flow bug, because it punishes the user for checking
  their own work.
- **Forward after back must not re-ask.** State persists in both directions.
- **Exiting mid-flow needs a decision:** save a draft, discard with confirmation, or block exit.
  Silently discarding is never acceptable. Silently blocking is a dark pattern
  → [../anti-patterns/dark-patterns.md](../anti-patterns/dark-patterns.md).

**Prefer undo over confirm.** A confirmation dialog interrupts every user to protect against a
rare mistake; an undo affordance costs nothing until it's needed. Reserve confirmation for
actions that are genuinely irreversible (permanent deletion, sending money, publishing
publicly) → [../components/modals-dialogs.md](../components/modals-dialogs.md).

## Rule 3: The Flow Must Survive Interruption

Users get phone calls, close tabs, lose connection, and switch devices. Assume every flow is
interrupted at its worst moment.

- **Persist progress** for anything longer than one screen. Local persistence for anonymous
  users, server-side for authenticated ones.
- **Make resumption explicit.** "You have an unfinished application — continue or start over?"
  Silently restoring old state confuses; silently discarding it enrages.
- **Never make a network failure lose user input.** Keep the input, show the error, offer retry
  → [feedback-systems.md](feedback-systems.md).
- **Long operations must survive navigation** or say clearly that they will not.

## Rule 4: Success Is a Step, Not an Ending

A success screen that only says "Done" is a dead end. Every terminal state answers three
questions:

1. **What happened?** Specific and verifiable — "Invoice #1043 sent to sara@example.com".
2. **What can I do about it?** Undo, view the result, download a receipt, share.
3. **What's next?** The most likely following action, or a clear route back to a known place.

The same applies to **partial success** — "3 of 5 items imported" needs to say which two failed
and what to do about them. Reporting a partial failure as a success is a correctness bug.

## Rule 5: Ask Late, Ask Little

- **Defer authentication** until it's actually required. Let users build the cart, configure the
  thing, see the value — then ask. Requiring signup before any value is the highest-abandonment
  pattern there is.
- **Defer every optional field** to after the goal is met. Profile completion belongs in the
  product, not in the doorway.
- **Never ask for what you can derive.** Country from locale, timezone from the browser, city
  from postal code.
- **Ask in the order the user thinks**, not the order the database stores.

## Onboarding Flows

Onboarding is the flow with the worst ratio of designer attention to user tolerance.

| Approach | Use when | Risk |
|---|---|---|
| **Do nothing** — product is self-evident | Simple, familiar products | Undersells non-obvious value |
| **Empty states as onboarding** *(default choice)* | Almost always | None — this is the strongest default |
| **Progressive / contextual hints** | Feature is non-obvious *at the moment it's needed* | Becomes nagging if not dismissible-forever |
| **Setup wizard** | Product genuinely cannot work unconfigured | Abandonment before first value |
| **Upfront tour / carousel** | Rarely justified | Skipped, unread, and forgotten |

**Rules:**

- **Empty states are the best onboarding surface** — they appear exactly where the user is
  stuck, explain what goes there, and offer the action that fills it. No separate flow, nothing
  to dismiss → [state-patterns.md](state-patterns.md).
- **Time-to-first-value is the metric.** Every onboarding step must shorten it. A step that
  only collects data for you lengthens it.
- **Everything must be skippable and re-findable.** A tour that can't be skipped is a dark
  pattern; a tour that can't be replayed is a waste.
- **Never gate the product behind a tour.**

## Destructive Flows

1. **Name the object in the confirmation.** "Delete project *Atlas*" — not "Are you sure?"
2. **State the consequence and its scope.** "This deletes 47 files and cannot be undone."
3. **The confirming button says the verb** — "Delete", not "OK". The user must be able to act on
   the button alone.
4. **Cancel is the default focus**, and Escape cancels → [../components/modals-dialogs.md](../components/modals-dialogs.md).
5. **Scale friction to consequence.** A single item: undo toast, no dialog. An account: a dialog.
   Everything, irreversibly: type-to-confirm. Type-to-confirm on a trivial action is
   condescending; a bare dialog on account deletion is negligent.
6. **Prefer soft delete + undo** wherever the data model allows it.

## Authentication Flows

- **Never make the user guess whether they have an account.** Detect it from the identifier and
  route accordingly, rather than presenting "Sign in" and "Sign up" as a quiz.
- **Preserve intent across authentication.** Deep link → login → *the deep link*, never the
  dashboard.
- Use the correct `autocomplete` tokens — `current-password` for sign-in, `new-password` for
  sign-up. Getting these backwards breaks every password manager →
  [forms-input.md](forms-input.md).
- **Password requirements are shown before typing**, validated live, and stated as what *is*
  required, not as what just failed.
- **Error messages must not leak account existence** ("No account with that email" tells an
  attacker which addresses are registered). Keep the message uniform.
- **Provide the recovery path on the failure screen**, not three clicks away.

## Flow Anti-Patterns

| Anti-pattern | Why it fails |
|---|---|
| Dead-end success screen | User is stranded at the moment of highest trust |
| Back button loses entered data | Punishes the user for verifying their own work |
| Forced linearity in a non-linear task | Users have information out of order |
| Signup wall before any value | Highest-abandonment pattern in existence |
| Error at the end that invalidates step 1 | Validate at the step where it's fixable |
| Confirmation with no information ("Are you sure?") | Confirms nothing; trains reflexive clicking |
| Irreversible action with no confirmation *or* undo | One misclick, permanent loss |
| Progress bar that lies or jumps backward | Destroys trust in every other indicator |
| Silent partial failure | Reports success for work that didn't happen |
| Re-asking for information already given | Signals the product isn't paying attention |

## Flow Audit Checklist

Run this on any multi-step interaction before delivering:

- [ ] Every entry point enumerated, including deep link and unauthenticated arrival
- [ ] Back works at every step and preserves input
- [ ] Mid-flow exit has a defined behaviour (draft / confirmed discard / blocked)
- [ ] Interruption and resumption defined; progress persists
- [ ] Every step has a failure mode designed, with the input preserved and a retry
- [ ] Errors surface at the step where they can be fixed
- [ ] Success state names what happened, offers undo/detail, and gives a next step
- [ ] Partial success is reported as partial, with the failed items identified
- [ ] Destructive steps have friction proportional to consequence
- [ ] No step asks for information already provided or derivable
- [ ] The whole flow is completable by keyboard alone → [../accessibility/keyboard-focus.md](../accessibility/keyboard-focus.md)
- [ ] Every state change is announced to assistive tech → [feedback-systems.md](feedback-systems.md)

## Cross-References

- Where the user *is* in the structure → [../foundations/information-architecture.md](../foundations/information-architecture.md)
- Step and field mechanics → [forms-input.md](forms-input.md)
- Loading, empty, and error states within a step → [state-patterns.md](state-patterns.md)
- Communicating progress and outcome → [feedback-systems.md](feedback-systems.md)
- Revealing complexity across steps → [progressive-disclosure.md](progressive-disclosure.md)
- Wording of confirmations, errors, and success → [../foundations/content-and-microcopy.md](../foundations/content-and-microcopy.md)
- Friction used against the user → [../anti-patterns/dark-patterns.md](../anti-patterns/dark-patterns.md)
