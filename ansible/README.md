# ansible/ (planned)

Nothing here has been written or run yet. This describes the planned layout so that work packages can be reviewed against it.

```
ansible/
├── ansible.cfg
├── requirements.yml           # pinned collections and roles
├── inventory/
│   ├── pilot/hosts.yml        # pilot KVM 2 (IP placeholder only)
│   └── group_vars/
│       ├── all.yml            # non-secret settings
│       └── all.vault.yml      # encrypted with Ansible Vault (ADR 0007)
├── playbooks/
│   ├── site.yml               # full server build: base → hardening → docker → platform
│   ├── platform.yml           # Caddy, PostgreSQL, backup, monitoring stacks
│   ├── school-add.yml         # one school: database, login, .env, app stack
│   └── school-release.yml     # one school: pull image tag, migrate, restart
└── roles/
    ├── base/                  # users, SSH keys, timezone Asia/Kolkata, swap, unattended upgrades
    ├── firewall/              # UFW: 22 (restricted), 80, 443 only
    ├── docker/                # Docker Engine + Compose plugin
    ├── platform/              # deploys stacks/platform
    └── school/                # deploys stacks/school-template for one school
```

## Planned external content (pin exact versions)

| Source | Use | Decision |
|---|---|---|
| `devsec.hardening` collection (`os_hardening`, `ssh_hardening`) | OS and SSH hardening | Use after configuration review (ADR 0002) |
| `geerlingguy.docker`, `geerlingguy.swap` | Docker install, swap file | Candidate; review before adopting |
| `community.docker`, `community.postgresql` | Manage Compose stacks, databases, roles | Candidate |

## Rules

- Every playbook must support `--check --diff`.
- Every role must be idempotent: a second run changes nothing.
- Test on a disposable VPS before the pilot server ([new-server runbook](../runbooks/new-server.md)).
- Keep DevSec SSH hardening from locking us out: confirm key login works and keep the Hostinger browser console available before applying it.
