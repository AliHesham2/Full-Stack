# Dark Patterns (Deceptive Design)

Patterns designed to trick users into actions they didn't intend. These are **never acceptable**, regardless of business pressure. They erode trust, violate regulations (EU Digital Services Act, US state laws), and have been classified as unfair commercial practices.

## Categories of Dark Patterns

### 1. Hidden Information
Concealing or delaying information the user needs to make an informed decision.
- **Pre-checked boxes** for email subscriptions, terms acceptance, or add-ons
- **Burying cancellation** in multiple sub-pages or requiring a phone call
- **Drip pricing:** Showing a base price and adding mandatory fees at the final step

### 2. Urgency / Pressure
Creating false urgency to prevent rational decision-making.
- **Fake countdowns:** "Only 2 left!" when inventory is not limited
- **Fake activity:** "5 people are looking at this" when no one is
- **Artificial scarcity:** "Sale ends in 10 minutes" that resets

### 3. Misdirection
Drawing attention away from important information.
- **Visual interference:** Making the "decline" option tiny and the "accept" option huge
- **Confirmshaming:** "No thanks, I don't want to save money"
- **Hidden costs:** Free trial that requires credit card, making cancellation hard

### 4. Forced Action
Requiring the user to do something unrelated to their goal.
- **Mandatory account creation** to view pricing or content
- **Forced email opt-in** to proceed
- **Required app download** for basic web functionality

### 5. Social Proof Manipulation
Fabricating or misrepresenting social validation.
- **Fake reviews** or selective review display
- **"Join 10,000 satisfied customers"** without evidence
- **Showing notifications** for actions by fake or non-recent users

## Why This Matters for AI Agents

AI agents are often prompted to "increase conversion" or "optimize sign-ups." Dark patterns may be generated if the agent optimizes for the requested metric without ethical constraints.

**If a design decision would make a user feel tricked, manipulated, or trapped, it's a dark pattern regardless of effectiveness.**

## Legal Context

All three primary instruments below were read directly from the official text.

- **EU Digital Services Act** — Regulation (EU) 2022/2065, **applicable from 17 February 2024**.
  **Article 25(1)** is the operative provision, and its wording is broader than "dark patterns":

  > *"Providers of online platforms shall not design, organise or operate their online
  > interfaces in a way that deceives or manipulates the recipients of their service or in a
  > way that otherwise materially distorts or impairs the ability of the recipients of their
  > service to make free and informed decisions."*

  Note **"materially distorts or impairs"** — a design does not have to be deceptive to be
  caught; impairing free decision-making is enough. Maximum fines are **6% of annual worldwide
  turnover** (Art. 52(3); Art. 74(1) for very large platforms).

- **California (CCPA as amended by CPRA)** — this is the US instrument that matters, not the
  bot-disclosure law. Cal. Civ. Code **§1798.140(l)** defines the term in statute:

  > *"'Dark pattern' means a user interface designed or manipulated with the substantial effect
  > of subverting or impairing user autonomy, decisionmaking, or choice…"*

  And the consent provision states plainly: *"agreement obtained through use of dark patterns
  does not constitute consent."* **Consequence for design:** a consent flow built with a dark
  pattern doesn't merely risk a fine — the consent it collects is **void**, so every downstream
  use of that data is unauthorised.

- **EU Consumer Rights Directive:** requires clear pricing; pre-checked boxes for extra payments
  are prohibited.
- **UK Consumer Protection from Unfair Trading Regulations 2008** and successor consumer
  legislation: misleading actions and omissions are unlawful.
- **US State AG actions:** multiple states have sued companies over deceptive design.

> **The practical test is not "is this illegal in my jurisdiction."** The DSA's "materially
> distorts or impairs" and California's "subverting or impairing user autonomy" both describe
> *effect*, not intent. If the design works because the user didn't fully understand, it is in
> scope.

## References
- Deceptive Design (Harry Brignull, who coined the term): https://deceptive.design — this
  replaced the original `darkpatterns.org`, which no longer serves the content. Cite the
  current domain; the old one appears in a lot of stale writing.
- NN/g: https://www.nngroup.com/articles/deceptive-patterns