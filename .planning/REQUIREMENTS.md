# Requirements: NUBs (Napplet Unified Blueprints)

**Defined:** 2026-04-07
**Core Value:** Napplet authors never touch crypto. They subscribe(), publish(), query(). The runtime handles identity, signing, ACL, and routing internally.

## v0.1.0 Requirements

Requirements for initial spec simplification. Each maps to roadmap phases.

### Foundation

- [ ] **FOUND-01**: SPEC.md defines the two-layer architecture (spec layer vs implementation layer) with clear boundary rules
- [ ] **FOUND-02**: SPEC.md defines canonical type vocabulary (`EventTemplate`, wire message shapes, opaque identity tokens)
- [ ] **FOUND-03**: SPEC.md defines the "runtime-internal interface" category for specs like NUB-SIGNER
- [ ] **FOUND-04**: SPEC.md defines the passthrough exception (napplet can forward externally-authored signed events, cannot author signatures)
- [ ] **FOUND-05**: SPEC.md resolves "no crypto" scope: no signing operations vs no raw pubkeys ever (PIPES peer identity decision)
- [ ] **FOUND-06**: Kind 29003 cross-spec audit completed (shared by IFC and STORAGE)

### Interface Specs

- [ ] **SPEC-01**: NUB-RELAY rewritten — `publish(template)` is self-contained, no `signEvent()` in napplet path, shell signs internally
- [ ] **SPEC-02**: NUB-STORAGE rewritten — crypto implementation details removed from napplet-facing sections
- [ ] **SPEC-03**: NUB-SIGNER reframed as runtime-internal interface with optional user-consent-only API surface
- [ ] **SPEC-04**: NUB-NOSTRDB rewritten — passthrough exception applied to `add()`, no napplet-authored signatures
- [ ] **SPEC-05**: NUB-IFC rewritten — drop signed event wrappers, sender verification via `MessageEvent.source`
- [ ] **SPEC-06**: NUB-PIPES rewritten — peer identity uses opaque token or dTag (per FOUND-05 decision), not raw pubkey
- [ ] **SPEC-07**: NUB-IFC naming resolved — canonical name decided and applied consistently (IPC vs IFC)
- [ ] **SPEC-08**: Security Considerations sections updated in all 6 specs to reflect runtime-owned crypto model

### Templates & Governance

- [ ] **GOVN-01**: TEMPLATE-WORD.md updated with two-layer section structure (Napplet API / Wire Protocol / Shell Behavior / Implementation Reference)
- [ ] **GOVN-02**: TEMPLATE-NN.md updated to reflect new spec philosophy
- [ ] **GOVN-03**: README.md registry table updated to reflect revised spec status and any naming changes

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
| FOUND-01 | — | Pending |
| FOUND-02 | — | Pending |
| FOUND-03 | — | Pending |
| FOUND-04 | — | Pending |
| FOUND-05 | — | Pending |
| FOUND-06 | — | Pending |
| SPEC-01 | — | Pending |
| SPEC-02 | — | Pending |
| SPEC-03 | — | Pending |
| SPEC-04 | — | Pending |
| SPEC-05 | — | Pending |
| SPEC-06 | — | Pending |
| SPEC-07 | — | Pending |
| SPEC-08 | — | Pending |
| GOVN-01 | — | Pending |
| GOVN-02 | — | Pending |
| GOVN-03 | — | Pending |

**Coverage:**
- v0.1.0 requirements: 17 total
- Mapped to phases: 0
- Unmapped: 17

---
*Requirements defined: 2026-04-07*
*Last updated: 2026-04-07 after initial definition*
