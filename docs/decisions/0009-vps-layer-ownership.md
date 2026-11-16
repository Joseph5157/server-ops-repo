# ADR 0009: One owner per server layer; VPS tools as references

- Status: **Proposed**
- Date: 2026-11-16

## Context

Several tools can configure a Hostinger VPS: Hostinger Agent Skills and MCP, vps-forge, vps-setup-skill and DevSec Ansible. If more than one changes the same setting (for example SSH), the server drifts from this repository and a rebuild no longer matches.

## Decision

| Layer | Owner |
|---|---|
| Outside the server (VPS lifecycle, plan, region, snapshots, Hostinger backups and firewall, hPanel SSH keys, metrics, malware scanner) | Hostinger Agent Skills / MCP or hPanel |
| Inside the server (users, SSH, UFW, fail2ban, updates, sysctl, Docker) | This repository's Ansible, using DevSec hardening |
| Apps, PostgreSQL, Caddy, backups, monitoring | This repository's Compose stacks and restic |

- **vps-forge** and **vps-setup-skill** are references. Run them only on a disposable VPS; copy useful safeguards (such as verifying key login before disabling passwords, and lockout protection) into our Ansible roles.
- DevSec's Nginx role is not used (we use Caddy).
- Hostinger snapshot and backup **restores** overwrite the whole VPS and are tier 3.
- Hostinger firewall and UFW carry identical rules.

## Consequences

- Any server change not made through the owning tool is drift and must be either reverted or added to the repository.
- Details and Hostinger specifics: [docs/hostinger-kvm.md](../hostinger-kvm.md).
