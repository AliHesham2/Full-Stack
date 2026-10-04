# Testing Strategy — What to Test, Where, and Why

> **Part of:** Testing — Backend Project Guide
> **Read when:** Writing tests for a backend change, deciding what to mock, structuring test suites and CI gates, or judging coverage quality.
> **Related:** `../05-data/concurrency-and-idempotency.md` §7, `../03-auth/authorization.md` §8, `../10-delivery/ci-cd.md`
> **Status:** Current consensus-flavored: integration tests against real dependencies dominate modern backend testing; mock-heavy "unit pyramids" are legacy advice

---

## 1. Position

Tests exist to catch **classes of defects** cheaply. Each test type is a tool for a specific defect class — coverage theater (tests that exist but catch nothing) is the failure mode this guide guards against. The current (and research-supported) backend consensus: most confidence per effort comes from **integration tests running real code against real infrastructure (in containers)**, with a focused unit layer for genuine logic and a thin E2E layer for critical journeys.

## 2. The Levels (and What Each Is FOR)

| Level | Target defect class | Infrastructure | Volume target |
|---|---|---|---|
| **Unit** | Pure logic: domain rules, calculations, state machines, mappers | none — no I/O | many, for real logic only |
| **Integration** | Wiring: SQL correctness, transactions, auth wiring, error mapping, serialization | real DB (container), faked externals | the BULK of the suite |
| **API/contract** | The HTTP contract: status codes, error envelope, pagination, validation | app booted, real DB, HTTP calls | every endpoint |
| **Concurrency** | Races: double-submit, parallel invariants, redelivery | real DB, parallel execution | few, for hot invariants |
| **E2E** | Critical user journeys across the real stack | full env | few (smoke + money paths) |
| **Failure-path** | Degradation: dependency down, timeout, retry, DLQ | fault injection (proxies/kills) | per critical dependency |

**In this project E2E is the user's:** the user checks every finished feature end to end, by hand (CLAUDE.md → Workflow step 10). The testers write every other level, each in its own folder.

**What NOT to test:** the framework (routing works), the ORM (it persists), mock implementations (testing your mocks tests nothing), and exhaustive permutations of trivial mapping code. Every test must name the defect it would catch — if you can't, it's coverage theater.

## 3. The Mocking Decision (Where Realism Wins)

| Dependency | Test double? | Why |
|---|---|---|
| Database | **REAL** (container per suite/session) | SQL, transactions, constraints, migrations are the defect surface — mocks verify nothing about them |
| Cache | real-or-embedded per tooling | invalidation logic interacts with real TTL/serialization behavior |
| External APIs (payment, email) | **FAKE/stub** at the HTTP boundary | You can't hit production third parties; DO simulate their failure modes (timeout, 4xx, 5xx, malformed bodies) |
| Time/clock | fakeable clock injected | deterministic time-dependent logic (`../11-specialized-concerns/time-and-money.md` §2) |
| Randomness/IDs | injected generator | reproducing specific scenarios |
| Message broker | embedded/testcontainer where feasible | delivery/redelivery semantics matter (§5) |

Mock-the-boundary, not-the-world: stub the external HTTP call (contract-shaped fixtures), but let YOUR code — routing, validation, use case, SQL — run real under it.

## 4. Test Data & Isolation

- **Isolation strategy per suite:** integration suites use per-test database (create/truncate via template-DB trick) or transaction-rollback-per-test; parallel-safe from day one (database-per-worker or schema-per-worker — retrofitting parallelism into a serial suite is a rewrite).
- **Factories over fixtures:** builders that construct valid entities with overrides (`OrderFactory.build({status: 'shipped'})`) — shared static fixture files rot and hide required fields; factories fail loudly when schemas change.
- **No test depends on another's data** (implicit ordering bugs) and **no test depends on real elapsed time** (flaky-by-design).
- Migration currency: the suite builds schema by RUNNING the migration chain (`../05-data/migrations.md` §6) — schema drift is caught by CI, not by production.

## 5. The Non-Negotiable Test Classes

1. **Authorization denial tests** per endpoint: user A on user B's object → 403/404; wrong role → 403; cross-tenant with valid token → denied; list endpoints scope-check by count (`../03-auth/authorization.md` §8). This class catches the #1 real-world vulnerability class.
2. **Validation tests:** every endpoint's malformed-input battery (type confusion, boundaries, oversized arrays/strings, unknown fields, unicode).
3. **Idempotency/concurrency tests:** double-submit same key ⇒ single effect; parallel submits ⇒ invariant holds exactly (`../05-data/concurrency-and-idempotency.md` §7).
4. **Failure-path tests:** dependency timeout/down ⇒ bounded behavior (503/queue, not hang); retry policy behaves; DLQ receives after max attempts (`../07-reliability/error-handling.md`).
5. **Migration safety:** chain runs clean on empty + realistic-volume DB (§4).
6. **Error-contract tests:** every mapped error produces the envelope shape (`../02-domain-and-api/api-design.md` §4).
7. **Money paths E2E** (the user's check, by hand): signup → core action → payment → receipt, as close to real as the environment allows. The testers cover each of these steps with API/contract tests.

## 6. CI Gates (What Blocks a Merge)

- Lint + typecheck (cheap, instant — catch the defect classes machines catch)
- Full test suite **against containers** (real DB per §3) — parallel, time-bounded
- Migration chain application (§4)
- Security scans: dependency audit (`../10-delivery/dependencies.md` §4), secret scanning
- Coverage as **smoke signal, not target**: track trends and uncovered critical paths (auth, money, idempotency must be covered); a mandated global % number produces theater, not safety
- Build artifact (fail the build on type/config errors that runtime would discover late)

Flaky-test policy: quarantine-with-ticket immediately (a flaky test left in the suite teaches the team red is ignorable — the most expensive lesson CI can teach).

## 7. Writing Tests for a Change (Agent Procedure)

For any backend change, before claiming done:

1. List the defect classes this change could introduce (logic? contract? concurrency? auth? migration?)
2. Pick levels per §2 targeting those classes — usually: unit for new logic + integration/API for wiring + the §5 classes touched
3. Write the failure paths FIRST (what happens when this fails?) — AI agents systematically under-write these
4. Verify tests fail meaningfully before the fix when bug-fixing (a bug test that never failed proves nothing)
5. Run the suite — full green, not "my tests"

## 8. Failure Modes

- **The mock pyramid:** 90% of tests mock the DB and repos, asserting on interaction counts; production defects live in SQL/wiring; suite is green and useless (§3 — this is the defining legacy anti-pattern).
- **Coverage theater:** `assert(result).toBeDefined()` × 300 to hit a number (§1).
- **Fixture rot:** shared JSON fixtures from 2024; new required field added; tests pass by constructing entities outside factories; production breaks.
- **Serial-suite lockdown:** one test mutates shared state, suite can't parallelize, CI takes 50 minutes, everyone stops running it before push.
- **Happy-path myopia:** every test feeds valid input to cooperative dependencies — the first malformed webhook body in production finds zero armor.
- **Flaky normalization:** re-running until green becomes culture; red loses meaning; real regressions ship.
- **Untested failure modes as "edge cases":** dependency down, double-delivery, concurrent edit — labeled edge cases and skipped, though they're the standard failure modes of production (§5).
