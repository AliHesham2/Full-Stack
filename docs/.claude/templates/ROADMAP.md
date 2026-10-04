# ROADMAP — <fill: feature name>

| | |
|---|---|
| PRD | `plans/<fill: feature-folder>/PRD.md`, version <fill: N>, approved on <fill: YYYY-MM-DD> |
| Status | planned |

`Status`: `planned` → `building` → `testing` → `ready-for-user-check` → `done`.

Ready means automated checks passed on the recorded revision, with manual checks explicitly pending. Done requires the user's own check of the API on
that revision/build. Pending manual checks remain UNVERIFIED, never PASS.

## How to Write This Roadmap

1. **One step = one prompt = one commit.** A step is small enough to build and verify in one go.
   Steps are numbered 1, 2, 3 … in order, and every column is filled (`—` where there is
   nothing), except `Commit`, which is filled when the step is verified (rule 5).
2. **The build order is fixed:** database → API. The feature `setup` follows its initialization guide's
   own order instead, where the app's FIRST step builds its skeleton (or scaffold) together
   with THE CHECK, so every later step can run it. Keep --passWithNoTests through the initial
   build. After the first test round has discovered/executed suites from both testers, add a
   CHANGE REQUEST step on setup to remove the flag. Anti2.0 updates the PRD Version and Change
   Log; Claude checks the PRD/ROADMAP; the implementer edits package.json; both testers rerun
   the same changed revision. Setup cannot be tested, and later features cannot begin, until
   nonempty suites pass with the strict script. A step never depends on a later step.
3. **A step builds code units (`MOD-n`) from the PRD's Code Structure.** `Covers` lists those MOD
   IDs (and the FL flows the step completes). `Files` is EXACTLY the paths of the MODs it covers,
   plus the app's `package.json` and `package-lock.json` when the step adds a package the PRD
   lists (a PKG-n, named in the step's `Covers`). No other file: a file the PRD's Code Structure
   does not have is a design change, back to the PRD first. A `scaffold` MOD's entry in
   `Files` is its app folder: the step's diff may hold what its command generates, plus the
   changes to the step's other MODs. `Covers`
   names only IDs the PRD defines and does not mark REMOVED. The orchestrator compares `Files`
   with `git diff`.
4. **Every MOD is built by a step, every PKG is added by one, and every FL flow is completed by
   one** (a change request later adds new steps for the MODs it changes). Through its MODs every
   DB, API and BR item of the PRD is covered. The AC items are checked by the testers.
5. **Step status** (set by the orchestrator only): `planned` → `built` (the implementer reported
   DONE) → `verified` (the check, the diff and the sources were checked) → `tested` (both testers
   PASS). `Commit` is filled when the step is verified.
6. **Never delete a section, and replace every `<fill…>` marker.** The Chain has one entry for
   every FL flow in the PRD that is not REMOVED.
7. **Before you report DONE, check every rule above against the WHOLE document, one by one**, and
   put each rule's result in your report, with the coverage list: every MOD, every PKG and every
   FL → its step. Claude checks them again by hand.

## Steps

| Step | What | Implementer | Depends on | Files (create / change) | Covers | Status | Commit |
|---|---|---|---|---|---|---|---|
| 1 | <fill> | IDE | — | <fill: the MODs' paths, e.g. backend/src/…> | <fill: MOD-1> | planned | |

## The Chain

For every flow in the PRD, the same path written with its real files.

### FL-1 — <fill: flow name>

```
client (another project)
  → API-1   backend/src/<fill>   <fill: METHOD /path>
    → DB-1   <fill: table>
```
