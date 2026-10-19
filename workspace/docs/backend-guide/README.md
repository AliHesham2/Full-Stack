# Backend Project Guide

A reusable engineering knowledge system for designing, building, securing, testing, operating, and evolving backend systems. Written for **future AI agents** (and the humans supervising them) who will build real backend projects phase-by-phase.

> This guide answers *HOW to decide* on the backend: architecture, boundaries, security,
> consistency, failure behavior. It deliberately does not duplicate technology-specific
> knowledge: that lives in the specialized guides. Routing between guides is not part of this
> guide.

---

**Every code path in this guide is relative to `backend/`**, the Express app's folder in the project
root (`src/` means `backend/src/`). Run its commands inside `backend/`.

## API-only scope

Read [backend-only scope](00-navigation/backend-only-scope.md). This template builds APIs only; external clients live elsewhere. Planner defines complete contracts/data/security, with contract review before code. There is no frontend guide, UI Designer or visual TODO. Read [source/version policy](00-navigation/source-version-policy.md) before relying on an assumed latest release.

## Start Here

Start at [GUIDE-MAP.md](GUIDE-MAP.md) for the reading flowchart and a short description of every file. It routes to the actual procedures and rules below; it does not replace them.

1. **`00-navigation/agent-workflow.md`** — the operating loop (understand → inspect → locate → plan → implement → validate → finish), phase playbooks, and the hard rules. Read this before any task.
2. **`00-navigation/decision-index.md`** — "I need to decide X → read Y." The token-efficient routing table. Use it instead of reading everything.

## The Map

| Area | Files | What it covers |
|---|---|---|
| **01-foundation** | initialization · architecture · project-structure · complexity-control | Bootstrapping a project without speculative infrastructure; architecture as a decision process (modular monolith default); where things live; the justification bar for ANY complexity |
| **02-domain-and-api** | domain-logic · api-design · validation | Business rules placement and use-case anatomy; HTTP contracts, status codes, pagination, idempotency, evolution; syntactic vs business validation and trust boundaries |
| **03-auth** | authentication · authorization | Session/token model selection, passwords (NIST 800-63-4, Argon2id), MFA, service-to-service identity; RBAC/ABAC, object-level authorization (BOLA — the #1 API threat), multi-tenant enforcement |
| **04-security** | security-foundations · secrets-management | Trust boundaries, the injection family, CSRF, SSRF, supply chain (2025–2026 npm attacks), leakage; secrets storage, rotation, encryption, key management |
| **05-data** | data-architecture · data-access · transactions · concurrency-and-idempotency · migrations | Ownership and schema reasoning, indexes, constraints; repositories and the ORM question; ACID, isolation, locking; races and idempotency as first-class; expand–contract schema evolution |
| **06-async-and-scale** | caching · background-jobs · events-and-messaging · file-and-object-storage · realtime · performance · scalability | Decision-first caching and invalidation; job design and DLQs; events vs commands and the outbox pattern; upload security; WebSockets/SSE lifecycle; measurement-driven performance; the scaling ladder |
| **07-reliability** | error-handling · reliability | Error taxonomy and retryability; health/readiness, timeouts, graceful shutdown, degradation, resource exhaustion |
| **08-observability** | observability | Structured logging, metrics (RED), tracing, correlation IDs, alerting on symptoms, audit logs as a distinct stream, the production-debugging workflow |
| **09-testing** | testing-strategy | Integration-first strategy with real databases, what each level is FOR, the non-negotiable test classes (authorization denials, failure paths, concurrency), CI gates |
| **10-delivery** | dependencies · configuration · ci-cd · deployment · documentation-and-decisions | Supply-chain-era dependency policy; typed config and the four config taxonomies; pipeline stages and gates; zero-downtime sequencing and rollback; README/ADR/runbook discipline |
| **11-specialized-concerns** | multi-tenancy · time-and-money · webhooks · feature-flags · privacy-compliance-audit · disaster-recovery | Isolation models and tenant context; UTC/timezones, integer money and ledgers; webhook verification and dispatch; flag lifecycle; PII inventory and rights endpoints; RPO/RTO and restore drills |
| **12-anti-patterns** | anti-patterns | 60+ cataloged failure patterns with signatures and fixes — including the AI-specific generation failure modes |

## Reading Priorities (If You Read Nothing Else)

For a new project: `agent-workflow` → `01-foundation/*` → `10-delivery/configuration.md`.
For a feature task: `decision-index` → the 2–3 routed files → `12-anti-patterns` (§ matching your design).
For a security review: `04-security/security-foundations.md` §8 checklist → `03-auth/authorization.md` §4 → `12-anti-patterns` §D.

## The Philosophy (Encoded in Every File)

- **Decision frameworks over rules** — criteria, trade-offs, and revisit triggers, not "always do X"
- **Security-first** — trust boundaries and authorization are design inputs, not patches; generated code carries defects at rates high enough that self-trust is a defect class of its own — verify, don't self-trust
- **Complexity resistance** — every structural addition passes a justification bar; boring solutions are the default; the modular monolith is the architecture until evidence says otherwise
- **Failure-mode thinking** — every topic documents what breaks, when, and who notices
- **Token efficiency** — cross-referenced, focused files; read only what the task needs (`decision-index.md` is the router)

## Conventions

- Each file header carries: **Read when** (trigger), **Related** (cross-refs), **Status** (freshness label — `stable`, `current as of 2026`, `ecosystem-volatile` items deserve external re-verification before reliance).
- Cross-references are pointers, not obligations.
- Illustrative snippets are pseudocode-shaped; exact syntax belongs to the technology guides.

## Adding to This Guide

New knowledge earns a file when: it's a decision domain future agents will repeatedly face, it survives the complexity bar (is it load-bearing?), and it can't live as a section of an existing file without bloating it. When adding: update `decision-index.md`, this map, and cross-references — a file unreachable from the navigation layer doesn't exist.

Database/provider code stays in Express on Node.js; Supabase is the current provider, not a frontend dependency.
