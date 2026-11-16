# Pilot test plan

Status: draft. Run on the pilot VPS with **fictional demo data only**. No real school moves until every "must pass" test passes and the owner signs off.

## Stages

1. Disposable VPS: build from Ansible, try vps-forge, destroy.
2. Pilot VPS: build from Ansible, deploy the demo school plus 2–3 fictional schools.
3. Run the tests below; record results in `docs/pilot-results.md` (to be created).
4. Server-lost drill.
5. Owner decision on moving one real school.

## Tests

| # | Test | Pass condition | Must pass |
|---|---|---|---|
| T1 | Rebuild from repository | Blank VPS → all schools live using only Ansible + backups, following [server-lost](../runbooks/server-lost.md) | Yes |
| T2 | Idempotency | Second `site.yml` run reports no changes | Yes |
| T3 | Database isolation | Isolation checks 1–5 in [security model](security-model.md) | Yes |
| T4 | In-school access review | Sentry `django-access-review` findings fixed or accepted; check 6 passes | Yes |
| T5 | Morning attendance peak | N schools × teachers submitting registers in one hour: p95 under 1.5 s, no errors | Yes |
| T6 | Report-card day | Generating PDFs for a full class does not push other schools over 3 s or cause out-of-memory | Yes |
| T7 | Backup | Nightly backup runs; every school has its own dump in restic | Yes |
| T8 | Single-school restore | Restore one school to a new database and switch it in under 1 hour, others unaffected ([runbook](../runbooks/restore-school.md)) | Yes |
| T9 | Release one school at a time | New tag on one school; others stay on the old tag and keep working ([runbook](../runbooks/rollout-release.md)) | Yes |
| T10 | Bad release rollback | A deliberately failing release is rolled back for that school only | Yes |
| T11 | Alerts | Stopping a school, filling disk to 80%, and skipping a backup each produce a Telegram alert naming the server or school | Yes |
| T12 | Secret rotation | Rotating one school's database password and secret key causes no downtime beyond a restart | No |
| T13 | Capacity | Measured memory and CPU per school recorded; [capacity](capacity.md) updated | Yes |
| T14 | SSH hardening safety | Key login works after DevSec; Hostinger console tested | Yes |

## Exit criteria

- All "must pass" tests pass, with evidence recorded.
- Open decisions D1–D4 and D7 decided.
- Owner approves moving one real school, with a fallback plan to stay on Railway.
