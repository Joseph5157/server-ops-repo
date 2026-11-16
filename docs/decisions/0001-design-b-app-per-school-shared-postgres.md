# ADR 0001: One app container per school, shared PostgreSQL with one database per school

- Status: **Proposed**
- Date: 2026-11-16
- Scope: pilot VPS; Railway production unchanged

## Context

Everclass ERP (`school-erp`) is a Django 5.2 + PostgreSQL monolith. Today each school is a separate Railway project (app + PostgreSQL). We want to host several schools more efficiently on a Hostinger KVM VPS without weakening isolation.

The application assumes **one school per database**: `School.objects.first()` is used in several places, no table carries a school ID, and 12 migration files install PL/pgSQL triggers that enforce "never delete, never rewrite history" rules.

## Options considered

| | Option | Code change | Isolation | Server efficiency | Main risk |
|---|---|---|---|---|---|
| A | Full stack per school (app + own Postgres) | None | Strongest | Lowest (~8–10 schools per KVM 2) | Many Postgres instances to tune and back up |
| **B** | **App per school, shared Postgres, database per school** | **None** | **Strong (separate database and login)** | **Good (~10–18 per KVM 2)** | **Shared database server load** |
| C | One app, database per school | DB router + middleware | Strong | Better | Custom per-request routing; one deploy hits all |
| D | One app, schema per school (django-tenants) | Moderate | Good | Better | `search_path` handling; one deploy hits all |
| E | One app, shared tables with `school_id` (django-multitenant) | Rewrite ~45 models | Weakest (per query) | Best | Data leak from one missed filter |

## Decision (proposed)

Adopt **B** for the pilot:

- One Everclass container per school, all from the same image, each with its own `.env`.
- One PostgreSQL server per VPS; each school has its own database and its own login that can connect only to that database (`pg_hba.conf` and `REVOKE CONNECT ... FROM PUBLIC`).
- One Caddy routes each school's domain to its container.

## Consequences

Good:

- No application code changes; the existing test suite stays valid.
- Releases can roll out one school at a time, so a bad migration affects one school first.
- One school can be backed up, restored, paused or moved independently.
- Keeps the commercial promise of a separate application and database per school (wording to adjust; see open decision D10).

Costs:

- Memory grows with each school (~180–400 MB per app container).
- One PostgreSQL server is a shared point of failure and shared load (the morning attendance rush hits every school at once).
- Connections grow with schools × workers; may need PgBouncer in session mode.

## When to revisit

- More than ~30 schools across servers, or running many containers becomes the main operating cost: evaluate D (schema per school).
- Hundreds of small schools: evaluate E.

## Verification

All items in the [pilot test plan](../pilot-test-plan.md) pass, especially the isolation, morning-peak and single-school restore tests.
