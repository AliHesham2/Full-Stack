# Investigating a failed check across system boundaries

Use this procedure when a failed test or unexpected result cannot be explained from its first error, especially across Browser → Next.js → supplied external API, or when a purported fix fails again. It adapts the distinct root-cause method in [Superpowers systematic-debugging](https://github.com/obra/superpowers/blob/bb92a77741419a4ab5f06e711a283343f1ada0c3/skills/systematic-debugging/SKILL.md) and [root-cause tracing](https://github.com/obra/superpowers/blob/bb92a77741419a4ab5f06e711a283343f1ada0c3/skills/systematic-debugging/root-cause-tracing.md) to the Tester role. Source: `obra/superpowers` v7.0.0, commit `bb92a77741419a4ab5f06e711a283343f1ada0c3`, MIT © Jesse Vincent; [local license](references/superpowers-LICENSE). This project procedure, not the upstream workflow, is what an assigned Tester follows.

## 1. Establish the failure

Capture the exact candidate, command, test ID, environment, input/fixture, full error and stack trace, and frequency. Re-run only in an authorized isolated environment. If it is intermittent, record pass/fail counts and vary one likely factor at a time: data, time, browser/OS, order of tests, concurrency or external response. `NOT REPRODUCED` is an observation, not proof that the report is false.

## 2. Locate the failing boundary

Trace the affected path backwards from symptom through permitted frontend boundaries: browser state, Next.js route/action, feature query and HTTP adapter. Compare supplied contract values/statuses and safe correlation evidence where available. External server/domain/database investigation needs separate owner authorization; report missing evidence rather than infer a backend cause. Instrument only the assigned test/harness scope, redact secrets/PII and preserve actual candidate evidence.

When a similar path works, compare its inputs, configuration, dependency version, timing and state with the failing path. List material differences before choosing a cause. A review of source can suggest where to measure, but it does not establish the runtime boundary that failed.

## 3. Test one explanation at a time

Write a falsifiable hypothesis: “If X is the cause, changing only Y in the test setup should change result Z.” Use the smallest safe experiment, keep other variables fixed, and record both outcomes. A failed hypothesis is useful evidence; replace it rather than stacking speculative fixes. If the environment cannot support an experiment, mark the cause `UNKNOWN` and specify what evidence would discriminate it.

Separate:

- **Observed defect:** expected/actual behavior reproduced on a named candidate.
- **Root cause:** causal mechanism supported by a boundary trace or discriminating experiment.
- **Hypothesis:** plausible explanation not yet tested.
- **Source/design gap:** expected behavior itself is missing or contradictory.

## 4. Hand off and verify a fix

Report the smallest reproduction, first bad boundary if known, tested hypotheses, likely affected paths and regression test to the Orchestrator. The Implementer proposes and makes production changes. Tester may write an assigned failing test in its own area during the initial implementation phase and record its original failure. Provide the reproducer and relevant affected suite/quality gates to Orchestrator, which personally checks the corrected candidate, including whether the failure was suppressed or moved to the next layer. Under the [delivery cycle](../../orchestrator/skillz/delivery-cycle.md), no automatic Tester return follows a correction; do not claim the post-fix outcome yourself without a separately owner-authorized assignment.

If a previous fix fails again, or different fixes keep exposing the same cross-cutting state/contract issue, stop recommending local patches. Report the repeated pattern and possible architecture or requirement conflict to the Orchestrator for decision. Do not declare an architectural change yourself.

## Evidence to put in Output

Record candidate, environment, exact command and output; minimal repro; expected/actual; observed boundary trace; each tested hypothesis and result; root-cause confidence (`CONFIRMED`, `LIKELY`, `UNKNOWN`); and remaining uncertainty. Avoid claiming certainty from a single stack-trace frame, a green retry, or another agent's explanation.
