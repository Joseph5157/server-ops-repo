# ADR 0003: Stay on PostgreSQL

- Status: **Proposed** (owner direction 2026-11-16)
- Date: 2026-11-16

## Context

MySQL or MariaDB was suggested as an alternative for efficiency.

## Findings in `school-erp`

- 12 migration files install **PL/pgSQL triggers** that enforce append-only rules (attendance, results, guardian access, teacher assignments).
- About **28 conditional unique constraints** (`UniqueConstraint(condition=...)`), such as one active enrollment per student. MySQL and MariaDB do not support these; Django skips them silently, so duplicates could appear.
- About 46 check constraints that would need re-testing.
- The test suite is verified on PostgreSQL.

## Decision

Keep PostgreSQL. Use **PostgreSQL 16**, the version `school-erp` CI and every phase closeout are tested on (`.github/workflows/ci.yml`). Confirm Railway production matches. Upgrading the major version is a separate, deliberate change: test it in `school-erp` CI first.

## Consequences

- No application rewrite.
- PostgreSQL rolls back failed migrations completely (transactional DDL); MySQL/MariaDB do not.
- Efficiency comes from tuning instead: `shared_buffers` ~2 GB on KVM 2, sensible `max_connections`, PgBouncer (session mode) if connections grow, and indexes on the fastest-growing tables.
