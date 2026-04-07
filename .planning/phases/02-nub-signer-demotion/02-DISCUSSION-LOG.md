# Phase 2: NUB-SIGNER Demotion - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-07
**Phase:** 02-nub-signer-demotion
**Areas discussed:** SIGNER's new role, IFC naming resolution, NIP-07 compatibility, Downstream references

---

## SIGNER's New Role

| Option | Description | Selected |
|--------|-------------|----------|
| Runtime implementation guide | Rewrite as non-normative shell implementation guide | |
| Delete entirely | Remove NUB-SIGNER spec. Signing details belong in SPEC.md | ✓ |
| Optional user-consent API | Keep minimal napplet-callable signer.* domain | |

**User's choice:** Initially selected "Runtime implementation guide", then clarified: NUB-SIGNER is unnecessary — NIP-07 is already in SPEC.md as a core requirement. Delete entirely, close PR, delete branch.
**Notes:** User pointed out SPEC.md line 33 mandates NIP-07. NUB-SIGNER has no remaining purpose.

---

## IFC Naming Resolution

| Option | Description | Selected |
|--------|-------------|----------|
| ifc (Inter-Frame Communication) | Already used in README and spec file. Accurate for iframes. | ✓ |
| ipc (Inter-Process Communication) | More conventional CS term. Branch named nub-ipc. Technically wrong. | |

**User's choice:** ifc
**Notes:** Branch name `nub-ipc` is the only stale reference.

---

## NIP-07 Compatibility

**User's choice:** Not a separate discussion — user clarified that NUB-SIGNER should simply be closed. NIP-07 is a SPEC.md concern, not a NUB concern.
**Notes:** User corrected Claude's assertion that NIP-07 was "absorbed" — the relationship is that SPEC.md mandates it directly.

---

## Downstream References

| Option | Description | Selected |
|--------|-------------|----------|
| Each rewrite cleans up naturally | Phases 3-7 rewrite each spec against SPEC.md. Stale refs disappear. | ✓ |
| Document a signing pattern in CONTEXT | Capture standard phrase for Shell Behavior sections. | |

**User's choice:** Each rewrite cleans up naturally
**Notes:** No special coordination needed across phases.

## Claude's Discretion

- PR close message wording
- Whether to rename `nub-ipc` branch or close/reopen during Phase 6

## Deferred Ideas

None.
