# Roster — role assignments

The single assignment file records who performs each role, with approved tools and permissions. It does not duplicate role responsibilities or project rules.

## Assignment authority

This file is the canonical project record of the owner's approved role-to-operator assignments, including Orchestrator. Any approved agent/model may fill any role; Claude, Codex and other names carry no permanent responsibility by default in these reusable role packages. Role `prompt.md` and `skillz/` define how to perform a role, not who performs it. Client entry filenames, the current chat provider and previous conversations do not assign roles.

Read this file before selecting an operator or preparing a dispatch. A blank assignment is unassigned, not an invitation to choose a model silently. Record an explicit owner assignment here when authorized; distinguish a proposed assignment from an approved one. When an assignment changes, update this record and hand over current task/candidate/artifact state; the role instructions remain model-neutral. An assignment does not itself grant tools, permission to send messages, live side effects or release authorization beyond actual owner/host permissions.

At direct startup from its prompt, Orchestrator maintains this file using the team and roles the owner supplies or approves, including its own role if explicitly assigned. Record the authorization/reference with actual assignments. Ask only for genuinely missing required operator choices; do not fabricate names or require an optional Advisor. Starting from the role prompt establishes the instructed responsibility, not permission to appoint the whole team without owner direction.

For every actual assignment, record the operator/model and runtime, approved tools, allowed project/file/environment scope, communication permissions, and the owner's authorization/reference. Record explicit limits or unresolved permissions rather than interpreting a blank field as unrestricted access. Keep any proposed assignment visibly `PROPOSED` until the owner approves it; neither proposal nor approval overrides host permissions. Use multiple identified operators only when the owner actually approves them for a role.

Before authorized bridge communication, operators must follow [bridge entry](../bridge.md) and its guide-only external access restriction. Transport master/runtime labels do not override these role assignments, and Roster does not grant access to the bridge project's other files.

## Current status

No new assignments are approved yet for the reusable role packages below. The project owner will define them later.

This starter-template record does not revoke or migrate the current legacy workflow: `CLAUDE.md` still assigns Claude as its orchestrator and `AGENTS.md` defines Codex's current advisor/authorized architect scope. Creating this file does not switch those live roles or activate the new packages. A transition requires an explicit owner decision and reconciliation of the affected entry instructions; do not silently choose between conflicting assignments.

| Role | Assignment | Required starting point | Status |
| --- | --- | --- | --- |
| Orchestrator | — | [prompt.md](../agents/orchestrator/prompt.md) | Prompt and orchestration Skill written; assignment pending |
| Planner | — | [prompt.md](../agents/planner/prompt.md) | Prompt and planning Skill written; assignment pending |
| Designer | — | [prompt.md](../agents/designer/prompt.md) | Prompt and design Skill written; assignment pending |
| Implementer | — | [prompt.md](../agents/implementer/prompt.md) | Prompt and implementation Skill written; assignment pending |
| Tester | — | [prompt.md](../agents/tester/prompt.md) | Prompt and verification Skill written; assignment pending |
| Advisor (optional) | — | [prompt.md](../agents/advisor/prompt.md) | Prompt and advisory Skill written; unassigned, not a required workflow gate |

The statuses describe file availability, not proof of a live workflow trial or product readiness. Paths in code are relative to project root; link pending entry points when their actual files are created.

Every assigned operator starts at the prompt identified in its row, then follows that prompt to `skillz/SKILL.md` and the task-relevant procedures, shared rules and permitted sources. The assignment/brief must name this entry explicitly; a model's default behavior is not a substitute. Keep these routes when updating operators. Reading a prompt does not activate an unassigned role or authorize dispatch; Advisor remains optional.

Advisor is optional. Select an operator only for an authorized useful review; no permanent named relationship to Orchestrator is assumed. Its absence alone is not a blocker. Advice is nonbinding and actual decisions/verification remain with their established owners; lack of an advisor does not waive missing mandatory evidence.

Role-structure guide: `workspace/agents/agents-guide.md` is not yet created. Do not treat that pending document as an available procedure; use the existing role prompts and project folder map for current navigation.
