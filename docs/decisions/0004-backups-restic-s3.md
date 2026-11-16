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
- **Protecting backups from a compromised server.** A key that simply cannot delete does not work with restic: restic writes and removes lock files on every run, and a no-delete key fails or leaves stale locks. Use instead:
  1. **Bucket versioning** (or the provider's equivalent, such as Backblaze B2 "hide" plus a lifecycle rule) with old versions kept for at least 30 days. The server key may delete, but a deletion only hides a version, which the owner can recover. Where the provider supports **object lock** with a retention period, it can add protection, but restic does not manage locks itself, so test it before relying on it.
  2. A **separate owner key** with full rights, kept off the server, for `forget --prune`, `check` and clearing stale locks. Pruning runs from the owner's machine, monthly.
  3. Alternative if the provider cannot do versioning: an append-only restic endpoint (`rest-server --append-only` or `rclone serve restic --append-only`) on a separate small host. More moving parts; consider only if needed.
- Prove it in the pilot (test T18): with the server key, delete a snapshot's files, then recover them with the owner key.
- The restic password lives outside the server too (ADR 0007). If it is lost, the backups cannot be opened.

## Consequences

- Need a bucket, an access key limited to that bucket, and a monitoring check for backup age.
- Bucket region should be acceptable for school data (consider in-country storage requirements).
