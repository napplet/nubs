# Phase 4: NUB-RELAY Rewrite - Context

**Gathered:** 2026-04-07
**Status:** Ready for planning

<domain>
## Phase Boundary

Fully rewrite NUB-RELAY using the SPEC.md wire format (`relay.*` message types). This is the highest-impact rewrite — relay access is the most fundamental shell capability. Napplets sign events themselves via `window.nostr.signEvent()` (NIP-07, provided by shell per SPEC.md line 33) and publish signed events. The shell forwards to relays.

</domain>

<decisions>
## Implementation Decisions

### Wire Protocol Message Design
- **D-01:** Request types: `relay.subscribe`, `relay.publish`, `relay.query`, `relay.close`
- **D-02:** Event delivery: `relay.event` — shell pushes matching events to napplet
- **D-03:** Subscription lifecycle: `relay.eose` (end of stored events), `relay.closed` (shell-terminated sub)
- **D-04:** Publish result: `relay.publish.result` with `{ ok: true, id: "<hex>" }` or `{ error: "..." }`
- **D-05:** Correlation IDs: `id` field on all request/result pairs, `subId` for subscription-scoped messages

### Event Representation
- **D-06:** `relay.event` delivers full `NostrEvent` objects (pubkey, id, sig, kind, tags, content, created_at) — these are externally-authored events from relays
- **D-07:** Napplets ALWAYS sign events themselves via `window.nostr.signEvent(template)` before publishing. `relay.publish` accepts a signed `NostrEvent`, NOT an unsigned template. The shell forwards it to relays. No dual-path (signed vs unsigned) — one path only.
- **D-08:** Filters use standard NIP-01 filter objects: `{ kinds, authors, #e, #p, since, until, limit }`

### Scoped Relays & Structure
- **D-09:** Keep scoped relay support — needed for NIP-29 groups, uses `relay` field in messages
- **D-10:** Spec document structure: Wire Protocol → Shell Behavior → Security Considerations (same as NUB-STORAGE, no Implementation Reference appendix)
- **D-11:** Keep TypeScript interface block with `// via relay.subscribe` annotations — consistent with STORAGE

### Claude's Discretion
- Exact payload field names beyond the ones specified above
- How to describe relay pool management in Shell Behavior
- Security Considerations wording for scoped relay URL validation

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Core protocol
- `SPEC.md` — NIP-5D wire format, identity model, NUB framework. Line 33: shells MUST provide NIP-07 `window.nostr`

### Current spec (to be rewritten)
- PR #2 on napplet/nubs (`nub-relay` branch) — existing NUB-RELAY spec with NIP-01 wire verbs and kind 29001

### Validated pattern
- `.planning/phases/03-nub-storage-rewrite/03-CONTEXT.md` — NUB-STORAGE established the document structure pattern
- `nub-storage` branch `NUB-STORAGE.md` — reference for section ordering and message type conventions

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- NUB-STORAGE spec on `nub-storage` branch — validated document structure to follow
- Existing NUB-RELAY API surface (subscribe/publish/query) is mostly preserved, but publish() signature changes (accepts signed NostrEvent, not template)

### Established Patterns
- SPEC.md wire format: `{ "type": "domain.action", "id": "...", ...payload }`
- NUB-STORAGE pattern: request types → `.result` response suffix → concrete JSON examples

### Integration Points
- README.md registry table (NUB-RELAY row already exists)
- PR #2 on nub-relay branch — force-push rewritten spec

</code_context>

<specifics>
## Specific Ideas

- Napplets use `window.nostr.signEvent(template)` to sign, then pass the signed event to `relay.publish`. This is the ONLY publish path — no shell-signs-for-you alternative.
- NIP-07 `window.nostr` is a core SPEC.md requirement (line 33), so every napplet has signing capability available.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 04-nub-relay-rewrite*
*Context gathered: 2026-04-07*
