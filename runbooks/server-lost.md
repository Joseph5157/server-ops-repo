# Runbook: server lost

Use when the VPS is gone, corrupted, compromised or unrecoverable. Highest tier: **T3**. Target: all schools back in under 4 hours.

## Decide first (owner)

- Is the old server really unrecoverable? Check Hostinger status and the browser console first.
- If **compromised**: do not reuse it. Rotate all secrets ([rotate-secrets](rotate-secrets.md)) as part of the rebuild.
- Tell the affected schools. Agree what to say.

## Steps

1. **[T3] Owner:** order a new VPS (same size or larger, same region), or rebuild the existing one from the Hostinger panel.
2. **[T2]** Follow [new-server](new-server.md) up to and including the platform step. Use the same `schools/<server>.yml`.
3. **[T0]** Confirm restic access from the new server: `restic snapshots` lists the latest per-school dumps.
4. For each school, highest priority first:
   1. **[T2]** Restore its database from the latest snapshot: `backup/restore-school.sh <slug> latest --target db_<slug>` (allowed to target the live name here only because the database does not exist yet).
   2. **[T2]** Restore its `.env` from restic (or regenerate from the vault).
   3. **[T2]** Start `school-<slug>`, check login.
5. **[T2]** Update DNS if the IP changed. With the standing 600-second TTL ([Hostinger KVM plan](../docs/hostinger-kvm.md)), the switch takes effect within about 10 minutes.
6. **[T1]** Re-enable monitoring and the external uptime check.
7. **[T2]** Run a full backup on the new server.

## Verify

- Every school's `https://<domain>/accounts/login/` returns 200 with a valid certificate.
- Each school administrator confirms the latest data (and is told the data-loss window, up to 24 hours).
- Isolation checks pass.

## Afterwards

- Incident note: cause, timeline, data loss, what to change.
- If compromised: review access logs, rotate everything, review Hostinger account security.
