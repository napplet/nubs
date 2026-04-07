# Requirements: NUBs (Napplet Unified Blueprints)

**Defined:** 2026-04-07
**Core Value:** Napplet authors never touch crypto. They subscribe(), publish(), query(). The runtime handles identity, signing, ACL, and routing internally.

## v0.1.0 Requirements

Requirements for initial spec simplification. Each maps to roadmap phases.

### Foundation (Complete)

- [x] **FOUND-01**: SPEC.md defines the two-layer architecture (spec layer vs implementation layer) with clear boundary rules
- [x] **FOUND-02**: SPEC.md defines canonical wire format (`{ "type": "domain.action", ...payload }`) replacing NIP-01 event-based protocol
- [x] **FOUND-03**: SPEC.md architecture implicitly supports runtime-internal interfaces (napplets have no access to `window.nostr`)
- [x] **FOUND-04**: SPEC.md wire format eliminates napplet signing entirely — napplets send typed messages, shell handles all crypto
- [x] **FOUND-05**: SPEC.md resolves "no crypto" scope — napplets have no signing, no keypairs, no `window.nostr` access
- [x] **FOUND-06**: Kind numbers removed from wire protocol entirely — NIP-01 event kinds are now shell implementation details only

### Interface Specs

- [x] **SPEC-01**: NUB-RELAY rewritten — `publish(event)` accepts signed NostrEvent, napplet signs via `window.nostr.signEvent()`, relay.* wire format
- [x] **SPEC-02**: NUB-STORAGE rewritten — crypto implementation details removed from napplet-facing sections
- [x] **SPEC-03**: NUB-SIGNER reframed as runtime-internal interface with optional user-consent-only API surface
- [x] **SPEC-04**: NUB-NOSTRDB rewritten — passthrough exception applied to `add()`, no napplet-authored signatures
- [x] **SPEC-05**: NUB-IFC rewritten — drop signed event wrappers, sender verification via `MessageEvent.source`
- [x] **SPEC-06**: NUB-PIPES rewritten — peer identity uses opaque token or dTag (per FOUND-05 decision), not raw pubkey
- [x] **SPEC-07**: NUB-IFC naming resolved — canonical name decided and applied consistently (IPC vs IFC)
- [x] **SPEC-08**: Security Considerations sections updated in all 6 specs to reflect runtime-owned crypto model

### Templates & Governance

- [x] **GOVN-01**: TEMPLATE-WORD.md updated with two-layer section structure (Napplet API / Wire Protocol / Shell Behavior / Implementation Reference)
- [x] **GOVN-02**: TEMPLATE-NN.md updated to reflect new spec philosophy
- [x] **GOVN-03**: README.md registry table updated to reflect revised spec status and any naming changes

## Future Requirements

### v0.1.x

- **NUB-NN-01**: First message protocol spec using simplified architecture
- **CAPS-01**: NUB-CAPABILITIES spec for runtime capability negotiation
- **IMPL-01**: Reference shim implementation reflecting simplified spec

## Out of Scope

| Feature | Reason |
|---------|--------|
| NUB-NN message protocol specs | v0.1.0 focuses on interface specs only |
| SDK/shim implementation | Specs first, code follows in napplet/napplet repo |
| NIP-5D changes | Core protocol lives in nostr-protocol/nips, not this repo |
| IDENTITY message privkey redesign | NIP-5D concern, not NUB concern (research flagged it) |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 | Phase 1 | Complete |
| FOUND-02 | Phase 1 | Complete |
| FOUND-03 | Phase 1 | Complete |
| FOUND-04 | Phase 1 | Complete |
| FOUND-05 | Phase 1 | Complete |
| FOUND-06 | Phase 1 | Complete |
| SPEC-01 | Phase 4 | Complete |
| SPEC-02 | Phase 3 | Complete |
| SPEC-03 | Phase 2 | Complete |
| SPEC-04 | Phase 5 | Complete |
| SPEC-05 | Phase 6 | Complete |
| SPEC-06 | Phase 7 | Complete |
| SPEC-07 | Phase 2 | Complete |
| SPEC-08 | Phase 7 | Complete |
| GOVN-01 | Phase 7 | Complete |
| GOVN-02 | Phase 7 | Complete |
| GOVN-03 | Phase 7 | Complete |

**Coverage:**
- v0.1.0 requirements: 17 total
- Mapped to phases: 17
- Unmapped: 0
- Complete: 6 (Foundation)

---
*Requirements defined: 2026-04-07*
*Last updated: 2026-04-07 after roadmap creation*
