---
gsd_state_version: 1.0
milestone: v0.1.0
milestone_name: milestone
status: planning
stopped_at: Phase 2 context gathered
last_updated: "2026-04-07T10:22:31.736Z"
last_activity: 2026-04-07 — Phase 1 completed (SPEC.md written externally), roadmap adjusted
progress:
  total_phases: 7
  completed_phases: 0
  total_plans: 0
  completed_plans: 0
  percent: 14
---

# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-04-07)

**Core value:** Napplet authors never touch crypto. They subscribe(), publish(), query().
**Current focus:** Milestone v0.1.0 — Phase 2: NUB-SIGNER Demotion (ready to plan)

## Current Position

Phase: 2 of 7 (NUB-SIGNER Demotion)
Plan: -- of -- in current phase
Status: Ready to plan
Last activity: 2026-04-07 — Phase 1 completed (SPEC.md written externally), roadmap adjusted

Progress: [█░░░░░░░░░] 14%

## Performance Metrics

**Velocity:**

- Total plans completed: 0
- Average duration: --
- Total execution time: 0 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 1 | -- | -- | -- (external) |

**Recent Trend:**

- Last 5 plans: --
- Trend: --

*Updated after each plan completion*

## Accumulated Context

### Decisions

- SPEC.md uses `{ "type": "domain.action", ...payload }` wire format instead of NIP-01 events — this is a larger shift than originally planned, meaning kind numbers are shell implementation details only
- Identity is `MessageEvent.source` based, assigned at iframe creation — no negotiation
- Napplets have no access to `window.nostr` — all signing proxied through shell

### Pending Todos

None yet.

### Blockers/Concerns

- 6 existing DRAFT PRs for NUB specs will need reconciliation once rewrites are complete
- NUB spec rewrites must adopt new `domain.action` wire format (not NIP-01 events)

## Session Continuity

Last session: 2026-04-07T10:22:31.733Z
Stopped at: Phase 2 context gathered
Resume file: .planning/phases/02-nub-signer-demotion/02-CONTEXT.md
