# Roadmap: NUBs v0.1.0 Spec Simplification

## Overview

Six phases that rewrite the 6 existing draft NUB interface specs to align with the new SPEC.md wire format (`{ "type": "domain.action", ...payload }`) and two-layer architecture. Napplet authors see only typed messages and domain primitives; the runtime handles identity, signing, and ACL invisibly. SPEC.md (Phase 1 — complete) established the foundation. The remaining work demotes NUB-SIGNER, rewrites specs in dependency order using the new wire format, and closes with governance artifacts.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] ~~**Phase 1: Foundation**~~ - SPEC.md written by external agent (complete)
- [ ] **Phase 2: NUB-SIGNER Demotion** - Close NUB-SIGNER PR entirely, resolve IFC/IPC naming
- [x] **Phase 3: NUB-STORAGE Rewrite** - Simplest spec rewrite; validates new wire format in practice (completed 2026-04-07)
- [x] **Phase 4: NUB-RELAY Rewrite** - Highest-impact rewrite; publish via typed messages, no crypto in napplet API (completed 2026-04-07)
- [ ] **Phase 5: NUB-NOSTRDB Rewrite** - Event caching interface aligned with new wire format
- [ ] **Phase 6: NUB-IFC Rewrite** - Most prose-heavy; sender verification model updated
- [ ] **Phase 7: Pipes, Security Audit & Governance** - PIPES rewrite, cross-spec security audit, templates and registry

## Phase Details

### Phase 1: Foundation (COMPLETE)
**Goal**: SPEC.md exists as the authoritative reference for the two-layer architecture
**Status**: Complete — SPEC.md written by external agent
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05, FOUND-06
**Notes**: SPEC.md defines `{ "type": "domain.action", ...payload }` wire format instead of NIP-01 events. This is a larger architectural shift than originally planned — kind numbers are no longer in the wire protocol, `EventTemplate` concept replaced by typed message payloads, identity via `MessageEvent.source`. All subsequent phases must use this wire format.

### Phase 2: NUB-SIGNER Demotion
**Goal**: NUB-SIGNER PR is closed entirely (NIP-07 `window.nostr` is a core SPEC.md requirement, nothing left for a separate spec) and the IPC/IFC naming is resolved so all other spec rewrites reference the correct canonical name NUB-IFC
**Depends on**: Phase 1 (complete)
**Requirements**: SPEC-03, SPEC-07
**Plans:** 1 plan
Plans:
- [x] 02-01-PLAN.md — Close NUB-SIGNER PR, delete branch, update README.md and CLAUDE.md with NUB-IFC naming
**Success Criteria** (what must be TRUE):
  1. PR #1 (NUB-SIGNER) is closed on GitHub with an explanatory comment; remote branch deleted
  2. NUB-SIGNER row removed from README.md registry table
  3. A single canonical name (NUB-IFC) is used consistently in README.md and CLAUDE.md — no remaining NUB-IPC references

### Phase 3: NUB-STORAGE Rewrite
**Goal**: NUB-STORAGE is fully rewritten using the SPEC.md wire format (`storage.*` message types) — validating the new spec structure before higher-complexity specs use it
**Depends on**: Phase 2
**Requirements**: SPEC-02
**Success Criteria** (what must be TRUE):
  1. NUB-STORAGE defines `storage.*` message types (e.g., `storage.get`, `storage.set`, `storage.remove`, `storage.keys`) with typed payloads per SPEC.md wire format
  2. Shell Behavior section describes composite key scoping and any internal event mechanics without exposing them to napplet authors
  3. No reference to signing, keypairs, pubkeys, kind numbers, or cryptographic operations appears in napplet-facing sections
**Plans:** 1/1 plans complete
Plans:
- [x] 03-01-PLAN.md — Rewrite NUB-STORAGE.md wire format and update PR #3

### Phase 4: NUB-RELAY Rewrite
**Goal**: NUB-RELAY is fully rewritten using `relay.*` message types — the central interface of the milestone — with napplets signing events themselves via `window.nostr.signEvent()` (NIP-07) before publishing
**Depends on**: Phase 3
**Requirements**: SPEC-01
**Success Criteria** (what must be TRUE):
  1. NUB-RELAY defines `relay.publish`, `relay.subscribe`, `relay.query` (and corresponding result types) using SPEC.md wire format — `relay.publish` accepts a signed NostrEvent, napplets sign via `window.nostr.signEvent()` before calling publish
  2. Shell Behavior section describes how the runtime forwards signed events to relays and delivers incoming events — this detail is not visible to napplets beyond the message types
  3. NUB-RELAY does not reference NUB-SIGNER as a napplet-visible dependency
  4. `subscribe` and `query` result payloads are defined with clear types
**Plans:** 1/1 plans complete
Plans:
- [x] 04-01-PLAN.md — Rewrite NUB-RELAY.md wire format and update PR #2

### Phase 5: NUB-NOSTRDB Rewrite
**Goal**: NUB-NOSTRDB is fully rewritten using `nostrdb.*` message types — napplets can cache and query events received from subscriptions
**Depends on**: Phase 4
**Requirements**: SPEC-04
**Success Criteria** (what must be TRUE):
  1. NUB-NOSTRDB defines `nostrdb.*` message types for add, query, subscribe operations using SPEC.md wire format
  2. The `add` operation documents that events are externally-authored (received from relay subscriptions), not napplet-signed
  3. No napplet-authored signing operation appears in the napplet-facing sections of the spec
**Plans:** 1 plan
Plans:
- [x] 05-01-PLAN.md — Rewrite NUB-NOSTRDB.md wire format and update PR #4

### Phase 6: NUB-IFC Rewrite
**Goal**: NUB-IFC is fully rewritten using `ifc.*` message types — inter-frame communication without signed event wrappers, sender verification via shell's `MessageEvent.source` identity mapping
**Depends on**: Phase 4
**Requirements**: SPEC-05
**Success Criteria** (what must be TRUE):
  1. NUB-IFC defines `ifc.*` message types (e.g., `ifc.emit`, `ifc.on`) with no signing or cryptographic operations in napplet-facing sections
  2. Shell Behavior section documents sender verification via `MessageEvent.source` → napplet identity mapping (per SPEC.md)
  3. Security Considerations describes sender identity as shell-enforced — no per-message Schnorr signing as napplet responsibility
**Plans:** 1 plan
Plans:
- [ ] 06-01-PLAN.md — Rewrite NUB-IFC.md wire format and update PR #5

### Phase 7: Pipes, Security Audit & Governance
**Goal**: NUB-PIPES is rewritten as a final validation of the new spec structure; cross-spec Security Considerations audit confirms consistency; templates and registry updated
**Depends on**: Phase 6
**Requirements**: SPEC-06, SPEC-08, GOVN-01, GOVN-02, GOVN-03
**Success Criteria** (what must be TRUE):
  1. NUB-PIPES defines `pipes.*` message types with peer identity using opaque token or dTag — not raw pubkey
  2. All 6 spec Security Considerations sections use the runtime as the subject of MUST clauses for crypto operations — no spec says napplets MUST sign or verify
  3. TEMPLATE-WORD.md uses the SPEC.md wire format structure (`type` strings, payload shapes, shell behavior) so future specs follow the pattern
  4. TEMPLATE-NN.md is updated to reflect the new spec philosophy
  5. README registry table reflects current canonical names and runtime-internal classification for NUB-SIGNER
**Plans**: TBD
**UI hint**: no

## Progress

**Execution Order:**
Phases execute in numeric order: 1 (done) → 2 → 3 → 4 → 5 → 6 → 7

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | -- | Complete | 2026-04-07 |
| 2. NUB-SIGNER Demotion | 0/1 | Not started | - |
| 3. NUB-STORAGE Rewrite | 1/1 | Complete   | 2026-04-07 |
| 4. NUB-RELAY Rewrite | 1/1 | Complete   | 2026-04-07 |
| 5. NUB-NOSTRDB Rewrite | 1/1 | Complete   | 2026-04-07 |
| 6. NUB-IFC Rewrite | 0/1 | Planned | - |
| 7. Pipes, Security Audit & Governance | 0/? | Not started | - |
