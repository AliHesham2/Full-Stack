# Superpowers skills review for the Tester role

Reviewed [obra/superpowers `skills/`](https://github.com/obra/superpowers/tree/bb92a77741419a4ab5f06e711a283343f1ada0c3/skills) at v7.0.0 / commit `bb92a77741419a4ab5f06e711a283343f1ada0c3` (2026-10-09). Its [MIT license](superpowers-LICENSE) is retained. This is a scope/duplication audit, not an installation or authority grant. Only the distinct Tester method from `systematic-debugging` is adapted into [../failure-investigation.md](../failure-investigation.md). No upstream skill file is copied or auto-invoked.

| Upstream skill | Tester decision and reason |
| --- | --- |
| `systematic-debugging` | **Add focused method:** cross-boundary evidence, working/failing comparison, one falsifiable hypothesis per experiment, and escalation on recurring failures. Tester diagnoses and reports; Implementer fixes. |
| `verification-before-completion` | Already covered by Tester candidate-specific evidence, AC status rules and Output; also by the project's Every Task gate. No duplicate skill. |
| `test-driven-development` | Primarily an Implementer coding method. Tester already writes a failing regression check when assigned, using `bug-reproduction` and `unit-testing` from the QA library. No duplicate role procedure. |
| `receiving-code-review` | Technical verification of feedback is already required when Orchestrator validates Tester findings; source's response style and implementation steps do not belong in Tester instructions. |
| `requesting-code-review` | Independent review is an Orchestrator handoff; Tester Output already provides reviewable evidence. No extra dispatch rule. |
| `brainstorming` | Product/design discovery belongs to Planner/Designer/owner, not independent Tester execution. |
| `writing-plans` | Implementation planning belongs to Planner; Tester uses its own coverage TODO. |
| `executing-plans` | Implementer workflow, not Tester. |
| `subagent-driven-development` | Orchestrator/Implementer delegation workflow; Roster and actual host permissions control dispatch. |
| `dispatching-parallel-agents` | Orchestrator-only dispatch decision; Tester cannot appoint or launch operators by reading a skill. |
| `finishing-a-development-branch` | Merge/branch disposition is outside Tester authority. |
| `using-git-worktrees` | Workspace isolation is host/project setup, not an automatic Tester requirement or permission. |
| `using-superpowers` | Its global skill-activation rule would conflict with this project's role entry and Roster; do not import. |
| `diagnosing-superpowers` | Diagnoses the Superpowers skill runtime itself, not product software quality. |
| `writing-skills` | Skill-authoring workflow, outside assigned Tester verification. |

The source's `systematic-debugging` implementation phase tells its agent to edit production code. That portion is deliberately not imported: the Tester records evidence and hands defects to the Orchestrator. Its condition-based waiting and test quality ideas are already covered by `test-reliability` and `bug-reproduction` in the existing QA library. Its verification-before-completion evidence rule is already part of the local Tester Output contract.
