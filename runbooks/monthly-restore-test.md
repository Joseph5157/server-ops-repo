# Runbook: monthly restore test

A backup counts only after it has been restored. Run on the first working Monday of each month. Highest tier: **T2** (dropping the scratch database); everything else is T0–T1. Time: about 30 minutes.

## Steps

1. **[T0]** Pick one school (rotate through schools month by month) and note the latest snapshot ID and time.
2. **[T1]** Restore into a scratch database on the same server, or better on a temporary VPS:
   `backup/verify-latest.sh <slug>` (planned). It should:
   - restore `db_<slug>` into `verify_<slug>_<yyyymmdd>`
   - run checks: tables exist, row counts for students, enrollments, attendance registers and assessments are non-zero, latest attendance date is within 1 day of the snapshot
   - run `python manage.py check` and `showmigrations` against it using the school's image
3. **[T1]** Record the result in `docs/restore-tests.md` (date, school, snapshot, duration, pass/fail).
4. **[T2]** Drop the scratch database (it is a copy created for this test; dropping the live database is never part of this runbook).
5. **[T1]** Send a Telegram note with the result; monitoring alerts if no passing test is recorded for 35 days.

## Every quarter, additionally

- Restore on a **separate** temporary VPS from scratch, including the `.env` and roles, to prove the server-lost path works. Destroy the VPS afterwards (**[T3]** owner).

## If the test fails

- Treat as an incident: backups may be unusable.
- Fix the cause, run a fresh backup and repeat the test the same day.
- Do not roll out releases until a test passes.
