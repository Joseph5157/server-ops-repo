# schools/ (planned)

A non-secret registry of which schools run on which server. Ansible reads it to generate Caddy routes, databases and app stacks.

Planned format, one file per server (for example `schools/pilot.yml`):

```yaml
server: pilot-kvm2-01
schools:
  - slug: demo                 # lowercase letters, digits, hyphens; used in names
    domain: demo.example.in    # placeholder
    release: v0.0.0            # school-erp image tag
    gunicorn_workers: 2
    memory_limit: 400m
    status: active             # active | paused | leaving
```

Derived names (never typed by hand):

| Item | Pattern |
|---|---|
| Database | `db_<slug>` (hyphens become underscores) |
| PostgreSQL login | `app_<slug>` |
| Compose project | `school-<slug>` |
| Server path | `/srv/schools/<slug>/` |

**Never** put names of real children, guardians or staff, contact details, passwords or `.env` values in this folder. A real school's name and domain may be listed once the owner approves it.
