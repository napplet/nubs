# Phase 2: NUB-SIGNER Demotion - Context

**Gathered:** 2026-04-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Close the NUB-SIGNER spec entirely (PR closed, remote branch deleted) and resolve the IFC/IPC naming inconsistency across the repo. NUB-SIGNER is not demoted to a runtime guide — it is removed. Signing is handled by SPEC.md's core protocol (shells MUST provide NIP-07 `window.nostr`). The IFC naming is locked as canonical.

</domain>

<decisions>
## Implementation Decisions

### NUB-SIGNER disposition
- **D-01:** NUB-SIGNER spec is deleted entirely — no runtime guide, no implementation doc. The PR is closed and the remote branch is deleted.
- **D-02:** NIP-07 `window.nostr` is a core SPEC.md requirement (line 33: "Shells MUST provide a NIP-07 window.nostr implementation to each napplet iframe"). There is nothing left for a separate NUB-SIGNER to specify.
- **D-03:** The README registry table row for NUB-SIGNER must be removed.

### IFC naming resolution
- **D-04:** Canonical name is `NUB-IFC` (Inter-Frame Communication). Domain name in wire protocol is `ifc`.
- **D-05:** The `nub-ipc` branch name on remote is the stale artifact. All references should use `ifc`, not `ipc`.

### Downstream references
- **D-06:** Stale NUB-SIGNER references in other specs (NUB-RELAY's `signEvent()` call, NUB-PIPES' signing overhead comparison) will be cleaned up naturally during their respective phase rewrites (Phases 3-7). No special coordination or shared language pattern needed.

### Claude's Discretion
- PR close message wording
- Whether to update the `nub-ipc` branch name on remote or just close the old PR and open a fresh one during Phase 6

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Core protocol
- `SPEC.md` — NIP-5D rewrite; line 33 mandates NIP-07 `window.nostr`; line 29 lists sandbox restrictions; wire format is `{ "type": "domain.action", ...payload }`

### Current NUB-SIGNER spec (to be closed)
- PR #1 on napplet/nubs (`nub-signer` branch) — the NIP-07 proxy spec being removed

### IFC naming
- `README.md` lines 24 — registry table already says NUB-IFC
- PR #5 on napplet/nubs (`nub-ipc` branch) — branch name is stale, spec content already says NUB-IFC

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — this is a spec repo, no executable code

### Established Patterns
- PRs are the delivery mechanism for specs (one PR per NUB)
- README registry table is the central index
- `gh` CLI available for PR operations

### Integration Points
- README.md registry table: remove NUB-SIGNER row, verify NUB-IFC row is correct
- Other spec branches reference NUB-SIGNER (NUB-RELAY, NUB-PIPES) — handled in later phases

</code_context>

<specifics>
## Specific Ideas

No specific requirements — straightforward closure and cleanup.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 02-nub-signer-demotion*
*Context gathered: 2026-04-07*
