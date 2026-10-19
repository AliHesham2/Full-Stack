# Implementer — mandatory project syntax authority

This owner-required constraint is mandatory from the [Implementer prompt](../prompt.md) and [implementation procedure](SKILL.md) before code is written. It does not assign an operator or activate a replacement for the current project workflow.

## Exclusive technical source

For every implementation assignment, use `workspace/docs/syntax/` as the **exclusive source for language syntax, framework/library APIs, technical patterns and version-specific behavior used in authored or modified code**. Begin at [syntax GUIDE-MAP](../../../docs/syntax/GUIDE-MAP.md), then read the relevant technology index and task-specific pages before implementation; the map alone is insufficient source support. Do not substitute pretrained recollection, a personal/global/provider-installed Skill, another project's instructions, an external skill catalog, generated suggestions, or unapproved online material for these local sources. Do not activate such Skills for implementation. This constraint applies regardless of operator, model or client.

The approved Planner Output and reviewed contracts specify required product behavior. The applicable `workspace/docs/backend-guide/` specify architecture, boundaries, conventions and quality constraints; `syntax/` supplies the permitted technical means. This rule does not authorize changing requirements, tests, contracts, dependencies or architecture to fit a syntax example. Project code, package manifests, lockfiles and compiler output are evidence of the actual implementation and environment, not alternative syntax authority. Higher-priority host instructions and actual permissions still apply.

Write original task-specific code by composing the documented constructs; this is not a requirement to paste examples verbatim. However, every material API, pattern, configuration construct or version-dependent choice introduced or changed must have relevant support in the local syntax reference. Existing code, generated code, copied snippets and tool suggestions must meet the same constraint before being incorporated. Do not retain a known unsupported choice merely because it already exists in the changed path; report the conflict without expanding into unrelated rewrites.

## Before and during implementation

1. Identify affected technologies and actual installed versions from each affected app's manifests/lockfiles and permitted runtime inspection. Do not assume that this starter template has an initialized app or installed dependencies.
2. In the task TODO, map each implementation item to its approved requirement/contract, applicable backend guide section, and the exact `syntax/` file and heading that supports its technical choices. Read those sections before writing the corresponding code; a filename or index listing alone is not sufficient.
3. Use the newest applicable, non-deprecated form **documented in the local reference and compatible with the project's approved installed version**. Historical examples, migration-only syntax and a feature documented for a different version are not current implementation choices. A dated local reference is not proof that it contains today's latest upstream release. Do not upgrade dependencies or silently replace the local reference to satisfy the word "latest".
4. If a necessary technology/API/construct is absent, ambiguous, stale relative to the required version, or conflicts with a project guide, stop the dependent implementation item. Report the exact requirement, missing/conflicting files and headings, version evidence and impact to Orchestrator. Continue only independent work whose sources are complete. Never fill the gap from memory, switch libraries, browse an alternative, or edit the reference without a separately authorized source-maintenance task.
5. Resume that item only after the authorized source owner resolves the issue in `syntax/` and any affected project guide/requirement, or the project owner explicitly changes this restriction. A peer suggestion or passing compiler/test does not waive the source requirement.

## Self-review and evidence

Before handoff, inspect the actual diff for unsupported, removed or deprecated constructs and confirm that the TODO's source mapping covers the final choices. Run the project's applicable checks; source conformity and successful execution are separate evidence and both must be reported accurately. Do not edit Tester's expectations or quality gates to make unsupported code pass.

In the task Output, record implementation item/requirement ID, changed code location, local syntax file/heading, relevant version, applicable guide section, checks actually run and any unresolved source gap. One mapping may cover a coherent group of uses; do not inflate the report with a citation on every code line. Missing coverage stays explicitly `UNVERIFIED` or `BLOCKED`, never a fabricated compliance claim.
