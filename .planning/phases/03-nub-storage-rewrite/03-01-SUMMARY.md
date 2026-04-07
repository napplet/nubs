---
phase: 03-nub-storage-rewrite
plan: 01
subsystem: spec
tags: [nub-storage, wire-format, postMessage, spec-rewrite]

# Dependency graph
requires:
  - phase: 01-spec-foundation
    provides: SPEC.md with NIP-5D wire format (domain.action typed messages)
provides:
  - NUB-STORAGE.md on nub-storage branch using storage.* typed messages (no kind numbers)
  - PR #3 updated with new wire format description
  - Validated wire format rewrite pattern for subsequent spec phases
affects: [04-nub-relay-rewrite, 05-nub-nostrdb-rewrite, 06-nub-ifc-rewrite, 07-nub-pipes-rewrite]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "NUB spec section order: Description -> API Surface -> Wire Protocol -> Shell Behavior -> Security Considerations"
    - "TypeScript interface block annotated with // via domain.action comments"
    - "Wire protocol table: Type | Direction | Payload fields"
    - "Inline error handling via error field in result messages (no separate error type)"
    - "shell identity scoped to (dTag, aggregateHash) without exposing internal key format"

key-files:
  created: []
  modified:
    - NUB-STORAGE.md (on nub-storage branch — rewritten spec)

key-decisions:
  - "storage.* message types mirror API method names exactly (storage.get, storage.set, storage.remove, storage.keys)"
  - "Errors are inline in result messages via error field — no separate error message type (D-04)"
  - "Shell Behavior describes composite key scoping outcome without prescribing the internal prefix format"
  - "API Surface TypeScript block annotated with // via storage.get comments bridging API to wire layer"

patterns-established:
  - "Wire Protocol section replaces Event Kinds — domain.action types replace kind numbers"
  - "Result suffix: storage.get.result, storage.set.result, etc."
  - "Correlation ID field is id in all request/response pairs"
  - "value: null in storage.get.result when key not found (localStorage semantics)"

requirements-completed: [SPEC-02]

# Metrics
duration: 15min
completed: 2026-04-07
---

# Phase 3 Plan 01: NUB-STORAGE Rewrite Summary

**NUB-STORAGE rewritten to use `storage.*` typed messages with JSON payloads, replacing kind 29003/IFC_PEER transport — API surface preserved, zero crypto references**

## Performance

- **Duration:** ~15 min
- **Started:** 2026-04-07T11:00:00Z
- **Completed:** 2026-04-07T11:15:00Z
- **Tasks:** 2
- **Files modified:** 1 (NUB-STORAGE.md on nub-storage branch)

## Accomplishments

- Rewrote NUB-STORAGE.md replacing Event Kinds/Response Tags/IFC_PEER sections with a Wire Protocol section using `storage.get`, `storage.set`, `storage.remove`, `storage.keys` typed messages
- Added concrete JSON examples for all four operations (get, set, remove, keys) plus error handling example (`quota exceeded`)
- Updated PR #3 body on GitHub to reflect new wire format with `storage.get`/`storage.set` references, removing IPC-PEER and kind 29003 mentions
- Established rewrite pattern and section ordering for all subsequent spec phases (04-07)

## Task Commits

Each task was committed atomically:

1. **Task 1: Rewrite NUB-STORAGE.md on nub-storage branch** - `55cde7d` (docs)
2. **Task 2: Force-push to PR branch and update PR body** - `55cde7d` (same commit — branch force-pushed, PR body updated via gh CLI)

**Plan metadata:** (docs commit to follow)

## Files Created/Modified

- `NUB-STORAGE.md` (nub-storage branch) — Complete rewrite: Wire Protocol section with typed message table and JSON examples; API Surface TypeScript block annotated with `// via storage.*` comments; Shell Behavior updated with scoping language that doesn't expose internal key format; Security Considerations trimmed of prescriptive internals

## Decisions Made

- TypeScript interface comments use `// via storage.get` pattern to bridge the API surface and wire protocol layers — makes the spec self-documenting at a glance
- Shell Behavior says "The shell maps each napplet's `(dTag, aggregateHash)` identity to an isolated storage namespace" rather than prescribing key prefix format — leaves implementation detail to shell authors
- Removed "validate storage key prefixes" from Security Considerations — the spec MUST enforce isolation (outcome), not prescribe the mechanism (prefix format)
- `value: null` in `storage.get.result` when key not found — explicit match to localStorage semantics documented inline

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None. The nub-storage branch was already checked out-capable; needed a stash/pop around the branch switch due to unstaged STATE.md changes on master.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- NUB-STORAGE rewrite is complete and force-pushed to origin/nub-storage (PR #3 updated)
- The wire format pattern established here (`storage.*` typed messages, `.result` suffix, inline error field, section ordering) serves as the template for all remaining spec rewrites (Phases 04-07: RELAY, NOSTRDB, IFC, PIPES)
- No blockers for Phase 04

---
*Phase: 03-nub-storage-rewrite*
*Completed: 2026-04-07*
