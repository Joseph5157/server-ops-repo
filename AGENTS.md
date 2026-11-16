# Instructions for AI agents (Claude Code, Codex)

This repository operates servers that will hold school records about children, guardians and staff. Mistakes here can lose data or expose it. Work carefully, prefer reading over changing, and stop to ask when unsure.

## Scope

- Server operations only: Ansible, Docker Compose, Caddy, PostgreSQL, backups, monitoring, runbooks.
- Application code belongs in `school-erp`. Do not add or change it here.
- Production runs on Railway and is outside this repository during the pilot.

## Hard security rules

Never add to this repository, even as examples:

- passwords, API keys, tokens, SSH private keys, restic keys or Ansible Vault passwords
- real school `.env` files
- database dumps, exports or any real student, guardian or staff information
- Hostinger API tokens

Use placeholders such as `CHANGE_ME` in `*.env.example` files. Encrypted Ansible Vault files are allowed only once ADR 0007 is approved.

## Approval tiers

Every action falls into one tier. When unsure, treat it as the next tier up.

| Tier | What | Examples | Rule |
|---|---|---|---|
| **0: Read** | Inspect only | `docker ps`, logs, `df -h`, `restic snapshots`, `ansible-playbook --check --diff`, reading files | Do freely |
| **1: Reversible change** | Changes that can be undone quickly and touch no data | Restarting one school's app container, editing docs, opening a draft PR, running a playbook in `--check` mode | State the plan in one line, then do it |
| **2: Controlled change** | Changes to a live server or school | Applying a playbook, adding a school, rolling out a release, rotating a secret, changing firewall or SSH settings | Show the exact commands and the expected result, wait for explicit owner approval, then follow the runbook step by step |
| **3: Destructive** | Anything that can lose data or access | Dropping a database or volume, `restic forget/prune`, deleting a VPS or snapshot, rebuilding a VPS, removing a school, restoring over existing data, disabling backups | Never run. Prepare the commands and the checklist, then hand them to the owner to run |

Also stop and ask whenever:

- a command would affect more than one school at once
- a backup is older than 26 hours or the last restore test failed
- the server state differs from what this repository describes (configuration drift)

## How to work

- Follow the matching runbook in `runbooks/`; the `server-ops` skill in `.claude/skills/server-ops/` explains which one.
- Run Ansible with `--check --diff` first and show the diff before any real run.
- Change one school at a time. Never loop a change across all schools without approval for each wave.
- Take a fresh backup before any tier 2 change that touches a database.
- After every change, verify it worked (health check, login page, logs) and say what you checked.

## Working conventions

- Branch names: `<type>/<short-description>` (for example `docs/foundation-plan`, `ansible/base-role`).
- Work packages are named `OPS-<AREA>-<NN>`; use that as the pull request title prefix.
- Open pull requests as drafts first.
- Decision records go in `docs/decisions/` as `NNNN-short-title.md`, status `Proposed` until the owner approves.
- Do not mark a decision Approved or a runbook Final; only the owner does that.
- Write for operators reading on a phone during an incident: short steps, exact commands, what success looks like.
