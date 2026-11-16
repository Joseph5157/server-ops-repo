# ADR 0006: Lightweight monitoring first

- Status: **Proposed** (owner chose lightweight 2026-11-16)
- Date: 2026-11-16

## Decision

- **Uptime Kuma** for login-page checks per school and certificate expiry.
- A small **host-checks script** (cron) for disk, memory, container restart loops, backup age and restore-test age.
- **Telegram** for alerts, reusing the operator chat already used by `school-erp` crash alerts.
- One **external** uptime check for the server itself, because Uptime Kuma on the same VPS cannot report the VPS being down.

Not now: Prometheus, Grafana, Alertmanager (too heavy for a KVM 2 running schools). Revisit with several servers.

## Consequences

- Thresholds live in `monitoring/README.md` and later `monitoring/alert-rules.md`.
- Each school's application alerts must include the school's name or slug.
- Logs: Docker json-file with rotation; Caddy access logs rotated. No central log system during the pilot.
