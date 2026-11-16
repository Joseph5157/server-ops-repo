# ADR 0002: Tooling choices

- Status: **Proposed** (records the owner's stated decisions of 2026-11-16)
- Date: 2026-11-16

## Decision

| Tool | Decision | Conditions |
|---|---|---|
| Hostinger official MCP (`hostinger-api-mcp`) | Use | Runs locally in Claude Code or Claude Desktop with a Hostinger API token. Token scoped as narrowly as Hostinger allows. VPS rebuild, delete, OS reinstall and snapshot restore are tier 3: owner confirms each. Token never committed. |
| Hostinger Agent Skills | Use | Read the skill source before installing; pin the version. |
| DevSec Ansible (`devsec.hardening`) | Use after configuration review | Review every variable that changes SSH, PAM, sysctl and kernel modules. Confirm key login and Hostinger console access before applying SSH hardening. |
| vps-forge | Evaluate on a disposable VPS first | Record what it changes; adopt only the parts we can reproduce in our own Ansible. |
| Docker Compose | Preferred starting deployment | One platform stack and one stack per school (ADR 0001). |
| Claude Code / Codex | Reviews, configuration and controlled operations | Bound by the approval tiers in `AGENTS.md`. |
| Sentry `django-access-review` skill | Use (in `school-erp`) | For IDOR review inside a school: teacher to another section, guardian to another child, revoked access. |
| Claude Code Django Toolkit | Optional | Second opinion on Django security; small user base, read the source first. |
| Codex multi-tenancy skill | Reference only | Borrow cross-tenant test ideas; review scripts before running. |
| django-multitenant | Not adopted | Shared-table design (option E); its published compatibility table lists Django up to 4.2. |

## Rationale

Prefer tools we can read, pin and reproduce. Anything that changes a server must be expressible in this repository so that a rebuilt server is identical.

## Consequences

- A `requirements.yml` pins Ansible collections and roles.
- Third-party skills are recorded in `docs/references.md` with the version reviewed.
