# Feature Research

**Domain:** Simplified spec API surface for sandboxed napplet extension specs
**Researched:** 2026-04-07
**Confidence:** HIGH (for API pattern analysis based on existing specs and analogous platforms); MEDIUM (for ecosystem comparison — napplet is novel, analogies are partial)

## Feature Landscape

### Table Stakes (Users Expect These)

For this milestone, the "users" are napplet authors reading and implementing against the NUB specs. Missing any of these makes the spec feel incomplete or forces developers back into crypto territory.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| `subscribe(filters, onEvent, onEose)` returns a closeable handle | Every sandboxed app platform (Figma, Chrome extensions, Service Workers, Pusher) uses subscription handles as the atomic primitive for live data. Napplet authors expect this shape. | LOW | Already present in NUB-RELAY. The spec must clearly state that the napplet passes filter criteria only — no pubkey, no auth token, no key material in the filter unless it is a semantic query value (e.g., filtering by `authors` to read someone else's events is fine). |
| `publish(template)` takes an unsigned payload | NIP-07's `window.nostr.signEvent(template)` established the precedent: app passes a template, runtime returns a signed result. The napplet spec must formalise that `publish()` accepts an unsigned `EventTemplate` and the shell handles signing invisibly. The napplet never calls `signEvent()` directly. | LOW | NUB-RELAY already has this shape but the description explicitly says "Signs the event template via `window.nostr.signEvent()` (NUB-SIGNER)". That sentence is the crypto leak. The spec must move the signing step into shell behavior, invisible to the napplet API narrative. |
| `query(filters)` returns a Promise of events | Collect-once, no streaming. Expected by analogy with every query API (IndexedDB `getAll()`, Firestore `getDocs()`, NIP-45 COUNT). Without it developers write boilerplate subscribe+collect+EOSE+close. | LOW | Already present in NUB-RELAY. No crypto concerns here. |
| Scoped storage with localStorage-like methods | Every sandboxed embed platform gates storage behind the host (Figma uses `clientStorage`, Chrome extensions use `chrome.storage`, Adobe Express proxies storage). Napplet devs expect `getItem/setItem/removeItem/keys`. Identity scoping is an implementation detail — the napplet never sees the composite key `(dTag, aggregateHash)`. | LOW | NUB-STORAGE API surface is already correct. The spec prose must stop describing how scoping works at the napplet-API level and move it to Shell Behavior. |
| Topic-based pub/sub `emit(topic, payload)` / `on(topic, cb)` | Inter-component event buses are table stakes for embedded app platforms (Salesforce LWC, Figma postMessage patterns, Chrome `runtime.sendMessage`). Napplets expect to broadcast `"stream:channel-switch"` without knowing who listens. | LOW | NUB-IPC API surface is correct. The crypto leak is in the event structure: "The event is signed with the napplet's delegated session key before posting to the shell." That sentence must disappear from the napplet-facing API section and move to the shell implementation layer. |
| `window.nostr` signer proxy (NIP-07 contract) | NIP-07 is already the standard abstraction for hiding private keys from web apps. Napplets expect `window.nostr.signEvent()` and `getPublicKey()`. The proxy pattern is well understood — the shell mediates, the napplet never touches the private key. | LOW | NUB-SIGNER is already correct in principle. However, once publish() in NUB-RELAY is moved to the shell-signs-transparently model, `window.nostr` is mainly needed for explicit user-directed signing (writing profile events, etc.). The spec should clarify this narrower scope. |
| Discovery via `shell.supports("NUB-X")` | Every capability-gated platform requires feature detection before use. Chrome extensions require `chrome.runtime.getManifest()` checks; Service Workers require `'pushManager' in serviceWorkerRegistration`. `shell.supports()` is the napplet equivalent. | LOW | Defined in NIP-5D. NUB specs just need to declare their discovery string consistently. Not a new feature — enforcement item. |
| Graceful degradation documented per spec | When a NUB is absent, what should a napplet do? Every web platform capability spec (Push API, Service Worker, Payment Request) documents the fallback. | LOW | Currently missing from all NUB specs. Each spec should include a short "If absent" note. |

### Differentiators (Competitive Advantage)

These are features that make the NUB spec system distinctively better than the alternatives.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Explicit two-layer split in each spec (napplet API section vs shell implementation section) | Most sandboxed app specs conflate the two layers. Figma plugins simply get `figma.ui.postMessage` — the underlying Realm sandbox and event routing are invisible and undocumented. Making the boundary explicit in every NUB spec lets shell implementers know exactly what to hide and lets napplet authors know exactly what they never need to think about. | LOW | Requires a structural template change: TEMPLATE-WORD.md should rename or reorder sections to make the boundary unmissable — "Napplet API" (what napplets call) then "Shell Implementation" (what the runtime does internally, including signing, correlation IDs, event kind mapping). |
| ACL capabilities declared, not enforced by napplet | Ably and Pusher token capabilities (`subscribe`, `publish` per channel) are server-scoped. The client code never inspects its own capabilities — it just makes calls and receives errors. The NUB spec system should follow this: capabilities (`relay:read`, `relay:write`, `storage:read`) are declared in the NIP-5A manifest and enforced by the shell. The napplet never checks ACL. | MEDIUM | Currently specs like NUB-RELAY say "The shell MAY enforce ACL checks on `relay:read`..." This is correct but the napplet API sections should explicitly note "napplets do not request or inspect capabilities — they are granted by the manifest and enforced by the shell." A NUB-CAPABILITIES foundational document (identified in CONCERNS.md) would lock this in. |
| Wire protocol sections scoped to shell-internal only | NUB-RELAY and NUB-IPC expose their postMessage wire format (event kinds, tag structures) as part of the napplet-facing narrative. This is unnecessary. Napplet authors using the `@napplet/shim` never send raw kind 29003 events — that is the shim's job. Moving event kind tables to a clearly labeled "Wire Format (Implementation Detail)" section signals clearly that napplet code should never construct raw postMessage events. | LOW | Structural change to template and specs. High signal-to-noise improvement for spec readers. |
| `AsyncGenerator` subscribe pattern for database (NUB-NOSTRDB) | `subscribe()` returning an `AsyncGenerator<NostrEvent>` using `for await...of` is more idiomatic in modern JS than callback-based subscriptions. This differentiates the local DB access from relay subscriptions and matches how JavaScript developers expect to consume streams today. | LOW | Already present in NUB-NOSTRDB. Preserve it. No crypto concerns in this API. |
| Per-spec "what the napplet never does" summary | A short prose paragraph at the top of each spec that lists what crypto/identity concern has been moved to the runtime. This is a DX differentiator — it communicates design intent immediately and prevents future spec contributors from inadvertently re-leaking crypto. | LOW | New addition to TEMPLATE-WORD.md. Example for NUB-RELAY: "The napplet never calls `signEvent()`, never handles keypairs, never manages subscription IDs, never sends raw REQ messages." |

### Anti-Features (Commonly Requested, Often Problematic)

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Expose `delegatedPubkey` or session identity to napplet API | Napplet authors may want to know "who am I" to build UIs, filter their own events, or show the user their identity. | Exposing a raw pubkey to the napplet API re-introduces identity into the napplet layer. The napplet now has a thing it has to reason about. It is a small step from "I have a pubkey" to "let me sign something with it." The delegated key should remain an implementation detail of NIP-5D AUTH, not a napplet API feature. | If a napplet needs "the current user's display name," it should use `window.napplet.relay.query([{kinds:[0], authors:[...]}])` — the shell can resolve the current user identity transparently. If it needs to filter its own events by author, the shell can inject a special filter value like `"self"` that the runtime resolves. Do not expose raw pubkeys in the napplet API layer. |
| Explicit signing method on NUB-RELAY `publish()` | Some napplet authors may want control over which key signs a published event — e.g., "sign this with the user's real key, not the delegated key." | Introducing signing-key selection as a parameter to `publish()` re-introduces crypto responsibility into the napplet call site. It creates two publishing modes the napplet must understand, and exposes the distinction between real keys and delegated keys as a napplet concern. | The shell decides which key to use based on event kind and ACL policy. Events that must carry the user's real signature (kind 0, kind 3) are always signed by the real signer (NUB-SIGNER proxy). Events that should use the delegated key (internal IPC events) are signed internally. This is a shell implementation decision, not a napplet API surface decision. |
| Raw postMessage access to NIP-01 wire format | Developers familiar with Nostr may want to drop down to raw `["REQ", subId, ...filters]` messages for full control or to work around API limitations. | Exposing raw NIP-01 wire format to napplets breaks the abstraction boundary entirely. It means the shim becomes optional, the spec becomes implementation-coupled, and any future wire format changes (e.g., NUB defining a new kind) become breaking changes for all napplets using raw messages. | If an API method is missing, add it to the spec. The right answer to "I need something the API does not offer" is a spec amendment, not a raw wire format escape hatch. |
| Per-message delivery receipts on `emit()` | Developers may want to know if their IPC message was received by another napplet. | Fire-and-forget is the correct model for broadcast pub/sub. Adding delivery acknowledgment requires the shell to track all receivers and coordinate responses, which is O(n receivers) complexity. It re-introduces the question "who received this?" which napplet authors should not need to answer in a loosely coupled system. | For tight-coupled point-to-point coordination where the napplet needs acknowledgment, use NUB-PIPES, which has connection lifecycle (PIPE_OPEN, PIPE_ACK, PIPE_CLOSE). |
| NUB-SIGNER `signEvent()` in normal publish flows | Some specs (the current NUB-RELAY) implicitly require napplets to call `window.nostr.signEvent()` before publishing. | This is the primary crypto leak identified in the milestone. It forces every napplet that publishes events to understand and call a crypto primitive. | `publish(template)` on NUB-RELAY takes an unsigned template. The shell signs it. `window.nostr.signEvent()` is reserved for cases where the napplet explicitly needs a user-signed event (profile updates, contact list updates) and should be called out in those specs as a deliberate user-action boundary, not a routine publish step. |
| Versioned spec files (NUB-RELAY-v2.md) | Maintainers may want to maintain backward-compatible spec history by keeping old versions as separate files. | This creates immediate confusion about which file is canonical, duplicates content, and turns the registry table into a version matrix. It is premature optimization for a spec system that has no deployed stable versions yet. | Use a `Last Updated` date and a `## Changelog` section within the single spec file. If a breaking change is necessary, the PR process surfaces the discussion. Formal versioning can be added when a spec reaches `stable` status. |

## Feature Dependencies

```
[NIP-5D REGISTER/IDENTITY/AUTH]
    └──required by──> [All NUB specs] (shell authentication must complete before any NUB capability is accessible)

[Two-Layer Template (TEMPLATE-WORD.md update)]
    └──required by──> [All 6 spec rewrites] (specs cannot be rewritten consistently without a clear template)
    └──required by──> [SPEC.md foundational doc] (architectural doc needs a template to reference)

[NUB-RELAY (simplified)]
    └──used by──> [NUB-IPC] (IPC uses kind 29003 events routed via the relay bus)
    └──used by──> [NUB-NOSTRDB] (NOSTRDB requests use kind 29006/29007 via the postMessage transport)

[NUB-SIGNER]
    └──used by (runtime-internal)──> [NUB-RELAY publish()] (shell uses signer internally; napplet does not call it)
    └──used by (explicit user action)──> [napplet code for kind 0, kind 3 writes]

[shell.supports() discovery]
    └──required before──> [Any NUB API call] (napplets must degrade gracefully if a NUB is absent)

[NUB-PIPES]
    └──depends on──> [NUB-IPC or NUB-RELAY transport] (PIPE_OPEN/PIPE_ACK use the postMessage bus)
    └──currently unimplemented──> [defer to post-v0.1.0 or mark clearly in registry]
```

### Dependency Notes

- **Two-layer template required before spec rewrites:** Without updating TEMPLATE-WORD.md first, each spec rewrite will independently invent its own section structure for separating napplet API from shell implementation. This will produce six inconsistent specs.
- **NUB-RELAY underpins IPC and NOSTRDB wire transport:** Even after the API simplification, the underlying wire transport for NUB-IPC (kind 29003) and NUB-NOSTRDB (kind 29006/29007) rides on the same postMessage channel as NUB-RELAY. This is a shell implementation detail. But it means any spec that changes NUB-RELAY's wire format ripples to IPC and NOSTRDB. Specs must clarify this dependency is at the shell layer, not at the napplet API layer.
- **NUB-SIGNER role narrows significantly:** In the current specs, NUB-SIGNER is load-bearing for publish flows. After simplification, its role narrows to explicit user-consent signing (profile, contacts, delegation). Specs that previously said "calls NUB-SIGNER internally" should say "the shell handles signing" — NUB-SIGNER becomes a user-facing API for deliberate actions, not a plumbing dependency.

## MVP Definition

The "MVP" here is the minimum spec surface that delivers the "napplets have zero crypto responsibilities" guarantee, sufficient for a napplet author to build a real read/write Nostr application.

### Launch With (v0.1.0)

- [ ] SPEC.md foundational doc — defines the two-layer split, vocabulary (spec layer vs implementation layer), and the "no crypto in napplet API" principle. This is the anchor every other spec references.
- [ ] Updated TEMPLATE-WORD.md — adds "Napplet API" and "Shell Behavior" as mandatory distinct sections, adds a "What the napplet never does" summary field, removes the assumption that napplets construct raw events.
- [ ] Simplified NUB-RELAY — `subscribe/publish/query` with unsigned template publish, crypto stripped from napplet API narrative, event kinds moved to shell behavior section.
- [ ] Simplified NUB-STORAGE — API surface already clean; prose cleanup only (remove composite key description from napplet-facing section).
- [ ] Simplified NUB-IPC — `emit/on` with signed-event language moved to shell behavior section.
- [ ] Simplified NUB-NOSTRDB — API surface is clean; `add(event)` takes a `NostrEvent` but validation is shell-internal; verify no crypto leaks.
- [ ] Simplified NUB-SIGNER — reframe as "user-consent signing proxy" not "publish pipeline component"; remove from required path for standard publish flows.
- [ ] Simplified NUB-IFC (resolve NUB-IPC vs NUB-IFC naming) — consistent name across README, template, spec file.

### Add After Validation (v0.1.x)

- [ ] NUB-CAPABILITIES foundational spec — define how capabilities are declared in NIP-5A manifest and enforced by the shell. Unblocks clean security sections in all other specs.
- [ ] "If absent" graceful degradation section added to all specs — once the core rewrites are done, add a consistent fallback pattern.
- [ ] NUB-PIPES spec rewrite — currently unimplemented; simplification is premature until there is at least one implementation to validate the wire format against.

### Future Consideration (v0.2+)

- [ ] NUB-NN message protocol specs — v0.1.0 is interface-only. Once interface specs are stable, define 1-2 message protocol specs (e.g., a basic feed or chat protocol) to exercise the NUB-NN track.
- [ ] Formal maturity levels — `draft`, `proposed`, `stable`. Add graduation criteria. Not needed until at least one spec has independent implementations.
- [ ] Schema validation for wire format examples — JSON Schema or TypeScript definitions for code examples in specs, CI check on PR. Valuable once spec surface stabilizes.

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| SPEC.md two-layer architecture doc | HIGH | LOW (writing only) | P1 |
| Updated TEMPLATE-WORD.md | HIGH | LOW (markdown edit) | P1 |
| NUB-RELAY: unsigned template publish, crypto removed from API | HIGH | LOW (prose rewrite) | P1 |
| NUB-IPC: signed-event language moved to shell behavior | HIGH | LOW (prose rewrite) | P1 |
| NUB-SIGNER: reframed as user-consent-only signing proxy | HIGH | LOW (prose rewrite) | P1 |
| NUB-STORAGE: composite key moved to shell behavior | MEDIUM | LOW (prose cleanup) | P1 |
| NUB-NOSTRDB: verify no crypto leaks, add graceful degradation | MEDIUM | LOW (audit + minor edits) | P1 |
| Resolve NUB-IPC vs NUB-IFC naming | HIGH | LOW (rename + grep) | P1 |
| NUB-CAPABILITIES spec | HIGH | MEDIUM (new spec) | P2 |
| Graceful degradation "If absent" per spec | MEDIUM | LOW per spec | P2 |
| NUB-PIPES rewrite | LOW (unimplemented) | HIGH (design + spec) | P3 |
| NUB-NN message protocol specs | MEDIUM | HIGH (new track) | P3 |
| Formal maturity levels + governance update | MEDIUM | MEDIUM | P3 |

**Priority key:**
- P1: Required for v0.1.0 milestone
- P2: High value, add when P1 is complete
- P3: Future milestone

## Competitor Feature Analysis

Sandboxed app platforms that hide identity/crypto from app authors and their API patterns:

| Feature | Figma Plugin API | Chrome Extension API | NIP-07 Web Apps | NUB Approach |
|---------|-----------------|---------------------|-----------------|--------------|
| Auth hidden from app code | YES — no concept of auth in plugin code | PARTIAL — content scripts have no tokens, background service worker may have OAuth token | YES — `window.nostr` proxy hides private key entirely | YES — AUTH is NIP-5D, invisible to napplet API |
| Publish/write | `figma.currentPage.appendChild()` — no signing concept | `chrome.storage.sync.set()` — no signing concept | `window.nostr.signEvent(template)` — explicit signing, returns signed event | Target: `relay.publish(template)` — shell signs, napplet never calls signEvent |
| Subscribe/read | `figma.on('selectionchange', cb)` — event listener | `chrome.storage.onChanged.addListener(cb)` — event listener | Manual REQ/EVENT via relay WebSocket or library | `relay.subscribe(filters, onEvent, onEose)` — NIP-01 semantics, no crypto |
| Query/read-once | `figma.currentPage.findAll(pred)` — sync query on in-memory tree | `chrome.storage.sync.get(keys, cb)` — async callback | `pool.querySync(relays, filters)` via library — explicit relay management | `relay.query(filters)` — Promise, shell handles relay pool |
| Storage | `figma.clientStorage.getAsync(key)` — async, scoped by plugin ID | `chrome.storage.local.get/set` — scoped by extension ID | `localStorage` (no sandboxing) or relay-based storage | `storage.getItem/setItem` — async, scoped by `(dTag, aggregateHash)` |
| IPC between app components | `figma.ui.postMessage(msg)` / `figma.ui.onmessage` — single channel | `chrome.runtime.sendMessage()` / `onMessage` — typed messages | No standard — custom relay subscriptions | `ipc.emit(topic, payload)` / `ipc.on(topic, cb)` — topic-based pub/sub |
| Capability/permission model | Plugin manifest declares `permissions` array | Extension manifest declares `permissions` array | None — app has full NIP-07 access if user approved | NIP-5A manifest declares `requires` tags, shell enforces ACL |

**Key observation:** Figma and Chrome extensions are the closest analogues. Neither exposes cryptographic identity to plugin/extension code. The platform handles persistence, routing, and identity internally. The app developer only sees domain-level primitives (`findAll`, `sendMessage`, `getAsync`). NUB specs should reach that same level of abstraction.

## Sources

- [NIP-07 specification](https://nostr-nips.com/nip-07) — establishes the `window.nostr` proxy as the standard pattern for hiding private keys from web apps; HIGH confidence
- [Figma Plugin how plugins run](https://developers.figma.com/docs/plugins/how-plugins-run/) — confirms Figma plugin model: no auth/crypto in plugin code, postMessage for communication; HIGH confidence
- [Figma Plugin API postMessage](https://www.figma.com/plugin-docs/api/properties/figma-ui-postmessage/) — wire format for plugin communication; HIGH confidence
- [Chrome Extension Message Passing](https://developer.chrome.com/docs/extensions/develop/concepts/messaging) — `sendMessage` / `onMessage` pattern, sender identity via `MessageEvent.source`; HIGH confidence
- [Ably Capabilities documentation](https://ably.com/docs/auth/capabilities) — capability model (subscribe/publish per channel) scoped by server-issued tokens, never exposed to client code; HIGH confidence
- [Ably Token Auth](https://ably.com/docs/auth/token) — "Never use API keys in client-side code. Use token authentication." Confirms the pattern of server-scoped capability, client never manages keys; HIGH confidence
- [Salesforce LWS iframe docs](https://developer.salesforce.com/docs/platform/lightning-components-security/guide/lws-iframes.html) — LWS sandbox model; MEDIUM confidence (secondary reference)
- [Push API W3C spec](https://www.w3.org/TR/push-api/) — subscribe/unsubscribe pattern for platform-delivered events, app code never sees server auth tokens; HIGH confidence

---
*Feature research for: NUB spec simplification (napplet zero-crypto API)*
*Researched: 2026-04-07*
