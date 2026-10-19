# Source freshness and compatible versions

This migration reuses documented reference snapshots; no installed backend or package upgrade exists yet. A dated reference or latest-doc URL does not prove the actual project uses the newest stable release.

For a real task Planner records actual approved Node.js/Express/provider/runtime/tool versions and local syntax coverage. Select supported stable versions compatible with project constraints; distinguish Active/Maintenance LTS, Current, prerelease and end-of-life status from the official release schedule rather than assuming the largest number is the production choice. Confirm dependencies/lockfile/runtime evidence when an app exists. [Official Node.js release schedule](https://nodejs.org/en/about/previous-releases).

Official Express security guidance recommends maintained, non-vulnerable releases and application-level input/transport protections. Check the relevant version/security advisories and actual deployment; a middleware list is not proof of enforcement. [Express security](https://expressjs.com/en/advanced/best-practice-security/) and [error handling](https://expressjs.com/en/guide/error-handling/).

Express 5 propagates rejected promises from returned route/middleware handlers to error handling. Callbacks, background work and streams still need their supported error paths. This project retains its central safe JSON envelope instead of exposing the default development error output. Exact implementation syntax must be present in the local reference; remote documentation is not an Implementer bypass.

For OpenAPI, record the exact approved specification version and validator/tool compatibility. The moving [official latest specification](https://spec.openapis.org/oas/latest.html) is a discovery source, not a mandate to upgrade an existing contract or use unsupported schema syntax. Keep one canonical contract and verified synchronization to handlers/types/docs.

Primary-source routes above were checked during migration on 2026-10-19 for the affected policies. This is not an exhaustive latest-version audit of every stored chapter, package, provider or security claim. Research/source maintenance needs actual authority; Implementer remains bound to [local syntax policy](../../../agents/implementer/skillz/syntax-policy.md). A missing/deprecated/incompatible construct blocks dependent code until an authorized source/version resolution is recorded. Never silently install, upgrade or improvise syntax to satisfy the word latest.
