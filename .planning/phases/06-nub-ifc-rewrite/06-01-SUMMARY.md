---
phase: 06-nub-ifc-rewrite
plan: 01
subsystem: api
tags: [nub, ifc, postmessage, wire-format, nip-5d]

# Dependency graph
requires:
  - phase: 02-nub-signer-demotion
    provides: "NUB-IFC canonical name established; NUB-IPC references replaced"
  - phase: 04-nub-relay-rewrite
    provides: "Validated wire protocol pattern (domain.action, id/subId field conventions)"
  - phase: 03-nub-storage-rewrite
    provides: "Validated document structure pattern (header, API Surface, Wire Protocol, Shell Behavior, Security)"
provides:
  - "NUB-IFC spec rewritten with ifc.* wire format (ifc.emit, ifc.subscribe, ifc.unsubscribe, ifc.event)"
  - "Sender identity via MessageEvent.source -> dTag (no Schnorr signing)"
  - "PR #5 (nub-ipc branch) updated with rewritten spec"
affects: [07-nub-pipes-rewrite]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "ifc.emit is fire-and-forget (no id field, no result message)"
    - "ifc.subscribe uses id for correlation; ifc.subscribe.result echoes id"
    - "ifc.event is shell-initiated delivery with topic + sender (dTag) fields"

key-files:
  created: []
  modified:
    - "NUB-IFC.md (on nub-ipc branch)"

key-decisions:
  - "ifc.emit is fire-and-forget — no correlation id, no result message, matching plan D-01 semantics"
  - "Sender identity in ifc.event uses dTag string (napplet type identifier), not pubkey — shell-enforced via MessageEvent.source"
  - "Topic conventions (shell:*, napplet:*, domain:*) are advisory — shell routes by topic match, not prefix parsing"
  - "Sender exclusion: shell MUST NOT deliver ifc.event back to emitting napplet (prevents echo loops)"

patterns-established:
  - "Fire-and-forget pattern: no id field on message, no corresponding result type"
  - "Shell-initiated delivery pattern: no id field, uses domain-specific routing field (topic) instead"

requirements-completed: [SPEC-05]

# Metrics
duration: 2min
completed: 2026-04-07
---

# Phase 06 Plan 01: NUB-IFC Rewrite Summary

**NUB-IFC rewritten with ifc.emit/subscribe/unsubscribe/event wire format, replacing kind 29003 signed events with shell-enforced MessageEvent.source identity**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-04-07T11:24:11Z
- **Completed:** 2026-04-07T11:25:39Z
- **Tasks:** 2
- **Files modified:** 1 (NUB-IFC.md on nub-ipc branch)

## Accomplishments

- Rewrote NUB-IFC.md from scratch following validated NUB-STORAGE/NUB-RELAY document structure
- Dropped all NIP-01 crypto (kind 29003, IFC_PEER, Schnorr signatures, delegated session keys) from napplet-facing API
- Implemented four ifc.* message types: emit (fire-and-forget), subscribe/result, unsubscribe, event (shell delivery)
- Preserved topic conventions (shell:*, napplet:*, domain:*) as advisory routing hints
- Force-pushed nub-ipc branch; PR #5 updated and remains OPEN

## Task Commits

Each task was committed atomically:

1. **Task 1: Rewrite NUB-IFC.md spec** - (file written, committed as part of Task 2 branch operation)
2. **Task 2: Force-push to nub-ipc branch and verify PR #5** - `0e0819d` (feat: rewrite NUB-IFC ifc.* typed messages)

## Files Created/Modified

- `NUB-IFC.md` (nub-ipc branch) - Complete rewrite with ifc.* wire format, SPEC.md-aligned sender identity model

## Decisions Made

- `ifc.emit` is fire-and-forget with no `id` field — matches D-01 semantics from the plan; no acknowledgment needed for pub/sub broadcast
- Sender identity in `ifc.event` uses `dTag` string per SPEC.md identity model, not pubkey — shell-enforced via `MessageEvent.source`
- `ifc.subscribe.result` confirms registration and MAY carry `error` field for ACL rejections — matches error pattern from NUB-STORAGE
- Topic conventions are explicitly described as advisory — shell routes by match, not prefix parsing

## Deviations from Plan

None — plan executed exactly as written. The NUB-IFC.md file existed as untracked on master when switching branches; moved to /tmp temporarily before checkout, which is the standard checkout procedure when untracked files would be overwritten (not a deviation, just a normal git workflow step).

## Issues Encountered

Minor: `git checkout nub-ipc` aborted because NUB-IFC.md was untracked on master and would be overwritten. Resolved by copying to /tmp, switching branches, then copying back. No functional impact.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- NUB-IFC spec complete and PR #5 updated
- Pattern established: fire-and-forget messages have no `id` field; shell-initiated deliveries use domain routing fields instead of correlation ids
- Ready for Phase 07: NUB-PIPES rewrite (final interface spec)

---
*Phase: 06-nub-ifc-rewrite*
*Completed: 2026-04-07*
