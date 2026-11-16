# Runbook: build a new server

Highest tier: **T2**. Time: about 1–2 hours the first time.

## Before you start

- [ ] Hostinger KVM ordered in the agreed region (D7; the location cannot be changed later), plain Ubuntu LTS template (no control panel), admin SSH public keys added in hPanel. See [Hostinger KVM plan](../docs/hostinger-kvm.md).
- [ ] Two-factor authentication on the Hostinger account.
- [ ] Ansible Vault password and restic password available (ADR 0007).
- [ ] DNS records for the server hostname and school domains ready (can point later).

## Steps

1. **[T0]** Confirm key login works: `ssh root@<ip> 'uname -a && lsb_release -a'`.
2. **[T0]** Open the Hostinger browser console once and confirm you can log in. This is the way back if SSH hardening goes wrong.
3. **[T2]** In hPanel, create the Hostinger firewall group with **SSH first**, then 80 and 443, and activate it. Confirm SSH still works.
4. **[T2]** Take a Hostinger snapshot (only one is kept, for 20 days) so the clean install can be rolled back while no schools are on it.
5. **[T1]** Add the server to `ansible/inventory/<server>/hosts.yml` on a branch; open a draft PR.
6. **[T0]** Dry run: `ansible-playbook -i inventory/<server> playbooks/site.yml --check --diff`. Review the diff, especially SSH, firewall and users.
7. **[T2]** Apply base, firewall and Docker: `ansible-playbook ... site.yml --tags base,firewall,docker`.
8. **[T0]** In a **second** terminal, confirm you can still log in as the admin user with your key before closing the first session.
9. **[T2]** Apply hardening: `--tags hardening`. Repeat step 8.
10. **[T2]** Apply the platform (Caddy, PostgreSQL, backup, monitoring): `--tags platform`.
11. **[T0]** Run the playbook again in check mode; expect **no changes** (idempotency).
12. **[T1]** Add the server to Uptime Kuma and the external uptime check.
13. **[T2]** If decided (D13), install Hostinger's malware scanner from hPanel (up to 60 minutes; the VPS cannot be managed in hPanel meanwhile).

## Verify

- `ss -tlnp` on the server shows only 22, 80, 443 listening publicly.
- `docker ps` shows caddy, postgres, backup, uptime-kuma healthy.
- From outside (`nmap -Pn <ip>` from another machine): only 22, 80 and 443 answer; 5432 is closed. This also checks that Docker has not published a port past UFW.
- Hostinger firewall rules match UFW rules.
- A manual backup run succeeds: on the server, `backup/backup-all.sh` (planned), then `restic snapshots` lists it.
- A Telegram test alert arrives.

## Roll back

- If locked out after hardening: Hostinger console → revert `/etc/ssh/sshd_config.d/` changes → fix the role variables → re-run.
- A brand-new server with no schools can be rolled back to the step-4 snapshot or reinstalled from hPanel: **[T3]** owner only (reinstalling deletes the snapshot).
