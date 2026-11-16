# Security model

Status: draft.

## What we protect

Student, guardian and staff records; attendance; results; guardian access grants; credentials. Many records concern children.

## Threats and controls

| Threat | Control |
|---|---|
| SSH brute force or stolen password | Key-only SSH, no root login (DevSec), UFW, optional source-IP restriction, fail2ban |
| Unpatched OS | Unattended security upgrades; monthly review |
| Exposed database | No public PostgreSQL port; Docker internal network only |
| One school reading another's data | Separate containers, `.env` files and database logins; `REVOKE CONNECT ON DATABASE ... FROM PUBLIC` |
| Teacher or guardian over-reaching inside a school | Existing Django authorization; review with Sentry `django-access-review`; IDOR tests |
| Secret leak through Git | No plain secrets in Git (ADR 0007); `.gitignore`; review every PR |
| Lost backups or keys | Off-server encrypted backups; two key holders (ADR 0007) |
| Ransomware or a compromised server deleting backups | Bucket versioning with old versions kept 30+ days, so deletions by the server key can be undone; a separate owner key, kept off the server, for pruning. A plain no-delete key is not used because restic must delete its lock files ([ADR 0004](decisions/0004-backups-restic-s3.md)). |
| AI agent mistake | Approval tiers (ADR 0008) |
| Hostinger account takeover | Two-factor authentication on Hostinger and GitHub; API token kept off servers |
| Noisy neighbour (one school overloading) | Container memory limits; monitoring |

## Isolation checks (automatable)

1. From `school-a`'s container, connecting to `db_school_b` with `app_school_a` fails.
2. `school-a`'s container cannot read `/srv/schools/school-b/.env`.
3. PostgreSQL port 5432 is closed from the internet (`nmap` from outside).
4. Only 22, 80 and 443 are open publicly.
5. Each school's `DJANGO_SECRET_KEY` is different.
6. Inside one school: a teacher URL for another section's student returns 403/404; a guardian URL for an ungranted child returns 403/404.

## Data protection notes

- Keep servers and backups in a region acceptable for school data (open decision D7, D2).
- Restores for testing use fictional demo data or a scratch database destroyed after the test.
- Never copy real school data to a laptop or into this repository.
