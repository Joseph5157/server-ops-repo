# Capacity planning

Status: **estimates, not measurements.** The pilot must replace these numbers with measured ones.

## Pilot server

Hostinger KVM 2: 2 vCPU, 8 GB RAM, 100 GB NVMe.

## Memory budget (design B)

| Item | Estimate |
|---|---|
| OS, Docker, Caddy, Uptime Kuma, backup job | ~1.0 GB |
| PostgreSQL base (`shared_buffers` ~2 GB plus overhead) | ~2.0 GB |
| PostgreSQL per school database (connections, caches) | ~50 MB |
| Everclass app, 2 Gunicorn workers | ~300 MB per school |
| Everclass app, 1 worker + threads | ~180 MB per school |

| Schools | 2 workers each | 1 worker + threads |
|---|---|---|
| 8 | ~5.8 GB, comfortable | ~4.8 GB |
| 12 | ~7.2 GB, tight | ~5.8 GB, comfortable |
| 16 | too much | ~6.9 GB, tight |
| 20 | too much | ~7.6 GB, use KVM 4 |

**Working limits:** about 10–12 schools comfortably, about 15–18 with tuning, use a KVM 4 or a second server for 20.

## The real limit: CPU at peak times

Every school takes attendance in the same morning window, and report-card days bring PDF generation (WeasyPrint is memory- and CPU-heavy). Two vCPUs are shared by all schools.

The pilot must measure:

- Requests per second and response time during a simulated 9:00–10:00 attendance rush for N schools.
- Memory peak while generating report-card PDFs for a full class.
- PostgreSQL connections and slow queries during the rush.

## Starting PostgreSQL settings for KVM 2 (to tune in the pilot)

| Setting | Start value |
|---|---|
| `shared_buffers` | 2GB |
| `effective_cache_size` | 4GB |
| `work_mem` | 16MB |
| `maintenance_work_mem` | 256MB |
| `max_connections` | 100 (PgBouncer in session mode if exceeded) |
| `wal_compression` | on |

## Disk

100 GB NVMe: OS and images ~15 GB; each school database is expected to stay small (attendance and results grow fastest). Alert at 80% use.

## Cost reference (checked 2026-11-16)

- Railway charges about $10 per GB of RAM and $20 per vCPU per month, so cost grows with every school.
- Hostinger KVM 2 in India: about ₹799/month introductory, ₹1,199/month on renewal (third-party price tracker, August 2026). Confirm on hostinger.in before buying.
