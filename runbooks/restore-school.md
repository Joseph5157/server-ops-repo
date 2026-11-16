# Runbook: restore one school

Highest tier: **T3** (switching a school to restored data loses anything entered after the backup). Target: under 1 hour.

## Decide first (owner)

- Which school, and why (data deleted by mistake, bad migration, corruption)?
- Which snapshot (time)? Anything entered after it will be lost unless re-entered.
- Is the school told? Agree a short maintenance message.

## Steps

1. **[T0]** List snapshots for the school: `restic snapshots --tag db_<slug>`.
2. **[T2]** Restore into a **new** database, never over the live one:
   `backup/restore-school.sh <slug> <snapshot-id> --target db_<slug>_restore_<yyyymmdd>` (planned). The script creates the database owned by `app_<slug>` and restores with `pg_restore --no-owner --role=app_<slug>`.
3. **[T0]** Check the restored data: row counts for students, attendance registers and assessments compared with expectations; latest attendance date matches the snapshot time.
4. **[T1]** Pause the school's Uptime Kuma checks so the planned downtime does not alert.
   **[T2]** Put the school in maintenance: stop the **whole** school stack, including scheduled jobs (`docker compose -p school-<slug> stop`), and show a maintenance page in Caddy. Then **[T0]** confirm no connections remain: `SELECT pid, usename, application_name FROM pg_stat_activity WHERE datname = 'db_<slug>';` If any remain, **[T2]** end them: `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'db_<slug>' AND pid <> pg_backend_pid();` (renaming a database fails while it has connections).
5. **[T2]** Take a final backup of the current (broken) database, so nothing is lost for good: `backup/backup-all.sh --only db_<slug>`.
6. **[T3] Owner:** switch the school to the restored database. Either rename databases (`ALTER DATABASE db_<slug> RENAME TO db_<slug>_broken_<yyyymmdd>; ALTER DATABASE db_<slug>_restore_<yyyymmdd> RENAME TO db_<slug>;`) or point `DB_NAME` in the school's `.env` at the restored database. Confirm `app_<slug>` owns the restored database and its objects (the restore script sets this).
7. **[T2]** Start the school (then **[T1]** resume its Uptime Kuma checks): `docker compose -p school-<slug> up -d`, run `migrate` if the release needs it.

## Verify

- Login works; the school's administrator confirms recent records look right.
- Isolation check 1 still passes for this school.
- Next nightly backup includes the restored database.

## Afterwards

- Keep `db_<slug>_broken_<date>` for 7 days, then **[T3]** owner drops it.
- Write a short incident note: cause, data loss window, fix.
