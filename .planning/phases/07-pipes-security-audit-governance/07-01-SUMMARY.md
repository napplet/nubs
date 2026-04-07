---
phase: 07-pipes-security-audit-governance
plan: 01
subsystem: api
tags: [nub, ifc, channels, pipes, postmessage, wire-format]

# Dependency graph
requires:
  - phase: 06-nub-ifc-rewrite
    provides: "NUB-IFC spec rewritten with ifc.* wire format on nub-ipc branch"
provides:
  - "PR #6 (NUB-PIPES) closed with explanatory comment"
  - "NUB-IFC spec extended with ifc.channel.* message types (9 wire protocol types)"
  - "auth-on-open channel model documented (validates once, no per-message checking)"
  - "Channels vs Topics section explaining design tradeoffs"
affects: [07-nub-security-audit]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "auth-on-open model: shell validates target once on ifc.channel.open, messages flow without per-message checks"
    - "Channel IDs are opaque (shell-assigned) — prevents enumeration/hijacking"
    - "ifc.channel.broadcast is fire-and-forget to all open channel peers"
    - "ifc.channel.closed is shell-initiated (peer destroyed, graceful close, ACL revocation)"

key-files:
  created: []
  modified:
    - "NUB-IFC.md (on nub-ipc branch) — added ifc.channel.* API, wire protocol, examples, shell behavior, security"

key-decisions:
  - "NUB-PIPES eliminated as separate spec — channel functionality merged into NUB-IFC as ifc.channel.* types"
  - "Auth-on-open model: shell validates channel target once at open time, no per-message validation after that"
  - "Channel IDs are opaque (shell-assigned) — napplets cannot enumerate or guess other channels' IDs"
  - "ifc.channel.broadcast excludes the sender (consistent with topic ifc.event sender exclusion)"

requirements-completed: [SPEC-06]

# Metrics
duration: 2min
completed: 2026-04-07
---

# Phase 07 Plan 01: Close NUB-PIPES and Add IFC Channels Summary

**NUB-PIPES PR #6 closed and absorbed into NUB-IFC as ifc.channel.* message types with auth-on-open semantics and 9 wire protocol types**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-04-07T11:40:38Z
- **Completed:** 2026-04-07T11:42:32Z
- **Tasks:** 2
- **Files modified:** 1 (NUB-IFC.md on nub-ipc branch, plus GitHub operations)

## Accomplishments

- Closed PR #6 (NUB-PIPES) with explanatory comment pointing to the NUB-IFC update
- Deleted remote and local `nub-pipes` branches
- Added point-to-point channel API surface to NUB-IFC: `channel.open`, `channel.list`, `channel.broadcast`, plus `ChannelHandle` with `emit`, `on`, `close`
- Added 9 `ifc.channel.*` wire protocol message types to the spec
- Added channel examples for all operations (open, emit, event, broadcast, list, close, error)
- Added "Channels vs Topics" section explaining design tradeoffs and auth-on-open model
- Added channel-specific Shell Behavior MUST/MAY clauses (11 requirements)
- Added channel Security Considerations (opaque IDs, broadcast warning, trust boundary)
- Removed "Relationship to NUB-PIPES" section and all NUB-PIPES references (0 remaining)
- Force-pushed nub-ipc branch; PR #5 updated and remains OPEN

## Task Commits

Each task was committed atomically:

1. **Task 1: Close NUB-PIPES PR #6 and delete remote branch** — GitHub operations only (no file commit needed); PR #6 closed, remote and local nub-pipes branches deleted
2. **Task 2: Add channel support to NUB-IFC spec on nub-ipc branch** — `6d220b6` (feat(07-01): add ifc.channel.* support (absorbs NUB-PIPES))

## Files Created/Modified

- `NUB-IFC.md` (nub-ipc branch) — Extended with ifc.channel.* API surface, 9 wire protocol types, examples, Channels vs Topics section, channel shell behavior, channel security considerations; all NUB-PIPES references removed

## Decisions Made

- NUB-PIPES eliminated as separate spec — its point-to-point channel concepts are a natural extension of NUB-IFC, not a separate interface
- Auth-on-open model: shell validates target dTag + ACL once at `ifc.channel.open`; subsequent messages flow without per-message validation — same trust boundary as topic IFC
- Channel IDs are opaque (shell-assigned) — napplets cannot enumerate or guess other channels' IDs, preventing channel hijacking
- `ifc.channel.broadcast` excludes the sender, consistent with the existing sender exclusion rule on `ifc.event`

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

None — all channel operations are fully specified in the wire protocol.

## Self-Check: PASSED

- NUB-IFC.md modified on nub-ipc branch: FOUND (6d220b6 confirmed)
- PR #6 state CLOSED: CONFIRMED
- PR #5 state OPEN: CONFIRMED
- ifc.channel.open occurrences >= 3: FOUND (11)
- ifc.channel.emit occurrences >= 2: FOUND (5)
- ifc.channel.event occurrences >= 2: FOUND (4)
- ifc.channel.broadcast occurrences >= 2: FOUND (6)
- ifc.channel.list occurrences >= 2: FOUND (6)
- ifc.channel.close occurrences >= 2: FOUND (11)
- NUB-PIPES occurrences == 0: CONFIRMED (0)
- auth-on-open occurrences >= 1: FOUND (4)
- ifc.channel total occurrences >= 20: FOUND (42)
