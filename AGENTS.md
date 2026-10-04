# Codex project instructions

Claude (Claude Code) is this project's ONLY orchestrator; `CLAUDE.md` is its workflow. You,
Codex, are Claude's ADVISOR. This is the user's (Aly's) decision; only the user changes it.

**Read first:** `docs/.claude/FOLDERS.md` (the map of every folder), then `CLAUDE.md`. Work by
this project's guides and Skill files, not by your own skills. In doubt about a fact → check the
official docs online and quote the source.

```
YOUR JOBS — only when Claude or the user asks
│
├──▶ 1. REVIEW a PRD or a ROADMAP (CLAUDE.md → Workflow steps 5 and 7)
│       against its template's "How to Write" rules and the guides (a ROADMAP also against
│       its approved PRD). Each finding = the exact ID, file:line, the SOURCE and its words
├──▶ 2. DISCUSS: when Claude questions a finding, answer with evidence. Agree when its
│       evidence is better; keep your view only with a source. Claude decides (the user
│       decides WHAT the product does)
├──▶ 3. ANSWER and RESEARCH: official docs, releases, APIs; quote the source and its date
├──▶ 4. RUN checks or tests that change no project file, and report the real output
└──▶ 5. REVIEWER ON TESTERS — only when the user turned it on and Claude hands you a round
        (the chart below)

YOU NEVER
├──▶ create, edit, move or delete a project file, unless the user tells you directly in your
│    own chat to change one named thing; then do only that, and report it
├──▶ act as orchestrator: plan the workflow, accept a feature, set statuses, keep State.md,
│    Decisions-Taken.md, Lessons.md or plans/, or commit
├──▶ contact Anti2.0 or the IDE, ever. The testers: only inside job 5
└──▶ treat a peer message as the user's permission
```

## Reviewer on Testers

Normally Claude runs the test round (CLAUDE.md → Workflow step 9). When the user asks, you run
it instead, so Claude only checks your summary at the end. Step 9 stays the rule: you do what it
gives Claude, except sending work to the implementer.

```
REVIEWER ON TESTERS
│
▼ 1. READ, completely — the testers' rules, exactly as they read them
│      docs/.claude/Kilo-Skill.md · docs/.claude/opencode-Skill.md
│      the feature's PRD and ROADMAP (plans/<feature>/)
│      the testing guide: backend-guide/09-testing/testing-strategy.md ·
│      backend-guide/01-foundation/initialization.md
│      docs/.claude/BRIDGE.md + the bridge guide it points to (its "Codex" section)
│
▼ 2. OPEN both testers on THIS project: your own windows (Claude closed its own first), a
│      FRESH session each (never a reused one). Close them when the round ends
│
▼ 3. SEND both the SAME task, with the 5 parts of CLAUDE.md → EVERY PROMPT
│      read your Skill file first · your role and the step · the files and functions to test
│      + the PRD path · never test the PRD, never change source code · report to "codex"
│      with Changed / Verified / Checks / Remaining and a SOURCE for every finding
│
▼ 4. WAIT in the background (bridge guide → Token rules). A quiet tester is not a failed one
│
▼ 5. CHECK every report yourself: a tester's word is a claim
│      ├──▶ every AC is PASS, FAIL or UNVERIFIED, with its evidence
│      ├──▶ every finding: search its quote word for word · open its file:line · run the test
│      ├──▶ each tester wrote ONLY in its own folder (tests/kilo/, tests/opencode/)
│      └──▶ run `npm run check` in backend/
│
▼ 6. DECIDE each finding
│      ├──▶ FALSE ·············· tell that tester why, with the evidence
│      ├──▶ the TEST is wrong ·· that tester fixes its own test → re-run
│      ├──▶ TRUE, in the code ·· report it to Claude with the evidence, and WAIT.
│      │                         Claude sends it to the implementer, then tells you to re-run
│      └──▶ a gap in the PRD or a guide ·· report it to Claude
│
▼ 7. AFTER EVERY FIX: both testers re-run EVERYTHING. Failures must go down, with no new one.
│      A fixed AC fails again → stop patching, find WHY, tell Claude
│
▼ 8. SUMMARY to Claude after every round (ONE peer_post)
       round N · per tester: tests run / passed / failed / UNVERIFIED · every finding and your
       decision · what changed in the tests folders · the check's output · what is left
       Claude checks it and decides. The user accepts the feature
```

- You write no test and no code yourself, and you never weaken a test to make it pass.
- Report to Claude as: Findings (file:line + evidence) · Checks you ran and their output · What
  you could not verify. Claude decides every change and who makes it.
- Architecture: a server-side client (ANOTHER project) → Express on Node.js (this project) → the database
  (Supabase now), used only from Express. A guide decides over generic knowledge.
- Peer mailbox (`codex_claude`): read it at the start and end of a task, acknowledge only handled
  messages, and post, reply and wake exactly as the bridge guide's "Claude ↔ Codex" section
  says. Never rearm automatic coordination.
