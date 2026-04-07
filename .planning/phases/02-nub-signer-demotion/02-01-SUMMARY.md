---
phase: 02-nub-signer-demotion
plan: 01
subsystem: governance
tags: [github, spec, cleanup, naming]

# Dependency graph
requires: []
provides:
  - PR #1 (NUB-SIGNER) closed with explanatory comment on GitHub
  - Remote branch nub-signer deleted from origin
  - README.md registry table updated to 5 rows (no NUB-SIGNER)
  - CLAUDE.md updated with NUB-IFC canonical name, NUB-SIGNER references removed
affects: [03-nub-relay-rewrite, 04-nub-storage-rewrite, 05-nub-nostrdb-rewrite, 06-nub-ifc-rewrite, 07-nub-pipes-rewrite]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "NUB-SIGNER is not a NUB — window.nostr is a core SPEC.md invariant provided by the shell"
    - "Canonical inter-frame communication spec name is NUB-IFC (not NUB-IPC)"

key-files:
  created:
    - CLAUDE.md
  modified:
    - README.md

key-decisions:
  - "NUB-SIGNER closed entirely — NIP-07 window.nostr is a SPEC.md core requirement (shells MUST provide it), so there is nothing left for a separate spec to define"
  - "NUB-IFC is the canonical name for inter-frame communication — all NUB-IPC references replaced"

patterns-established:
  - "Signing is shell-internal: napplets never touch window.nostr, no NUB spec for it"
  - "Registry table in README.md is the single source of truth for active NUB specs"

requirements-completed: [SPEC-03, SPEC-07]

# Metrics
duration: 2min
completed: 2026-04-07
---

# Phase 02: NUB-SIGNER Demotion Summary

**NUB-SIGNER PR closed and remote branch deleted; NUB-IFC canonicalized across README.md and CLAUDE.md, collapsing the 6-spec registry to 5**

## Performance

- **Duration:** 2 min
- **Started:** 2026-04-07T10:33:54Z
- **Completed:** 2026-04-07T10:35:54Z
- **Tasks:** 2
- **Files modified:** 2 (README.md, CLAUDE.md)

## Accomplishments

- Closed PR #1 (NUB-SIGNER) on GitHub with a detailed comment explaining the two-layer architecture rationale
- Deleted remote branch `nub-signer` from origin — no stale branches remain
- Removed NUB-SIGNER row from README.md registry table (5 specs remain: RELAY, STORAGE, NOSTRDB, IFC, PIPES)
- Replaced all NUB-IPC references with NUB-IFC in both CLAUDE.md and README.md prose

## Task Commits

Each task was committed atomically:

1. **Task 1: Close NUB-SIGNER PR and delete remote branch** - GitHub-only operations (no local files changed; no commit needed)
2. **Task 2: Update README.md and CLAUDE.md** - `0a07457` (chore)

**Plan metadata:** (final docs commit below)

## Files Created/Modified

- `/home/sandwich/Develop/nubs/README.md` - Removed NUB-SIGNER registry row; changed "IPC" to "IFC" in prose
- `/home/sandwich/Develop/nubs/CLAUDE.md` - Removed NUB-SIGNER from spec list, example, and PR table; renamed NUB-IPC to NUB-IFC

## Decisions Made

- NUB-SIGNER is not a NUB at all — NIP-07 `window.nostr` is mandated by SPEC.md as a core shell invariant. A dedicated spec would duplicate what is already required. Closed without replacement.
- NUB-IFC is the locked canonical name. The `nub-ipc` remote branch remains (stale name artifact) but spec content already says NUB-IFC. That branch rename is deferred to Phase 6 (IFC rewrite) per D-05.

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- NUB-SIGNER is fully removed from the project; downstream phases (3-7) can proceed without encountering stale NUB-SIGNER references in README.md or CLAUDE.md
- Remaining stale NUB-SIGNER references in other spec branches (NUB-RELAY's signEvent() call, NUB-PIPES' signing comparison) will be cleaned up naturally during their respective phase rewrites per D-06
- The `nub-ipc` remote branch name is a known stale artifact — to be handled in Phase 6 (NUB-IFC rewrite)

---
*Phase: 02-nub-signer-demotion*
*Completed: 2026-04-07*
