# Phase 5: NUB-NOSTRDB Rewrite - Context

**Gathered:** 2026-04-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Fully rewrite NUB-NOSTRDB using the SPEC.md wire format (`nostrdb.*` message types). Napplets can cache and query events received from relay subscriptions. The `add()` method accepts signed NostrEvent objects (externally authored).

</domain>

<decisions>
## Implementation Decisions

### Wire Protocol
- **D-01:** Message types: `nostrdb.query`, `nostrdb.add`, `nostrdb.event` (by id), `nostrdb.replaceable`, `nostrdb.count`, `nostrdb.subscribe`, `nostrdb.unsubscribe` with `.result` response suffixes
- **D-02:** `nostrdb.add` accepts a signed `NostrEvent` — these are externally-authored events (e.g., from relay subscriptions) being cached locally
- **D-03:** Subscribe delivery uses `nostrdb.event` pushes with `subId` — same pattern as `relay.event`
- **D-04:** Correlation IDs: `id` for request/response, `subId` for subscription-scoped delivery

### Structure
- **D-05:** Same document structure as NUB-STORAGE and NUB-RELAY: Wire Protocol → Shell Behavior → Security Considerations
- **D-06:** TypeScript interface with `// via nostrdb.*` annotations
- **D-07:** No backward-compatibility or migration notes — unpublished spec

### Claude's Discretion
- Exact payload field names beyond id, subId, event, filters
- How to describe OPFS/persistence in Shell Behavior
- Security Considerations wording

</decisions>

<canonical_refs>
## Canonical References

### Core protocol
- `SPEC.md` — Wire format, identity model, NUB framework

### Current spec (to be rewritten)
- PR #4 on napplet/nubs (`nub-nostrdb` branch) — existing spec with kind 29006/29007

### Validated patterns
- `nub-storage` branch `NUB-STORAGE.md` — document structure reference
- `nub-relay` branch `NUB-RELAY.md` — subscription/event delivery pattern reference

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- NUB-NOSTRDB API surface (query, add, event, replaceable, count, subscribe) is clean — no crypto
- NUB-RELAY's subscription pattern (`relay.event` with `subId`) applies to `nostrdb.subscribe`

### Integration Points
- PR #4 on nub-nostrdb branch — force-push rewritten spec

</code_context>

<specifics>
## Specific Ideas

No specific requirements — follow established STORAGE/RELAY patterns.

</specifics>

<deferred>
## Deferred Ideas

None.

</deferred>

---

*Phase: 05-nub-nostrdb-rewrite*
*Context gathered: 2026-04-07*
