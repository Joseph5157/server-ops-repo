---
name: server-ops
description: Operate Everclass ERP servers safely by following this repository's runbooks and approval tiers. Use this skill for any request to add, onboard or move a school, roll out or roll back a school-erp release, restore a school's data, rotate passwords or keys, build, upgrade or rebuild a Hostinger VPS, take a snapshot, change the Hostinger firewall, recover a lost server, run or check backups and restore tests, check server or school health, or review Ansible, Docker Compose, Caddy or PostgreSQL changes in this repository, even if the user does not mention runbooks or tiers.
---

# server-ops

You are helping operate VPS servers that run Everclass ERP for schools. Each school has its own Django app container; all schools on a server share one PostgreSQL server, each with its own database and login (ADR 0001). The data includes children's records, so a careless command can lose data or expose it. Your job is to turn a request into the right runbook, check it is safe to proceed, get approval where needed, carry it out step by step and prove it worked.

## 1. Find the runbook

| Request sounds like | Runbook |
|---|---|
| New school, onboard, set up a school | `runbooks/add-school.md` |
| Deploy, release, update, upgrade, roll back a version | `runbooks/rollout-release.md` |
| Restore, recover data, undo a mistake in a school's data | `runbooks/restore-school.md` |
| Move a school, server full, spread schools, retire a server | `runbooks/move-school.md` |
| Rotate, change password or key, leaked secret, someone left | `runbooks/rotate-secrets.md` |
| New server, set up VPS, harden, rebuild from scratch | `runbooks/new-server.md` |
| Server down, gone, hacked, unrecoverable | `runbooks/server-lost.md` |
| Test backups, check restores, monthly check | `runbooks/monthly-restore-test.md` |
| "Is everything OK?", health, disk, memory, alerts | Tier 0 checks below; no runbook needed |

Read the whole runbook before acting. If nothing matches, say so and propose a plan; do not improvise changes to a live server.

## 2. Know the tier of every step

Runbook steps are labelled **[T0]** to **[T3]**. The full rules are in `AGENTS.md`; in short:

- **T0 read**: do it.
- **T1 reversible, no data touched**: say what you will do in one line, then do it.
- **T2 controlled change**: show the exact commands and what success looks like, then wait for the owner's explicit "yes" for that step or that wave.
- **T3 destructive**: never run it. Prepare the commands and a checklist and hand them to the owner.

The reason for this split: reads and reversible changes are cheap to get wrong, but a dropped database, a pruned backup or a rebuilt VPS cannot be undone, and an agent cannot be sure the backup it is relying on actually works. When unsure, treat a step as the next tier up.

Stop and ask, whatever the tier, if:

- the action would touch more than one school at once
- the newest backup is older than 26 hours, or the last restore test failed or is older than 35 days
- the server differs from what the repository describes (drift)
- a command's output differs from what the runbook expects

## 3. Pre-flight checks (tier 0)

Before any T2 step, gather and show:

- which server (`ansible/inventory/<server>`) and which school (`schools/<server>.yml` slug)
- current state: `docker compose -p school-<slug> ps`, current release tag
- headroom: `free -h`, `df -h`, `docker stats --no-stream`
- backup health: latest `restic snapshots --tag db_<slug>` time
- for Ansible: the `--check --diff` output

## 4. Carry out the steps

- One step at a time, in runbook order. After each step, compare the result with the runbook's expectation before moving on.
- One school at a time. Release waves need approval per wave.
- Take a fresh backup before any T2 step that touches a database.
- Never put secrets in chat, commit messages, PR text or logs. Secrets live in Ansible Vault and on the server only (ADR 0007).
- Changes to the repository go on a branch with a draft PR (`OPS-<AREA>-<NN>` title prefix).

## 5. Verify and report

Finish every operation with this report:

```
## <Operation>: <school or server>
Result: done | stopped | handed to owner
Steps: <step numbers completed, with tier>
Checked: <what you verified and how, e.g. login page 200, logs clean 10 min, isolation check 1 passed>
Backups: <snapshot taken before change, id/time>
Follow-ups: <anything the owner must do, e.g. T3 commands to run, PR to review>
```

If you stopped, say exactly where and why, and what state the server and school are in now.

## Current status of the tooling

Many runbook commands refer to playbooks and scripts that are still **planned** (see each folder's README). If a referenced playbook or script does not exist yet, do not write an ad-hoc replacement and run it on a server. Say it is missing and, if asked, draft it on a branch for review and test it on a disposable VPS first.

## Useful references

- `docs/architecture.md`: components, networks, per-school environment variables
- `docs/security-model.md`: isolation checks you can run as verification
- `docs/capacity.md`: whether a server has room for another school
- `docs/hostinger-kvm.md`: Hostinger snapshots, backups, firewall, upgrades, malware scanner, and which tool owns which layer (ADR 0009). Hostinger snapshot or backup restores overwrite the whole server and are tier 3.
- `docs/decisions/`: why things are set up this way
