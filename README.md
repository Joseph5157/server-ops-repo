# server-ops

Server operations for **Everclass ERP** on self-managed VPS servers.

This repository holds what we need to rebuild a secure server from nothing, recover from failures and, later, manage several VPS servers the same way:

- approved Ansible configuration
- Docker Compose stacks
- backup and restore scripts
- monitoring and alert rules
- runbooks
- instructions and a skill for Claude Code and Codex

> **Status: planning.** This branch contains decisions, architecture and runbooks only. No Ansible roles, Compose files or scripts have been written or run yet. Every decision record is **Proposed** until the owner approves it.

## Scope

| In scope | Out of scope |
|---|---|
| Server setup, hardening, Docker, Caddy, PostgreSQL | Application code (lives in `school-erp`) |
| One app container per school, shared PostgreSQL | Marketing and sales material (lives in `school-erp-go-to-market`) |
| Backups, restores, monitoring, alerts | Real school data, exports or credentials |
| Runbooks and AI-agent operating rules | Current Railway production (unchanged during the pilot) |

Production stays on **Railway**. This repository supports a **pilot** on a Hostinger KVM VPS to test hosting several schools efficiently.

## Chosen architecture (proposed)

**Design B: one Django app container per school, one shared PostgreSQL server with one database per school.**

```
                Internet
                   │
             Caddy (HTTPS)
     ┌─────────────┼─────────────┐
 school-a.app  school-b.app  school-c.app     ← same Everclass image, own .env
     └─────────────┼─────────────┘
          PostgreSQL 16 (one server)
        db_school_a · db_school_b · db_school_c   ← own database and own login each
                   │
       restic → encrypted off-server backups (S3-compatible)
```

See [docs/architecture.md](docs/architecture.md) and [ADR 0001](docs/decisions/0001-design-b-app-per-school-shared-postgres.md).

## Repository layout

| Path | Purpose | Status |
|---|---|---|
| `AGENTS.md`, `CLAUDE.md` | Rules for Claude Code and Codex | Draft |
| `.claude/skills/server-ops/` | Claude Code skill that follows the runbooks | Draft |
| `docs/` | Architecture, capacity, Hostinger KVM plan, security model, pilot test plan, open decisions, references | Draft |
| `docs/decisions/` | Architecture decision records (ADRs) | Proposed |
| `runbooks/` | Step-by-step operating procedures | Draft |
| `ansible/` | Inventory, roles and playbooks | Planned |
| `stacks/` | Docker Compose: shared platform and per-school template | Planned |
| `backup/` | restic backup, restore and verification scripts | Planned |
| `monitoring/` | Uptime Kuma checks, host checks, Telegram alerts | Planned |
| `schools/` | Non-secret registry of schools on each server | Planned |

## Start here

1. [Architecture](docs/architecture.md)
1. [Hostinger KVM plan](docs/hostinger-kvm.md)
1. [Decision records](docs/decisions/)
1. [Open decisions](docs/open-decisions.md): what the owner still needs to decide
1. [Pilot test plan](docs/pilot-test-plan.md): what must pass before any real school moves
1. [Runbooks](runbooks/)

## Hard rules

- No secrets, passwords, tokens, keys or school data in Git. See [ADR 0007](docs/decisions/0007-secrets-management.md).
- A backup counts only after it has been restored. See [monthly restore test](runbooks/monthly-restore-test.md).
- Every server must be rebuildable from this repository plus the encrypted backups.
- AI agents follow the approval tiers in [AGENTS.md](AGENTS.md).
