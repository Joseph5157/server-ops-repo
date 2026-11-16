# monitoring/ (planned)

Lightweight monitoring for one or a few VPS servers. Design: [ADR 0006](../docs/decisions/0006-lightweight-monitoring.md).

```
monitoring/
├── uptime-kuma/           # monitor list (exported config, no secrets)
├── host-checks.sh         # disk, memory, load, container restarts, backup age
└── alert-rules.md         # thresholds and who gets alerted
```

## Planned checks

| Check | Threshold | Alert |
|---|---|---|
| Each school login page `https://<domain>/accounts/login/` (expect HTTP 200) | Down for 2 checks (2 minutes) | Telegram |
| HTTPS certificate expiry | Under 14 days | Telegram |
| Disk use | Over 80% | Telegram |
| Memory available | Under 10% for 10 minutes | Telegram |
| Container restart loop | More than 3 restarts in 15 minutes | Telegram |
| Newest backup snapshot | Older than 26 hours | Telegram |
| Last monthly restore test | Older than 35 days or failed | Telegram |
| PostgreSQL connections | Over 80% of `max_connections` | Telegram |
| Hostinger malware scanner findings (if installed, D13) | Any finding | Weekly manual review in hPanel; treat as incident |

Application crash alerts already exist in `school-erp` (Telegram error alerts and optional email). Each school's alerts must name the school.

## Where Uptime Kuma runs

Running it on the same VPS cannot alert when the whole VPS is down. During the pilot, add one free external uptime check (or a second small server) for the server itself.
