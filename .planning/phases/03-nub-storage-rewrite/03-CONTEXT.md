# Phase 3: NUB-STORAGE Rewrite - Context

**Gathered:** 2026-04-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Fully rewrite NUB-STORAGE using the SPEC.md wire format (`storage.*` message types). This is the simplest spec rewrite and validates the new structure before higher-complexity specs use it. The existing kind 29003/IFC_PEER wire format is replaced entirely.

</domain>

<decisions>
## Implementation Decisions

### Wire Protocol Message Design
- **D-01:** Message types mirror API method names: `storage.get`, `storage.set`, `storage.remove`, `storage.keys`
- **D-02:** Response types use `.result` suffix per SPEC.md pattern: `storage.get.result`, `storage.set.result`, etc.
- **D-03:** Correlation ID field is `id` — matches SPEC.md examples
- **D-04:** Errors are inline in result messages (`{ "type": "storage.get.result", "id": "abc", "error": "quota exceeded" }`) — no separate error message type

### Spec Document Structure
- **D-05:** Keep TypeScript interface block showing napplet-facing API, annotated with `// via storage.get message` comments
- **D-06:** Section ordering: Wire Protocol → Shell Behavior → Security Considerations (no Implementation Reference appendix — clean break from old wire format)

### Storage Semantics
- **D-07:** Keep 512KB quota recommendation as concrete guidance for shell implementers
- **D-08:** Quota errors returned as result message with `error` field, not a separate error type
- **D-09:** Remove backward-compatible migration note entirely — unpublished spec, no backward compatibility needed

### Claude's Discretion
- Exact payload field names beyond `id`, `key`, `value`, `error`
- How to describe composite key scoping in Shell Behavior without exposing internals
- Security Considerations wording updates

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Core protocol
- `SPEC.md` — NIP-5D wire format (`{ "type": "domain.action", ...payload }`), identity model, NUB extension framework

### Current spec (to be rewritten)
- PR #3 on napplet/nubs (`nub-storage` branch) — existing NUB-STORAGE spec with kind 29003 wire format that must be replaced

### Prior phase context
- `.planning/phases/02-nub-signer-demotion/02-CONTEXT.md` — NUB-IFC is canonical name (not NUB-IPC)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- Existing NUB-STORAGE API surface (`getItem`, `setItem`, `removeItem`, `keys`) is already clean — no crypto leakage, can be preserved as-is

### Established Patterns
- SPEC.md example: `{ "type": "foo.bar", "id": "abc", "data": {...} }` / `{ "type": "foo.bar.result", "id": "abc", "result": {...} }`
- Specs submitted as PRs, one per NUB

### Integration Points
- README.md registry table (NUB-STORAGE row already exists, may need status update)
- This rewrite validates the template structure for all subsequent spec rewrites (Phases 4-7)

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches within the decisions above.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 03-nub-storage-rewrite*
*Context gathered: 2026-04-07*
