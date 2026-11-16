# ADR 0004: Encrypted off-server backups with restic to S3-compatible storage

- Status: **Proposed** (storage provider not chosen: open decision D2)
- Date: 2026-11-16

## Context

Railway handled database backups. On a VPS we must provide our own, off the server and off the provider, and prove they restore.

## Decision

- Tool: **restic** (encrypted, deduplicated, incremental).
- Target: an **S3-compatible bucket** (Backblaze B2, Cloudflare R2 or Wasabi; D2).
- Each school database dumped **separately** with `pg_dump -Fc`, streamed into restic.
- Also back up: PostgreSQL roles, school `.env` files, Caddy data. (Logos are stored in the database; there are no media files on disk today.)
- Schedule: nightly at 01:30 IST, plus an on-demand backup before every release or database change.
- Retention (D9, proposed): 7 daily, 4 weekly, 12 monthly.
- Hostinger's own weekly VPS backups or snapshots are an extra layer, not the primary backup.

## Recovery targets (proposed)

| Target | Value |
|---|---|
| Maximum data loss (RPO) | 24 hours; lower to 1 hour later with WAL archiving if schools need it |
| Time to restore one school (RTO) | Under 1 hour |
| Time to rebuild a lost server (RTO) | Under 4 hours |

## Rules

- A backup counts only after a restore succeeds: monthly test ([runbook](../../runbooks/monthly-restore-test.md)).
- Restores go to a new database name first, never over the live one. Only exception: rebuilding a lost server ([server-lost](../../runbooks/server-lost.md)), where the database does not exist yet.
- `restic forget --prune` and deleting snapshots are tier 3.
- Two bucket keys: the **server key** can write and read but not delete (append-only), so a compromised server cannot wipe backups; the **owner's prune key** can delete and is kept off the server. Pruning runs from the owner's machine, monthly.
- The restic password lives outside the server too (ADR 0007). If it is lost, the backups cannot be opened.

## Consequences

- Need a bucket, an access key limited to that bucket, and a monitoring check for backup age.
- Bucket region should be acceptable for school data (consider in-country storage requirements).
