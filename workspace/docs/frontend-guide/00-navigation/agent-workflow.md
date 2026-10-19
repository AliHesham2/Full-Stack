# Agent Workflow

This file defines how an AI coding agent should consume this guide system.

## Workflow Overview

Start from [Roster](../../../rostar/roster.md), your assigned role's prompt and Skill, and the shared [delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md). This engineering guide routes technical reading; it does not appoint operators or replace role sequencing, ownership or evidence requirements. The steps below apply to the assigned phase, not permission to perform every role yourself.

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
- Read the current approved Planner Output/DEC/CHG entries, software TODO, actual design TODO/version and the linked task control record (see [delivery-and-observability](../05-delivery/delivery-and-observability.md)). Use their actual authorized paths; do not require historical planning templates.
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

Follow the assigned role's source-gap and escalation procedure:

- Missing required behavior, technical source, contract or permission blocks the dependent work. Report exact sources, affected items and the needed decision through the assigned Orchestrator; Planner records requirement/decision gaps in its current Output.
- Bounded primary-source research or source maintenance requires actual authority; it does not automatically change project policy or override Implementer's local syntax restriction. The responsible role records source-backed proposals, and the actual owner approves decisions within its authority.
- Engineering constraints and role procedures govern different concerns. If they conflict materially, cite both and report it rather than silently overriding either. Continue only independent work whose prerequisites remain valid.

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
  (`tests/<assigned-tester>/`, with identities/write boundaries from Roster and the brief; see [testing.md](../04-quality/testing.md)).
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

- Deliver the assigned role's Output: actual changes, candidate identity, sources, checks and limitations. Proposed work is not execution evidence.
- Report plan/contract/design changes through Orchestrator to their owning role. Planner maintains canonical requirement/DEC/CHG records; Orchestrator maintains the single linked coordination control record. Workers do not edit another role's decision/state artifacts.
- After Tester's initial report, Orchestrator reviews every result and personally verifies Designer/Implementer corrections under the shared cycle; do not automatically return fixes to Tester.

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
8. **When in doubt, never assume.** A missing required decision/source blocks affected work and returns through Orchestrator to the actual owner. Within an already approved range, use the assigned role's rules; do not invent contract, schema, security or product behavior.
