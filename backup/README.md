# backup/ (planned)

Backup, restore and verification scripts. Nothing here has been written yet. Design: [ADR 0004](../docs/decisions/0004-backups-restic-s3.md).

```
backup/
├── backup-all.sh          # pg_dump each school database separately → restic
├── restore-school.sh      # restore one school's database into a NEW database name (pg_restore --no-owner --role=app_<slug>)
├── verify-latest.sh       # restore latest snapshot to a scratch database and run checks
├── backup.env.example     # RESTIC_REPOSITORY, S3 keys, RESTIC_PASSWORD_FILE (placeholders)
└── README.md
```

## What gets backed up

| Item | How | Why |
|---|---|---|
| Each school database | `pg_dump -Fc` per database | Restore one school without touching others |
| PostgreSQL roles | `pg_dumpall --roles-only` | Recreate school logins |
| School `.env` files | restic, encrypted | Needed to bring a school back |
| Caddy data (certificates) | restic | Faster recovery; certificates can also be re-issued |

School logos are stored in the database (`SchoolLogo.image`), so the database dump already includes them. No media files are stored on disk today.

## Rules

- Dumps go straight into restic; no unencrypted dump is left on disk.
- A run that fails sends a Telegram alert.
- Monitoring alerts if the newest snapshot is older than 26 hours.
- `restic forget --prune` is tier 3: owner runs it with the owner key, kept off the server. The server key can delete (restic needs to remove lock files), so the bucket keeps old versions for 30+ days to undo any deletion ([ADR 0004](../docs/decisions/0004-backups-restic-s3.md)).
- Restores always go to a new database name first, never over the live one (exception: rebuilding a lost server, where it does not exist yet).

## Inspiration (patterns only, not adopted)

- `restic-pg-dump`: dumps every database separately into restic.
- `tiredofit/docker-db-backup`: multi-database scheduled backups with S3.

We prefer a short, reviewed script of our own. See [references](../docs/references.md).
