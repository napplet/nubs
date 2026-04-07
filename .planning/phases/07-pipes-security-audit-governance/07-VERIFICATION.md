---
phase: 07-pipes-security-audit-governance
verified: 2026-04-07T12:00:00Z
status: passed
score: 5/5 must-haves verified
re_verification: false
---

# Phase 7: Pipes, Security Audit & Governance — Verification Report

**Phase Goal:** NUB-PIPES is closed and its channel concepts absorbed into NUB-IFC; cross-spec Security Considerations audit confirms consistency; templates and registry updated
**Verified:** 2026-04-07
**Status:** PASSED
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| #  | Truth | Status | Evidence |
|----|-------|--------|----------|
| 1  | PR #6 (NUB-PIPES) closed, nub-pipes branch deleted, channel support added to NUB-IFC as `ifc.channel.*` message types | VERIFIED | `gh pr view 6` returns `state: CLOSED`; remote nub-pipes confirmed deleted via `git fetch --prune` (pruned stale local tracking ref); nub-ipc:NUB-IFC.md has ifc.channel.open (11 occurrences), emit (5), event (4), broadcast (6), list (6), close (11); NUB-PIPES count = 0 |
| 2  | All 4 spec Security Considerations sections use the runtime as subject of MUST clauses for crypto — no spec says napplets MUST sign or verify | VERIFIED | `grep -ic "napplet.*MUST.*sign\|napplet.*MUST.*verify"` = 0 across all 4 branches; MUST clauses in each spec use "The shell MUST" as subject; NUB-RELAY's napplet-signs language is descriptive only ("napplet signs the event") |
| 3  | TEMPLATE-WORD.md uses SPEC.md wire format structure (`type` strings, payload shapes, shell behavior) | VERIFIED | `grep -c "Wire Protocol" TEMPLATE-WORD.md` = 1; `grep -c "Event Kinds" TEMPLATE-WORD.md` = 0; `grep -c "domain.action" TEMPLATE-WORD.md` = 1 |
| 4  | TEMPLATE-NN.md updated to reflect new spec philosophy | VERIFIED | `grep -ic "wire format" TEMPLATE-NN.md` = 1; `grep -c '"kind"' TEMPLATE-NN.md` = 0; NIP-5D wire format referenced as primary; NIP-01 event kind structure absent |
| 5  | README registry table has exactly 4 rows (RELAY, STORAGE, NOSTRDB, IFC) — NUB-PIPES removed | VERIFIED | Registry table row count = 4 (RELAY, STORAGE, NOSTRDB, IFC); `grep -c "NUB-PIPES" README.md` = 0; no "pipes" in intro text |

**Score:** 5/5 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `NUB-IFC.md` (nub-ipc branch) | Channel support with 9 wire protocol types, auth-on-open model, zero NUB-PIPES refs | VERIFIED | All 6 channel message families present with correct occurrence counts; `ifc.channel.open.result`, `ifc.channel.emit`, `ifc.channel.event`, `ifc.channel.broadcast`, `ifc.channel.list`, `ifc.channel.list.result`, `ifc.channel.close`, `ifc.channel.closed` all present; auth-on-open phrase appears 4 times; NUB-PIPES = 0 |
| `TEMPLATE-WORD.md` (master) | Wire Protocol section replacing Event Kinds, domain.action format | VERIFIED | Wire Protocol section present, Event Kinds absent, shell-as-MUST-subject note in Security Considerations |
| `TEMPLATE-NN.md` (master) | NIP-5D wire format references, no NIP-01 kind JSON structure | VERIFIED | "wire format" present, `"kind"` JSON key absent, NUB-RELAY/NUB-IFC listed as transport mechanisms |
| `README.md` (master) | 4-row registry, no NUB-PIPES | VERIFIED | Exactly 4 rows; "pipes" absent from both registry and intro text |

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| NUB-IFC.md channel section | SPEC.md identity model | dTag-based channel targeting (`target` field = dTag string) | VERIFIED | `ifc.channel.open` uses `target` (dTag string); `ifc.channel.event` carries `sender` (dTag); shell validates via SPEC.md `MessageEvent.source` identity |
| TEMPLATE-WORD.md | SPEC.md wire format | Template section structure with `domain.action` pattern | VERIFIED | Template uses `{name}.action` / `{name}.action.result` pattern; explicitly cites NIP-5D wire format |
| PR #6 closure | NUB-IFC PR #5 | Closing comment pointing to ifc.channel.* | VERIFIED | Comment text: "Closing — channel functionality has been merged into NUB-IFC (PR #5) as `ifc.channel.*` message types..." |

### Data-Flow Trace (Level 4)

Not applicable — this phase produces specification documents (Markdown), not runnable code with data flows.

### Behavioral Spot-Checks

| Behavior | Command | Result | Status |
|----------|---------|--------|--------|
| PR #6 is closed on GitHub | `gh pr view 6 -R napplet/nubs --json state --jq '.state'` | CLOSED | PASS |
| Remote nub-pipes branch deleted | `git ls-remote origin nub-pipes` (after fetch --prune) | empty | PASS |
| NUB-IFC has all channel types | `git show nub-ipc:NUB-IFC.md | grep -c "ifc.channel.open"` | 11 | PASS |
| NUB-IFC has zero NUB-PIPES refs | `git show nub-ipc:NUB-IFC.md | grep -c "NUB-PIPES"` | 0 | PASS |
| Zero napplet MUST sign/verify across all 4 specs | pattern check on each branch | 0 on all 4 | PASS |
| TEMPLATE-WORD.md Wire Protocol present | `grep -c "Wire Protocol" TEMPLATE-WORD.md` | 1 | PASS |
| TEMPLATE-WORD.md Event Kinds absent | `grep -c "Event Kinds" TEMPLATE-WORD.md` | 0 | PASS |
| TEMPLATE-NN.md kind JSON absent | `grep -c '"kind"' TEMPLATE-NN.md` | 0 | PASS |
| README registry exactly 4 rows | count of `^\| [NUB-` rows | 4 | PASS |
| README no NUB-PIPES row | `grep -c "NUB-PIPES" README.md` | 0 | PASS |

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| SPEC-06 | 07-01-PLAN.md | NUB-PIPES rewritten — peer identity uses opaque token or dTag, not raw pubkey | SATISFIED | NUB-PIPES absorbed into NUB-IFC; channel open uses dTag targeting; PR #6 closed |
| SPEC-08 | 07-02-PLAN.md | Security Considerations updated in all 6 specs to reflect runtime-owned crypto model | SATISFIED | All 4 active specs have zero "napplet MUST sign/verify" clauses; all MUST crypto clauses have shell as subject |
| GOVN-01 | 07-02-PLAN.md | TEMPLATE-WORD.md updated with two-layer section structure | SATISFIED | Wire Protocol section present, shell-behavior notes present, shell-as-MUST-subject note in Security Considerations template |
| GOVN-02 | 07-02-PLAN.md | TEMPLATE-NN.md updated to reflect new spec philosophy | SATISFIED | NIP-5D wire format primary; NIP-01 kind structure absent; NUB-IFC/NUB-RELAY as transport |
| GOVN-03 | 07-02-PLAN.md | README registry table updated | SATISFIED | 4 rows, NUB-PIPES removed, NUB-IFC row present |

No orphaned requirements — all 5 Phase 7 requirements (SPEC-06, SPEC-08, GOVN-01, GOVN-02, GOVN-03) were claimed by plans and have implementation evidence.

### Anti-Patterns Found

No blocker anti-patterns found.

Notes on reviewed content:
- NUB-RELAY still describes the napplet-signs-via-window.nostr flow in the API section (descriptive language: "napplet signs"), but this is correct — it documents the NIP-07 workflow, not a normative crypto MUST clause. All MUST clauses in NUB-RELAY use the shell as subject.
- `nub-signer` local branch and remote still exist, but NUB-SIGNER was closed in Phase 2 (not Phase 7 scope). Phase 7 audited the 4 active specs only.
- Remote `remotes/origin/nub-pipes` appeared in local `git branch -a` output at the time of verification as a stale tracking ref. A `git fetch --prune` confirmed the remote branch is deleted — the ref was stale.

### Human Verification Required

None required — all success criteria are verifiable from the git object store and GitHub API.

### Gaps Summary

No gaps. All 5 success criteria are satisfied by the actual codebase state.

---

_Verified: 2026-04-07_
_Verifier: Claude (gsd-verifier)_
