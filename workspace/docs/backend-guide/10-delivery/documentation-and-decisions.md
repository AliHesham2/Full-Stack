# Documentation & Architectural Decisions — Memory for Humans and Agents

> **Part of:** Delivery — Backend Project Guide
> **Read when:** Deciding what to document, writing an ADR, maintaining runbooks — or when a future agent (including you) will need to understand why the system is shaped this way.
> **Related:** `../01-foundation/architecture.md` §8, `../08-observability/observability.md` §4 (runbook-linked alerts), `../00-navigation/agent-workflow.md`
> **Status:** Stable principles

---

## 1. Position

Documentation is **the system's memory** — for the next developer, the on-call engineer at 3AM, and the AI agent that will modify this code six months from now (statistically: all the same category — context-free arrivals). Code tells them WHAT happens; docs must carry the WHY and the HOW-TO-OPERATE that code cannot express. Documentation that no context-free reader can act on is inventory, not memory.

## 2. The Minimum Set (Per Project)

| Artifact | Answers | Lives | Freshness bar |
|---|---|---|---|
| **README** | How do I run/test/migrate this locally? | repo root | Always true — verified by a newcomer-boot test |
| **Architecture overview** | What are the modules/boundaries and why? | `docs/architecture.md` + ADRs | Per structural change |
| **ADRs** | Why is it this way? (the decision log) | the feature's PRD and `docs/.claude/Decisions-Taken.md` (§4) | Append-only |
| **API contract docs** | What does the surface promise? | generated from code/schema where possible | Per API change |
| **Runbooks** | The service is down — now what? | linked from alerts | Reviewed per incident postmortem |
| **Operational notes** | Deploys, migrations, backups, restore drills | `docs/ops/` | Per change to those flows |

Everything else is optional; these six are the difference between a maintainable system and archaeology.

## 3. README Discipline

The README must get a context-free engineer (or agent) to a running, tested local instance: prerequisites, env setup (pointing at `.env.example`), run, test, migrate, and the ONE command per common task. If any step exists only in someone's head or chat history, the README is broken. Verify by actually performing it fresh — the "newcomer test" is the only honest documentation CI.

## 4. ADRs (Architecture Decision Records)

**In this project an ADR is not a separate file.** It lives in exactly one of two places:

- **Decided before the build** → a DEC-n row in the feature's PRD (`plans/<feature>/PRD.md` →
  Code Structure → Decisions), written by the PRD writer and approved by the user. A choice the
  guides leave open is first an Open Question; its answer (the user's, or Claude's for a
  technical choice) becomes the row. A later change is a new PRD version with a Change Log line;
  a superseded row stays, marked `(REMOVED in vN)`.
- **Decided during the work** → an entry in `docs/.claude/Decisions-Taken.md`, written only by
  Claude (the orchestrator): its own decisions, and the ones from the workers' reports that must
  be kept.

Workers report each decision with its source and never write a decision file. Both places keep
the fields of the format below: Context, Decision, Alternatives (and why each lost),
Consequences, Assumptions & revisit triggers (with the direction to take then). The date and
status come from the PRD's version and Change Log, or from the entry's date.

Format (short, structured, append-only):

```markdown
# ADR-NNN: <Decision title>
Date / Status (proposed | accepted | superseded by ADR-MMM)

## Context
The forces at play: constraints, requirements, measured facts.

## Decision
What we chose — stated in one crisp paragraph.

## Alternatives considered
What else was on the table, and the specific reason each lost.

## Consequences
What we accept by choosing this (including new failure modes and costs).

## Assumptions & revisit triggers
What must stay true for this to remain right; what signal means re-open it; which direction to
take then.
```

Rules:

- ADRs record **decisions with alternatives** — where a wrong choice is expensive to reverse (`../01-foundation/complexity-control.md` §5): architecture shape, consistency model, queue introduction, sharding, auth model, buy-vs-build. Not styling choices; not trivia.
- **Assumptions + revisit triggers are mandatory fields** — they convert the ADR from history into a live instrument (`../01-foundation/architecture.md` §8): future agents re-evaluate against them instead of treating the decision as physics.
- Superseded ADRs stay, marked — the trail of WHY-the-old-way-lost prevents circular re-litigation (and re-introduction of known-bad ideas).
- AI-agent-specific value: an agent inheriting a repo with real ADRs reads context in minutes; without them it infers intent from code — confidently, wrongly.

## 5. Runbooks (Incident Documentation)

Written for a stressed human at 3AM (or an agent mid-incident): linked directly from the alert that fires them (`../08-observability/observability.md` §4). Per alert:

1. What this alert means (in product terms)
2. First three checks (specific: which dashboard, which query, which log filter)
3. Mitigations in order of safety (flag-off → rollback → scale → page owner)
4. Escalation path

A runbook nobody has executed is a draft. Postmortems (blameless, per incident: timeline → contributing causes → action items with owners) feed runbook updates — the incident-to-docs loop is the actual learning mechanism of an operating team.

## 6. Documenting for AI Consumption Specifically

Because future maintainers include agents, structure beats prose:

- Machine-greppable conventions: consistent module anatomy, named use-cases, ADR numbering — agents navigate structure far better than narrative
- Inline `WHY` comments ONLY where the code's shape defies expectation ("expand-phase migration: dual-writing for ADR-014 rollout — do not 'simplify'") — these are landmine-defusal notes for exactly the refactoring-happy agent
- Prefer generated docs (OpenAPI/schema-derived) over hand-maintained twins — the hand-maintained copy is always stale, and stale docs are worse than none (they're confidently wrong)
- The repository's own copy of decision-relevant facts (this README + `docs/`) is what a context-limited agent will actually read first: put the load-bearing facts there, not in chat history or tickets

## 7. Failure Modes

- **Documentation theater:** a beautiful wiki from launch week, stale since — every page wrong in the dangerous direction (confidently outdated). Fix: generate what can be generated, newcomer-test what can't, delete what fails.
- **The tribal-knowledge subsystem:** "ask Ahmed about billing" — a single-person bus factor plus an undocumented blast radius; the README test surfaces it as the gap it is.
- **ADRs as press releases:** records of conclusions without alternatives or assumptions — no one can later tell whether conditions changed, so the decision calcifies into pseudo-law.
- **Runbook rot:** alerts without runbook links; runbooks referencing dashboards that were deleted in a re-org. Reviewed-per-postmortem is the maintenance loop.
- **The stale-comment trap:** comments describing what code USED to do (the code changed, the comment lied) — worse than no comment; comments explain WHY, code review checks the match.
- **Chat-history-as-docs:** decisions live in Slack threads; the repo shows only the aftermath; future agents find contradictions they can't resolve. Decisions graduate to ADRs or they didn't happen.
