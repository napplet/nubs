# NUBs (Napplet Unified Blueprints)

## What This Is

The extension proposal system for the napplet protocol. NUBs are markdown specs that extend NIP-5D with interface and message protocol definitions for the napplet ecosystem. This repo hosts the governance docs, templates, and spec proposals (via PRs). Four interface specs are active: NUB-RELAY, NUB-STORAGE, NUB-NOSTRDB, NUB-IFC.

## Core Value

Napplet authors use simple typed messages over postMessage. The shell handles identity, relay connections, storage scoping, and signing. Specs define the simplest possible wire protocol using NIP-5D's `{ "type": "domain.action", ...payload }` format.

## Requirements

### Validated

- ✓ SPEC.md (NIP-5D) defines wire format, identity model, NUB extension framework — v0.1.0
- ✓ NUB-RELAY rewritten with `relay.*` typed messages, napplet signs via `window.nostr` — v0.1.0
- ✓ NUB-STORAGE rewritten with `storage.*` typed messages, no crypto in napplet API — v0.1.0
- ✓ NUB-SIGNER eliminated (NIP-07 is a core SPEC.md requirement) — v0.1.0
- ✓ NUB-NOSTRDB rewritten with `nostrdb.*` typed messages — v0.1.0
- ✓ NUB-IFC rewritten with `ifc.*` typed messages + `ifc.channel.*` pre-authorized channels — v0.1.0
- ✓ NUB-PIPES absorbed into NUB-IFC as channels — v0.1.0
- ✓ Templates updated to reflect wire format architecture — v0.1.0
- ✓ Cross-spec security audit passed — v0.1.0

### Active

(None — next milestone will define new requirements)

### Out of Scope

- NUB-NN message protocol specs — interface specs came first
- SDK/shim implementation — specs first, code follows in napplet/napplet repo
- NIP-5D upstream changes — core protocol lives in nostr-protocol/nips

## Context

- 4 active NUB interface specs on PR branches (RELAY #2, STORAGE #3, NOSTRDB #4, IFC #5)
- NIP-5D (SPEC.md) defines: transport (postMessage), identity (MessageEvent.source), wire format ({ type, ...payload }), NUB extension framework
- Shells MUST provide NIP-07 `window.nostr` to napplet iframes (SPEC.md line 33)
- Related repos: napplet/napplet (SDK), sandwichfarm/hyprgate (reference shell), nostr-protocol/nips (NIP-5D)

## Constraints

- **Format**: All specs are markdown files following TEMPLATE-WORD.md / TEMPLATE-NN.md
- **Process**: Specs submitted as PRs, NIP-style informal governance
- **Compatibility**: Must remain consistent with NIP-5D (SPEC.md) wire format and identity model
- **Naming**: NUB-WORD for interfaces (short domain name for discovery), NUB-NN for message protocols

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Wire format: `{ "type": "domain.action", ...payload }` | Simple typed messages over postMessage, consistent with NIP-5D | ✓ Good — all 4 specs use it |
| Napplets sign via `window.nostr.signEvent()` | NIP-07 is shell-provided (SPEC.md line 33), napplet signs before publish | ✓ Good — simpler than shell-signs-for-you |
| NUB-SIGNER eliminated | NIP-07 is a core SPEC.md requirement, nothing left for a separate spec | ✓ Good |
| NUB-PIPES absorbed into NUB-IFC channels | `ifc.channel.*` pre-authorized channels cover pipes use case, one less spec | ✓ Good |
| `window.nostrdb` namespace (not `window.napplet.nostrdb`) | Matches nostrdb library convention | ✓ Good — documented with design note |
| Sender identity uses dTag (not pubkey) in IFC | Opaque identifier, consistent with NIP-5D identity model | ✓ Good |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition:**
1. Requirements invalidated? -> Move to Out of Scope with reason
2. Requirements validated? -> Move to Validated with phase reference
3. New requirements emerged? -> Add to Active
4. Decisions to log? -> Add to Key Decisions
5. "What This Is" still accurate? -> Update if drifted

**After each milestone:**
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-07 after v0.1.0 milestone*
