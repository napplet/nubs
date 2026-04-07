---
phase: 04-nub-relay-rewrite
verified: 2026-04-07T12:00:00Z
status: passed
score: 4/4 must-haves verified
re_verification: false
---

# Phase 4: NUB-RELAY Rewrite Verification Report

**Phase Goal:** NUB-RELAY is fully rewritten using relay.* message types — the central interface of the milestone — with no crypto in the napplet-visible API (napplets sign via window.nostr.signEvent then pass signed events to relay.publish)
**Verified:** 2026-04-07T12:00:00Z
**Status:** passed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | NUB-RELAY defines relay.publish, relay.subscribe, relay.query (and corresponding result types) using SPEC.md wire format — napplet signs via window.nostr.signEvent() and passes signed event to publish | VERIFIED | relay.subscribe: 4 occurrences, relay.publish: 9, relay.query: 7, relay.publish.result: 6, relay.query.result: 4; window.nostr.signEvent: 3 occurrences; publish() signature is `publish(event: NostrEvent)` not EventTemplate |
| 2 | Shell Behavior section describes how the runtime forwards signed events to relays — not visible to napplets | VERIFIED | "## Shell Behavior" section exists at line 118; describes relay pool forwarding, signature verification, ACL, subscription lifecycle — all framed as runtime behavior |
| 3 | NUB-RELAY does not reference NUB-SIGNER as a napplet-visible dependency | VERIFIED | grep -ci 'NUB-SIGNER' returns 0; signing is described only via window.nostr.signEvent() (NIP-07, a core SPEC.md/NIP-5D requirement) |
| 4 | subscribe and query result payloads are defined with clear types | VERIFIED | Wire Protocol table defines relay.event (subId, event: NostrEvent), relay.eose (subId), relay.query.result (id, events: NostrEvent array); TypeScript Subscription interface defines on('event', cb) and on('eose', cb) |

**Score:** 4/4 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `NUB-RELAY.md` (nub-relay branch) | Rewritten NUB-RELAY spec using SPEC.md wire format | VERIFIED | Exists on nub-relay branch at commit de2f6a5; 142 lines; contains "relay.subscribe" (4x), "relay.publish" (9x), complete Wire Protocol table |

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| NUB-RELAY.md Wire Protocol | NUB-RELAY.md API Surface | message type names mirror API methods (relay.subscribe, relay.publish, relay.query, relay.close) | WIRED | Wire Protocol table rows map directly to TypeScript interface methods with `// via relay.*` annotations |
| NUB-RELAY.md Wire Protocol | SPEC.md wire format | domain.action pattern with .result suffix (relay.publish.result, relay.query.result) | WIRED | Both .result suffixes present in table and examples; relay.publish.result: 6 occurrences, relay.query.result: 4 |
| NUB-RELAY.md API Surface | SPEC.md line 33 | window.nostr.signEvent() for signing before relay.publish | WIRED | "The napplet MUST sign the event via `window.nostr.signEvent(template)` before calling publish" in API Surface; also in publish example and Security Considerations |

### Data-Flow Trace (Level 4)

Not applicable — this phase produces a markdown specification document, not runnable code with data sources.

### Behavioral Spot-Checks

Step 7b: SKIPPED (no runnable entry points — this phase produces a markdown spec document only)

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|-------------|-------------|--------|----------|
| SPEC-01 | 04-01-PLAN.md | NUB-RELAY rewritten — per ROADMAP.md Phase 4 success criteria: relay.publish accepts signed NostrEvent, napplets sign via window.nostr.signEvent() | SATISFIED | See truths 1-4 above; all four ROADMAP success criteria met |

**Note on REQUIREMENTS.md SPEC-01 wording:** REQUIREMENTS.md line 21 describes SPEC-01 as "`publish(template)` is self-contained, no `signEvent()` in napplet path, shell signs internally" — this contradicts the ROADMAP Phase 4 goal, CONTEXT.md decision D-07, and the actual implemented spec (which has napplets sign via window.nostr.signEvent()). The ROADMAP is the authoritative source for phase success criteria. The REQUIREMENTS.md description appears to be a stale artifact from an earlier design direction and should be corrected in a future pass.

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| — | — | None found | — | — |

No TODO/FIXME/placeholder comments, no empty implementations, no forbidden terms (NUB-SIGNER, EventTemplate, delegated, kind 29001, NIP-01 wire verbs) found in the spec.

### Human Verification Required

None — all four success criteria are verifiable programmatically from the spec document content.

### Gaps Summary

No gaps. All four success criteria from the ROADMAP are satisfied:

1. relay.publish, relay.subscribe, relay.query plus all result types defined with SPEC.md wire format — napplet signs via window.nostr.signEvent() (3 occurrences confirming this path).
2. Shell Behavior section (line 118) isolates runtime forwarding detail from napplet-visible API.
3. Zero occurrences of NUB-SIGNER; signing attributed solely to window.nostr (NIP-07).
4. Subscribe result payloads: relay.event {subId, event: NostrEvent}, relay.eose {subId}; query result: relay.query.result {id, events: NostrEvent[]}; TypeScript Subscription interface with typed event listener signatures.

PR #2 is OPEN, title is "NUB-RELAY: Relay proxy interface" (no "NIP-01"), body references relay.subscribe, relay.publish, and window.nostr.signEvent. Working tree is on master.

---

_Verified: 2026-04-07T12:00:00Z_
_Verifier: Claude (gsd-verifier)_
