# Privacy, Compliance & Audit — Personal Data as an Engineering Concern

> **Part of:** Specialized Concerns — Backend Project Guide
> **Read when:** Storing personal data, designing deletion/export, audit trails, retention, data residency, or evaluating "do we need to worry about GDPR-class rules?"
> **Related:** `../08-observability/observability.md` §7 (audit logs), `../04-security/secrets-management.md` §5 (encryption), `../06-async-and-scale/file-and-object-storage.md` §6
> **Status:** Principles stable · Legal specifics are jurisdiction-volatile — verify current obligations per product market; this file is engineering discipline, not legal advice

---

## 1. Position

Privacy compliance (GDPR-class regulations, CCPA-class statutes, sector rules like health/finance) is a **backend architecture concern**, not a policy document: the regulated behaviors — deletion, export, minimization, residency, breach notification — are implemented in schemas, jobs, and pipelines, or they exist nowhere. Discovered importance: retrofitting deletion onto a system built without data inventory is one of the hardest rewrites there is; the cheap moment to comply was at schema design.

## 2. Data Inventory (The Prerequisite for Everything)

You cannot delete, export, or minimize what you cannot locate. Maintain a **data map**: every PII field — where stored (tables, caches, logs, files, backups, third parties), its purpose, retention, sensitivity class. Living artifact (schema comments → generated inventory is the sustainable shape), reviewed when schemas change. This inventory is also your breach-response map and your "what do we even have" answer for every compliance questionnaire.

## 3. Minimization & Purpose (Design-Time Rules)

- **Don't collect what you don't need now** ("might be useful for analytics someday" is not a purpose — collect when the purpose arrives; retroactive collection is impossible).
- **Identifiers over attributes:** store the minimal reference, not the dossier (user ID, not full profile copies scattered across tables).
- **Derived/synthetic keys for analytics** (pseudonymized IDs in event streams; raw PII stays in the systems of record).
- Sensitive classes (health, biometrics, precise location, payment data) get a **deliberate decision per field**: encryption-at-rest beyond baseline, access logging, and a written justification. Payment specifics belong to the processor (Stripe-class — store references, never PANs).

## 4. The Rights Endpoints (Delete / Export / Correct)

Engineering behaviors behind the legal rights:

- **Deletion (right to erasure):** cascade defined per data map — rows owned, references nullified/anonymized, files deleted (`file-and-object-storage.md` §6 lifecycle), **caches flushed or TTL-bounded**, search/projection stores purged, third parties notified via their deletion APIs, **backups: documented exception** (deletion applies prospectively; backup-restore procedures must re-apply erasures on restore — write that runbook now, not during an incident).
- **Export (data portability):** a job that assembles the user's data (all inventory locations) into a machine-readable archive with a verified-complete checklist; rate-limited, authenticated, audited.
- **Correction:** normal update flows + propagation to derived stores (the same staleness discipline as cache invalidation — `../06-async-and-scale/caching.md` §4).
- All three: **request logging for audit** (who requested, what was done, when), processing SLA tracking (regulations carry deadlines), and idempotent execution as background work (`../06-async-and-scale/background-jobs.md` — deletion cascades are jobs, not request-time loops).

## 5. Retention & Lifecycle

- Per-data-class retention policy with **enforcement jobs** (not policy PDFs): sessions/logs 30–90d, deleted-account PII N days post-deletion-request, backups cycling per `disaster-recovery.md`.
- **Soft-deleted ≠ deleted:** the privacy-relevant clock starts at erasure request; audit the actual physical/anonymous state.
- Aggregate/anonymized data is exempt-ish — IF anonymization is real (re-identification via joins is a documented failure mode; true anonymization is harder than it looks — prefer deletion or irreversible aggregation).

## 6. Audit Trails (Who Did What)

Separate stream from app logs (`../08-observability/observability.md` §7): append-only, tamper-evident storage, retention aligned to regulation, capturing: auth events, permission changes, data exports/deletions, admin/privileged actions, data-access events for sensitive classes. Alert-worthy anomalies: bulk exports, off-hours admin access, permission self-elevation.

## 7. Residency & Regionalization

Where data MUST live (jurisdiction-driven): region-pinned storage/jobs per tenant or per market (`../11-specialized-concerns/multi-tenancy.md` §2 region-per-tenant), cross-region flows enumerated and justified (a "global" analytics pipeline ingesting EU PII is a compliance incident on a schedule). Logging/backup infrastructure has a residency too — the data map covers the replicas, not just the primary.

## 8. Breach Posture (Designed Before Needed)

- Detection paths: audit anomalies, access-log alerts, third-party breach notifications
- Response: the data map IS your scope-assessment instrument ("what classes, whose, where"); notification obligations have SHORT clocks — the runbook names the legal trigger-owner and the engineering evidence-supplier
- Practice: the incident path exercises the same inventory tooling as deletion/export

## 9. Testing

- Deletion completeness tests: erasure request ⇒ every inventory location verifies empty/anonymized (including cache and search stores — the forgotten projection is the classic leak)
- Export completeness: inventory checklist vs archive contents
- Audit-stream assertions: sensitive operations emit audit records (test per operation class)
- Access control on sensitive classes: role restrictions tested as denial tests (`../09-testing/testing-strategy.md` §5.1)

## 10. Failure Modes

- **The scattered dossier:** PII copies in 9 tables + logs + a spreadsheet export + an analytics event — deletion reaches 4 of 12; the data map would have known (§2).
- **Logs-that-remember:** erasure deletes rows but log lines still carry emails (PII in logs is data — log fields are inventory items; minimize at emission, `../08-observability/observability.md` §2).
- **Backup amnesia:** restore brings deleted users back to life; nobody wrote the re-erasure runbook (§4 deletion row).
- **Consent-theater:** consent recorded nowhere technical (no consent event with timestamp/version — the audit trail regulators actually ask for).
- **Anonymization-by-renaming:** "anonymized" dataset still keyed by email-hash joinable to the original — re-identification, not anonymization (§5).
- **Compliance-as-PDF:** policies promising minimization/deletion that no schema, job, or pipeline implements — the gap surfaces during a breach response, the worst possible discovery moment.
