# Architecture (pilot, design B)

Status: draft, based on [ADR 0001](decisions/0001-design-b-app-per-school-shared-postgres.md).

## Overview

```
                       Internet
                          │  80/443 only
   ┌──────────────────────┼──────────────────────────────┐
   │ Hostinger KVM VPS (Ubuntu LTS, Asia/Kolkata)        │
   │                      │                              │
   │                ┌─────┴─────┐                        │
   │                │   Caddy   │  HTTPS, routes domains │
   │                └─────┬─────┘                        │
   │        edge network  │                              │
   │   ┌──────────┬───────┴──┬──────────┐                │
   │   │ school-a │ school-b │ school-c │  Everclass     │
   │   │  web     │  web     │  web     │  (same image,  │
   │   │ +jobs    │ +jobs    │ +jobs    │   own .env)    │
   │   └────┬─────┴────┬─────┴────┬─────┘                │
   │        db network │          │                      │
   │            ┌──────┴──────────┴─┐                    │
   │            │ PostgreSQL        │  no public port    │
   │            │ db_a  db_b  db_c  │  login per school  │
   │            └──────┬────────────┘                    │
   │                   │                                 │
   │   backup (restic) │   Uptime Kuma, host checks      │
   └───────────────────┼─────────────────────────────────┘
                       ▼
          S3-compatible bucket (encrypted)     Telegram alerts
```

## Components

| Component | One per | Notes |
|---|---|---|
| Ubuntu LTS + DevSec hardening | server | UFW allows 22 (restricted), 80, 443 |
| Docker Engine + Compose | server | |
| Caddy | server | Routes from `schools/<server>.yml` ([ADR 0005](decisions/0005-caddy-reverse-proxy.md)) |
| PostgreSQL | server | One database and one login per school ([ADR 0003](decisions/0003-stay-on-postgresql.md)) |
| Everclass `web` container | school | Gunicorn; image `ghcr.io/joseph5157/school-erp:<tag>` |
| Scheduled jobs | school | `purge_security_events`, `expire_import_batches`; `refresh_demo_school` on demo only |
| Backup job | server | restic, per-school dumps ([ADR 0004](decisions/0004-backups-restic-s3.md)) |
| Monitoring | server | Uptime Kuma + host checks + external check ([ADR 0006](decisions/0006-lightweight-monitoring.md)) |

## Request flow

1. A teacher opens `school-a.<domain>`.
2. Caddy terminates HTTPS and forwards to `school-a`'s `web` container only.
3. School A's container reads its own `.env`: its own secret key and database login.
4. The login `app_school_a` can connect only to `db_school_a`; PostgreSQL rejects any other database.
5. The existing Django authorization (roles, teacher assignments, guardian grants) applies inside that school.

## Isolation layers

| Layer | Prevents |
|---|---|
| Separate container and `.env` per school | One school's process reading another's secrets |
| Separate database login, `pg_hba.conf`, `REVOKE CONNECT FROM PUBLIC` | One school's app opening another school's database |
| No public PostgreSQL port | Direct database access from the internet |
| Django authorization inside each school | Teacher or guardian reaching records they are not granted (review with Sentry `django-access-review`) |
| Memory limits per container | One school's heavy day (report cards, PDFs) starving others |

## Environment per school

Variables `school-erp` reads (values are placeholders here):

| Variable | Per school value |
|---|---|
| `DJANGO_SECRET_KEY` | Unique, generated |
| `DJANGO_DEBUG` | `False` |
| `DJANGO_ALLOWED_HOSTS` | School domain |
| `DJANGO_TRUSTED_PROXIES` | Caddy network range |
| `PORTAL_BASE_URL` | `https://<school domain>` |
| `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT` | `db_<slug>`, `app_<slug>`, generated, `postgres`, `5432` |
| `TELEGRAM_ERROR_BOT_TOKEN`, `TELEGRAM_ERROR_CHAT_ID` | Shared operator bot (both or neither) |
| `EMAIL_*`, `DJANGO_ADMINS` | Optional |

## What changes from Railway

| Railway gave us | Replacement here |
|---|---|
| Automatic builds | Image built in CI and pushed to GitHub Container Registry |
| Managed PostgreSQL and backups | Our PostgreSQL + restic backups + restore tests |
| Logs in the dashboard | Docker logs with rotation |
| Deploy notifications | Uptime Kuma + Telegram |
| Scheduled jobs | Per-school cron or scheduler container |
| HTTPS | Caddy |

## Prerequisites outside this repository

- `school-erp`: a production Dockerfile (WeasyPrint libraries, Noto fonts), a health endpoint or use of the login page, and CI that publishes tagged images.
- Confirm whether any uploaded files (for example the school logo) are stored on disk; if so, add a volume and back it up.
