---
gsd_state_version: 1.0
milestone: v0.1.0
milestone_name: milestone
status: verifying
stopped_at: Completed 06-nub-ifc-rewrite/06-01-PLAN.md
last_updated: "2026-04-07T11:29:00.791Z"
last_activity: 2026-04-07
progress:
  total_phases: 7
  completed_phases: 5
  total_plans: 5
  completed_plans: 5
  percent: 14
---

# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-04-07)

**Core value:** Napplet authors never touch crypto. They subscribe(), publish(), query().
**Current focus:** Phase 04 — nub-relay-rewrite

## Current Position

Phase: 7
Plan: Not started
Status: Phase complete — ready for verification
Last activity: 2026-04-07

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
| Phase 02-nub-signer-demotion P01 | 2 | 2 tasks | 2 files |
| Phase 03-nub-storage-rewrite P01 | 15 | 2 tasks | 1 files |
| Phase 04-nub-relay-rewrite P01 | 2 | 2 tasks | 1 files |
| Phase 05-nub-nostrdb-rewrite P01 | 2 | 2 tasks | 1 files |
| Phase 06-nub-ifc-rewrite P01 | 2 | 2 tasks | 1 files |

## Accumulated Context

### Decisions

- SPEC.md uses `{ "type": "domain.action", ...payload }` wire format instead of NIP-01 events — this is a larger shift than originally planned, meaning kind numbers are shell implementation details only
- Identity is `MessageEvent.source` based, assigned at iframe creation — no negotiation
- Napplets have no access to `window.nostr` — all signing proxied through shell
- [Phase 02-nub-signer-demotion]: NUB-SIGNER closed entirely — NIP-07 window.nostr is a SPEC.md core requirement (shells MUST provide it), so there is nothing left for a separate spec to define
- [Phase 02-nub-signer-demotion]: NUB-IFC is the canonical name for inter-frame communication — all NUB-IPC references replaced in README.md and CLAUDE.md
- [Phase 03-nub-storage-rewrite]: storage.* message types mirror API method names; errors are inline in result messages via error field; shell behavior describes composite key scoping outcome without prescribing internal key format
- [Phase 04-nub-relay-rewrite]: relay.publish accepts a signed NostrEvent only — napplets sign via window.nostr.signEvent(template) before publishing, one path only (D-07)
- [Phase 04-nub-relay-rewrite]: subId field distinguishes subscription-scoped messages from request/result pairs (which use id)
- [Phase 04-nub-relay-rewrite]: Shell MUST verify event signatures before relay broadcast — invalid signatures rejected with relay.publish.result ok:false
- [Phase 05-nub-nostrdb-rewrite]: nostrdb.sub.event and nostrdb.sub.eose used for subscription push to disambiguate from nostrdb.event by-id lookup
- [Phase 05-nub-nostrdb-rewrite]: nostrdb.add documents externally-authored pattern: napplets do NOT sign events, they cache relay.event-received events
- [Phase 06-nub-ifc-rewrite]: ifc.emit is fire-and-forget with no id field — no acknowledgment needed for pub/sub broadcast
- [Phase 06-nub-ifc-rewrite]: Sender identity in ifc.event uses dTag (not pubkey) — shell-enforced via MessageEvent.source mapping
- [Phase 06-nub-ifc-rewrite]: Topic conventions (shell:*, napplet:*, domain:*) are advisory — shell routes by topic match, not prefix parsing

### Pending Todos

None yet.

### Blockers/Concerns

- 6 existing DRAFT PRs for NUB specs will need reconciliation once rewrites are complete
- NUB spec rewrites must adopt new `domain.action` wire format (not NIP-01 events)

## Session Continuity

Last session: 2026-04-07T11:26:41.416Z
Stopped at: Completed 06-nub-ifc-rewrite/06-01-PLAN.md
Resume file: None
