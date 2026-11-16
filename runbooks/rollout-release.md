# Runbook: roll out a school-erp release

Highest tier: **T2**. Releases go **one school at a time, in waves**. Never update every school at once.

## Before you start

- [ ] The release is tagged in `school-erp` (for example `v1.8.0`), CI passed, and the image `ghcr.io/joseph5157/school-erp:v1.8.0` exists.
- [ ] Release notes read: do they include migrations? Any long-running data migration?
- [ ] Nightly backups healthy for every school in this wave.
- [ ] Not during school hours for schools in the wave (avoid 08:00–16:00 IST on working days).

## Waves

| Wave | Schools | Wait before next wave |
|---|---|---|
| 1 | Demo school | 1 hour of checks |
| 2 | One pilot school | 1 school day |
| 3 | Remaining schools, a few at a time | Monitor after each |

Each wave needs owner approval.

## Steps for one school

1. **[T0]** Current state: `docker compose -p school-<slug> ps`, note the current tag.
2. **[T2]** Fresh backup of this school: `backup/backup-all.sh --only db_<slug>` (planned). Confirm the snapshot exists.
3. **[T1]** Change `release:` for this school in `schools/<server>.yml` on a branch; draft PR.
4. **[T0]** Dry run: `ansible-playbook ... playbooks/school-release.yml -e slug=<slug> --check --diff`.
5. **[T1]** Pause the school's Uptime Kuma checks for the restart. **[T2]** Apply: the playbook pulls the image, runs `migrate`, then restarts `web` and the jobs. **[T1]** Resume the checks once the login page answers.
6. **[T0]** Verify (below). If anything fails, roll back straight away.

## Verify

- `https://<domain>/accounts/login/` returns 200; sign in with the school's test or admin account if agreed.
- `docker compose -p school-<slug> logs --since 10m web` shows no errors.
- `python manage.py showmigrations | grep '\[ \]'` returns nothing.
- Uptime Kuma green; no Telegram crash alerts for 30 minutes.

## Roll back

- **No migrations in the release:** set `release:` back to the previous tag and re-run step 5 (**[T2]**).
- **Migrations already applied:** the previous image may not work with the new schema. Options, owner decides:
  1. Fix forward with a patch release (preferred if the problem is small).
  2. **[T3]** Restore this school's pre-release backup to a new database and switch the school back to the old tag (follow [restore-school](restore-school.md)). Data entered since the backup is lost.
- Pause later waves until the cause is understood.
