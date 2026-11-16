# References and inspiration

We copy **patterns**, not code, and pin anything we adopt. Record the version reviewed when something is adopted.

## Infrastructure

| Source | What we take | Status |
|---|---|---|
| [dev-sec/ansible-collection-hardening](https://github.com/dev-sec/ansible-collection-hardening) | OS and SSH hardening roles | Use after review (ADR 0002) |
| [geerlingguy Ansible roles](https://github.com/geerlingguy) (`docker`, `swap`, `security`) | Readable, widely used roles | Candidate |
| [cookiecutter-django](https://github.com/cookiecutter/cookiecutter-django) | Production Dockerfile and Compose patterns for Django | Patterns only (for `school-erp` Dockerfile) |
| [lucaslorentz/caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) | Label-based Caddy routing | Considered; Ansible-generated Caddyfile preferred (ADR 0005) |
| restic-pg-dump (Docker image) | Dump every database separately into restic | Pattern for `backup/` |
| [tiredofit/docker-db-backup](https://github.com/tiredofit/docker-db-backup) | Multi-database scheduled backups to S3 | Pattern for `backup/` |
| [Uptime Kuma](https://github.com/louislam/uptime-kuma) | Uptime and certificate checks | Planned (ADR 0006) |
| Dokploy, Coolify, Kamal | How self-hosted platforms organise per-app stacks | Comparison only |
| Hostinger official MCP (`hostinger-api-mcp`), Hostinger Agent Skills | VPS operations from Claude Code | Use with tier rules (ADR 0002) |
| vps-forge | Hardening ideas, lockout safeguards, fail2ban, Docker network rules | Evaluate on disposable VPS; reference only (ADR 0009) |
| vps-setup-skill | Guided fresh-VPS setup; verifies key login before disabling passwords | Reference only (ADR 0009) |
| Hostinger support pages | Snapshots, backups, upgrades, firewall, locations, malware scanner | Summarised in [hostinger-kvm.md](hostinger-kvm.md) |

## Skills for agents

| Skill | Use |
|---|---|
| Anthropic `skill-creator` | Build and test the `server-ops` skill in this repository |
| Sentry `django-access-review` (`getsentry/skills`) | IDOR and authorization review in `school-erp` |
| Claude Code Django Toolkit (`django-security`) | Optional second opinion |
| Codex multi-tenancy skill | Cross-tenant test checklist ideas only; review scripts first |

## Not adopted

| Source | Why |
|---|---|
| django-multitenant | Shared-table design needs a rewrite of ~45 models; published compatibility lists Django up to 4.2 |
| django-tenants | Not needed for design B; revisit if moving to schema per school |
| MySQL / MariaDB | See ADR 0003 |

## Pricing references (checked 2026-11-16)

- [Railway pricing](https://docs.railway.com/pricing)
- [Hostinger VPS prices in India, third-party tracker](https://zoutons.com/news/vps-hosting-price-hostinger-vs-godaddy-india-2026)
