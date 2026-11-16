# Runbooks

Step-by-step procedures. Each step is labelled with its approval tier from [AGENTS.md](../AGENTS.md): **[T0]** read, **[T1]** reversible, **[T2]** needs owner approval, **[T3]** owner runs it.

> **Status: draft.** Commands refer to playbooks and scripts that are planned but not yet written. Update each runbook when its tooling exists and has been tested on the pilot.

| Runbook | Use when | Highest tier |
|---|---|---|
| [new-server](new-server.md) | Building a server from blank | T2 |
| [add-school](add-school.md) | Onboarding a school onto a server | T2 |
| [rollout-release](rollout-release.md) | Releasing a new `school-erp` version | T2 |
| [restore-school](restore-school.md) | Bringing back one school's data | T3 |
| [rotate-secrets](rotate-secrets.md) | Routine rotation or suspected leak | T2 |
| [server-lost](server-lost.md) | The server is gone or unusable | T3 |
| [monthly-restore-test](monthly-restore-test.md) | Proving backups work, every month | T2 |

## Conventions

- `<slug>`: the school's slug from `schools/<server>.yml`.
- `<server>`: the inventory name, for example `pilot`.
- Commands run from the operator's machine unless they say "on the server".
- Every runbook ends with **Verify** and, where relevant, **Roll back**.
- If any step's result differs from what the runbook expects, stop and report.
