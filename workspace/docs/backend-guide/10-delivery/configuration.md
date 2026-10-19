# Configuration — Environment, Validation, and the Secrets Boundary

> **Part of:** Delivery — Backend Project Guide
> **Read when:** Structuring config, adding environment variables, deciding config vs feature flags, or debugging "works in staging, breaks in prod."
> **Related:** `../01-foundation/initialization.md` §2.2, `../04-security/secrets-management.md` (the secrets half), `../11-specialized-concerns/feature-flags.md`
> **Status:** Stable principles

---

## 1. The Config Model

All configuration flows through **one typed, validated object**, loaded and checked at startup:

```
environment (vars/files/secret manager)
        ↓ (single loader)
  ConfigSchema (validated: types, ranges, required)
        ↓ (fail fast on invalid)
  typed config object  →  injected where used
```

Properties this buys: a missing/invalid setting crashes the process at boot with a precise message (not at 2AM traffic); the full setting surface is greppable in one schema; tests construct configs explicitly; docs generate from the schema.

## 2. Rules

1. **No scattered `process.env` reads** in business code. The process boundary touches ONE loader. (Reading env in a handler is config-invisibility: untestable, typo-prone, undocumented.)
2. **Schema with teeth:** required fields required; typed with ranges/enums (port 1–65535, `log_level ∈ {debug,info,warn,error}`); unknown-var warnings to catch spelling drift between env files and schema.
3. **Fail fast at startup** on invalid config — deployment errors must surface at deploy time.
4. **No computed magic in env handling:** parsing/normalization/coercion lives in the loader, once, tested.
5. **Defaults discipline:** safe defaults only, and never for secrets or anything environment-specific (a default DB URL guarantees some environment silently points at the wrong database).
6. **Document every variable** in a committed `.env.example` (names + meaning + example values — never real secrets, `../04-security/secrets-management.md` §2).

## 3. Taxonomy (Four Different Things, Not One "Config")

| Type | Changes | Lives in | Examples |
|---|---|---|---|
| **Deployment variance** | per environment, at deploy | env vars / platform config | DB URL, pool size, feature of the region |
| **Secrets** | rotated, access-controlled | secret manager → injected | API keys, tokens (`../04-security/secrets-management.md`) |
| **Business settings** | by operators, runtime-ish | config table / admin surface | pricing tiers, limits — often better as DATA |
| **Feature flags** | per-release/per-user, runtime | flag system (`../11-specialized-concerns/feature-flags.md`) | rollout switches |

Collapsing all four into one `.env` is the classic failure: developers with prod-credential visibility, "redeploys" required for pricing changes, and flags that can't be toggled. Keep the channels separate even when they share the injection mechanism.

**Runtime-vs-build-time:** config is read at RUNTIME by the deployed artifact; anything baked at build time (env-specific bundles) breaks the one-artifact-many-environments model and promotes env drift — the artifact moves through environments unchanged, config differs (`deployment.md` §1).

## 4. Environment Hygiene

- **Parity** (staging ≈ prod in structure and config SHAPE, differing only in values): "works in staging" is only meaningful when staging's config topology matches production's — same flags, same feature-set, same secrets count.
- **Named environments, explicit promotion:** dev → staging → prod, values reviewed at each hop; ad-hoc "test-2" environments with hand-edited drift are incident factories.
- **Local dev:** `.env` files (git-ignored), realistic seed data, and NO production credentials ever reachable from a dev machine — the laptop-is-the-perimeter era is over.
- **Config-diff in deploys:** the release checklist includes what config changed alongside what code changed (config-only incidents are real and invisible in code diffs).

## 5. Testing

- Config loader unit tests: valid env → typed object; each invalid shape → precise startup error; defaults applied where safe.
- One integration boot test with intentionally-broken required config asserting the process refuses to start (the fail-fast guarantee is itself a feature with a test).
- Tests never read the developer's ambient env (hermetic: config constructed per test) — flaky suites caused by a stray local env var are a classic self-own.

## 6. Failure Modes

- **The ambient-env crawler:** 47 files each reading `process.env.X` — nobody can list the settings surface; typo'd vars read as `undefined` and become mystery behavior.
- **Silent default to production:** `DB_URL ?? "prod-host"` as a fallback default — one missing var in staging points it at prod. Defaults never target real environments (§2.5).
- **Config-as-tombstone:** 80 env vars where 60 are legacy-unused (verified by nothing, deleted by nobody) — every new environment setup transcribes archaeological noise. Prune with usage-checks.
- **Business-settings-in-env:** changing a pricing limit requires a redeploy (and a developer) — that's data or an admin surface, not env (§3).
- **The shared mega-.env:** one file holding config + secrets for every service; broad human access; no rotation path — split per §3 and move secrets to their channel.
- **Build-time env baking:** environment-specific artifacts; config changes require rebuilds; the same artifact is never what was tested.
