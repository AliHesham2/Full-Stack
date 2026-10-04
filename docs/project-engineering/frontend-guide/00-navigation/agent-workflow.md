# Agent Workflow

This file defines how an AI coding agent should consume this guide system.

## Workflow Overview

In this project, the stages of every task and when it is done are set by your Skill file
(`docs/.claude/<you>-Skill.md`). The sections below give each stage its knowledge: 1–3 →
understand and inspect · 4 → not covered · 5 → your criteria · 6 → the work · 7–8 → verify ·
9 → your report.

```
Understand → Inspect → Identify → (not covered → report) → Plan → Implement → Test → Audit → Finish
```

### 1. Understand

- What is the current project phase? (Initialization, or Phase N)
- What exactly is being asked to build?
- What are the acceptance criteria?

**For initialization (Phase 0):** Read [initialization.md](../01-foundation/initialization.md) completely.

**For feature phases:** Identify the feature type and read only the relevant sections.

### 2. Inspect

- Read the existing project structure (if it exists).
- Read the decisions already made: the feature's PRD (`plans/<feature>/PRD.md` → Code Structure → New Packages and Decisions) and `docs/.claude/Decisions-Taken.md` (see [delivery-and-observability](../05-delivery/delivery-and-observability.md)).
- Identify existing patterns, boundaries, and conventions.
- Identify which files will be affected.

### 3. Identify Relevant Knowledge

Use this decision tree:

Paths are relative to `frontend-guide/`.

```
What kind of work is this?
├── Project setup / first-time structure
│   → 01-foundation/initialization, architecture, project-structure
│     + 02-ui/theme-and-dark-mode (the theme files are created at setup)
│
├── Styling / colors / spacing / fonts
│   → 02-ui/design-tokens
│
├── Theme / dark mode / brand change
│   → 02-ui/theme-and-dark-mode
│
├── Shared UI component (components/ui/)
│   → 02-ui/components, 02-ui/portability, 02-ui/accessibility
│
├── Page layout / responsive behavior
│   → 02-ui/layout-and-responsive
│
├── Tables, cards, lists, empty/error states, toasts, dialogs
│   → 02-ui/data-display-and-feedback
│
├── Form / user input
│   → 02-ui/forms, 02-ui/accessibility
│
├── Interactive behavior / state change
│   → 02-ui/components, 03-data/state-and-data
│
├── Navigation / routes / protected pages / URL state
│   → 01-foundation/routing, 03-data/state-and-data
│
├── Data from backend / API / auth
│   → 03-data/backend-integration, 03-data/state-and-data, 04-quality/security
│
├── Performance · Security · Tests · Error/loading handling · Dependencies · Lint/format
│   → 04-quality/performance · security · testing · resilience · dependencies · code-quality
│
├── Env variables / config / feature flags · CI / deployment / production debugging
│   → 05-delivery/environments-and-config · delivery-and-observability
│
├── Starting a new project from this kit
│   → 02-ui/portability
│
└── Unsure about a decision
    → 00-navigation/decision-framework
```

**Rule:** Read the minimum necessary. Do not load every guide file for every task.

### 4. Not Covered → Stop and Report

The same rule as your Skill file, RULES 1:

- **Neither this guide nor the Skills cover it** → stop, and report it in the mailbox: what is not
  covered, where you looked, and the decision that is needed (the PRD writer instead writes it as
  an Open Question in the PRD). Do not silently invent a requirement. Report the gap; Claude may authorize bounded primary-source research/reproduction and record the resulting technical decision. Evidence can challenge a factual claim without overriding the fixed architecture.
- **A Skills file disagrees with this guide** → the guide wins: follow it, carry on, and report
  the conflict so the Skills file gets fixed.
- **Two files of this guide disagree** → stop and report it, as for a gap.

### 5. Plan

Before writing code, determine:

- **What files will change?** List them.
- **What new files (if any) will be created?** List them and justify each.
- **Which existing patterns apply?** Reference the specific guide section.
- **Are there accessibility implications?** Check [accessibility.md](../02-ui/accessibility.md).
- **Are there performance implications?** Check [performance.md](../04-quality/performance.md).
- **Are there security implications?** Check [security.md](../04-quality/security.md).
- **What tests are needed?** Check [testing.md](../04-quality/testing.md).
- **Is this a new directory or abstraction?** Check [decision-framework.md](./decision-framework.md) for the justification criteria.

### 6. Implement

- Follow the established patterns from the relevant guide sections.
- Do not violate existing architectural boundaries.
- Write code that a future agent can understand without context.
- Use semantic HTML. Use accessible patterns. Follow security defaults.
- Prefer composition over abstraction.
- Prefer explicit over implicit.

### 7. Test

- The unit and component tests are written by the testers, each in its own folder
  (`tests/kilo/`, `tests/opencode/`; see [testing.md](../04-quality/testing.md)).
- The implementer runs every suite with `npm run check` and verifies all tests pass.
- A tester's test fails because of your change → fix the code. The test itself looks wrong →
  report BLOCKED. Never edit or delete a tester's test.
- Tests prove behavior, not implementation: nothing for trivial prop forwarding, CSS classes, or
  third-party library behavior.

### 8. Audit

After implementation, verify:

| Check | Question | Reference |
|-------|----------|-----------|
| Architecture | Does this fit the existing structure? | [architecture](../01-foundation/architecture.md) |
| Placement | Is this file in the right directory? | [project-structure](../01-foundation/project-structure.md) |
| Accessibility | Are there a11y regressions? | [accessibility](../02-ui/accessibility.md) |
| Performance | Does this introduce unnecessary JS/renders/bundle? | [performance](../04-quality/performance.md) |
| Security | Does this handle user data safely? | [security](../04-quality/security.md) |
| Tests | Do the tests provide real value? | [testing.md](../04-quality/testing.md) |
| Consistency | Does this follow existing conventions? | [code-quality](../04-quality/code-quality.md) |
| Regressions | Did anything unrelated break? | Run full test suite |
| Complexity | Did we add unnecessary abstraction? | [anti-patterns](../04-quality/anti-patterns.md) |

### 9. Finish

- Report as your Skill file's REPORT rule says: what changed, what you verified, the checks, what
  remains, and every decision you made with its source.
- Never write a decision file yourself: Claude records what must be kept in
  `docs/.claude/Decisions-Taken.md`. A new architectural boundary is decided in the PRD, never
  during the build.

## Phase Lifecycle

```
Project description
       ↓
Phase 0 — Read 01-foundation/initialization completely
       ↓
Strong foundation established
       ↓
Phase 1 — Read relevant sections, implement
       ↓
Audit against all quality dimensions
       ↓
Phase 2 — Read relevant sections, implement
       ↓
Audit against all quality dimensions
       ↓
Phase 3 ...
```

## Key Rules

1. **Never redesign the project for a single feature.** Extend the existing architecture.
2. **Never create speculative infrastructure.** Build what is needed now.
3. **Never skip the audit step.** Every phase must be verified.
4. **Never duplicate logic.** If it exists, reuse it. If it's similar, abstract only when justified by current requirements.
5. **Never assume the frontend enforces security.** Security boundaries are backend-enforced; frontend contributes defense in depth.
6. **Never treat accessibility as a cleanup step.** It is part of every implementation decision.
7. **Never optimize prematurely.** Establish performance foundations, then measure before optimizing.
8. **When in doubt, never assume.** A choice that neither the PRD, the guides nor the Skills
   decide → handle it as your Skill file's RULES 1 says (a report in the mailbox; for the PRD
   writer, an Open Question in the PRD). Where they allow several options, choose the simplest
   reversible one.