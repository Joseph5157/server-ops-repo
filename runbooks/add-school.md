# Runbook: add a school to a server

Highest tier: **T2**. Time: about 30 minutes plus data setup.

## Before you start

- [ ] Owner approved this school on this server.
- [ ] Capacity check: `docs/capacity.md` limit not exceeded; **[T0]** `free -h` and `docker stats --no-stream` show headroom for ~400 MB more.
- [ ] Domain chosen and DNS A record points to the server.
- [ ] Release tag to deploy (normally the latest approved tag).
- [ ] Last nightly backup succeeded (**[T0]** check monitoring).

## Steps

1. **[T1]** On a branch, add the school to `schools/<server>.yml`:
   ```yaml
   - slug: <slug>
     domain: <domain>
     release: <tag>
     gunicorn_workers: 2
     memory_limit: 400m
     status: active
   ```
   Open a draft PR.
2. **[T1]** Add the school's generated secrets to the vault: `ansible-vault edit inventory/group_vars/all.vault.yml` (new `DJANGO_SECRET_KEY`, database password). Never paste them anywhere else.
3. **[T0]** Dry run: `ansible-playbook -i inventory/<server> playbooks/school-add.yml -e slug=<slug> --check --diff`.
4. **[T2]** Apply. The playbook should:
   - create database `db_<slug>` and login `app_<slug>`, `REVOKE CONNECT ... FROM PUBLIC`, grant connect to `app_<slug>` only
   - write `/srv/schools/<slug>/.env` (mode 0600)
   - start `school-<slug>`, run `migrate`, `collectstatic`
   - regenerate the Caddyfile and reload Caddy
   - add Uptime Kuma checks
5. **[T2]** Create the operator's own superuser for this school (kept by us, not given to the school): `docker compose -p school-<slug> exec web python manage.py createsuperuser`. Store its password in the vault.
6. **[T2]** Create the school's administrator as a **normal user in the School Administrators group**, not as a superuser: sign in to `https://<domain>/admin/` with the operator superuser, add the user, add them to **School Administrators**, leave *superuser* and *staff* unticked. That group holds every app permission (`school-erp` `docs/PERMISSIONS.md`) without giving the school Django admin access. Hand the credentials over through an agreed secure channel, never chat or email in plain text, and ask the administrator to change the password at first sign-in.
7. **[T1]** School data: follow `school-erp-go-to-market/docs/implementation/` (initial configuration, Excel import). That is the school's work with our help, not a server task.
8. **[T2]** Run a backup now: `backup/backup-all.sh` (planned).

## Verify

- `https://<domain>/accounts/login/` returns 200 with a valid certificate.
- **[T0]** Isolation: from `school-<slug>`, connecting to another school's database fails (security model check 1).
- Uptime Kuma shows the school green; a Telegram test alert names `<slug>`.
- `restic snapshots` includes `db_<slug>`.

## Roll back (school not yet in use)

- **[T2]** Stop the stack: `docker compose -p school-<slug> down`.
- **[T3]** Dropping `db_<slug>` and deleting `/srv/schools/<slug>/` is owner-only.
- Revert the PR.
