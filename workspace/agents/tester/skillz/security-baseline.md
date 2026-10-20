# Versioned security verification baseline

Read [frontend-only scope](../../../docs/frontend-guide/00-navigation/frontend-only-scope.md). Verify frontend/Next.js transport controls and supplied identity/error behavior only; no UI restriction proves external service authorization, and no backend audit/source access is granted.

Use with the SECURITY stream, actual frontend security/integration guides and owner-supplied auth/API sources and selected `security-testing` methods through the [QA adapter](qa-skills-integration.md). This is technical verification guidance, not a new role, scan authorization, comprehensive audit claim or permission to change approved security policy.

## Establish the requirements and methods

Primary-source review on 2026-10-20 found [OWASP ASVS 5.0.0](https://owasp.org/projects/asvs) as the published stable baseline and [WSTG 4.2](https://wstg.owasp.org/v4.2/) as the versioned stable testing guide. ASVS supplies verifiable security requirements; WSTG supplies testing methods. Pin the actual task versions and read selected requirements/methods, rather than assuming a link to a moving latest page is reproducible coverage. A pre-existing approved baseline stays identified; proposing a newer baseline does not silently approve a policy change. Recheck relevant official release status for later tasks; this dated check is not a perpetual newest-version guarantee.

Record the actual approved assurance target/ASVS level if one exists. Do not choose a level, required control set or exception for the owner. Use the task's actual risk/requirements to select relevant controls, identifying their scope and justified exclusions. Missing required policy blocks the affected check and returns through Orchestrator. No blanket obligation to run the entire standard for a low-risk task is created here.

## Map controls to observable checks

For each selected requirement record its exact version-qualified ASVS identifier, title/source location, related project SEC/AC IDs, applicable surface and expected observable enforcement. Follow the identifier format documented by ASVS; never invent numbers or reuse an old ID under another version without checking its meaning. When a WSTG method helps, record its actual version/identifier/section and source; do not imply that ASVS and WSTG have a universal one-to-one mapping. Project-specific checks can have no matching standard control with a reason; do not fabricate a standard citation.

Plan relevant cases for authentication/session handling, user versus service identity, object/tenant authorization, validation/injection, secrets/sensitive responses, abuse/resource limits and other affected boundaries. Distinguish valid negative/denial cases from a scan warning. Establish exact permitted test accounts/roles, isolated data, targets, allowed techniques and expected safe outcomes before active execution. Tool availability, a public URL or this baseline grants no scanning, load, destructive action, credential use or production access.

In Output, link each selected control to actual check IDs, candidate/environment, method, result/evidence and coverage limits. A tool warning must be investigated; no findings is not proof of all controls. A document-phase result verifies only the specified design/control definition, not runtime enforcement. Missing runtime evidence stays UNVERIFIED at its AC, even if the source review passed. Partial coverage never supports a whole-ASVS, penetration-test or compliance certificate claim.

## Related accessibility target

Accessibility remains a separate A11Y result. Use the actual project target; the current frontend guide specifies WCAG 2.2 AA. The primary [WCAG 2.2 Recommendation](https://www.w3.org/TR/WCAG22/) provides success criteria; an automated scan or isolated criterion pass is not a full conformance claim. Record the selected version/criteria, actual method and manual follow-up under the existing [axe procedure](axe-core-results.md), not as ASVS coverage.

## Authority and ownership

Do not install scanners, edit product code/quality thresholds, accept risk or send findings outside the authorized route from this procedure. External standards inform Tester verification; they do not bypass Implementer's local syntax policy or assign operators. Report exact source conflicts and actual evidence through Orchestrator. Preserve the original Tester report; later corrections are personally verified by Orchestrator under the existing delivery cycle, not automatically retested by Tester.
