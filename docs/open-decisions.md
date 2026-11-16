# Open decisions

Decisions the owner still needs to make before the pilot server is built. Each links to the document it affects.

| # | Decision | Options | Suggested default | Affects |
|---|---|---|---|---|
| D1 | Approve design B | B (app per school, shared Postgres) / schema per school / full stack per school | B | [ADR 0001](decisions/0001-design-b-app-per-school-shared-postgres.md) |
| D2 | Backup storage provider | Backblaze B2 / Cloudflare R2 / Wasabi / second VPS | An S3-compatible bucket in a region you accept for school data **that supports versioning (or hide + lifecycle) and per-bucket keys** | [ADR 0004](decisions/0004-backups-restic-s3.md) |
| D3 | Who holds the restic key and Ansible Vault password | Owner only / owner + one named deputy | Owner + one deputy, stored in a password manager and on paper in a safe place | [ADR 0007](decisions/0007-secrets-management.md) |
| D4 | School domains | Subdomains of one product domain / each school's own domain / both | Subdomains first; own domains later via Caddy on-demand TLS | [ADR 0005](decisions/0005-caddy-reverse-proxy.md) |
| D5 | Pilot server size | KVM 2 / KVM 4 | KVM 2 for the pilot | [capacity](capacity.md) |
| D6 | Separate demo/staging server | Same VPS as pilot schools / separate KVM 1 | Same VPS during the pilot, separate before real schools | [architecture](architecture.md) |
| D7 | Server data-centre region | India / other | India (latency, and to keep school data in-country). Fixed after ordering | [Hostinger KVM plan](hostinger-kvm.md) |
| D8 | Image registry | GitHub Container Registry / build on server | GitHub Container Registry, images tagged by release | [rollout runbook](../runbooks/rollout-release.md) |
| D9 | Data retention for backups | e.g. 7 daily, 4 weekly, 12 monthly | As suggested | [ADR 0004](decisions/0004-backups-restic-s3.md) |
| D10 | Commercial wording | Keep "separate installation" / reword to "separate database and application per school" | Reword, as design B shares the server and PostgreSQL | `school-erp-go-to-market` sales material |
| D11 | Use vps-forge | Adopt / borrow ideas only / skip | Evaluate on a disposable VPS first (already decided) | [ADR 0002](decisions/0002-tooling-choices.md) |
| D12 | Hostinger daily backups add-on | Buy / weekly only | Weekly only during the pilot; restic is primary | [Hostinger KVM plan](hostinger-kvm.md) |
| D13 | Hostinger malware scanner (Monarx) | Detection only / paid cleanup licence / none | Detection only, reviewed weekly | [Hostinger KVM plan](hostinger-kvm.md) |
| D14 | Billing term for the pilot KVM | Monthly / 12 / 24 months | Shortest term you would not regret if the pilot stops; note renewal price | [Hostinger KVM plan](hostinger-kvm.md) |

When a decision is made, record it in the matching ADR and move it to the table below.

## Decided

| # | Decision | Outcome | Date |
|---|---|---|---|
| — | Production stays on Railway during the pilot | Yes | 2026-11-16 |
| — | Database engine | PostgreSQL (not MySQL/MariaDB) | 2026-11-16 |
| — | Reverse proxy | Caddy | 2026-11-16 |
| — | Monitoring level | Lightweight (Uptime Kuma + host checks + Telegram) | 2026-11-16 |
