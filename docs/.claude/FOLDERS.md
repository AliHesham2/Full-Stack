# FOLDERS — The Map

Everyone reads this map before looking for a file: the orchestrator, Codex and every worker.
Find the folder you need here, then open only that. All paths are from the project root.
Guide files and the `docs/.claude` files are listed by their real names, without `.md`.

```
<project root>/
│
├── CLAUDE.md ················ the orchestrator's workflow (Claude)
├── AGENTS.md ················ Codex's instructions (advisor)
│
├── docs/ ···················· RULES. Read-only for workers
│   ├── .claude/ ············· who does what, and the project's memory
│   │   ├── FOLDERS ·········· this map
│   │   ├── BRIDGE ··········· how the orchestrator reaches the workers
│   │   ├── Anti2-Skill ······ the PRD writer's rules
│   │   ├── IDE-Skill ········ the implementer's rules
│   │   ├── Kilo-Skill ······· tester 1's rules
│   │   ├── opencode-Skill ··· tester 2's rules
│   │   ├── templates/ ······· PRD · ROADMAP (copied for every feature)
│   │   ├── State ············ where the work stands (orchestrator only)
│   │   ├── Decisions-Taken ·· decisions taken during the work (orchestrator only)
│   │   └── Lessons ·········· this project's lessons (orchestrator only)
│   │
│   ├── project-engineering/ · THE GUIDE: this project's decisions. It wins over Skills
│   │   └── frontend-guide/ ·· README (the index)
│   │       ├── 00-navigation : agent-workflow · decision-framework        ← START HERE
│   │       ├── 01-foundation : architecture · initialization · project-structure · routing
│   │       ├── 02-ui ········: design-tokens · theme-and-dark-mode · components · forms ·
│   │       │                   layout-and-responsive · data-display-and-feedback ·
│   │       │                   accessibility · portability
│   │       ├── 03-data ······: backend-integration · state-and-data
│   │       ├── 04-quality ···: security · performance · resilience · testing · dependencies ·
│   │       │                   code-quality · anti-patterns
│   │       └── 05-delivery ··: delivery-and-observability · environments-and-config
│   │
│   └── Skills/ ·············· HOW a technology works (the guide decides WHAT we do)
│       └── react · Tailwind css · javascript
│
├── plans/ ··················· one folder per feature
│   ├── INDEX.md ············· every feature → its folder → its status
│   └── <feature>/ ··········· PRD.md · ROADMAP.md · the user's material (the backend's API contract too)
│
└── frontend/ ················ the Next.js app: the ONLY app in this project
    ├── src/ ················· the code (layout: frontend-guide/01-foundation/project-structure)
    ├── tests/kilo/ · tests/opencode/ ·· each tester writes ONLY in its own folder
    └── node_modules/next/dist/docs/ ··· Next.js's own docs for the installed version
```

The backend (Express and its database) is ANOTHER project: none of its code lives here.

## Who Writes Where

| Folder | Writes it | Everyone else |
|---|---|---|
| `plans/<feature>/` and its line in `plans/INDEX.md` | Anti2.0 (PRD writer) | read |
| `frontend/src/` | IDE (implementer) | read |
| `frontend/tests/kilo/` | Kilo | read |
| `frontend/tests/opencode/` | opencode | read |
| `docs/.claude/State.md`, `Decisions-Taken.md`, `Lessons.md`, statuses in `plans/` | Claude (orchestrator) | read |
| `docs/` (everything else) | nobody during the work; a validated fix: Claude | read |

## Outside the Project

```
├── the TEMPLATE ·········· the folder this project was copied from (CLAUDE.md → Template)
│   └── learn/ ············ lessons and issues for the template: never copied
└── the BRIDGE ············ the worker bridge and its guide (docs/.claude/BRIDGE.md → BRIDGE GUIDE)
```
