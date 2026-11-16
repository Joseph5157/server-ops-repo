# ADR 0008: Approval tiers for AI agents

- Status: **Proposed**
- Date: 2026-11-16

## Context

Claude Code and Codex will review configuration and carry out controlled operations, possibly with the Hostinger MCP connected. The servers hold children's records, so an agent mistake must not be able to lose data or widen access.

## Decision

Every operation falls into one of four tiers, defined in [AGENTS.md](../../AGENTS.md):

| Tier | Meaning | Agent may |
|---|---|---|
| 0 | Read | Do freely |
| 1 | Reversible, no data touched | Do after stating the plan |
| 2 | Controlled change to a live server or school | Do only after explicit owner approval, following a runbook |
| 3 | Destructive or access-removing | Never; prepare commands for the owner |

Agents follow the `server-ops` skill, which maps requests to runbooks and marks each runbook step with its tier.

## Consequences

- Runbooks label every step with its tier.
- The Hostinger MCP token should not allow more than needed; destructive VPS actions remain owner-only regardless of token scope.
- Pull requests from agents are drafts until a person reviews them.
