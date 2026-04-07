# Phase 6: NUB-IFC Rewrite - Context

**Gathered:** 2026-04-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Fully rewrite NUB-IFC using the SPEC.md wire format (`ifc.*` message types). Replace kind 29003 signed event wrappers with typed messages. Sender verification moves to shell's `MessageEvent.source` identity mapping per SPEC.md.

</domain>

<decisions>
## Implementation Decisions

### Wire Protocol
- **D-01:** Message types: `ifc.emit` (napplet→shell broadcast), `ifc.subscribe`, `ifc.unsubscribe` (topic subscription management), `ifc.event` (shell→napplet delivery)
- **D-02:** Topic conventions preserved: `shell:*` (napplet→shell commands), `napplet:*` (shell→napplet responses), `{domain}:*` (inter-napplet). Advisory, not enforced by shell.
- **D-03:** Sender identity in `ifc.event` delivery includes sender `dTag` (napplet type identifier) — not raw pubkey
- **D-04:** Sender exclusion preserved — shell MUST NOT deliver `ifc.event` back to the emitting napplet

### Structure
- **D-05:** Same document structure as STORAGE/RELAY/NOSTRDB: Wire Protocol → Shell Behavior → Security Considerations
- **D-06:** TypeScript interface with `// via ifc.*` annotations
- **D-07:** Security Considerations describes sender identity as shell-enforced via `MessageEvent.source` — no per-message signing

### Claude's Discretion
- Exact payload field names
- How to describe shell topic interception in Shell Behavior
- Fire-and-forget vs acknowledgment semantics for emit

</decisions>

<canonical_refs>
## Canonical References

### Core protocol
- `SPEC.md` — Wire format, `MessageEvent.source` identity, NUB framework

### Current spec (to be rewritten)
- PR #5 on napplet/nubs (`nub-ipc` branch) — existing spec with kind 29003/IFC_PEER. Note: branch name is `nub-ipc` but spec is `NUB-IFC`

### Validated patterns
- `nub-storage`, `nub-relay` branches — document structure and message type references

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- NUB-IFC API surface (emit/on) is clean — pattern preserved
- Topic conventions are well-defined and useful

### Integration Points
- PR #5 on `nub-ipc` branch — force-push rewritten spec (branch name stays as-is, spec file says NUB-IFC)

</code_context>

<specifics>
## Specific Ideas

No specific requirements — follow established patterns.

</specifics>

<deferred>
## Deferred Ideas

None.

</deferred>

---

*Phase: 06-nub-ifc-rewrite*
*Context gathered: 2026-04-07*
