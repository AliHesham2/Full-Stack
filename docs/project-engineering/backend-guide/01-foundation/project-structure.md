# Project Structure — Organizing the Repository

> **Part of:** Foundation — Backend Project Guide
> **Read when:** Creating directories, deciding where a file belongs, or judging whether a new module/directory is justified.
> **Related:** `architecture.md`, `../02-domain-and-api/domain-logic.md`, `../05-data/data-architecture.md`
> **Status:** Stable principles · Exact layout is ecosystem-dependent (see framework guide)

---

## 1. Position

There is no universal directory tree. There is a universal **decision process** for what belongs where. The structure below is a reference anatomy for a modular monolith; the framework/stack guide may adjust names, but ownership rules stay.

## 2. Reference Anatomy (Modular Monolith)

```
src/
├── main.ts / index.ts          # composition root: wiring only
├── config/                     # typed, validated configuration object
├── shared/                     # cross-cutting primitives (see §4 — strict bar)
│   ├── errors/                 # error taxonomy types
│   ├── logging/                # logger setup + correlation IDs
│   └── http/                   # edge helpers: validation glue, serializers
├── modules/                    # domain modules live here
│   └── orders/
│       ├── orders.routes.ts        # edge: HTTP handlers
│       ├── orders.controller.ts    # edge: request→use-case translation (optional)
│       ├── orders.use-cases.ts     # application: orchestration
│       ├── orders.domain.ts        # domain: entities + invariants (if non-trivial)
│       ├── orders.repo.ts          # infra: DB access (interface + impl)
│       ├── orders.events.ts        # contracts this module publishes/subscribes
│       └── orders.types.ts         # public DTOs = the module's contract
└── infra/                      # process-level infrastructure
    ├── db/                      # pool, migration bootstrap
    ├── cache/                   # cache client (if introduced deliberately)
    └── jobs/                    # worker entrypoints (if background jobs exist)
```

**The tests themselves live outside `src/`, one folder per tester:** `backend/tests/kilo/` and
`backend/tests/opencode/`. Each mirrors the modules and endpoints it proves
(`tests/kilo/modules/orders/orders.use-cases.test.ts`), and every test's name starts with its
acceptance check (`AC-3: …`). Two independent suites for the same code; nobody else writes in a
tester's folder, and each tester keeps its own helpers (fixtures, factories) inside it: the suites
share nothing.

Rules encoded here:

1. **Composition root wires, nothing else.** `main.ts` instantiates infra, injects it into modules. Business logic never reads config globals directly.
2. **A module's public surface is its types + use-cases.** Other modules import `orders.types`/`orders.use-cases`, never `orders.repo` or `orders.domain` internals.
3. **`infra/` holds process-level machinery** (DB pool, clients). Module-level data access lives in the module's repo file, implemented against the shared pool.

## 3. Ownership Rules (What Goes Where)

| Artifact | Home | Never in |
|---|---|---|
| HTTP parsing, status mapping, serialization | module edge layer | use-cases, domain |
| Orchestration ("do A, then B, transactionally") | use-cases | controllers, repos |
| Business invariants, policies, calculations | domain | controllers, repos |
| SQL / ORM queries | module repo | use-cases, domain |
| Authorization policy definitions | module domain/use-case; enforcement at edge + use-case | client |
| Input schema validation | edge (wrapped helper) | deep in use-cases |
| External API clients (Stripe, email…) | infra clients, wrapped behind an interface owned by the consuming module | domain |
| Cross-module contracts (DTOs, events) | publishing module's `types`/`events` files | `shared/` |
| Migrations | global `migrations/` (ordered, single timeline) | per-module ad-hoc scripts |

**Anti-diagnostic:** if a search for a business term (e.g., "refund") spans 7 directories, structure is technical, not domain-driven — refactor toward ownership.

## 4. The `shared/` Bar (Keep It Nearly Empty)

`shared/` (or `common/`, `utils/`) is the entropy sink of every rotting codebase. Admission bar for a shared artifact:

1. Used by ≥2 modules **today** (not "soon")
2. Contains **no business concepts** (no `OrderHelpers`, no `UserUtils` — those belong to their modules)
3. Stable, generic, and plausibly publishable as a standalone library (logging, error base types, date/clock wrapper, money type)

Anything business-flavored goes to its owning module even if two modules currently duplicate it. Duplication with clear ownership beats shared code with unclear ownership — the shared version couples the two modules' change cycles forever.

## 5. When Is a New Module Justified?

Create a new domain module when ALL hold:

- A coherent business capability with a name non-engineers recognize
- It owns data (at least one table or one external resource)
- Its rules change for business reasons distinct from existing modules

Create a new directory (non-module) only when a distinct responsibility exists that doesn't fit existing directories — and then ask why the reference anatomy didn't anticipate it. Create a new layer only per `architecture.md` §5.

**Not justified:** "one file got long" (split within the module), "different technology" (that's infra, and probably premature), "the tutorial had this folder".

## 6. Repositories with Multiple Deployables

If (and only if) the architecture decision (`architecture.md` §3) produced separate services:

- One repo per independently-deployed service, plus a `contracts/` package (or repo) holding shared API/event schemas with independent versioning
- Shared **libraries** (not services) may live in a common package — but shared **code** is a coupling decision: treat every addition as an architecture decision
- Never a monorepo subdirectory structure where one service's deploy implicitly ships another's code

**This project holds only `backend/`.** Its clients (a Next.js frontend's server, another service)
are other projects with their own repos: no code is shared, and they meet only over HTTP.

## 7. Structure Audits (Run When Adding a Module)

- [ ] Does any module import another module's repo/domain internals?
- [ ] Does `shared/` contain business vocabulary?
- [ ] Is there a directory with <3 files that hasn't grown in a month (candidate for deletion)?
- [ ] Can you name, for every table, exactly one owning module?
- [ ] Would a new engineer (or agent) locate the refund logic in ≤2 guesses from the repo root?

## 8. Failure Modes

- **Folder-per-type** (`controllers/`, `services/`, `repos/` at top level with 40 files each): technical partitioning; one feature's change touches 4 directories with 40-way merge conflicts. Migrate incrementally by extracting one domain module at a time.
- **Utils gravity well:** `utils.ts` grows one "temporary" helper at a time until it is imported everywhere and owns nothing. Enforce the §4 bar ruthlessly.
- **Config sprawl:** 12 `.env`-reading modules. Config is loaded once, typed once, injected (`../10-delivery/configuration.md`).
- **Test mirror confusion:** tests structured as a mirror of `src/` for unit tests but nowhere for integration tests. Integration tests should mirror **behavior** (modules/endpoints), not files (`../09-testing/testing-strategy.md`).
- **Zombie scaffolding:** empty directories from init templates (`.github/ISSUE_TEMPLATE`, `docs/`, `services/`) that signal capabilities that don't exist. Delete; recreate on need.
