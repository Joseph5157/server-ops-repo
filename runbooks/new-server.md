# Runbook: build a new server

Highest tier: **T2**. Time: about 1–2 hours the first time.

## Before you start

- [ ] Hostinger KVM ordered in the agreed region (D7), Ubuntu LTS image, your SSH public key added in hPanel.
- [ ] Two-factor authentication on the Hostinger account.
- [ ] Ansible Vault password and restic password available (ADR 0007).
- [ ] DNS records for the server hostname and school domains ready (can point later).

## Steps

1. **[T0]** Confirm key login works: `ssh root@<ip> 'uname -a && lsb_release -a'`.
2. **[T0]** Open the Hostinger browser console once and confirm you can log in. This is the way back if SSH hardening goes wrong.
3. **[T1]** Add the server to `ansible/inventory/<server>/hosts.yml` on a branch; open a draft PR.
4. **[T0]** Dry run: `ansible-playbook -i inventory/<server> playbooks/site.yml --check --diff`. Review the diff, especially SSH, firewall and users.
5. **[T2]** Apply base, firewall and Docker: `ansible-playbook ... site.yml --tags base,firewall,docker`.
6. **[T0]** In a **second** terminal, confirm you can still log in as the admin user with your key before closing the first session.
7. **[T2]** Apply hardening: `--tags hardening`. Repeat step 6.
8. **[T2]** Apply the platform (Caddy, PostgreSQL, backup, monitoring): `--tags platform`.
9. **[T0]** Run the playbook again in check mode; expect **no changes** (idempotency).
10. **[T1]** Add the server to Uptime Kuma and the external uptime check.

## Verify

- `ss -tlnp` on the server shows only 22, 80, 443 listening publicly.
- `docker ps` shows caddy, postgres, backup, uptime-kuma healthy.
- From outside: port 5432 closed.
- A manual backup run succeeds: on the server, `backup/backup-all.sh` (planned), then `restic snapshots` lists it.
- A Telegram test alert arrives.

## Roll back

- If locked out after hardening: Hostinger console → revert `/etc/ssh/sshd_config.d/` changes → fix the role variables → re-run.
- A brand-new server with no schools can be rebuilt from the Hostinger panel: **[T3]** owner only.
