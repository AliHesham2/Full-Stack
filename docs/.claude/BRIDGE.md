# BRIDGE — How the Orchestrator Reaches the Workers

The bridge has its own short manual. Where it lives (the user changes this line by hand):

```
BRIDGE GUIDE: C:\Users\aly_m\OneDrive\Desktop\custom_cmp\BRIDGE-GUIDE.md
```

**Read that ONE file before the first send.** It holds the tools, the loop, how to wait in the
background, the errors, the token rules and the paths. Open a worker's own manual next to it
(`kilo.md`, `opencode.md`, `antigravity-2.0.md`, `antigravity-ide.md`) only when the guide is
not enough. This file holds only what THIS project adds or fixes on top of it.

## Who Is Who

| Role | Worker | Model | Mode | Port |
|---|---|---|---|---|
| PRD writer | Anti2.0 (Antigravity 2.0) | `Claude Sonnet 4.6 (Thinking)` | — | — |
| Implementer | IDE (Antigravity IDE) | `Gemini 3.8 Flash (High)` | — | — |
| Tester | Kilo | the free Nemotron Ultra, `kilo/…` (exact id: `kc_models` with `contains:"nemotron"`) | `agent: code` | 4466 |
| Tester | opencode | `opencode/nemotron-3-ultra-free` (`oc_models`) | `agent: build` | 4488 |

## This Project's Rules, on Top of the Guide

```
PROJECT RULES
├──▶ start the orchestrator from THIS project's root, and open every window with
│    -Project "<this project's root>": the gate allows writes only inside it
├──▶ chats: ONE chat per app for this project (ag_chat finds it by the project's folder, or
│    creates the first one). Record every conversation id, port and session id in
│    docs/.claude/State.md
├──▶ testers: a FRESH session for every test round, never the reused one. Both get the same
│    code; each writes only in its own tests folder
├──▶ permissions: nothing is approved blind
│    ├──▶ Kilo / opencode → the gate answers by policy (inside the project only, never secrets,
│    │    never destructive). A refusal is usually right: change the task, never the gate
│    ├──▶ Anti2.0 → decide its waiting prompt by the PRD writer's policy: read anywhere except
│    │    secrets; write ONLY plans/<feature>/ and plans/INDEX.md; nothing in backend/
│    │    or outside the project; never destructive. Answer it as the guide says
│    └──▶ IDE → asks nothing: its permissions are the USER's
├──▶ quota
│    ├──▶ Kilo / opencode → another FREE model (never a paid one), log it in
│    │    Decisions-Taken.md, tell the user
│    └──▶ Anti2.0 / IDE → STOP and tell the user: the user switches the account and sends
│         "continue". Never switch the model, never retry
├──▶ reports: THIS project's rule wins over the guide's "ONE line". A worker reports the four
│    parts (Changed · Verified · Checks · Remaining) with a SOURCE for every decision or
│    finding, as its Skill file says. You still verify on disk, never by the report alone
├──▶ a stop in the middle of a task → "continue" in the SAME session or chat, same model
├──▶ never send a task twice into a busy worker; an unknown delivery is held, never resent
└──▶ the guide does not cover it, or it contradicts this file → tell the user; never guess
```
