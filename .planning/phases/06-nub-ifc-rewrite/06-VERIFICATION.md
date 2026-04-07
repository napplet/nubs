---
phase: 06-nub-ifc-rewrite
verified: 2026-04-07T11:28:16Z
status: passed
score: 6/6 must-haves verified
re_verification: false
---

# Phase 06: NUB-IFC Rewrite Verification Report

**Phase Goal:** NUB-IFC is fully rewritten using ifc.* message types — inter-frame communication without signed event wrappers, sender verification via shell's MessageEvent.source identity mapping
**Verified:** 2026-04-07T11:28:16Z
**Status:** passed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| #  | Truth                                                                                                 | Status     | Evidence                                                                                   |
|----|-------------------------------------------------------------------------------------------------------|------------|--------------------------------------------------------------------------------------------|
| 1  | NUB-IFC defines ifc.emit, ifc.subscribe, ifc.unsubscribe, ifc.event message types using SPEC.md wire format | VERIFIED | 5 occurrences of `ifc.emit`, 10 of `ifc.subscribe`, 5 of `ifc.event`, 3 of `ifc.unsubscribe` found in spec |
| 2  | Shell Behavior documents sender verification via MessageEvent.source -> napplet identity mapping      | VERIFIED   | Shell Behavior section: "The shell MUST identify the sender via `MessageEvent.source` and include the sender's `dTag`" |
| 3  | Security Considerations describes sender identity as shell-enforced — no per-message Schnorr signing  | VERIFIED   | "There is no per-message signing — the shell's sender identification is the trust boundary" |
| 4  | No reference to kind 29003, IFC_PEER, signed events, or NIP-01 subscriptions in napplet-facing sections | VERIFIED | grep confirmed zero matches for kind 29003, IFC_PEER, Schnorr, NIP-01, "signed event", "delegated session" |
| 5  | Topic conventions (shell:*, napplet:*, domain:*) are preserved                                        | VERIFIED   | Topic Conventions table present with all three prefix types and advisory routing note       |
| 6  | Sender identity in ifc.event delivery uses dTag, not pubkey                                           | VERIFIED   | Wire Protocol table: `ifc.event` payload field `sender` annotated as "(dTag)"; IfcEvent interface has `sender: string // sender dTag` |

**Score:** 6/6 truths verified

### Required Artifacts

| Artifact     | Expected                                              | Status   | Details                                                                      |
|--------------|-------------------------------------------------------|----------|------------------------------------------------------------------------------|
| `NUB-IFC.md` | Complete NUB-IFC spec rewritten with ifc.* wire format | VERIFIED | Exists on `nub-ipc` branch (commit `0e0819d`); not on master (correct)      |

### Key Link Verification

| From         | To      | Via                       | Status   | Details                                                                   |
|--------------|---------|---------------------------|----------|---------------------------------------------------------------------------|
| `NUB-IFC.md` | SPEC.md | wire format reference     | VERIFIED | "IFC operations use the NIP-5D wire format (`{ "type": "domain.action", ...payload }`)"; MessageEvent.source referenced with "(per SPEC.md identity model)" |

### Data-Flow Trace (Level 4)

Not applicable — this phase produces a markdown specification document, not runnable code with data flows.

### Behavioral Spot-Checks

| Behavior                                | Command                                                | Result          | Status |
|-----------------------------------------|--------------------------------------------------------|-----------------|--------|
| Rewritten commit exists on nub-ipc      | `git log nub-ipc --oneline -1`                        | `0e0819d rewrite NUB-IFC: ifc.* typed messages, drop kind 29003 signed events` | PASS |
| PR #5 remains open                      | `gh pr view 5 --json state,title`                     | `OPEN: NUB-IFC: Inter-frame communication` | PASS |
| NUB-IFC.md not on master                | `git show master:NUB-IFC.md`                          | `FILE_NOT_ON_MASTER` | PASS |
| ifc.emit present                        | `git show nub-ipc:NUB-IFC.md \| grep -c "ifc\.emit"` | 5 matches       | PASS |
| MessageEvent.source present             | count grep                                             | 3 matches       | PASS |
| Forbidden crypto terms absent           | grep for kind 29003, IFC_PEER, Schnorr, NIP-01, signed event, delegated session | All returned NOT_FOUND | PASS |

### Requirements Coverage

| Requirement | Source Plan   | Description                                                                        | Status    | Evidence                                                                          |
|-------------|---------------|------------------------------------------------------------------------------------|-----------|-----------------------------------------------------------------------------------|
| SPEC-05     | 06-01-PLAN.md | NUB-IFC rewritten — drop signed event wrappers, sender verification via `MessageEvent.source` | SATISFIED | Spec fully rewritten; shell identity model documented; all forbidden crypto terms absent |

No orphaned requirements — REQUIREMENTS.md maps SPEC-05 to Phase 6 only, and it is the single requirement declared in the plan.

### Anti-Patterns Found

None. The spec is a markdown document. No TODO/FIXME/placeholder patterns are present in NUB-IFC.md. The document follows the validated NUB-STORAGE/NUB-RELAY structural pattern with complete sections (Description, API Surface, Wire Protocol, Topic Conventions, Shell Behavior, Security Considerations, Relationship to NUB-PIPES).

### Human Verification Required

None. All success criteria are verifiable programmatically against the spec text.

### Gaps Summary

No gaps. All six must-have truths are verified against the actual content of NUB-IFC.md on the `nub-ipc` branch. The rewrite fully achieves the phase goal: `ifc.*` typed messages replace kind 29003 signed events, `MessageEvent.source` identity mapping is the documented trust boundary, and no per-message Schnorr signing appears anywhere in napplet-facing sections.

---

_Verified: 2026-04-07T11:28:16Z_
_Verifier: Claude (gsd-verifier)_
