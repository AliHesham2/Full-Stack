# Superpowers — focused Planner adaptation and provenance

Reviewed the official [v7.0.0 release](https://github.com/obra/superpowers/releases/tag/v7.0.0), commit `bb92a77741419a4ab5f06e711a283343f1ada0c3`, on 2026-10-19. The [MIT license](superpowers-LICENSE) is retained. This records the source of adapted methods, not an upstream installation, role assignment, fresh reviewer dispatch or live workflow trial. Upstream source text is not loaded as an alternate instruction hierarchy.

| Source method | Distinct addition | Local procedure and evidence |
| --- | --- | --- |
| [brainstorming](https://github.com/obra/superpowers/blob/v7.0.0/skills/brainstorming/SKILL.md): intent playback and anti-goals | Summarize each meaningful module, distinguish owner statements from interpretation, and identify unacceptable outcomes without inventing requirements. | SKILL.md section 1; internal TODO and canonical Output understanding/anti-goal fields. |
| [builder check](https://github.com/obra/superpowers/blob/v7.0.0/skills/brainstorming/builder-check-prompt.md) | Read the handoff as a recipient without chat history; surface consequential unanswered questions and rank them. | SKILL.md section 5 and handoff-todos.md readiness; Output records document versions, reviewer mode, questions and disposition. |
| [writing-plans](https://github.com/obra/superpowers/blob/v7.0.0/skills/writing-plans/SKILL.md): interfaces | Make consumed/produced boundaries explicit and compare names, shapes and versions across tasks. | SKILL.md section 4; role TODO boundary tables referencing canonical definitions. |
| writing-plans: task sizing | Prefer coherent, independently reviewable deliverables; fold supporting setup into the work that needs it. | SKILL.md section 4 and role TODO instructions; no forced minute estimates. |
| writing-plans: review focus and proportion | Find prioritized uncovered failure cases, attach them to owning tasks/verification, and remove vague or code-heavy planning detail. | SKILL.md section 5; Output review-focus table and recorded planning checks. |

Existing source/requirement coverage, risks, evidence, resumable records, role separation and approval constraints remain at their current owners; no duplicate general-purpose Skill is added.

## Source instructions deliberately not imported

- Global auto-triggering, provider installers/hooks, mandatory upstream sub-skill invocations and `docs/superpowers/` output paths. Roster, project role procedures and authorized task-record paths control this system.
- Planner-led implementation, executable test authoring, TDD execution, automatic commits/worktrees, merging, publication and worker/reviewer dispatch. Those remain with actual assigned roles and permissions.
- Mandatory conversational scripts, a question after every message, fixed word counts, or a fixed number of uncovered cases. Ask only genuine consequential gaps and preserve the owner's already-settled choices.
- Upstream brainstorming's skip-question/spike shortcuts as a way around project approval, source coverage, security or role boundaries. A throwaway prototype still needs a separately authorized assignment and cannot become production evidence automatically.
- Writing-plans' self-review instruction to fix without re-review. Here, changed scope/interfaces/coverage are rechecked under the project's evidence rules.
- The execution/parallel-agent skills' ability to choose operators, alter tests or settle product ambiguity unilaterally. Planner proposes dependencies and questions; Orchestrator coordinates, Tester owns tests, and the owner decides product scope.

The `[method]` references are source evidence. The local integrated procedure is [../SKILL.md](../SKILL.md), and downstream work follows [../handoff-todos.md](../handoff-todos.md). No upstream tool invocation or copied setup step is required to use these methods.
