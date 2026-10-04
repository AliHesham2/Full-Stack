# Node.js Versions & Release Policy

Which Node to target and when to upgrade. For what breaks between majors, see [deprecations.md](./deprecations.md).

Latest checked targets (2026-09-26): **24.21.0 LTS** for production; **26.10.0 Current**, not yet the production default.

## Status table (snapshot 2026-09-02)

Values read directly from [nodejs/Release `schedule.json`](https://github.com/nodejs/Release/blob/main/schedule.json) — the primary source the website table is generated from.

| Version | Codename | Status | First release | Enters Maintenance | EOL |
|---|---|---|---|---|---|
| 27 | – | not released (alpha from 2026-10-28) | 2027-04-22 | 2027-10-20 | 2030-04-30 |
| 26 | – | **Current** | 2026-05-05 | 2027-10-20 | 2029-04-30 (LTS from 2026-10-28) |
| 25 | – | **EOL** | 2025-10-15 | – | 2026-06-01 (dead — never use) |
| **24** | Krypton | **Active LTS** | 2025-05-06 | **2026-10-20** | 2028-04-30 |
| 22 | Jod | Maintenance LTS | 2024-04-24 | 2025-10-21 | 2027-04-30 |
| 20 | Iron | **EOL** | 2023-04-17 | – | 2026-04-30 (dead) |

Historical builds as of 2026-09-02 (from [nodejs.org/dist/index.json](https://nodejs.org/dist/index.json)): **24.20.0** (V8 13.6.233.17, npm 11.19.0) · **26.8.1** (V8 14.6.202.34, npm 11.19.0) · **22.23.2** (V8 12.4, npm 10.9.8).

> **Node 24 leaves Active LTS on 2026-10-20** — less than four weeks out as of 2026-09-26. It keeps receiving
> critical fixes until 2028-04-30, but no new features. Plan the 24 → 26 jump for when 26
> becomes LTS on 2026-10-28.

**Note on the v24 EOL date:** `schedule.json` says **2028-04-30**, and that is the
authoritative value. endoflife.date has listed 2027-10-27, applying the shortened-LTS
proposal ([nodejs/Release#1113](https://github.com/nodejs/Release/issues/1113)) to v24;
the official cadence statement says the new model starts **with Node 27**, so ≤ 26 keeps
legacy dates. Trust 2028-04-30, but there is no cost to planning the migration for 2027 —
you would be moving to Node 26 LTS by then anyway.

## Cadence change (from Node 27)

- Release cycle becomes **annual** (one major per year, instead of two).
- Every major version becomes LTS after a 6-month Current phase plus ~6-month alpha phase — no more dead odd-numbered releases. Confirmed by v27's schedule entry: `alpha` 2026-10-28 → `start` 2027-04-22 → EOL 2030-04-30 (≈36 months).
- LTS guarantees critical-bug fixes for ~30 months; total lifetime ≈ 36 months from first release.
- Node ≤ 26 keeps the legacy model: odd majors (21/23/25) die after 6 months, only even majors become LTS.

Sources: official statement on [nodejs.org releases page](https://nodejs.org/en/about/previous-releases); [InfoQ summary](https://www.infoq.com/news/2026/06/nodejs-release-changes); [nodejsdesignpatterns.com analysis](https://nodejsdesignpatterns.com/blog/nodejs-release-schedule-changes).

## Rules

**R1 — Target Node 24 LTS for production. [Stable]**
Why: Active LTS gets new features + all fixes until 2026-10, then maintenance (critical fixes) until EOL. "Current" (26) is for compat testing, not production. Official guidance: production apps should only use Active LTS or Maintenance LTS.
```json
// package.json
"engines": { "node": ">=24.0.0 <25.0.0" }
```
Caveat: if you need Temporal now, that requires Node 26+ ([runtime-apis.md](./runtime-apis.md)) — it is **not** in 24 LTS. Same for `Map.prototype.getOrInsert` and `Iterator.concat`.

**R2 — Never run EOL or non-LTS majors in production. [Stable]**
Node 20 and 25 receive no security patches. Node 22 is maintenance-only — start migrating to 24 now if you haven't.

**R3 — Pin the runtime everywhere it appears. [Stable]**
- Dev: `.nvmrc` (`24`), mise/fnm/volta config.
- package.json: `engines` + `engine-strict=true` in `.npmrc` (npm ignores `engines` without it).
- Docker: `FROM node:24-slim`.
- CI: test the **lowest** version you claim in `engines`, not just the newest.
Why: "works on my Node" drift is the top source of CI-vs-prod discrepancies (module resolution and API behavior differ per major).

**R4 — Upgrade majors deliberately, LTS-to-LTS. [Context]**
24 → 26 (after 2026-10) → 27+ (annual). Before each jump, read the target major's release-note section "Deprecations and Removals" ([v24](https://nodejs.org/en/blog/release/v24.0.0), [v26](https://nodejs.org/en/blog/release/v26.0.0)) and check [deprecations.md](./deprecations.md).

## What each modern major gives you

- **24 (LTS)** — npm 11; V8 13.6 (`Float16Array`, `RegExp.escape`, `Error.isError`, explicit resource management `using`); `--permission` (renamed from `--experimental-permission`); `URLPattern` global; WebSocket stable; `require(esm)` stable; type stripping stable (24.12+); AsyncLocalStorage perf (AsyncContextFrame default); `node:sqlite` unflagged.
- **26 (Current)** — Temporal API by default; V8 14.6 (`Map.prototype.getOrInsert`, `Iterator.concat`); Undici 8; removals: `http.Server.prototype.writeHeader`, legacy `_stream_*` modules, `--experimental-transform-types`; runtime deprecation of `module.register()` (use `module.registerHooks()`); `--build-sea` for single executables.

- **26 (Current)** also bumps `NODE_MODULE_VERSION` to **147** — every native addon must be rebuilt for it. Check before upgrading anything with a compiled dependency (`sharp`, `bcrypt`, `better-sqlite3`, `canvas`).

Verify any of this locally: `node -p "process.versions"` (first-party check on 24.18.0 returned V8 13.6.233.17, OpenSSL 3.5.7, uv 1.52.1; 24.20.0 was the snapshot build; latest checked target is 24.21.0 (2026-09-26)).
