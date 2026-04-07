# Architecture Research

**Domain:** Specification registry with two-layer architecture (spec vs implementation)
**Researched:** 2026-04-07
**Confidence:** HIGH (derived from direct inspection of all 6 draft specs and NIP-5D context)

## Standard Architecture

### System Overview

The core insight driving v0.1.0 is a clean separation between two layers that currently bleed into each other in the draft specs:

```
┌──────────────────────────────────────────────────────────────────┐
│                        SPEC LAYER                                │
│             (Napplet-facing API — zero crypto)                   │
│                                                                  │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │
│  │   RELAY   │  │  STORAGE  │  │  NOSTRDB  │  │    IFC    │    │
│  │subscribe()│  │ getItem() │  │  query()  │  │  emit()   │    │
│  │ publish() │  │ setItem() │  │   add()   │  │   on()    │    │
│  │  query()  │  │ remove()  │  │subscribe()│  │           │    │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘   │
│        │              │              │              │           │
│  ┌─────┴──────────────────────────────────────────┴─────┐      │
│  │              postMessage Transport (NIP-5D)           │      │
│  └─────┬──────────────────────────────────────────────--┘      │
├────────┼─────────────────────────────────────────────────────── ┤
│        │             IMPLEMENTATION LAYER                        │
│        │     (Runtime-internal — full Nostr crypto)             │
│                                                                  │
│  ┌─────▼──────────────────────────────────────────────────┐    │
│  │                     Shell / Runtime                     │    │
│  │  ┌────────────┐  ┌──────────┐  ┌────────────────────┐  │    │
│  │  │  Identity  │  │  Signer  │  │    ACL / Session    │  │    │
│  │  │ (keypairs) │  │ (NIP-07) │  │  (AUTH handshake)  │  │    │
│  │  └────────────┘  └──────────┘  └────────────────────┘  │    │
│  │  ┌────────────┐  ┌──────────┐  ┌────────────────────┐  │    │
│  │  │  Relay     │  │ Storage  │  │  Event DB (OPFS)   │  │    │
│  │  │  Pool      │  │ Backend  │  │  (nostrdb worker)  │  │    │
│  │  └────────────┘  └──────────┘  └────────────────────┘  │    │
│  └──────────────────────────────────────────────────────── ┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │           Nostr Network / External Relays               │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

The spec layer is what NUBs define. The implementation layer is what shells like hyprgate implement. SPEC.md defines the boundary between them.

### The Two Layers Defined

**Spec Layer (what NUBs specify):**
- Napplet-facing API surfaces on `window.napplet.*`, `window.nostrdb`
- Message verbs and payload shapes that cross the postMessage boundary
- What behavior the napplet can rely on (MUST/MAY/SHOULD for shells)
- Zero crypto objects: no `NostrEvent`, no pubkeys, no signatures in napplet code

**Implementation Layer (what shells implement, NOT what NUBs specify):**
- NIP-5D AUTH handshake (REGISTER/IDENTITY/AUTH)
- Session key generation and delegated signing
- NIP-07 signer interaction (NUB-SIGNER is internal, not napplet-facing)
- Event construction, Schnorr signing, signature verification
- ACL policy enforcement
- Relay pool management

### Component Responsibilities

| Component | Layer | Responsibility |
|-----------|-------|----------------|
| NUB-RELAY | Spec | `subscribe(filters, onEvent)`, `publish(template)`, `query(filters)` — napplet API only |
| NUB-STORAGE | Spec | `getItem/setItem/removeItem/keys` — scoped KV, no wire format exposed |
| NUB-IFC | Spec | `emit(topic, payload)`, `on(topic, cb)` — payload is plain JSON, no NostrEvent in callbacks |
| NUB-NOSTRDB | Spec | `query/add/subscribe` with plain filters and unsigned templates — no signatures in API |
| NUB-PIPES | Spec | `open/onOpen/broadcast/send` — already nearly clean, minor pubkey exposure to remove |
| NUB-SIGNER | Implementation | Runtime-internal only. Napplets never call `window.nostr`. Shell uses it invisibly. |
| Shell runtime | Implementation | Auth, signing, ACL, routing, relay pool, storage backend, event DB |
| SPEC.md | Governance | Defines the two-layer boundary, what belongs in each layer, terminology |

## Recommended Spec Document Structure

### SPEC.md (new foundational doc)

SPEC.md is the entry point for understanding all NUBs. It defines:

1. **The two-layer model** — what "spec layer" and "implementation layer" mean, with explicit examples of what belongs where
2. **The napplet contract** — what a napplet author can assume: no crypto, no keypairs, no event construction
3. **The shell contract** — what any shell implementing NUBs must do internally
4. **Vocabulary** — define terms used consistently across all specs: "template", "filter", "subscription", "session"
5. **Wire format policy** — NUBs define message shapes, not internal event kinds; event kinds are implementation detail
6. **Relationship to NIP-5D** — SPEC.md extends NIP-5D by defining how extensions are structured, not by replacing core protocol

SPEC.md does NOT contain individual API surfaces. Those stay in NUB-*.md files.

### Per-Spec NUB-*.md Structure (revised sections)

Each spec should contain exactly these sections after simplification:

```
1. Header    — NUB ID, Namespace, Discovery, Status
2. Summary   — One paragraph: what it provides and why (no crypto leakage here)
3. API Surface — TypeScript interface, method descriptions, NO NostrEvent in napplet return types
4. Shell Behavior — MUST/MAY/SHOULD for runtime implementation
5. Wire Protocol — The postMessage message shapes (verb + payload), no event kinds unless genuinely needed
6. Security Model — Shell enforces, napplet trusts shell
7. Implementations — Links to SDK shim and shell implementations
```

The current "Event Kinds" section is problematic — it leaks NIP-01 implementation details into the spec. Replace it with "Wire Protocol" that describes only the postMessage message shapes visible to the napplet, without requiring the napplet to know about Nostr event kinds.

## Per-NUB Integration Map and Crypto Concerns

### NUB-RELAY — Significant changes needed

**Current crypto exposure:**
- `publish(template)` calls `window.nostr.signEvent()` — napplet must import/know about signer
- "Event Kinds" section exposes kind 29001 scoped relay events to napplets
- Shell behavior mentions signature verification as if napplet cares

**Simplified model:**
- `publish(template)` sends an unsigned EventTemplate; runtime signs internally
- Napplet never sees signed event; `publish()` can return void or a minimal receipt
- Event kinds section removed or renamed to "Implementation Notes" in shell docs
- Shell behavior: sign-then-forward is implementation detail, not contract

**Integration point:** NUB-RELAY depends on NUB-SIGNER internally (runtime signs before publish). This is NOT exposed in the spec — it's a shell implementation dependency.

### NUB-STORAGE — Minor changes needed

**Current crypto exposure:**
- Uses kind 29003 events for transport (leaks NIP-01 into spec)
- Shell scopes by `(dTag, aggregateHash)` — these are implementation details
- Events are signed with session keys — napplet does not need to know this

**Simplified model:**
- API surface is already clean: `getItem/setItem/removeItem/keys`
- Wire protocol section should describe the postMessage shape, not NIP-01 event kinds
- Scoping by napplet identity is shell behavior — spec says "MUST scope per napplet identity" without saying how
- Remove correlation ID discussion from spec (implementation detail)

**Integration point:** None with other NUBs at the spec layer. Shell-internal dependency on the AUTH session for scoping.

### NUB-SIGNER — Becomes implementation-only

**Current state:** Full NIP-07 `window.nostr` exposed to napplets. This is the most aggressive change.

**Simplified model:** NUB-SIGNER is removed as a napplet-facing interface. The spec documents the shell's internal signing requirement:
- Shell MUST have a signer available before processing any publish/emit request
- NUB-SIGNER spec becomes a shell implementation guide, not a napplet API spec
- Discovery via `shell.supports("signer")` is removed — napplets never call this
- `window.nostr` is NOT installed in napplet iframes

**Impact on other specs:** NUB-RELAY `publish()` no longer references NUB-SIGNER. Signing is invisible.

**Critical note:** This is a breaking departure from NIP-07 ecosystem conventions. SPEC.md should explain why this is intentional and how advanced napplets that genuinely need signing (rare) should handle it.

### NUB-IFC — Moderate changes needed

**Current crypto exposure:**
- `emit()` creates a signed kind 29003 event — napplet is aware of this
- `on()` callback receives `(payload, event: NostrEvent)` — exposes raw event with sig/pubkey
- "signed with delegated session key" mentioned in spec text
- Sender exclusion described in terms of Schnorr verification

**Simplified model:**
- `emit(topic, payload)` where payload is any JSON-serializable value — napplet sends data, not events
- `on(topic, cb)` where `cb` receives only `payload` — no `event` second argument
- Sender exclusion is a shell behavior, not something napplet code handles
- Wire protocol: shell internally creates kind 29003 events; napplet just sends/receives JSON

**Integration point:** NUB-IFC depends on NUB-RELAY at the shell level for routing (kind 29003 subscription). This is an implementation dependency. The spec should note that NUB-IFC requires NUB-RELAY support in the shell, but napplets don't see this.

### NUB-NOSTRDB — Moderate changes needed

**Current crypto exposure:**
- `add(event: NostrEvent)` — requires fully signed event; napplet must construct/sign
- `query()` returns `NostrEvent[]` — exposes raw events with pubkeys and signatures
- Shell validates signatures before storing
- `subscribe()` returns `AsyncGenerator<NostrEvent>` — yields raw events

**Simplified model:**
- `add(template: EventTemplate)` — napplet provides unsigned template; shell signs and adds
- `query()` returns typed data objects, or at minimum, events without requiring napplet crypto knowledge
- Alternatively: keep returning `NostrEvent` for query/subscribe (read-only is less problematic) but remove `add()` requiring a signed event
- The cleaner choice: `add()` takes EventTemplate, `query()` returns EventTemplate-shaped objects (kind/content/tags) without id/sig/pubkey. Shell adds those fields.

**Integration point:** NUB-NOSTRDB is the local cache for NUB-RELAY. Shells typically back NUB-RELAY queries from NUB-NOSTRDB. This is a shell-level dependency. The spec should note these can be combined but are separate interfaces.

### NUB-PIPES — Minor changes needed

**Current crypto exposure:**
- `PipeHandle.peer` exposes `{ pubkey: string; dTag: string }` — pubkey leaks identity
- "not authenticated" error references AUTH handshake that napplet didn't do
- PIPE_ACK includes `peer` pubkey in the wire format

**Simplified model:**
- `PipeHandle.peer` becomes `{ dTag: string }` only — dTag is the app-level identifier, pubkey is runtime-internal
- PIPE_ACK delivers peer dTag only; pubkey is implementation detail
- Auth errors say "target not available" rather than "not authenticated" (which implies napplet knows about auth)
- Otherwise the spec is already well-structured for the simplified model

**Integration point:** NUB-PIPES has no spec-layer dependencies on other NUBs. Shell internally may use session keys for routing, but this is not exposed.

## Data Flow Under the Simplified Model

### Publish Flow (NUB-RELAY)

```
Napplet                        Shell (Runtime)               Nostr Network
    |                               |                              |
    | publish({kind, tags, content})|                              |
    |-----------------------------> |                              |
    |                               | [sign with session key]      |
    |                               | [apply ACL check]            |
    |                               | [forward to relay pool]      |
    |                               |----------------------------> |
    |  (optional receipt)           |  OK/CLOSED                   |
    | <---------------------------- |                              |
```

### Subscribe Flow (NUB-RELAY)

```
Napplet                        Shell (Runtime)               Nostr Network
    |                               |                              |
    | subscribe(filters, onEvent)   |                              |
    |-----------------------------> |                              |
    |                               | [open REQ to relay pool]     |
    |                               |----------------------------> |
    |                               |  EVENT events                |
    |  onEvent(event)               | <--------------------------- |
    | <---------------------------- |                              |
    |  (events as plain objects)    |                              |
```

### IFC Emit Flow

```
Napplet A                      Shell (Runtime)              Napplet B
    |                               |                            |
    | emit("profile:open", payload) |                            |
    |-----------------------------> |                            |
    |                               | [construct kind 29003]     |
    |                               | [sign with session key]    |
    |                               | [route by #t filter match] |
    |                               |                            |
    |                               | on("profile:open", cb)     |
    |                               | <------------------------- |
    |                               |                            |
    |                               | cb(payload)                |
    |                               | -------------------------> |
```

## Spec Build Order

Build order is determined by dependency analysis. Specs with no spec-layer dependencies come first. Specs that reference other NUBs (even just in prose) come after.

```
SPEC.md         — Foundation. Defines two-layer model. Everything else references this.
    |
    ├── NUB-STORAGE  — No spec-layer dependencies. Cleanest API surface. Good first spec to validate template.
    |
    ├── NUB-SIGNER   — Becomes implementation-only doc. Write it early so other specs can correctly omit signing.
    |
    ├── NUB-RELAY    — Depends on SIGNER being implementation-only (publish() simplification)
    |
    ├── NUB-NOSTRDB  — Logically paired with RELAY (same data model). Write after RELAY.
    |
    ├── NUB-IFC      — Depends on RELAY wire format being settled (uses relay routing internally)
    |
    └── NUB-PIPES    — No spec-layer dependencies. Can be written any time after SPEC.md.
                       Ordered last because it's already the cleanest and needs least change.
```

**Rationale for this order:**

1. SPEC.md must land first — it defines the vocabulary and two-layer terminology used in all other specs
2. NUB-SIGNER second — its demotion to implementation-only removes the reference other specs must not make; resolving this early prevents back-and-forth edits
3. NUB-STORAGE third — simplest API, good test of the new template and spec philosophy before tackling complex ones
4. NUB-RELAY fourth — central to the protocol; once RELAY's wire protocol is settled, NOSTRDB and IFC can align to it
5. NUB-NOSTRDB fifth — shares filter/event data model with RELAY; write together or sequentially
6. NUB-IFC sixth — most prose-heavy spec with routing conventions; write after RELAY wire format is stable
7. NUB-PIPES last — already close to the target model, minor tweaks only; good final validation pass

## Architectural Patterns

### Pattern 1: EventTemplate vs NostrEvent

**What:** Napplets pass unsigned EventTemplates in; receive query results that may be read as plain data objects.
**When to use:** Any spec method that currently takes or returns `NostrEvent`
**Implication for specs:** Define an `EventTemplate` type (`{ kind, tags, content }`) in SPEC.md. Each spec references it. Do not define it per-spec.

```typescript
// Spec layer (napplet sees this)
interface EventTemplate {
  kind: number;
  tags: string[][];
  content: string;
}

// NOT in spec layer:
// interface NostrEvent extends EventTemplate {
//   id: string; pubkey: string; created_at: number; sig: string;
// }
```

### Pattern 2: Wire Protocol Section Replaces Event Kinds

**What:** Current specs have an "Event Kinds" section listing NIP-01 event kind numbers. This leaks implementation into the spec.
**Do instead:** Replace with "Wire Protocol" section listing only the message verbs and payload shapes that cross the postMessage boundary and are visible to napplet code.

```
Wire Protocol (what napplets send/receive):
["PIPE_OPEN", pipe_id, {target, name}]
["PIPE_ACK",  pipe_id, {peerDTag}]       ← no pubkey
["PIPE",      pipe_id, payload]

Implementation Notes (not normative for napplets):
Shell encodes pipe control as kind 29004 internally.
```

### Pattern 3: Shell Behavior Hides Crypto

**What:** Shell Behavior sections currently describe signing, verification, and AUTH in ways that imply napplet awareness.
**Do instead:** Shell Behavior sections describe observable outcomes, not mechanisms.

```
CURRENT (bad): "The shell MUST verify Schnorr signatures before routing kind 29003 events."
REVISED (good): "The shell MUST only route messages from authenticated napplets."

CURRENT (bad): "Events signed with delegated keys MUST NOT be published to external relays."
REVISED (good): "The shell MUST enforce the napplet's publish permissions before relaying events."
```

### Pattern 4: Correlation IDs Are Implementation Details

**What:** Current specs (STORAGE, NOSTRDB) describe correlation IDs in the spec layer. Napplets don't need to manage these.
**Do instead:** Correlation IDs are implementation details of the postMessage shim. The spec says "each method call returns a Promise that resolves with the result." How that Promise is correlated to a response is the shim's concern.

## Anti-Patterns

### Anti-Pattern 1: Exposing `NostrEvent` to Napplets

**What people do:** Return `NostrEvent[]` from query methods, pass `NostrEvent` to callbacks, require signed events in `add()`.
**Why it's wrong:** Forces napplet authors to understand NIP-01 event structure, id computation, and signature format. Creates coupling to Nostr internals that may change.
**Do this instead:** Return `EventTemplate`-shaped objects from reads. Accept `EventTemplate` for writes. Let the shell add `id/sig/pubkey` invisibly.

### Anti-Pattern 2: Referencing NUB-SIGNER in Other Specs

**What people do:** `publish()` docs say "signs the event via window.nostr.signEvent() (NUB-SIGNER)".
**Why it's wrong:** Exposes the signing dependency to napplet authors. They now need to know NUB-SIGNER exists and that it's a prereq for NUB-RELAY.
**Do this instead:** `publish()` docs say "the shell signs and broadcasts the event on behalf of the napplet." No reference to NUB-SIGNER.

### Anti-Pattern 3: Topic Prefix Conventions in Wire Protocol

**What people do:** NUB-IFC documents `shell:*` and `napplet:*` topic prefixes as part of the wire protocol.
**Why it's wrong:** These conventions are useful but are advisory coordination patterns, not protocol requirements. Calling them protocol spec creates false normative weight.
**Do this instead:** Move topic conventions to a non-normative appendix or to a separate "Conventions" doc. The wire protocol section only specifies required behavior.

### Anti-Pattern 4: Documenting Internal Event Kinds as Napplet-Visible

**What people do:** "This interface uses Kind 29001 for scoped relay connect" in the spec for napplets to read.
**Why it's wrong:** Napplets never construct kind 29001 events. The kind is only meaningful to the shell router.
**Do this instead:** Move kind tables to an "Implementation Reference" section clearly marked as non-normative for napplet authors, or to a separate shell implementation guide.

## Integration Points Between NUBs

### Spec-Layer Integration (napplet-visible)

There is intentionally minimal spec-layer integration. NUBs are independent. A napplet using NUB-RELAY does not need to know about NUB-STORAGE. This is by design.

The only legitimate cross-spec reference at the spec layer is:
- NUB-IFC may note "for high-frequency data, see NUB-PIPES" — this is informational, not normative.

### Shell-Layer Integration (implementation-visible)

| Integration | Description |
|-------------|-------------|
| NUB-RELAY + NUB-SIGNER | Shell uses its signer to sign events before publish |
| NUB-RELAY + NUB-NOSTRDB | Shell may serve NUB-RELAY queries from local NUB-NOSTRDB cache |
| NUB-IFC + NUB-RELAY | Shell routes IFC messages using the relay subscription infrastructure |
| NUB-STORAGE + NIP-5D AUTH | Shell scopes storage to the authenticated napplet identity |
| NUB-PIPES + NIP-5D AUTH | Shell validates both pipe endpoints are authenticated before PIPE_ACK |

These integrations are shell implementation details. They MUST NOT appear as normative requirements in the napplet-facing spec sections.

### Dependency Graph

```
NIP-5D (core)
    |
    ├── NUB-RELAY         ← foundation for event delivery
    │     └── (internally) NUB-NOSTRDB (cache)
    │     └── (internally) NUB-SIGNER (signing)
    │
    ├── NUB-STORAGE       ← independent, scoped to auth session
    │
    ├── NUB-IFC           ← uses relay routing internally
    │
    ├── NUB-NOSTRDB       ← independent, local cache
    │
    └── NUB-PIPES         ← independent, uses auth session
```

Solid line = spec-layer dependency (explicit in napplet API).
All arrows above are shell-level only — napplet code has no visible dependencies between NUBs.

## Sources

- Direct inspection of all 6 NUB draft specs (nub-relay, nub-signer, nub-ipc, nub-storage, nub-nostrdb, nub-pipes branches)
- .planning/codebase/ARCHITECTURE.md (codebase analysis)
- .planning/codebase/STRUCTURE.md (codebase structure)
- .planning/PROJECT.md (milestone goals)

---
*Architecture research for: NUBs spec simplification — two-layer model*
*Researched: 2026-04-07*
