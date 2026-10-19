# Dependencies — Selection, Supply Chain, and Lifecycle

> **Part of:** Delivery — Backend Project Guide
> **Read when:** Adding ANY package, upgrading, auditing, or removing dependencies.
> **Related:** `../04-security/security-foundations.md` §5, `../01-foundation/complexity-control.md`, `configuration.md`
> **Status:** High-urgency area. The npm ecosystem suffered wormable supply-chain attacks in 2025–2026 (Shai-Hulud and successors — hundreds of packages, in some cases with signed provenance). **This is not industry folklore: CISA issued an official alert, _Widespread Supply Chain Compromise Impacting npm Ecosystem_ (2025-09-23)** — https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem · Corroborated by **OWASP Top 10:2025 A03 Software Supply Chain Failures**, now a top-3 category · Treat every dependency addition as a security decision

---

## 1. Position

Every dependency is: code you didn't review, running with your privileges, updated on someone else's schedule, with an attack surface (install scripts, transitive graph, account compromise of a maintainer). The 2025–2026 npm worm families proved this is an active battlefield, not a theoretical one. The policy is therefore twofold: **fewer dependencies, and hostile-grade handling of the ones you have.**

## 2. Selection Bar (Before Adding a Package)

1. **Necessity:** does this problem justify an external dependency, or is the solution ≤ ~50 lines of obvious code? (Date formatting: no. Timezone math with DST: yes — that's a real library problem.)
2. **Maintenance health:** recent commits/releases within a reasonable window; issues responded to; not "done and abandoned" UNLESS it's genuinely finished code (a date parser can be done; a web framework can't).
3. **Security posture:** dependency-audit clean; maintainers respond to security reports; ideally provenance/sigstore signing on releases (note: signed provenance was compromised in one 2026 incident — signing is a signal, not a guarantee).
4. **Transitive weight:** check the tree, not the package — a 4KB utility dragging 80 transitive packages is an 84-package liability.
5. **Runtime impact:** startup cost, bundle/memory size, global side effects (patching built-ins is disqualifying).
6. **Escape hatch:** how hard is replacement? Prefer thin wrappers around dependencies at module boundaries so swapping doesn't touch business code (`../01-foundation/architecture.md` — infra behind interfaces owned by the consuming module).

If a package fails 2 of 1–5, write it yourself or find another. **Hallucinated dependencies are a documented AI-agent failure mode: verify every package name exists in the real registry, with the real version, from the real scope — before it enters the lockfile.**

## 3. Lifecycle Discipline

- **Lockfiles committed and enforced** (frozen installs in CI): builds are reproducible; "works on my machine" and "works on Tuesday" are the same class of bug.
- **Install-time hardening:** block/limit lifecycle scripts (`postinstall` is the primary worm vector — npm `--ignore-scripts` semantics where workable, allowlist where not); scoped registry tokens; no auto-running of arbitrary native builds from unvetted packages.
- **Upgrade cadence:** scheduled dependency updates (weekly-ish review windows) instead of panic-upgrades; **security patches jump the queue** always. Automate the PRs (renovate-class tooling), review the diffs like code.
- **Deliberate major upgrades:** read the changelog/migration guide (freshness discipline — `../00-navigation/agent-workflow.md` §6), run the full test suite, stage-rollout per `deployment.md`.
- **Downgrade discipline:** never `^` floating ranges on critical infra (frameworks, DB drivers, auth libraries) — pin exact, upgrade deliberately.
- **Removal reviews:** quarterly scan for unused/one-line-used dependencies — dead dependencies carry full attack surface for zero value. Deleting an unused package is a pure-security win.

## 4. Auditing & Monitoring

- CI gate: dependency vulnerability audit on every build (fail on high/critical with an explicit override-and-ticket path — the override must have an owner and expiry, or it becomes a permanent wall-tape).
- Periodic: license compliance scan. AGPL-family licenses need deliberate product decisions —
  e.g. **Redis Open Source from Redis 8 offers a choice of three licenses (RSALv2, SSPLv1, or
  AGPLv3)**, of which only AGPLv3 is OSI-approved open source and it carries network-copyleft
  obligations; **Valkey** is the BSD-licensed fork. Verified against Redis's own license page.
  Saying "Redis is AGPL now" is the common shorthand and it is imprecise: you pick one of the
  three, and which one you pick is a legal question with product consequences.
- SBOM generation where the org requires it; at minimum, keep the dependency tree inventoried and diffable in PRs.
- Subscribe to advisory feeds for the critical few (framework, auth, DB driver) — those patches move same-day.

## 5. Categories & Their Risk Profile

| Category | Risk | Policy |
|---|---|---|
| Framework/runtime (Express, Next, Fastify…) | deep code execution, high CVE attention | pin major, upgrade deliberately, follow advisories |
| Auth/crypto libraries | catastrophic-if-wrong | vetted implementations only; never novel; never patched locally |
| DB drivers/ORMs | data-layer execution, native bindings | pin, test upgrades against real DB |
| Utility packages (date, validation, lodash-class) | supply-chain favorite (small, widely depended-on) | minimize count; prefer platform built-ins (modern runtimes absorbed most lodash/date utilities) |
| Dev-only tooling (linters, test runners) | lower runtime risk, still CI execution risk | normal cadence; scripts restrictions in CI |
| Wrappers around external services | medium; often stale vs the real API | thin + replaceable; check maintenance against the actual API's current version |

## 6. Failure Modes

- **The 2-line package:** `left-pad` history — micro-utilities as dependencies: infinite supply-chain surface for trivial code. Platform built-ins first.
- **Install-script blind trust:** wormable postinstall chains did the 2025–2026 damage; an npm install with scripts enabled is remote code execution by invitation.
- **Hallucinated/slopsquatting packages:** AI agents (and humans) typing plausible-but-wrong package names; attackers register the plausible names. Existence + ownership + download-history verification before install.
- **Lockfile rot:** lockfile deleted/regenerated casually ("it was conflicting") — silent transitive jumps ship untested.
- **Zombie dependencies:** packages nothing imports anymore, still installed, still executing install scripts, still in the SBOM.
- **Security-exception permanence:** audit override "until we fix it properly" (never fixed); overrides need expiry dates and owners.
- **Framework-chasing:** rewriting working code to the new framework because the old one felt dated — dependency decisions driven by novelty instead of necessity (`../01-foundation/complexity-control.md` §6).
