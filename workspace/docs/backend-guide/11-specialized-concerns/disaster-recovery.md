# Disaster Recovery & Backups — Surviving the Worst Day

> **Part of:** Specialized Concerns — Backend Project Guide
> **Read when:** Defining backup strategy, restore procedures, RPO/RTO, or the "what if we lose the database / region / provider" question.
> **Related:** `../10-delivery/deployment.md`, `privacy-compliance-audit.md` §4 (deletion vs backups), `../08-observability/observability.md`
> **Status:** Stable principles · Platform mechanics in the platform/cloud guide · Discovered importance: untested backups are a belief system, not a recovery capability

---

## 1. Position

Backups are not a checkbox; they are a **recovery capability with measured objectives**. The difference is the restore test: a backup that has never been restored is a hope with storage costs. Discovered importance: data-loss incidents almost always reveal backups that existed, succeeded on paper, and couldn't restore — wrong scope, corrupted archives, missing dependencies, or restore times longer than the business survives.

## 2. Objectives (Decide Them Explicitly)

| Metric | Definition | Set by |
|---|---|---|
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss window (how much recent data may vanish) | business tolerance per data class |
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime for full recovery | business tolerance + technical reality, rehearsed |

Per data class where it matters (orders DB: RPO minutes via PITR; analytics events: RPO daily tolerable). These two numbers drive every mechanism choice below — a product that can tolerate 1h/24h loss needs radically less machinery than one that can't tolerate 5 minutes.

## 3. The Backup Stack (Layered)

- **Continuous/point-in-time recovery (PITR)** for transactional data: WAL archiving / platform-managed PITR — the RPO-minutes layer. Verify the archive lag is monitored (a silently stopped WAL archive is a widening RPO nobody watches).
- **Snapshot/physical backups** on schedule: full + incremental cadence per RPO calculus; **encrypted**, **cross-region/offsite copies** (the region-loss scenario), access-controlled (backup access = database access — treat archives as production data, `../04-security/secrets-management.md`).
- **Configuration/infrastructure as state**: the DB restores, but the app config, secrets inventory, DNS, and infra definitions must be re-creatable — IaC in version control is part of DR (the "we recovered the database but nobody remembers the env vars" postmortem is common).
- **Retention policy**: long enough for regulatory/rollback needs (`privacy-compliance-audit.md` §5 — and the backup re-erasure runbook), short enough to be operationally real.

## 4. Restore Testing (The Only Truth)

- **Scheduled restore drills** (monthly-ish per critical system): restore to isolated infra from the latest backup, run smoke tests + integrity checks (row counts vs expectations, app boots, auth works, money paths consistent).
- **Measure the actual RTO** during drills — the number in the doc vs the stopwatch in the drill diverge by multiples until rehearsed.
- **PITR point-in-time drill**: restore to a specific past moment (not just latest snapshot) — that's the mechanism you'll actually use after a logical corruption (bad migration, rogue job — the most likely real scenario).
- Drill output: the runbook's corrections. A runbook updated only by memory of the last drill is already drifting.

## 5. Failure Scenarios (Rehearse Per Class)

| Scenario | Recovery path | What must exist |
|---|---|---|
| Bad deploy / bad migration (logical corruption) | rollback artifact (`deployment.md` §4) + PITR to pre-incident point + data-repair job | PITR, artifact history, repair tooling |
| Datacenter/zone loss | regional failover (multi-AZ minimum; multi-region only if justified — `../06-async-and-scale/scalability.md` §3) | replicated data, infra-as-code re-provision |
| Database total loss | restore from backups + WAL replay | tested restore path + secrets/config re-creation |
| Provider compromise / egress | portability: exportable data + IaC + offsite copies | non-provider-bound formats somewhere |
| Ransomware/malicious deletion (incl. backup deletion) | immutable/offline backup tier | write-once storage or access-isolated copies; backup credentials ≠ prod admin |

## 6. Runbooks & Ownership

- Per critical system: the 3AM restoration runbook — exact commands, decision points (which PITR timestamp — how to choose it: incident start minus safety margin), verification steps, comms plan (status page, notification triggers per `privacy-compliance-audit.md` §8).
- **Named owner per backup set** and alerting on backup job failures (a failed backup must page like a failed deploy — silent backup rot is the classic discovered-too-late).
- DR is rehearsed chaos-drill material: inject "restore prod to yesterday into staging" quarterly; the team that has done it once does it in 1/10th the time under real pressure.

## 7. Failure Modes

- **Backup success theater:** dashboard green (job ran) — archives corrupt, scope incomplete (DB yes, object storage no, secrets no), or restore takes 40 hours against a 2-hour RTO. Restore drills are the only counter (§4).
- **The same-region backup:** primary and backup destroyed by the same event — offsite/cross-region copies exist for this (§3).
- **Backup admin = prod admin:** ransomware/insider deletes production AND its backups with one credential set — isolation/immutable tier (§5).
- **Unmonitored WAL lag:** PITR "exists" but the archive stopped last week; actual RPO is a week; discovered during the incident (§3).
- **Config amnesia:** data restored, application can't boot — env/secrets/IaC weren't part of the recovery story (§3 third bullet).
- **Never-rehearsed PITR selection:** team must choose a restore timestamp during an incident, under pressure, for the first time; they restore too far back (losing good transactions) or not far enough (corruption persists) — the decision procedure is written in the runbook, not improvised (§6).
- **Deletion-vs-backup deadlock:** erasure requests met with "we can't delete from backups" — because the re-erasure-on-restore runbook never existed (`privacy-compliance-audit.md` §4).
