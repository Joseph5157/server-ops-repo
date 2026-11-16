# Hostinger KVM plan

Status: draft. Hostinger facts below were checked against Hostinger's support pages on 2026-11-16; recheck before relying on them, as plans and features change.

## 1. Which tool owns which layer

One tool owns each layer, so a rebuilt server comes out identical and two tools never fight over the same setting.

| Layer | Owner | Not used for |
|---|---|---|
| Outside the server: create VPS, plan, region, snapshots, Hostinger backups, Hostinger firewall, hPanel SSH keys, metrics, malware scanner | Hostinger Agent Skills / official MCP, or hPanel by hand | Configuring anything inside the server |
| Inside the server: users, SSH, UFW, fail2ban, unattended upgrades, sysctl, Docker | Our Ansible + DevSec hardening collection | — |
| Apps, PostgreSQL, Caddy, backups, monitoring | Our Compose stacks + restic | — |
| vps-forge, vps-setup-skill | Reference only: run on a disposable VPS, copy good ideas into Ansible | Running on the pilot or any school server (causes drift) |

See [ADR 0009](decisions/0009-vps-layer-ownership.md).

## 2. Plan and sizing

| Stage | Plan | Notes |
|---|---|---|
| Disposable test VPS | KVM 1 (or the smallest available) | Destroy after testing Ansible and vps-forge |
| Pilot | **KVM 2** (2 vCPU, 8 GB, 100 GB NVMe) | About 10–12 schools comfortably ([capacity](capacity.md)) |
| Grow | KVM 4 **or** a second KVM 2 | See section 7 |

**Upgrading:** Hostinger upgrades the whole plan (RAM, CPU and disk cannot be bought separately) and says an upgrade can take up to 10 minutes. Treat it as downtime for every school on the server: do it outside school hours, take a backup first, tell schools. Hostinger documents upgrades only; assume you **cannot downgrade**, so do not upgrade "just to try".

**Billing term:** introductory prices need a longer term and renew higher (KVM 2 in India: about ₹799/month introductory, ₹1,199/month on renewal, per a third-party tracker, August 2026). For the pilot, prefer a term you would not regret if the pilot ends; record the renewal date in the owner's calendar.

## 3. Region

- Hostinger offers India among its VPS locations; choose **India** (lowest latency for Telangana and Andhra Pradesh schools, and keeps school data in-country). Open decision D7.
- Hostinger's support page says a VPS's **location is fixed after initial setup**; moving means backing up and setting up a new VPS elsewhere. Choose carefully at order time.

## 4. Operating system

- Plain **Ubuntu LTS** template (24.04 at the time of writing). No control-panel templates (CyberPanel and similar), because they add services we do not manage in Ansible.
- Reinstalling the OS **deletes the snapshot**. Treat OS reinstall as tier 3.

## 5. Hostinger backups and snapshots (extra safety layer, not our backup)

| Feature | What Hostinger provides (as documented) | How we use it |
|---|---|---|
| Weekly auto-backup | One weekly backup kept; each new one replaces the last | Leave on. Last-resort whole-server rollback |
| Daily auto-backups | Paid add-on; daily copies kept for about a week (Hostinger pages say 7 or 14 days) | Optional (open decision D12); restic remains primary |
| Snapshot | Manual; **only one at a time**; new one overwrites the old; expires after **20 days**; deleted on OS reinstall or restore from backup | Take one immediately before risky server changes (hardening, plan upgrade, major Docker or PostgreSQL upgrade) |
| Restore | Overwrites the **whole** VPS; the VPS is locked while restoring | **Tier 3**: rolls back every school on the server, losing their recent data |

Why restic is still the real backup: Hostinger backups live with the same provider and account, restore only the whole server, and cannot restore one school. Restic backups are off-provider, encrypted and per school ([ADR 0004](decisions/0004-backups-restic-s3.md)).

## 6. Firewalls: Hostinger firewall and UFW

Hostinger's managed firewall filters traffic **before** it reaches the VPS and is managed in hPanel. Activating a firewall group drops everything not allowed, so add the SSH rule first.

Use both layers, with **identical** rules:

| Port | Hostinger firewall | UFW |
|---|---|---|
| 22/tcp | Allow (restrict to admin IPs if they are stable) | Allow (same) |
| 80/tcp | Allow | Allow |
| 443/tcp | Allow | Allow |
| Everything else | Drop | Deny |

Why both: Docker writes its own iptables rules for published ports and can bypass UFW. Our stacks publish only Caddy's 80 and 443, but the Hostinger firewall is the safety net if a container ever publishes a port by mistake. A test in the [new-server runbook](../runbooks/new-server.md) checks from outside that only 22, 80 and 443 answer.

## 7. Growing beyond one server

| Signal | Action |
|---|---|
| Memory over ~80% at the morning peak, or the next school would break the [capacity](capacity.md) limit | Add a second KVM 2, or upgrade to KVM 4 |
| Want to limit how many schools one failure affects | Prefer **more servers** over a bigger one |
| Upgrade is cheaper and downtime is acceptable | Upgrade to KVM 4 |

Rules for several servers:

- Each server has its own inventory entry, `schools/<server>.yml`, PostgreSQL, Caddy, backups and monitoring. No server depends on another.
- Spread schools so no server holds all large schools.
- Move schools between servers with the [move-school runbook](../runbooks/move-school.md).

## 8. DNS

- Standing TTL for every school domain and the server hostname: **600 seconds** (10 minutes). Short enough that [server-lost](../runbooks/server-lost.md) and [move-school](../runbooks/move-school.md) switch quickly, long enough to avoid needless lookups.
- Lower to 300 seconds a day before a planned move; raise back afterwards.
- Keep DNS with a provider whose account also has two-factor authentication; whoever controls DNS controls the schools' traffic.

## 9. Account security

- Two-factor authentication on the Hostinger account for every person with access.
- Owner and one named deputy only (matches ADR 0007 key holders).
- Hostinger API token for the MCP: kept on the operator's machine, never in Git or on servers; rotate when anyone with access leaves.
- Invoices and renewal alerts go to a shared team mailbox, not one person's inbox.
- hPanel SSH keys: add the admin public keys there for console recovery; Ansible manages the keys inside the server.

## 10. Malware scanning

Hostinger's VPS Malware Scanner uses the Monarx agent:

- Installing it takes up to 60 minutes, during which the VPS cannot be managed in hPanel.
- Detection is included; **automatic cleanup needs a paid Monarx licence**.
- It scans files on disk, **not database contents**.

Plan (open decision D13): install the detection agent (a named exception to layer ownership, see [ADR 0009](decisions/0009-vps-layer-ownership.md)) on the pilot after hardening, and review results weekly. Treat any finding on a school server as an incident: do not just delete the file; check how it got there, rotate secrets, consider rebuilding from the repository and backups. It is an extra layer, not a replacement for hardening and updates.

## 11. Hostinger metrics vs our monitoring

hPanel shows CPU, memory, disk and network. Useful for a quick look and for choosing when to upgrade, but alerts come from our monitoring ([ADR 0006](decisions/0006-lightweight-monitoring.md)), which knows about schools and backups.

## Sources

- [Hostinger: back up or restore a VPS](https://www.hostinger.com/support/1583232-how-to-back-up-or-restore-a-vps)
- [Hostinger: upgrade a VPS](https://www.hostinger.com/support/1583229-how-to-upgrade-a-vps-server/)
- [Hostinger: server locations](https://www.hostinger.com/support/1583267-where-are-your-servers-located)
- [Hostinger: VPS malware scanner](https://www.hostinger.com/support/8450363-hostinger-vps-malware-scanner/)
- [Hostinger: managed VPS firewall](https://www.hostinger.com/support/8172641-how-to-use-a-managed-vps-firewall-at-hostinger/)
