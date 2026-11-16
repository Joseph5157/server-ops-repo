# Runbook: move a school to another server

Use when a server is full, to spread load, or before retiring a server. Highest tier: **T3** (the final cut-over and removing the school from the old server). Time: about 1 hour plus DNS propagation. Schedule outside school hours.

## Before you start

- [ ] Owner approved the move, the target server and the time window.
- [ ] Target server built with [new-server](new-server.md) and has capacity ([capacity](../docs/capacity.md)).
- [ ] Target server runs the **same PostgreSQL major version** and can run the school's release tag.
- [ ] School told about a short maintenance window.
- [ ] **[T1]** A day ahead, lower the school domain's DNS TTL to 300 seconds, so the switch takes effect quickly.

## Steps

1. **[T1]** On a branch, add the school to `schools/<target>.yml` with the **same** slug, domain and release, `status: paused`. Draft PR.
2. **[T2]** Prepare the target: run `school-add.yml -e slug=<slug>` on the target with `--check --diff` first, then for real, but **do not migrate or start** the app yet (planned flag, for example `-e prepare_only=true`). This creates `db_<slug>`, `app_<slug>` and the `.env`.
3. **[T2]** Maintenance on the source: stop the whole school stack (`docker compose -p school-<slug> stop`) and show the maintenance page. Confirm no connections remain to `db_<slug>` (see [restore-school](restore-school.md) step 4).
4. **[T2]** Final backup on the source: `backup/backup-all.sh --only db_<slug>` (planned). Note the snapshot ID.
5. **[T2]** Restore that snapshot into the empty `db_<slug>` on the target (allowed because it is new and empty): `backup/restore-school.sh <slug> <snapshot-id> --target db_<slug>` (planned). Re-apply ownership and grants for `app_<slug>`.
6. **[T0]** Compare row counts for students, enrollments, attendance registers and assessments between source and target.
7. **[T2]** Start the school on the target; set `status: active` in `schools/<target>.yml` and apply so Caddy on the target serves the domain. For the brief window before DNS switches, Caddy may not get a certificate yet; that is expected.
8. **[T1]** Point the domain's DNS A record (and AAAA, if used) to the target server's IP.
9. **[T0]** When `https://<domain>/accounts/login/` returns 200 from the target with a valid certificate, ask the school administrator to check recent records.
10. **[T2]** Run a backup on the target and confirm `db_<slug>` appears in its restic snapshots.

## Clean up the source (after 7 days)

- **[T1]** Set `status: leaving` in `schools/<source>.yml`, remove its Caddy route and monitoring.
- **[T3] Owner:** drop `db_<slug>` and `app_<slug>` on the source, delete `/srv/schools/<slug>/`, remove it from `schools/<source>.yml`.
- **[T1]** Raise the DNS TTL back to normal.

## Roll back (before step 8)

- Stop the school on the target, restart it on the source (`docker compose -p school-<slug> start`), remove the maintenance page. Nothing was entered on the target, so no data is lost.

## Roll back (after step 8)

- Data may already have been entered on the target. Owner decides: move back with this same runbook in reverse (target becomes source), never by pointing DNS back alone.
