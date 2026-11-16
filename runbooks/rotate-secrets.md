# Runbook: rotate secrets

Highest tier: **T2**. Rotate on schedule (every 12 months) and **immediately** when someone with access leaves or a leak is suspected.

## What and how

| Secret | Scope | Downtime |
|---|---|---|
| School database password | One school | Restart of that school |
| School `DJANGO_SECRET_KEY` | One school | Restart; all its users are signed out |
| Telegram bot token | All schools | Restart of each school, one at a time |
| S3 access key for backups | Server | None |
| restic password | Server | None (add new key, remove old) |
| PostgreSQL superuser password | Server | None for schools |
| Admin user SSH keys | Server | None |
| Ansible Vault password | Repository | None |

## Steps (one school's database password and secret key)

1. **[T1]** Generate new values and update them in the vault on a branch (`ansible-vault edit`).
2. **[T0]** Dry run `school-add.yml -e slug=<slug> --tags env,dbuser --check --diff` (planned tags).
3. **[T2]** Apply: sets the new password on `app_<slug>`, rewrites `.env`, restarts `school-<slug>`.
4. **[T0]** Verify login page and logs.

## restic password

1. **[T2]** `restic key add` with the new password, confirm `restic snapshots` works with it.
2. **[T2]** Update the password file on the server and in the vault.
3. **[T3]** Owner removes the old key: `restic key remove <id>`.

## Suspected leak

- Rotate the leaked secret first, then investigate.
- If a database password leaked, also check PostgreSQL logs for connections from unexpected addresses.
- If the Hostinger or GitHub token leaked, revoke it in their dashboards immediately (owner).
