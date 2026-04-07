# NUBs (Napplet Unified Blueprints)

## What This Is

The extension proposal system for the napplet protocol. NUBs are markdown specs that extend NIP-5D with interface and message protocol definitions for the napplet ecosystem. This repo hosts the governance docs, templates, and spec proposals (via PRs).

## Core Value

Napplet authors never touch crypto. They subscribe(), publish(), query(). The runtime handles identity, signing, ACL, and routing internally. Specs define the simplest possible wire protocol.

## Current Milestone: v0.1.0 Spec Simplification

**Goal:** Rewrite NUB specs so napplets have zero crypto responsibilities — the wire protocol is just message types + payloads, and the runtime handles all identity/signing/ACL internally.

**Target features:**
- SPEC.md foundational document defining the two-layer architecture (spec layer vs implementation layer)
- Revised NUB interface specs (RELAY, STORAGE, SIGNER, NOSTRDB, IFC, PIPES) reflecting simplified napplet API
- Clear boundary: napplet never imports a signing library, never handles keypairs, never thinks about AUTH
- Updated templates reflecting the new spec philosophy

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

(None yet — ship to validate)

### Active

<!-- Current scope. Building toward these. -->

- [ ] SPEC.md defining two-layer architecture (spec layer vs implementation layer)
- [ ] Simplified NUB-RELAY spec (subscribe/publish/query, no crypto)
- [ ] Simplified NUB-STORAGE spec (scoped KV, identity handled by runtime)
- [ ] Simplified NUB-SIGNER spec (runtime-internal signing, napplet-invisible)
- [ ] Simplified NUB-NOSTRDB spec (local event DB, no crypto in napplet API)
- [ ] Simplified NUB-IFC spec (inter-frame communication, identity by shell)
- [ ] Simplified NUB-PIPES spec (point-to-point, auth handled by runtime)
- [ ] Updated TEMPLATE-WORD.md reflecting new philosophy
- [ ] Updated TEMPLATE-NN.md reflecting new philosophy

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- NUB-NN message protocol specs — v0.1.0 focuses on interface specs only
- SDK/shim implementation — specs first, code follows in napplet/napplet repo
- NIP-5D changes — core protocol lives in nostr-protocol/nips

## Context

- 6 existing DRAFT PRs open for the original NUB specs (may need crypto concerns stripped)
- NIP-5D defines core: transport (postMessage), authentication (REGISTER -> IDENTITY -> AUTH), extension discovery (shell.supports()), security model
- Two-layer insight: spec layer (napplet-facing, no crypto) vs implementation layer (runtime-internal, full Nostr crypto identity)
- Related repos: napplet/napplet (SDK), sandwichfarm/hyprgate (reference shell), nostr-protocol/nips (NIP-5D)

## Constraints

- **Format**: All specs are markdown files following established templates
- **Process**: Specs submitted as PRs, NIP-style informal governance
- **Compatibility**: Must remain consistent with NIP-5D core protocol
- **Naming**: NUB-WORD for interfaces, NUB-NN for message protocols

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Napplets have zero crypto responsibilities | Simpler spec, thinner shim, better DX — runtime handles identity internally | -- Pending |
| Wire protocol = message types + payloads only | Keeps spec layer minimal, implementation details stay in runtime | -- Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? -> Move to Out of Scope with reason
2. Requirements validated? -> Move to Validated with phase reference
3. New requirements emerged? -> Add to Active
4. Decisions to log? -> Add to Key Decisions
5. "What This Is" still accurate? -> Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-07 after milestone v0.1.0 initialization*
