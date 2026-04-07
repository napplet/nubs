# Project Research Summary

**Project:** NUB Spec Simplification — Zero-Crypto Napplet API
**Domain:** Protocol specification — postMessage wire protocol with host-delegated identity for sandboxed iframe apps
**Researched:** 2026-04-07
**Confidence:** HIGH

## Executive Summary

The NUB spec rewrite is a documentation and specification architecture project, not a software implementation. The goal is to impose a clean two-layer split across 6 existing draft specs (RELAY, STORAGE, IFC, NOSTRDB, SIGNER, PIPES): a spec layer where napplet authors see only domain-level primitives with zero crypto, and an implementation layer where shells handle signing, key derivation, ACL enforcement, and event construction invisibly. This pattern is established precedent — Figma plugins, Chrome extensions, NIP-07, and MCP Apps all follow the same model of host-mediated identity where app code never touches key material. The NUB specs already have the correct API shapes; the problem is that crypto implementation details have bled into the napplet-facing narrative.

The recommended approach is SPEC.md-first. Write the foundational document that defines the two-layer boundary, establishes canonical vocabulary, documents the "passthrough exception" for signed events received from relays, and demotes NUB-SIGNER to a runtime-internal interface. Without SPEC.md as a reference, parallel spec rewrites will independently invent different phrasings for the same architectural invariants, producing six inconsistent specs. Once SPEC.md is written, the spec rewrites follow a clear dependency order: STORAGE (simplest, validates the template), then SIGNER (establishes its runtime-internal status so other specs can correctly omit signing references), then RELAY (central), then NOSTRDB (shares RELAY data model), then IFC (depends on settled RELAY wire format), then PIPES (already nearest to the target model).

The primary risk is crypto re-leaking after the rewrites: stripping explicit signEvent() calls from the API surface but leaving NostrEvent as a return type (which exposes pubkey, sig, id to the napplet), or updating the API surface section without updating the Security Considerations section to match. The mitigation is treating each spec as a complete document rewrite — not a search-and-delete — and using a verification checklist that confirms subject-verb agreement on every MUST clause and every return type across all 6 specs.

---

## Key Findings

### Recommended Stack

The "stack" for this project is protocol vocabulary, not npm packages. NIP-5D already establishes the correct transport (postMessage with MessageEvent.source for sender identity), the correct auth model (verify-once AUTH handshake, then trust source — no per-message signing), and the correct wire format (verb arrays ["VERB", ...args]). The rewrite draws on NIP-01 verb semantics (REQ/EVENT/CLOSE/EOSE/OK), the NIP-46 remote signer pattern (napplet submits unsigned EventTemplate; runtime signs and returns result), and the MCP Apps host-tool-delegation model (sandboxed iframe sends intent; host executes using user's credentials; app sees only result).

**Core protocol building blocks:**
- **NIP-5D transport + AUTH**: Existing foundation — do not change. "Verify once, trust source" is the security model. Per-message Schnorr signing in NUB-IFC is redundant with this.
- **Verb arrays throughout**: ["VERB", ...args] with positional correlation IDs — not JSON-RPC. Stays consistent with NIP-01 idiom.
- **EventTemplate as napplet-facing write type**: { kind, tags, content } — no pubkey, no id, no sig. NIP-46 prior art. Shell adds crypto fields invisibly.
- **NUB-SIGNER as runtime-internal only**: The NIP-07 window.nostr proxy is NOT exposed to napplet iframes in the simplified model. NUB-SIGNER describes the shell's internal signing capability, not a napplet-callable interface.
- **NIP-26 must not appear**: Explicitly deprecated. Use NIP-5D HMAC-derived session key model instead.

### Expected Features

**Must have (table stakes for v0.1.0):**
- subscribe(filters, onEvent, onEose) returning a closeable handle — every sandboxed platform uses this shape
- publish(template) accepting unsigned EventTemplate — no signEvent() call in napplet code; spec description must not leak the NUB-SIGNER dependency
- query(filters) returning Promise of events — eliminates boilerplate subscribe+collect+EOSE+close
- Scoped storage with getItem/setItem/removeItem/keys — NUB-STORAGE API surface is already correct; only prose cleanup needed
- Topic-based emit(topic, payload) / on(topic, cb) — NUB-IFC API shape is correct; the signed-event language must move to shell behavior
- shell.supports("NUB-X") discovery documented consistently per spec
- SPEC.md foundational doc and updated TEMPLATE-WORD.md — required before any individual spec rewrite
- Resolve NUB-IPC vs NUB-IFC naming — README, template, and spec file must be consistent

**Should have (competitive differentiators):**
- Explicit two-layer split (Napplet API section vs Shell Behavior section) in every spec — makes the abstraction boundary unmissable
- "What the napplet never does" summary paragraph at the top of each spec — DX differentiator that communicates design intent and prevents future crypto re-leaks
- ACL capabilities declared by manifest, enforced by shell, invisible to napplet — document this pattern consistently across specs
- Wire Protocol section replacing "Event Kinds" section — event kind numbers are implementation details, not napplet-visible protocol
- NUB-CAPABILITIES foundational spec — locks in the capability model referenced by all 6 specs

**Defer (v0.1.x or later):**
- "If absent" graceful degradation section per spec — add after core rewrites stabilize
- NUB-PIPES full rewrite — currently unimplemented; validate wire format against a real implementation first
- NUB-NN message protocol specs — interface specs must stabilize before defining event semantics napplets agree on
- Formal maturity levels (draft/proposed/stable) with graduation criteria
- Schema validation / TypeScript CI on wire format examples

### Architecture Approach

The architecture is a strict two-layer document model enforced by SPEC.md. The spec layer defines only what napplet authors call — zero crypto types, no NIP-01 event kinds, no session key references. The implementation layer is what shells build — AUTH handshake, session keypair management, ACL enforcement, relay pool, event signing, signature verification. The key structural change to every spec is replacing the "Event Kinds" section (which exposes kind 29001/29003/29006 as if napplets construct them) with a "Wire Protocol" section (which shows only the postMessage verb shapes napplets send and receive) and a "Shell Behavior" section (which documents the runtime's crypto obligations using observable outcomes, not mechanisms).

**Major components:**
1. **SPEC.md** — Foundational doc. Defines two-layer boundary, EventTemplate type, NostrEvent passthrough exception, runtime-internal interface category, canonical glossary phrases, capability model reference. Written first.
2. **Updated TEMPLATE-WORD.md** — Enforces two-layer section structure for all future NUB specs. Written immediately after SPEC.md.
3. **NUB-SIGNER (runtime-internal)** — Demoted from napplet-callable to shell implementation guide. Written early so all other specs can correctly omit signing references.
4. **NUB-RELAY, NUB-STORAGE, NUB-IFC, NUB-NOSTRDB** — Core spec rewrites following SPEC.md vocabulary. RELAY is the highest-impact rewrite; STORAGE is the simplest and validates the template.
5. **NUB-PIPES** — Already nearest to the target model. Written last as a final validation pass.

### Critical Pitfalls

1. **Crypto leaks through return types** — Stripping signEvent() from the API surface but leaving Promise<NostrEvent> as the return type of publish(). The napplet sees pubkey, sig, id in the resolved value. Prevention: define return types in SPEC.md before any spec is rewritten; run a type audit after each spec draft.

2. **SPEC.md written after spec rewrites** — If SPEC.md is written post-hoc, each spec author makes local terminology decisions. The resulting six specs describe the same architectural invariant with six different phrasings. Prevention: SPEC.md must ship before the first individual spec PR is opened.

3. **Security Considerations sections left stale** — API surface gets simplified but Security Considerations still says "napplet signs" or "delegated session key" as a napplet responsibility. Creates contradictory contracts. Prevention: treat each spec as a full document rewrite; Security Considerations is a required output, not optional cleanup. Verify subject-verb on every MUST clause.

4. **NUB-SIGNER category ambiguity** — Left as napplet-callable in the registry while other specs no longer reference it. New contributors are confused; implementors re-expose window.nostr. Prevention: SPEC.md defines a "runtime-internal" interface category; README registry marks NUB-SIGNER as non-callable.

5. **Kind 29003 shared between NUB-IFC and NUB-STORAGE** — A wire format change to one spec silently breaks the other. Prevention: before any spec rewrite begins, produce a kind-number-to-spec mapping table; no two specs share a kind without documented rationale.

6. **NUB-NOSTRDB add() passthrough ambiguity** — After "napplets never sign" rule is applied, spec authors may change add(event: NostrEvent) to add(template: EventTemplate), which breaks the legitimate use case of caching externally-authored events received from subscriptions. Prevention: SPEC.md explicitly defines the passthrough exception; NUB-NOSTRDB references it by name.

---

## Implications for Roadmap

### Phase 1: Foundation — SPEC.md and Template
**Rationale:** SPEC.md is the dependency for everything else. Without it, parallel spec rewrites produce inconsistent terminology. This phase has no prerequisites and unblocks all subsequent phases.
**Delivers:** SPEC.md foundational doc + updated TEMPLATE-WORD.md + kind-number audit table. The two-layer boundary is defined. Vocabulary is canonical. The passthrough exception is documented. NUB-SIGNER's runtime-internal status is established.
**Addresses:** SPEC.md (FEATURES P1), TEMPLATE-WORD.md update (FEATURES P1), NUB-IFC naming resolution (FEATURES P1)
**Avoids:** Pitfall 2 (SPEC.md written post-hoc), Pitfall 5 (inconsistent identity vocabulary), Pitfall 6 (NUB-PIPES pubkey definition scope), Pitfall 4 (NOSTRDB passthrough ambiguity defined here)

### Phase 2: NUB-SIGNER Demotion
**Rationale:** NUB-SIGNER's demotion to runtime-internal must happen before any other spec is rewritten. Every other spec currently cross-references NUB-SIGNER for signing. Until SIGNER's new category is locked in, RELAY, IFC, and NOSTRDB cannot be rewritten cleanly.
**Delivers:** NUB-SIGNER spec rewritten as a shell implementation guide. Registry table updated. window.nostr removed from napplet-visible namespace. All other specs now have a canonical way to say "the shell handles signing" without citing SIGNER as a napplet dependency.
**Avoids:** Pitfall 2 (NUB-SIGNER category ambiguity), Integration gotcha (NUB-RELAY publish() calling NUB-SIGNER)

### Phase 3: NUB-STORAGE Rewrite (Template Validation)
**Rationale:** NUB-STORAGE has the cleanest existing API surface — getItem/setItem/removeItem/keys has no crypto in the call signatures. Ideal first spec to rewrite against the new TEMPLATE-WORD.md. Validates the template structure before tackling higher-complexity specs.
**Delivers:** NUB-STORAGE rewritten to the two-layer format. Template validated in practice. Composite key scoping and correlation IDs moved to Shell Behavior. No crypto in napplet-facing prose.
**Avoids:** Pitfall 5 (first use of canonical SPEC.md glossary phrases in a real spec)

### Phase 4: NUB-RELAY Rewrite
**Rationale:** NUB-RELAY is the highest-impact spec — it underpins relay access, and its publish() simplification is the central goal of the milestone. Write it after STORAGE has validated the template structure.
**Delivers:** NUB-RELAY with unsigned EventTemplate publish, crypto removed from napplet API narrative, event kinds moved to Wire Protocol / Shell Behavior. publish() no longer references NUB-SIGNER. Return type no longer exposes NostrEvent.
**Uses:** NIP-01 verb semantics, NIP-46 EventTemplate pattern, NIP-5D session key model
**Avoids:** Pitfall 1 (crypto leaks through return types), Pitfall 3 (Security Considerations stale)

### Phase 5: NUB-NOSTRDB Rewrite
**Rationale:** Shares the EventTemplate / NostrEvent data model with RELAY. Write after RELAY so data type definitions are settled. The passthrough exception (add() accepts externally-authored signed events) is already defined in SPEC.md.
**Delivers:** NUB-NOSTRDB with add() passthrough semantics documented, query() and subscribe() returning read-only event data, no napplet-authored signing in the spec.
**Avoids:** Pitfall 4 (NOSTRDB add() passthrough ambiguity)

### Phase 6: NUB-IFC Rewrite
**Rationale:** NUB-IFC depends on the RELAY wire format being settled (IFC routes through the relay postMessage bus internally). The IFC rewrite is the most prose-heavy — it must update the Security Considerations sender-verification model (from Schnorr signature verification to MessageEvent.source identity).
**Delivers:** NUB-IFC with emit(topic, payload) / on(topic, cb) fully crypto-free, sender verification via MessageEvent.source documented in Shell Behavior, topic convention appendix separated from normative wire protocol.
**Avoids:** Pitfall 3 (IFC emit loses identity silently), Integration gotcha (NUB-IFC / NUB-PIPES cross-reference rationale updated)

### Phase 7: NUB-PIPES Rewrite
**Rationale:** NUB-PIPES is already the closest to the target model and has no spec-layer dependencies on other NUBs. Write last as a final validation pass of the two-layer pattern. Main change: removing peer.pubkey from PipeHandle or scoping it per SPEC.md definition.
**Delivers:** NUB-PIPES with PipeHandle.peer exposing only dTag (or explicitly scoped pubkey-as-display-only per SPEC.md), PIPE_ACK wire format updated, "not authenticated" error language replaced with "target not available."
**Avoids:** Pitfall 6 (NUB-PIPES pubkey exposure)

### Phase Ordering Rationale

- SPEC.md is an absolute prerequisite for all spec rewrites — it defines the vocabulary every spec uses. Most important finding from PITFALLS.md.
- NUB-SIGNER demotion must precede RELAY, IFC, and NOSTRDB rewrites because those specs currently cite SIGNER as a napplet-visible dependency.
- STORAGE validates the template before it is applied to more complex specs — low-risk first use.
- RELAY before NOSTRDB and IFC because both have shell-layer dependencies on relay infrastructure; settling RELAY's wire format first prevents ripple edits.
- PIPES last because it is already clean and serves as a final quality check.

### Research Flags

Phases with well-documented patterns (no additional research needed):
- **Phase 1 (SPEC.md):** Writing-only task; architecture fully defined by existing specs and research.
- **Phase 2 (NUB-SIGNER):** Category change well-understood; no new protocol design required.
- **Phase 3 (NUB-STORAGE):** API surface unchanged; prose cleanup only.
- **Phase 4 (NUB-RELAY):** NIP-01/NIP-46 patterns fully documented in STACK.md.
- **Phase 7 (NUB-PIPES):** Already closest to target; changes are minor and bounded.

Phases that may benefit from deeper review before drafting:
- **Phase 5 (NUB-NOSTRDB):** The passthrough exception may generate community discussion. Draft the SPEC.md passthrough section with examples before the NUB-NOSTRDB PR is opened.
- **Phase 6 (NUB-IFC):** Sender verification model changes from Schnorr to MessageEvent.source — a security model change that shell implementers need explicit guidance on.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All protocol vocabulary drawn from canonical sources: NIP-5D (local file), NIP-01, NIP-42, NIP-46 (official NIPs repo), MCP Apps (live production spec). |
| Features | HIGH | Feature analysis based on direct inspection of all 6 draft spec files plus well-documented analogues (Figma, Chrome extensions, NIP-07). MVP scope is conservative and grounded in existing work. |
| Architecture | HIGH | Derived from direct analysis of existing specs — not inferred from training data. Two-layer model is explicitly a stated goal in PROJECT.md. Build order derived from actual dependency analysis. |
| Pitfalls | HIGH | All pitfalls identified from direct spec text analysis — specific sentences and line-level issues cited. Not speculative. |

**Overall confidence:** HIGH

### Gaps to Address

- **NUB-CAPABILITIES scope**: All 6 specs reference capabilities (relay:read, storage:write) but the capability grant mechanism is undefined. v0.1.0 specs will need a placeholder reference in SPEC.md that can be upgraded to a normative citation when NUB-CAPABILITIES ships.

- **NUB-IFC vs NUB-IPC naming**: CLAUDE.md confirms the rename to IFC is decided. README registry, TEMPLATE-WORD.md, and spec file need to be updated atomically — not left as a partial rename across phases.

- **Kind 29003 conflict audit**: Both NUB-IFC and NUB-STORAGE currently route through kind 29003. A pre-rewrite audit is needed to determine whether this is intentional sharing or an accidental collision. Must be resolved in Phase 1 before any wire format sections are written.

---

## Sources

### Primary (HIGH confidence)
- `/home/sandwich/Develop/napplet/specs/NIP-5D.md` — transport, AUTH handshake, wire format, security model
- `https://github.com/nostr-protocol/nips/blob/master/01.md` — REQ/EVENT/CLOSE/EOSE/OK/CLOSED/NOTICE message shapes
- `https://github.com/nostr-protocol/nips/blob/master/46.md` — remote signer, EventTemplate pattern
- `https://nips.nostr.com/42` — AUTH challenge-response, kind 22242
- `https://nips.nostr.com/26` — NIP-26 deprecation
- `https://modelcontextprotocol.io/extensions/apps/overview` — MCP Apps host-delegated tool calling from sandboxed iframe (Jan 2026)
- `https://developer.chrome.com/docs/extensions/develop/concepts/messaging` — sendMessage/onMessage, MessageEvent.source identity
- `https://developers.figma.com/docs/plugins/how-plugins-run/` — no auth/crypto in plugin code
- `https://ably.com/docs/auth/capabilities` — capability model scoped by server-issued tokens
- Direct inspection of all 6 NUB draft specs (nub-relay, nub-signer, nub-ipc, nub-storage, nub-nostrdb, nub-pipes branches)
- `.planning/codebase/ARCHITECTURE.md`, `.planning/codebase/STRUCTURE.md`, `.planning/codebase/CONCERNS.md`, `.planning/PROJECT.md`

### Secondary (MEDIUM confidence)
- `https://engineering.mercari.com/en/blog/entry/20220930-building-secure-apps-using-web-workers/` — UUID correlation pattern, app-layer zero-crypto pattern
- `https://www.figma.com/plugin-docs/api/properties/figma-ui-postmessage/` — message queuing on startup, minimal envelope pattern
- `https://github.com/krakenjs/zoid` — xprops identity delegation, auth-on-open model

---
*Research completed: 2026-04-07*
*Ready for roadmap: yes*
