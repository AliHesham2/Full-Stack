# Web Interface Guidelines — Designer review adapter

Use this for a **targeted review of an existing web interface or its code** when the brief permits reading that artifact. The source is Vercel Labs' [Web Design Guidelines skill](https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines), which delegates its rules to [Web Interface Guidelines](https://github.com/vercel-labs/web-interface-guidelines/blob/main/command.md). This local [MIT-licensed snapshot](references/web-interface-guidelines-upstream/command.md) is pinned to `434b7f91364665f2f733b310ec54809bf8f37937` (retrieved 2026-10-19). It makes the reference available without a provider-specific skill install or a network dependency.

## When and how

1. Name the exact routes, components, states or file patterns in the TODO. A broad audit is only useful when the assignment covers that surface. If current online access is authorized, check the upstream `command.md` before review and record its revision or access date; otherwise use the pinned local snapshot and say so.
2. Read the relevant source files or rendered states. Check the matching headings in `command.md`: accessibility and focus; forms; motion; typography and long content; images and performance; navigation, touch and safe areas; themes, locale and hydration; copy. Cross-check with the approved PRD, [design guide](../../../docs/design-guide/INDEX.md) and [frontend guide](../../../docs/frontend-guide/README.md).
3. Record only applicable, supported findings. For code, cite `file:line`, the exact observed pattern, impact, source rule and proposed observable fix. For a mockup or textual specification, cite its artifact/section and mark code-only checks `UNVERIFIED`. A static screenshot cannot prove keyboard behavior, hydration, focus handling or performance.
4. Group findings by artifact in the Designer Output, link each to the relevant `UI-n`/`DS-n`/`AC-n` where possible, and hand actual implementation defects to Implementer and verification criteria to Tester. The Designer does not edit code unless a separate brief grants it.

## Project translation

The upstream document is a review checklist, not a higher-priority project policy. Its generic examples, including a numerical list-virtualization threshold, input `autocomplete` choices, English title casing and blanket keyboard-handler wording, require context and must not override semantic native controls, localization, the actual frontend architecture or the project's guides. Do not report a violation solely because a generic example differs. Conversely, preserve useful checks such as skip links, focus obscured by sticky UI, first-error focus, paste support, unsaved-change recovery, reduced motion, stable image layout and locale formatting when relevant. Avoid copying the whole checklist into every Output; cite the source heading and the particular evidence.

The upstream skill's `WebFetch` command and slash-command installation are transport suggestions, not assignment or tool authority. Use the host's authorized read tools. If the reference is unavailable, mark the affected review `NOT RUN` rather than implying compliance.
