---
phase: 04-nub-relay-rewrite
plan: 01
subsystem: spec
tags: [nub-relay, wire-format, postMessage, spec-rewrite, nostr-relay]

# Dependency graph
requires:
  - phase: 01-spec-foundation
    provides: SPEC.md with NIP-5D wire format (domain.action typed messages)
  - phase: 03-nub-storage-rewrite
    provides: Validated NUB spec section order and wire format rewrite pattern
provides:
  - NUB-RELAY.md on nub-relay branch using relay.* typed messages (no NIP-01 wire verbs, no kind 29001)
  - PR #2 updated with new wire format description and title (removed "NIP-01" reference)
  - relay.subscribe, relay.publish, relay.query, relay.close, relay.event, relay.eose, relay.closed, relay.publish.result, relay.query.result message types defined
affects: [05-nub-nostrdb-rewrite, 06-nub-ifc-rewrite, 07-nub-pipes-rewrite]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "NUB spec section order: Description -> API Surface -> Wire Protocol -> Shell Behavior -> Security Considerations"
    - "TypeScript interface block annotated with // via relay.* comments"
    - "Wire protocol table: Type | Direction | Payload fields"
    - "Subscription-scoped messages use subId field; request/result pairs use id field"
    - "Napplets sign via window.nostr.signEvent() before relay.publish — one path only, no dual-path"
    - "Scoped relay support via optional relay field in relay.subscribe"

key-files:
  created: []
  modified:
    - NUB-RELAY.md (on nub-relay branch — fully rewritten spec)

key-decisions:
  - "relay.publish accepts a signed NostrEvent only — napplets MUST sign via window.nostr.signEvent(template) first (D-07)"
  - "subId field distinguishes subscription-scoped messages (relay.event, relay.eose, relay.closed) from request/result pairs (id field)"
  - "Shell MUST verify event signatures before relay broadcast — invalid signatures rejected with relay.publish.result { ok: false, error: 'invalid signature' }"
  - "Scoped relay support preserved via optional relay field — needed for NIP-29 group relays (D-09)"
  - "Removed NUB-SIGNER reference from spec — NIP-07 window.nostr is a core SPEC.md/NIP-5D requirement (D-10)"

patterns-established:
  - "Wire Protocol section replaces Event Kinds — relay.* typed messages replace NIP-01 verbs"
  - "Result suffix: relay.publish.result, relay.query.result"
  - "Publish result: { ok, eventId?, error? } — explicit success/failure pattern"
  - "relay.closed for shell-initiated subscription termination (vs relay.close for napplet-initiated)"

requirements-completed: [SPEC-01]

# Metrics
duration: 2min
completed: 2026-04-07
---

# Phase 4 Plan 01: NUB-RELAY Rewrite Summary

**NUB-RELAY rewritten to use `relay.*` typed messages replacing NIP-01 wire verbs and kind 29001 — publish() now takes signed NostrEvent signed via window.nostr.signEvent()**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-04-07T11:04:52Z
- **Completed:** 2026-04-07T11:07:11Z
- **Tasks:** 2
- **Files modified:** 1 (NUB-RELAY.md on nub-relay branch)

## Accomplishments

- Rewrote NUB-RELAY.md replacing NIP-01 wire verbs (REQ/EVENT/CLOSE/EOSE/OK/CLOSED/NOTICE) and kind 29001 scoped relay events with Wire Protocol section using `relay.subscribe`, `relay.publish`, `relay.query`, `relay.close`, `relay.event`, `relay.eose`, `relay.closed`, `relay.publish.result`, `relay.query.result` typed messages
- Changed `publish()` signature from `EventTemplate` + NUB-SIGNER path to `signed NostrEvent` — napplets sign via `window.nostr.signEvent(template)` first, one path only
- Added `Subscription.on('event')` / `Subscription.on('eose')` event listener pattern replacing inline callback parameters
- Added concrete JSON examples for subscribe, publish (with sign step), query, close subscription, scoped relay, and publish error
- Force-pushed to `origin/nub-relay` and updated PR #2 title ("NUB-RELAY: Relay proxy interface") and body with new wire format description

## Task Commits

Each task was committed atomically:

1. **Task 1: Rewrite NUB-RELAY.md on nub-relay branch** - `de2f6a5` (docs)
2. **Task 2: Force-push to PR branch and update PR body** - `de2f6a5` (same commit — branch force-pushed, PR body updated via gh CLI)

**Plan metadata:** (docs commit to follow)

## Files Created/Modified

- `NUB-RELAY.md` (nub-relay branch) — Complete rewrite: Wire Protocol section with 9-row typed message table; API Surface TypeScript block with updated `subscribe()` signature (options-only, no inline callbacks), `publish(event: NostrEvent)` (signed only); Shell Behavior updated with relay pool forwarding, signature verification requirement, subId/id routing; Security Considerations with SSRF warning for scoped relay URLs, filter exhaustion, and signing model

## Decisions Made

- TypeScript `Subscription` interface changed to event-listener pattern (`on('event', cb)`, `on('eose', cb)`) instead of constructor-time callbacks — more natural API consistent with EventEmitter style
- `relay.close` includes `id` field for correlation even though it's a fire-and-forget — ensures shell can acknowledge with a result if needed in the future
- PR #2 title updated to remove "NIP-01" reference — spec no longer uses NIP-01 wire verbs, title was misleading
- Shell Behavior note: "which relays to connect to, reconnection strategy, and load balancing are implementation details" — keeps spec at the interface level

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None. Stash/pop around branch switch required due to unstaged STATE.md and TEMPLATE-NN.md changes on master (same pattern as Phase 03).

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- NUB-RELAY rewrite is complete and force-pushed to origin/nub-relay (PR #2 updated)
- The wire format pattern established across Phase 03 (STORAGE) and Phase 04 (RELAY) serves as the template for all remaining spec rewrites (Phases 05-07: NOSTRDB, IFC, PIPES)
- No blockers for Phase 05

## Self-Check: PASSED

- NUB-RELAY.md exists on nub-relay branch: FOUND
- Commit de2f6a5 exists: FOUND
- PR #2 state OPEN: CONFIRMED
- All forbidden terms absent: CONFIRMED (0 occurrences of 29001, NIP-01 verbs, NUB-SIGNER, EventTemplate, delegated)

---
*Phase: 04-nub-relay-rewrite*
*Completed: 2026-04-07*
