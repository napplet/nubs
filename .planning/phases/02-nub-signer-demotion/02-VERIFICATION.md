---
phase: 02-nub-signer-demotion
verified: 2026-04-07T10:39:06Z
status: passed
score: 5/5 must-haves verified
re_verification: false
---

# Phase 2: NUB-SIGNER Demotion Verification Report

**Phase Goal:** NUB-SIGNER PR is closed entirely (NIP-07 `window.nostr` is a core SPEC.md requirement, nothing left for a separate spec) and the IPC/IFC naming is resolved so all other spec rewrites reference the correct canonical name NUB-IFC
**Verified:** 2026-04-07T10:39:06Z
**Status:** passed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|---------|
| 1 | PR #1 (nub-signer) is closed on GitHub with an explanatory message | VERIFIED | `gh pr view 1` returns state=CLOSED, closedAt=2026-04-07T10:34:09Z; last comment contains "SPEC.md" and "window.nostr" |
| 2 | The remote branch origin/nub-signer no longer exists | VERIFIED | `git ls-remote --heads origin nub-signer` returns empty; remote is absent |
| 3 | The README.md registry table has no NUB-SIGNER row | VERIFIED | README.md has exactly 5 data rows: RELAY, STORAGE, NOSTRDB, IFC, PIPES — no NUB-SIGNER row |
| 4 | All references to NUB-IPC in README.md and CLAUDE.md use the canonical name NUB-IFC | VERIFIED | `grep NUB-IPC README.md CLAUDE.md` returns nothing; `grep NUB-IFC` returns hits in both files |
| 5 | CLAUDE.md no longer lists NUB-SIGNER as a spec to submit | VERIFIED | `grep NUB-SIGNER CLAUDE.md` returns nothing; PR table has 5 rows with NUB-IFC in place of NUB-SIGNER |

**Score:** 5/5 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `README.md` | Registry table without NUB-SIGNER row; contains NUB-IFC | VERIFIED | 5-row table. NUB-IFC links to pull/5. `grep NUB-SIGNER README.md` = 0 matches. `grep NUB-IFC README.md` = 1 match (registry row). Commit 0a07457. |
| `CLAUDE.md` | No NUB-SIGNER; NUB-IFC naming throughout | VERIFIED | `grep NUB-SIGNER CLAUDE.md` = 0. `grep NUB-IFC CLAUDE.md` = 3 matches (spec list, example, PR table). NUB-SIGNER row removed from PR table; 5 rows remain. |

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| README.md registry table | GitHub PR #5 | NUB-IFC link in table row | VERIFIED | Line 23: `[NUB-IFC](https://github.com/napplet/nubs/pull/5)` — pattern `NUB-IFC.*pull/5` matches |

### Data-Flow Trace (Level 4)

Not applicable — this phase produces documentation/governance artifacts (markdown files and GitHub PR operations), not runnable code with data flows.

### Behavioral Spot-Checks

Step 7b: SKIPPED — no runnable entry points. This phase modifies only markdown files and GitHub state.

GitHub state checks (equivalent verification):

| Behavior | Command | Result | Status |
|----------|---------|--------|--------|
| PR #1 is closed | `gh pr view 1 --json state` | `CLOSED` | PASS |
| PR #1 has explanatory comment | `gh pr view 1 --json comments` | last comment contains "SPEC.md" and "window.nostr" | PASS |
| Remote nub-signer branch absent | `git ls-remote --heads origin nub-signer` | empty | PASS |
| README.md has no NUB-SIGNER | `grep -c NUB-SIGNER README.md` | 0 | PASS |
| README.md has no NUB-IPC | `grep -c NUB-IPC README.md` | 0 | PASS |
| CLAUDE.md has no NUB-SIGNER | `grep -c NUB-SIGNER CLAUDE.md` | 0 | PASS |
| CLAUDE.md has no NUB-IPC | `grep -c NUB-IPC CLAUDE.md` | 0 | PASS |
| README.md registry has 5 rows | `grep -c '| \[NUB-' README.md` | 5 | PASS |
| CLAUDE.md PR table has 5 rows | `grep -c '| NUB-' CLAUDE.md` | 5 | PASS |

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|-------------|-------------|--------|---------|
| SPEC-03 | 02-01-PLAN.md | NUB-SIGNER reframed as runtime-internal interface — no napplet-callable surface | SATISFIED | PR #1 closed with comment explaining NIP-07 window.nostr is a SPEC.md core invariant. NUB-SIGNER removed from README.md and CLAUDE.md entirely. REQUIREMENTS.md traceability row marked Complete. |
| SPEC-07 | 02-01-PLAN.md | NUB-IFC naming resolved — canonical name applied consistently | SATISFIED | All NUB-IPC references replaced with NUB-IFC in README.md and CLAUDE.md. NUB-IFC links to the correct PR. REQUIREMENTS.md traceability row marked Complete. |

No orphaned requirements: REQUIREMENTS.md traceability table maps SPEC-03 and SPEC-07 to Phase 2 only, and both are covered by 02-01-PLAN.md.

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| `specs/README.md` | 22, 24 | NUB-SIGNER and NUB-IPC rows still present | Info | Not in Phase 2 scope. `specs/` contains source spec files received from the napplet repo (commit b319422). These live on feature branches (nub-signer, nub-ipc) and will be updated in Phases 3-7. The PLAN explicitly scoped changes to repo-root `README.md` and `CLAUDE.md` only. |
| `specs/NUB-SIGNER.md` | entire file | NUB-SIGNER spec file still tracked | Info | Not in Phase 2 scope. Phase 2 PLAN only required closing the remote PR and branch, and updating README.md/CLAUDE.md. The spec file itself is on the nub-signer feature branch, which was deleted from remote. Local branches (nub-signer, nub-ifc, etc.) remain as local working copies. |
| `specs/NUB-IPC.md` | entire file | NUB-IPC spec file — stale name | Info | Not in Phase 2 scope. Deferred to Phase 6 (NUB-IFC rewrite) per SUMMARY decision D-05. |
| `TEMPLATE-NN.md` | 11 | `NUB-IPC` in placeholder example comment | Info | Not in Phase 2 scope. TEMPLATE-NN.md updates are Phase 7 / GOVN-02. The occurrence is a template placeholder `{e.g., NUB-RELAY, NUB-IPC}` — a fill-in example, not a live spec reference. |
| `specs/NUB-RELAY.md` | 56 | References `signEvent()` (NUB-SIGNER) | Info | Not in Phase 2 scope. Deferred to Phase 4 (NUB-RELAY rewrite). Documented in SUMMARY as expected remaining stale reference. |
| `specs/NUB-PIPES.md` | multiple | References NUB-IPC comparison | Info | Not in Phase 2 scope. Deferred to Phase 7 (NUB-PIPES rewrite). |

No blockers. All anti-patterns are either: (a) in files outside Phase 2 scope, (b) explicitly deferred in the SUMMARY to later phases, or (c) template placeholders covered by GOVN-02 in Phase 7.

### Human Verification Required

None. All observable truths for this phase are verifiable programmatically:
- GitHub PR state via `gh` CLI
- Remote branch existence via `git ls-remote`
- File content via grep

## Gaps Summary

No gaps. All 5 must-have truths are verified against the actual codebase and GitHub state.

The phase goal is achieved: NUB-SIGNER is closed and removed from the active spec registry (README.md), the remote branch is deleted, and NUB-IFC is the canonical name used consistently in the two files that were in scope (README.md and CLAUDE.md). The stale references remaining in `specs/` source files and TEMPLATE-NN.md are correctly deferred to their respective later phases (3-7).

---

_Verified: 2026-04-07T10:39:06Z_
_Verifier: Claude (gsd-verifier)_
