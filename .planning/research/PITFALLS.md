# Pitfalls Research

**Domain:** Protocol spec simplification — hiding crypto/identity from app developers across 6 interdependent specs
**Researched:** 2026-04-07
**Confidence:** HIGH (derived from direct spec analysis, not training data)

---

## Critical Pitfalls

### Pitfall 1: Crypto Leaks Through the Publish API

**What goes wrong:**
The current NUB-RELAY spec has `publish(template: EventTemplate)` and explicitly says "Signs the event template via `window.nostr.signEvent()` (NUB-SIGNER)." After simplification, the napplet still receives a `NostrEvent` back (signed, with `id`, `sig`, `pubkey` populated). Even if the napplet never calls `signEvent()` directly, it still handles signed event objects, exposing crypto concepts. The abstraction says "no crypto" but the return type is `Promise<NostrEvent>` — the napplet sees key material in the returned object.

**Why it happens:**
Spec authors strip the explicit signing call but forget to audit the return type. `NostrEvent` carries `pubkey`, `id`, and `sig`. A napplet that inspects `event.pubkey` after publish is now reasoning about identity in a way the simplification intended to remove.

**How to avoid:**
Define a `PublishedEvent` return type (or just return `void`/`Promise<string>` for event ID only) that does not expose signing artifacts to the napplet. The SPEC.md boundary definition must state explicitly: napplet-facing types MUST NOT contain `pubkey`, `sig`, or raw keypair material. Run a type audit against all six API surfaces before writing a single word of new spec prose.

**Warning signs:**
- Any return type referencing `NostrEvent` (as opposed to a stripped `AppEvent` or `EventRecord`)
- `publish()` resolving with a full signed event object
- Any spec note saying "returns the signed event" pointing at napplet code

**Phase to address:**
Phase 1 — SPEC.md boundary definition. This must be resolved before touching any individual NUB spec, because it defines the vocabulary all 6 rewrites will use.

---

### Pitfall 2: NUB-SIGNER Becomes Invisible But the Interface Contract Remains

**What goes wrong:**
The goal is "napplet never imports a signing library, never handles keypairs, never thinks about AUTH." But NUB-SIGNER currently exposes `window.nostr` with `signEvent()`, `getPublicKey()`, `nip04.encrypt()`, `nip44.encrypt()`. After stripping crypto from the napplet-facing layer, NUB-SIGNER has no napplet-visible API surface left — it becomes entirely a runtime-internal interface. Spec authors may either: (a) delete NUB-SIGNER entirely (breaking the registry and any spec that cross-references it), or (b) keep the spec but not clearly mark what role it plays in the simplified architecture.

**Why it happens:**
The two-layer model requires NUB-SIGNER to shift from "app-callable" to "runtime-internal." This is a category change, not just a simplification. The template has no affordance for "internal-only interfaces that napplets cannot call directly." Without a defined category for this, each spec author handles the tension differently.

**How to avoid:**
SPEC.md must define a third interface category: "runtime-internal interfaces." NUB-SIGNER stays in the registry with an explicit note that it is not callable from napplet code — it is a shell capability used internally when another interface (NUB-RELAY's `publish()`, NUB-IPC's `emit()`) needs signing. The template should gain a `Layer: spec | runtime-internal` field. This prevents spec authors from accidentally re-exposing signing methods to napplets when they try to "keep" NUB-SIGNER.

**Warning signs:**
- NUB-SIGNER spec draft still includes `window.nostr` in its namespace
- `getPublicKey()` or `signEvent()` appearing in any napplet-facing API surface in any of the 6 specs
- Any spec saying "napplet calls NUB-SIGNER to..." rather than "the runtime uses NUB-SIGNER when..."

**Phase to address:**
Phase 1 — SPEC.md layer model definition. The distinction between spec-layer (napplet-callable) and implementation-layer (runtime-internal) must be written before NUB-SIGNER is rewritten.

---

### Pitfall 3: IPC Emit Loses Identity Silently

**What goes wrong:**
NUB-IPC's `emit()` currently builds a signed kind 29003 event with a delegated session key. After simplification, the napplet just calls `emit(topic, payload)` — it never touches the key. But the Security Considerations section currently says: "IPC messages are signed with the napplet's delegated session key. The shell verifies signatures before routing." After rewriting to hide crypto, this section must be updated to reflect that the shell signs on behalf of the napplet. If the security section is not updated in sync with the API surface, reviewers and shell implementers read two contradictory contracts: the API says "just emit," the security section says "napplet signs."

**Why it happens:**
Sections are edited independently. The API surface gets simplified, the security considerations stay stale. Spec rewrites typically start at the top (description, API) and tire before reaching security.

**How to avoid:**
Treat each spec's Security Considerations section as a required output of the rewrite, not optional cleanup. Write a rewrite checklist that explicitly requires: after changing the API surface, re-read every sentence in Security Considerations for subject-verb agreement — specifically whether "napplet" or "runtime/shell" is the actor for any crypto operation.

**Warning signs:**
- Security section says "napplet signs" anywhere after the API surface has been stripped of explicit signing calls
- Security section still references `delegated session key` as something the napplet manages
- Wire format examples in Event Kinds section showing napplet-populated `sig` or `pubkey` fields

**Phase to address:**
Phase 2 — individual spec rewrites. Each NUB spec must be treated as a complete document rewrite, not a search-and-delete of signing references.

---

### Pitfall 4: NUB-NOSTRDB's `add()` Requires Signed Events — But Who Signs?

**What goes wrong:**
NUB-NOSTRDB has `add(event: NostrEvent): Promise<boolean>`. This method inserts an event into the local database. In the current spec, it says "Returns `true` on success, `false` if the event was rejected (duplicate, invalid signature, etc.)." The phrase "invalid signature" implies the napplet is expected to provide a fully signed `NostrEvent`, with `sig`, `id`, and `pubkey` already populated. After simplification where napplets never sign anything, what does `add()` accept? An unsigned template? A pre-signed event obtained from a relay? This is the most ambiguous case in the 6 specs because the napplet may legitimately want to add events it received from subscriptions (already signed, from external authors), but it must never sign its own events.

**Why it happens:**
The "zero crypto for napplets" rule has an implicit exception: napplets can pass through signed events they received from the relay/DB without needing to sign them. The spec does not distinguish between "napplet-authored unsigned content" and "externally-authored signed events the napplet is caching." The simplification goal (strip napplet crypto) collapses this distinction.

**How to avoid:**
SPEC.md must define the "passthrough" case explicitly: napplets MAY pass `NostrEvent` objects (fully signed, received from subscriptions or queries) back to `add()`. They MUST NOT sign new events themselves. The `add()` signature can remain `add(event: NostrEvent)` — the constraint is behavioral (napplet receives the event from somewhere else, does not generate the signature). NUB-NOSTRDB should document this in a "Napplet Responsibilities" note: "Events passed to `add()` MUST originate from `subscribe()` or `query()` responses, not be locally authored."

**Warning signs:**
- `add()` spec example shows napplet constructing a new event object with a `sig` field it populated
- Security section says "the napplet MUST sign the event before calling add()"
- `add()` accepting `EventTemplate` (unsigned) rather than `NostrEvent` (already signed)

**Phase to address:**
Phase 2 — NUB-NOSTRDB rewrite. Requires the passthrough distinction from SPEC.md to already be defined.

---

### Pitfall 5: Inconsistency Across 6 Specs on How Identity Is Described

**What goes wrong:**
Each spec currently describes identity differently. NUB-RELAY says "the shell signs via NUB-SIGNER." NUB-IPC says "the napplet's delegated session key." NUB-STORAGE says "scoped by composite key `(dTag, aggregateHash)`" (not a signing identity, a content hash identity). NUB-PIPES says the pipe carries `peer: { pubkey: string; dTag: string }` (the peer's pubkey is visible to the napplet). After rewriting 6 specs independently, each author will phrase the runtime-handles-identity invariant differently unless a shared vocabulary is established first. Six differently-phrased security sections about the same architectural property creates confusion for shell implementers reading across specs.

**Why it happens:**
Parallel spec rewrites without a shared terminology glossary. The SPEC.md document is intended to establish this, but if it is written at too high a level or without specific canonical phrases, each spec author improvises.

**How to avoid:**
SPEC.md must include a glossary of canonical phrases:
- "The runtime signs this message on behalf of the napplet" (not "the napplet signs")
- "The shell's identity context is opaque to the napplet" (not "napplets cannot see keys")
- "The napplet's session identity is established by the AUTH handshake and managed by the shell"
These phrases should be used verbatim in each spec's Security Considerations section (copy-paste is acceptable here — consistency is more important than elegance). When the 6 specs are reviewed side by side, the same phrase appearing in each section confirms the invariant was applied uniformly.

**Warning signs:**
- Any two specs describing the same architectural invariant (e.g., "signing is hidden from napplets") using different subject-verb constructions
- One spec saying "the shell" signs while another says "the runtime" signs (same concept, different names)
- One spec omitting the signing responsibility entirely from Security Considerations

**Phase to address:**
Phase 1 — SPEC.md glossary. Before any of the 6 spec rewrites begin.

---

### Pitfall 6: NUB-PIPES Wire Format Exposes Session Pubkeys

**What goes wrong:**
NUB-PIPES `PIPE_ACK` delivers `{"peer": "<pubkey>", "peerDTag": "<dTag>"}` to the napplet. The napplet receives the peer's session pubkey directly. In a fully simplified model where napplets have zero crypto responsibilities, receiving a raw pubkey is at minimum inconsistent — at worst, it requires the napplet to reason about Nostr identity for peer validation. The spec says napplets can immediately call `pipe.send()` after receiving `PIPE_ACK` without validating the pubkey. But the `PipeHandle.peer` field is typed `{ pubkey: string; dTag: string }`, and the current spec does not say the napplet should ignore `peer.pubkey`.

**Why it happens:**
The pipe's identity handshake exists for shell-level authentication, but the identity data gets forwarded into the napplet-visible `PipeHandle` as a debugging/informational field. The spec does not consider whether exposing a raw pubkey violates the "no crypto" boundary, because the napplet isn't being asked to DO anything with the pubkey — it's just there.

**How to avoid:**
Decide during Phase 1 (SPEC.md) whether "no crypto" means "napplets never see raw pubkeys" or just "napplets never perform signing operations." If the former, `PipeHandle.peer` should expose only `dTag` and an opaque identity token. If the latter, document explicitly in SPEC.md that receiving pubkeys for informational display is acceptable, and each spec that exposes a pubkey must include a note that the napplet MUST NOT use the pubkey for cryptographic operations. Do not leave this ambiguous — it will resurface in every spec review.

**Warning signs:**
- `PipeHandle.peer.pubkey` in the API surface while other specs have removed pubkey exposure
- Any spec showing napplet code that calls `nostr.getPublicKey()` and compares the result to a received pubkey
- NUB-PIPES Security Considerations section not addressing what the napplet is allowed to do with the received pubkey

**Phase to address:**
Phase 1 — "no crypto" definition in SPEC.md. Phase 2 — NUB-PIPES rewrite, which is already marked `unimplemented` and may be the best candidate to draft fresh against the new model.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Rewrite only the API Surface section of each spec, leave Shell Behavior and Security Considerations unchanged | Faster per-spec rewrite | Security sections remain contradictory; shell implementers follow the old signing model | Never — this causes silent implementation divergence |
| Leave NUB-SIGNER spec unchanged and just say "napplets don't use it" | Avoids hard decision about spec category | NUB-SIGNER still appears in the registry as a napplet-callable interface; confuses new contributors | Never for the v0.1.0 milestone |
| Write SPEC.md after the individual spec rewrites as a post-hoc summary | Lets spec content drive architecture document | SPEC.md cannot define the boundary before it's enforced; each spec author makes local decisions; SPEC.md becomes descriptive not prescriptive | Never — SPEC.md must come first |
| Use `NostrEvent` as the napplet-facing type everywhere to avoid defining new types | Avoids type proliferation | Forces napplets to handle crypto fields they shouldn't need; contradicts the zero-crypto goal | Acceptable only for passthrough cases (events received from subscriptions) that the spec explicitly documents |
| Handle IPC/STORAGE/NOSTRDB as NIP-01 events signed by napplets but claim "that's just the wire format, not crypto" | Preserves existing wire format | The signing cost (Schnorr per message) is real and the napplet IS doing crypto whether the spec calls it that | Never — this is exactly the leaky abstraction the milestone is trying to fix |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| NUB-RELAY publish() calling NUB-SIGNER | Rewriting NUB-RELAY publish() in isolation while NUB-SIGNER spec still exposes `window.nostr.signEvent()` to napplets | Rewrite NUB-SIGNER first (or simultaneously) so the signing call is removed from the napplet-visible surface before NUB-RELAY references it |
| NUB-IPC emit() building kind 29003 events | Treating the kind 29003 wire format as an implementation detail separate from the API surface, so the spec says "emit a topic-tagged message" but examples show napplet constructing a signed event | Separate the napplet API (emit topic + payload) from the wire format (kind 29003, signed by runtime). The spec should show the napplet API only; shell behavior should describe the wire format |
| NUB-STORAGE and NUB-IPC sharing kind 29003 | After rewriting NUB-IPC, writers may not notice that NUB-STORAGE also routes through kind 29003. If the NUB-IPC rewrite changes the 29003 schema, NUB-STORAGE breaks silently | Before any wire format changes, map all specs that share the same kind number. Kind 29003 appears in both NUB-IPC and NUB-STORAGE — changes to one must be reviewed against the other |
| NUB-NOSTRDB add() and the signed event passthrough | After simplification, napplets no longer sign events. Writers may remove all `NostrEvent` types from napplet-facing APIs. But `add()` legitimately needs to accept fully-signed events received from subscriptions | Distinguish in SPEC.md: napplets cannot author signed events but CAN pass through externally-authored signed events. `NostrEvent` as an input type is acceptable in passthrough roles |
| NUB-PIPES and NUB-IPC cross-references | NUB-IPC spec currently recommends NUB-PIPES "for high-frequency communication where per-message signing overhead is unacceptable." After simplification removes per-message signing from NUB-IPC, this rationale disappears and the cross-reference becomes misleading | Update the NUB-IPC / NUB-PIPES Relationship section to reflect the new architectural reason to use each (performance overhead from shell-side signing, not napplet-side signing) |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Spec says "runtime handles signing" without defining what runtime means | Shell implementers disagree on whether "runtime" means the shell process, a worker, or a native signer extension. Each implements the signing boundary differently | SPEC.md must define "runtime" precisely: it is the shell's host-page JavaScript, not the napplet iframe. The napplet cannot reach across the iframe boundary to trigger signing |
| Stripping signing from IPC emit() without updating the threat model for sender forgery | IPC sender exclusion currently relies on the shell verifying event signatures. If the shell no longer receives signed events from napplets, it must use an alternative sender verification method (MessageEvent.source). The threat model changes even if the visible API does not | Each spec that removes napplet signing MUST add or update the sender verification mechanism. The Security Considerations section must name the new mechanism explicitly |
| ACL capabilities defined per-spec but capability grant mechanism undefined | Six specs reference capabilities (`relay:read`, `storage:write`, `sign:event`, `pipe:connect`) but none define how the napplet requests them or how the shell checks them. After simplification, napplets interact less directly with auth flows, making the capability gap even less visible | SPEC.md or a NUB-CAPABILITIES spec must define the capability model before any individual spec references capabilities. All six specs should reference the same authority |
| NUB-NOSTRDB `add()` accepting externally-authored events without signature verification documented | A napplet could forward a malformed or replayed event to the local DB. If the spec says only "add() inserts an event," shell implementers may skip signature verification | NUB-NOSTRDB Security Considerations must state: "The shell MUST verify event signatures in `add()` requests before storing. The napplet cannot forge signatures but may pass malformed events." |

---

## "Looks Done But Isn't" Checklist

For each of the 6 specs after rewrite:

- [ ] **API Surface:** No `signEvent()`, `getPublicKey()`, `nip04.*`, or `nip44.*` calls reachable from napplet code — verify by reading the TypeScript interface definitions, not just the prose
- [ ] **API Surface:** No method signatures with `EventTemplate` as a required napplet-authored input (passthrough `NostrEvent` is acceptable; napplet-constructed templates are not)
- [ ] **Shell Behavior:** Every MUST rule that previously said "napplet signs" now says "the shell/runtime signs" — verify subject-verb agreement on every MUST clause
- [ ] **Security Considerations:** No sentence where "napplet" is the subject of a signing or key-handling verb — search for "napplet signs," "napplet keys," "delegated key" as napplet responsibility
- [ ] **Event Kinds tables:** Wire format examples do not show napplet-populated `sig`, `pubkey`, or `id` fields (these are populated by the runtime before the event reaches the bus)
- [ ] **Cross-references:** Any spec that cites another NUB spec for a security property (e.g., NUB-RELAY citing NUB-SIGNER for signing) is updated to reflect that the cited interface is now runtime-internal, not napplet-callable
- [ ] **SPEC.md:** Written before any individual spec draft is finalized — check that SPEC.md defines the two-layer boundary, the passthrough exception, the glossary, and the capability model reference

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Crypto leaked into spec layer after PR merge | HIGH | Re-open PR as breaking change; bump spec version; notify existing implementation maintainers; update shim and shell reference implementations before re-merging |
| Six specs use inconsistent terminology for the same architectural invariant | MEDIUM | Create a single normative SPEC.md update PR that defines the canonical phrases; then open amendment PRs for each of the 6 specs that substitutes the canonical phrases; reviewers can diff each amendment against SPEC.md |
| NUB-SIGNER left as napplet-callable interface by mistake | MEDIUM | Move NUB-SIGNER to a "Runtime-Internal Interfaces" section in the README registry table; add a one-line note to the spec: "This interface is not callable by napplets. It describes the shell-internal signing capability used by NUB-RELAY, NUB-IPC, and NUB-NOSTRDB." |
| NUB-PIPES PIPE_ACK exposes raw pubkeys against the no-crypto rule | LOW | Add a prose note to the spec: "The `peer.pubkey` field is informational. Napplets MUST NOT use it for cryptographic operations. For identity comparison, use `peer.dTag`." This is an additive change with no wire format impact |
| Kind 29003 conflict between NUB-IPC and NUB-STORAGE after rewrite | HIGH | Assign separate kind numbers before writing new wire formats. This requires a cross-spec audit PRE-rewrite. If already merged with conflicting formats: open an errata PR for the spec that was written second, assign a new kind number, notify shell implementers |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Crypto leaks through return types | Phase 1: SPEC.md type vocabulary | After SPEC.md is written, grep all 6 spec rewrites for `NostrEvent` in return positions and verify each is documented as passthrough or replaced with an opaque type |
| NUB-SIGNER category ambiguity | Phase 1: SPEC.md layer model | SPEC.md contains a "Runtime-Internal Interfaces" category; NUB-SIGNER spec draft includes `Layer: runtime-internal`; README registry table notes it as non-callable |
| IPC emit() signing model inconsistency | Phase 2: per-spec Security Considerations audit | Read each spec's Security section aloud; confirm subject of every signing statement is "runtime" or "shell," never "napplet" |
| NUB-NOSTRDB add() passthrough ambiguity | Phase 1: SPEC.md passthrough exception | SPEC.md contains explicit passthrough definition; NUB-NOSTRDB spec references it by name |
| Inconsistent identity vocabulary across 6 specs | Phase 1: SPEC.md glossary | After all 6 spec drafts are written, run a cross-spec review: pick the 3 most important invariants from the glossary and confirm the exact canonical phrase appears in each relevant spec |
| NUB-PIPES pubkey exposure | Phase 1: "no crypto" definition scope | SPEC.md states whether receiving pubkeys for display is acceptable; NUB-PIPES PIPE_ACK section references SPEC.md definition |
| Kind 29003 conflict between NUB-IPC and NUB-STORAGE | Pre-Phase 2: wire format audit | Before any spec rewrite begins, produce a table mapping kind numbers to owning specs; no two specs share a kind number without explicit documented rationale |
| Stale cross-references after NUB-IPC / NUB-PIPES rationale changes | Phase 2: NUB-IPC rewrite | NUB-IPC Relationship to NUB-PIPES section is explicitly in scope for the rewrite, not just the API surface section |

---

## Sources

- Direct analysis of `/home/sandwich/Develop/nubs/specs/` — all 6 draft spec files
- `/home/sandwich/Develop/nubs/.planning/codebase/CONCERNS.md` — naming inconsistency, cross-spec dependency gaps, ACL framework gaps
- `/home/sandwich/Develop/nubs/.planning/codebase/TESTING.md` — validation patterns, review criteria
- `/home/sandwich/Develop/nubs/.planning/PROJECT.md` — milestone scope and key decisions

---
*Pitfalls research for: NUB spec simplification (crypto abstraction, two-layer architecture)*
*Researched: 2026-04-07*
