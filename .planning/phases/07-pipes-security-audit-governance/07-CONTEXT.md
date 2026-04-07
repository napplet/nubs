# Phase 7: Pipes, Security Audit & Governance - Context

**Gathered:** 2026-04-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Close NUB-PIPES PR (#6, branch `nub-pipes`) — channels are merged into NUB-IFC. Update the NUB-IFC spec (on `nub-ipc` branch) to add channel support. Run cross-spec security audit across all 5 rewritten specs. Update TEMPLATE-WORD.md, TEMPLATE-NN.md, and README.md registry.

</domain>

<decisions>
## Implementation Decisions

### NUB-PIPES Disposition
- **D-01:** NUB-PIPES spec is deleted entirely — PR #6 closed, `nub-pipes` branch deleted on remote. Channels are a feature of NUB-IFC, not a separate NUB.
- **D-02:** README registry: remove NUB-PIPES row (5 specs → 4: RELAY, STORAGE, NOSTRDB, IFC)

### IFC Channel Support (added to existing NUB-IFC spec)
- **D-03:** `ifc.channel.open` targets a specific napplet by dTag — point-to-point, pre-authorized channel. Shell checks/approves/denies once on open. Returns a channel ID.
- **D-04:** SDK convenience: `ifc.channel.open(dTag)` returns `{ emit, on, id }` object. Without SDK, the channel ID is used in low-level emit/on calls.
- **D-05:** Channel messages are NOT checked per-message by the shell — only the initial `ifc.channel.open` is validated. If revoked, subsequent opens return null/error.
- **D-06:** `ifc.channel.emit` / `ifc.channel.event` (delivery) — consistent with `ifc.emit` / `ifc.event` naming
- **D-07:** `ifc.channel.broadcast` — send to all open channels at once
- **D-08:** `ifc.channel.list` / `ifc.channel.list.result` — retrieve/list active channels
- **D-09:** `ifc.channel.close` — tear down a channel, both sides notified

### Cross-Spec Security Audit
- **D-10:** Automated grep across all 5 rewritten specs (RELAY, STORAGE, NOSTRDB, IFC, PIPES-removed) for stale crypto references: sign, Schnorr, delegated, kind numbers. Fix any found.
- **D-11:** All Security Considerations sections must use the runtime/shell as the subject of MUST clauses for crypto — no spec says napplets MUST sign or verify

### Templates & Governance
- **D-12:** TEMPLATE-WORD.md updated with validated structure: Description → API Surface → Wire Protocol → Shell Behavior → Security Considerations
- **D-13:** TEMPLATE-NN.md updated to reference SPEC.md wire format, remove NIP-01 event kind references
- **D-14:** README.md registry: 4 rows (RELAY, STORAGE, NOSTRDB, IFC), NUB-PIPES removed, NUB-SIGNER already removed

### Claude's Discretion
- Exact channel message payload field names
- PR #6 close message wording
- TEMPLATE-NN.md example content
- How to describe the channel vs topic distinction in the IFC spec

</decisions>

<canonical_refs>
## Canonical References

### Core protocol
- `SPEC.md` — Wire format, identity model, NUB framework

### Specs to update
- PR #5 on napplet/nubs (`nub-ipc` branch) — NUB-IFC spec getting channel support added

### Spec to close
- PR #6 on napplet/nubs (`nub-pipes` branch) — being merged into NUB-IFC

### Validated patterns (for security audit)
- `nub-relay` branch `NUB-RELAY.md`
- `nub-storage` branch `NUB-STORAGE.md`
- `nub-nostrdb` branch `NUB-NOSTRDB.md`

### Templates
- `TEMPLATE-WORD.md` — to be updated
- `TEMPLATE-NN.md` — to be updated

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- NUB-IFC spec on `nub-ipc` branch — channel support added to existing spec
- NUB-PIPES API concepts (open, send, broadcast, close) inform the channel API design

### Integration Points
- README.md registry table — remove PIPES row
- TEMPLATE-WORD.md and TEMPLATE-NN.md — update structure
- All 5 spec PR branches — security audit targets

</code_context>

<specifics>
## Specific Ideas

- Channels are "pre-authorized IFC" — shell validates once on open, then messages flow without per-message checking. This is the key distinction from regular IFC topic messaging.
- The old NUB-PIPES comparison to IFC signing overhead is no longer relevant (IFC doesn't sign per-message anymore). The channel distinction is now about authorization granularity, not crypto overhead.

</specifics>

<deferred>
## Deferred Ideas

None.

</deferred>

---

*Phase: 07-pipes-security-audit-governance*
*Context gathered: 2026-04-07*
