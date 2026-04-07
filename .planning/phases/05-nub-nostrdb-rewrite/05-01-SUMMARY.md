---
phase: 05-nub-nostrdb-rewrite
plan: "01"
subsystem: specs
tags: [nostrdb, postmessage, nub, nostr, wire-protocol, typed-messages]

requires:
  - phase: 04-nub-relay-rewrite
    provides: subscription/event delivery pattern (relay.event, subId, Subscription interface) that nostrdb mirrors

provides:
  - NUB-NOSTRDB.md on nub-nostrdb branch using nostrdb.* typed messages
  - PR #4 updated with nostrdb.* wire format summary and correct namespace

affects:
  - 06-nub-ifc-rewrite
  - 07-nub-pipes-rewrite

tech-stack:
  added: []
  patterns:
    - "nostrdb.sub.event / nostrdb.sub.eose for subscription push delivery (disambiguated from by-id lookup)"
    - "externally-authored event caching: add() accepts pre-signed NostrEvent, napplet never signs"
    - "nostrdb.event for by-id lookup uses eventId field to avoid field name collision with event payload"

key-files:
  created: []
  modified:
    - NUB-NOSTRDB.md (on nub-nostrdb branch)

key-decisions:
  - "nostrdb.sub.event and nostrdb.sub.eose used for subscription push (not nostrdb.event) to avoid naming collision with by-id lookup"
  - "nostrdb.event by-id lookup uses eventId field (hex string) to distinguish from event payload field in result"
  - "add() documents externally-authored pattern explicitly: napplet does NOT sign, events come from relay.event"

patterns-established:
  - "Sub-namespace pattern: domain.sub.* for subscription push messages when domain.* is already used for a distinct operation"

requirements-completed: [SPEC-04]

duration: 2min
completed: "2026-04-07"
---

# Phase 5 Plan 1: NUB-NOSTRDB Rewrite Summary

**NUB-NOSTRDB rewritten to use nostrdb.* typed messages with 14-type wire protocol, subscription delivery via nostrdb.sub.event/eose, and externally-authored event caching via add()**

## Performance

- **Duration:** 2 min
- **Started:** 2026-04-07T11:17:33Z
- **Completed:** 2026-04-07T11:20:19Z
- **Tasks:** 2
- **Files modified:** 1 (NUB-NOSTRDB.md on nub-nostrdb branch)

## Accomplishments

- Rewrote NUB-NOSTRDB.md replacing kind 29006/29007 NIP-01 event wrapping with nostrdb.* typed messages
- Added complete Wire Protocol table (14 message types), JSON examples for all 7 operations, and TypeScript interface with `// via nostrdb.*` annotations
- Force-pushed nub-nostrdb branch and updated PR #4 body with nostrdb.* wire format summary

## Task Commits

Each task was committed atomically:

1. **Task 1: Rewrite NUB-NOSTRDB.md on nub-nostrdb branch** - `de4010d` (docs)
2. **Task 2: Force-push to PR branch and update PR #4 body** - no code commit (remote branch update + GitHub PR edit only)

**Plan metadata:** (final commit — see below)

## Files Created/Modified

- `NUB-NOSTRDB.md` (nub-nostrdb branch) - Complete spec rewrite using nostrdb.* wire format; 14 message types, subscription delivery via nostrdb.sub.event/eose, add() documented as externally-authored only

## Decisions Made

- Used `nostrdb.sub.event` and `nostrdb.sub.eose` for subscription push delivery (not `nostrdb.event`) to disambiguate from the by-ID lookup — the plan spec explicitly required this naming
- The `nostrdb.event` by-ID request uses `eventId` field (not `id`) to avoid collision with the `event` payload field in results
- `add()` documentation makes explicit that napplets do NOT sign events — they are externally-authored events received from relay subscriptions

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- NUB-NOSTRDB.md on nub-nostrdb branch is complete and consistent with NUB-RELAY and NUB-STORAGE patterns
- PR #4 updated and open on GitHub
- Ready for Phase 06 (NUB-IFC rewrite) — nostrdb subscription pattern established for reference
