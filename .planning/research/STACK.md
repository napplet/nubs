# Stack Research

**Domain:** postMessage wire protocol design for iframe-sandboxed apps with host-delegated identity
**Researched:** 2026-04-07
**Confidence:** HIGH (NIP-01/NIP-5D from canonical source; prior art from MCP Apps, NIP-46, NIP-42 from official docs; iframe sandboxing from MDN)

---

## What This Research Covers

The "stack" for a spec rewrite is not npm packages. It is the **protocol vocabulary** the rewrite draws on: wire format conventions, identity delegation patterns, and the specific parts of Nostr wire format that map cleanly to a simplified napplet API. This document answers three questions:

1. What does the right wire format look like when napplets have zero crypto responsibility?
2. How do real systems delegate identity from a sandboxed iframe to a trusted host?
3. Which parts of NIP-01 map cleanly to napplet-facing API, and which are runtime-internal?

---

## Core Protocol Building Blocks

### NIP-5D Baseline (HIGH confidence — source: `/home/sandwich/Develop/napplet/specs/NIP-5D.md`)

NIP-5D already establishes the right transport layer. The rewrite does not change it. Key properties:

- **Verb-array wire format**: `["VERB", ...args]` throughout, identical to NIP-01
- **`MessageEvent.source` for sender auth**: unforgeable Window reference, no per-message crypto needed
- **"Verify once, trust source"**: AUTH handshake is one-time Schnorr signature; after that, the shell trusts the Window reference
- **Pre-AUTH queue**: shell queues up to 50 messages before AUTH completes, then replays or drops

The critical insight already in NIP-5D: the AUTH handshake establishes identity *once*. After that, message sender identity comes from `MessageEvent.source`, not cryptographic signatures. NUB-IPC's current design (per-message Schnorr signing of kind 29003 events) is redundant with the security model NIP-5D already provides.

### NIP-01 Wire Format (HIGH confidence — source: official NIP-01)

The six NIP-01 verbs and their shapes:

```
# Client → Relay
["REQ", <sub_id>, <filter>, ...]   -- open subscription
["EVENT", <event_json>]            -- publish event
["CLOSE", <sub_id>]                -- close subscription
["COUNT", <sub_id>, <filter>, ...] -- count events (NIP-45)

# Relay → Client
["EVENT", <sub_id>, <event_json>]  -- deliver matching event
["EOSE", <sub_id>]                 -- end of stored events
["OK", <event_id>, <bool>, <msg>]  -- publish ack/reject
["CLOSED", <sub_id>, <msg>]        -- subscription closed by relay
["NOTICE", <msg>]                  -- human-readable status
```

NIP-5D's napplet-to-shell and shell-to-napplet tables map 1:1 to these, plus `REGISTER`/`IDENTITY`. The rewrite keeps all of this.

**Filter shape** (the napplet-facing `subscribe(filters, ...)` and `query(filters)` already use this):
```json
{
  "kinds": [1, 6],
  "authors": ["pubkey_hex"],
  "ids": ["event_id_hex"],
  "#t": ["topic"],
  "since": 1700000000,
  "until":  1800000000,
  "limit": 100
}
```

### NIP-46 Remote Signer Pattern (HIGH confidence — source: official NIP-46)

NIP-46 ("bunker") is the canonical prior art for "app does not hold private key; host does signing." The app sends an unsigned event template; the bunker signs it and returns the result. The app never sees the private key.

Wire format (encrypted over Nostr relay, but the pattern is the same for postMessage):
```json
// Request (app → signer)
{ "id": "<uuid>", "method": "sign_event", "params": [<event_template_json>] }

// Response (signer → app)
{ "id": "<uuid>", "result": "<signed_event_json>", "error": null }
```

Methods: `sign_event`, `get_public_key`, `nip44_encrypt`, `nip44_decrypt`, `connect`, `ping`.

**Direct mapping to NUB-RELAY rewrite**: `publish(template)` already follows this — napplet submits an `EventTemplate` (no pubkey, no id, no sig), shell signs and broadcasts. The current spec's mention of `window.nostr.signEvent()` calling into the napplet layer is a spec artifact to remove; the shell handles it transparently.

**Note**: NIP-26 (delegated event signing) is explicitly deprecated/unrecommended as of 2024 per official NIPs repo: "adds unnecessary burden for little gain." The NUB spec rewrite should not reference NIP-26.

### NIP-42 AUTH Pattern (HIGH confidence — source: official NIP-42 + NIP-5D)

NIP-42 defines the challenge-response handshake NIP-5D builds on:
```
Relay:  ["AUTH", "<challenge_string>"]
Client: ["AUTH", { "kind": 22242, "tags": [["relay", "..."], ["challenge", "..."]], ... }]
Relay:  ["OK", "<event_id>", true, ""]
```

NIP-5D extends this for napplets with the REGISTER/IDENTITY preamble, then the same AUTH challenge/response. The signed event uses the *delegated* session key (not the user's key). This is entirely the shell's concern. The napplet's AUTH response is constructed and signed by the shim using the IDENTITY-delivered private key — the napplet never "does" this consciously; the shim handles it on first load.

---

## Identity Delegation Patterns in Iframe Contexts

### Pattern 1: Verify-Once, Trust-Source (NIP-5D's model — HIGH confidence)

**How it works**: Single AUTH handshake at connection time. After handshake, the shell trusts all messages from the verified `MessageEvent.source` (unforgeable Window reference). No per-message signature.

**Prior art**: NIP-5D Section "Post-AUTH" states explicitly "verify once, trust source." This is why NUB-IPC's per-message Schnorr signing is over-engineering — the shell already knows which iframe sent which message.

**Implication for rewrite**: IPC messages do not need to be NIP-01 events with `kind`, `tags`, `pubkey`, `id`, `sig`. They can be lightweight verb arrays: `["IPC", topic, payload]`.

### Pattern 2: Host-Mediated Tool Delegation (MCP Apps — HIGH confidence, source: official MCP Apps spec)

MCP Apps (Jan 2026, supported by Claude, VS Code Copilot, Goose) use sandboxed iframes with the same "host owns identity/capabilities, app sends requests" model:

```json
// App → Host (sandboxed iframe → parent)
{ "jsonrpc": "2.0", "id": 42, "method": "tools/call", "params": { "name": "toolName", "arguments": {} } }

// Host → App (result delivery)
{ "jsonrpc": "2.0", "id": 42, "result": { "content": [{ "type": "text", "text": "..." }] } }
```

The app never holds credentials, API keys, or signing material. The host validates that the tool call is permitted, executes it using the user's connected capabilities, and returns the result. The app only sees the result.

**Mapping to napplet**: The shell is the "host." NUB-RELAY `publish(template)` is equivalent to `tools/call` with `name: "relay/publish"`. The napplet sends an intent; the shell resolves it using the user's identity.

**Key MCP Apps security rule**: "Host MUST reject tool calls from apps that aren't declared in the app's capability manifest." This maps to NIP-5D's `requires` tags in NIP-5A manifests.

### Pattern 3: Web Worker Signing Proxy (Mercari pattern — MEDIUM confidence, source: Mercari Engineering blog)

Web Workers isolate private tokens from the main thread. The main thread sends requests without credentials; the worker injects credentials before forwarding to the backend. The main thread never sees the token.

The postMessage protocol between main thread and worker uses UUID correlation:
```
Main thread: { id: "<uuid>", url: "/api/resource", method: "GET" }
Worker:      { id: "<uuid>", status: 200, body: { ... } }
```

**Mapping to napplet**: The shell is the "worker" (in the role sense, not literally). NUB-STORAGE's correlation ID pattern (`id` tag in kind 29003 events) is already following this correctly. The rewrite simplifies by dropping the NIP-01 event wrapper around storage requests.

### Pattern 4: Figma Plugin postMessage (MEDIUM confidence, source: Figma Plugin API docs)

Figma uses a minimal `{ pluginMessage: <payload> }` envelope for plugin-to-UI communication. The plugin code (privileged, document access) is separate from the UI (sandboxed iframe). Identity is entirely in the plugin layer; the UI just sends/receives data.

Key pattern: **message queuing on startup**. Figma queues messages sent before the UI finishes loading, then replays them. NIP-5D already has this (50-message pre-AUTH queue). The rewrite can rely on this existing behavior.

### Pattern 5: Zoid Cross-Domain Components (PayPal — MEDIUM confidence, source: krakenjs/zoid GitHub)

Zoid passes props from parent to child iframe via `window.xprops`. Identity/auth tokens flow parent → child as props, not the other way. The child receives what it needs; it cannot read the parent's full context.

**Relevant for NUB-PIPES**: The auth-on-open model (PIPE_OPEN → PIPE_ACK with peer identity delivered by shell) is the right pattern. The napplet does not hold peer pubkeys in raw form for security decisions; the shell delivers opaque peer identity at pipe open time.

---

## Wire Format: Verb Arrays vs JSON-RPC Objects

Both patterns are well-established. The choice determines spec readability and implementation complexity.

### Verb Arrays (`["VERB", arg1, arg2]`) — NIP-01 / NIP-5D model

Pros:
- Matches existing NIP-01 format exactly — relay developers already know it
- Minimal bytes per message (no `jsonrpc`, `method`, `params` keys)
- Simple to parse: `const [verb, ...args] = msg`
- NIP-5D and all 6 existing NUBs already use this

Cons:
- No standard correlation ID mechanism (must be embedded as an arg position or named object)
- Less self-documenting than named fields

### JSON-RPC 2.0 (`{ "jsonrpc": "2.0", "id": N, "method": "...", "params": {} }`) — MCP Apps / LSP model

Pros:
- Standardized correlation: `id` field in every request/response pair
- Well-known by frontend devs
- Tooling support (JSON-RPC inspector tools)

Cons:
- 3-4x larger per message for simple cases
- Not idiomatic in Nostr ecosystem
- Would break consistency with NIP-5D's existing verb-array format

**Recommendation**: Stay with verb arrays. The rewrite is building on NIP-5D, not replacing it. The NUB-PIPES spec already demonstrates that verb arrays work cleanly for non-NIP-01 message types (`PIPE_OPEN`, `PIPE_ACK`, `PIPE`, `PIPE_CLOSE`, `PIPE_CLOSED`, `PIPE_ERROR`). This should be the pattern for all NUBs.

For request/response correlation (STORAGE, NOSTRDB, SIGNER), embed a correlation ID as the second argument:
```
["STORAGE_GET", "<id>", "<key>"]
["STORAGE_RESULT", "<id>", "<value_or_null>"]
```

---

## What the Napplet API Should Expose vs Keep Internal

### Napplet-facing (spec layer): zero crypto

The napplet sees:

| Interface | What napplet sees | What it does NOT see |
|-----------|------------------|----------------------|
| NUB-RELAY `publish(template)` | `{ kind, tags, content }` — no pubkey, no id, no sig | Signing, delegation, event ID computation |
| NUB-RELAY `subscribe(filters, cb)` | `NostrEvent` objects (fully signed, readable) | Which relay served the event, session key mechanics |
| NUB-IFC `emit(topic, payload)` | `emit("topic", { data })` — plain JS object | Signed event envelope, Schnorr overhead |
| NUB-IFC `on(topic, cb)` | `(payload: unknown) => void` — parsed data | Raw event pubkey, raw tags, raw kind |
| NUB-STORAGE | `getItem/setItem/removeItem/keys` — plain strings | Scoping key, composite key computation |
| NUB-NOSTRDB `add(event)` | Full `NostrEvent` (already signed, readable) | Signature verification (shell does this) |
| NUB-PIPES `send(payload)` | Plain JS value, pipe ID opaque | Session auth, peer pubkey resolution |

### Runtime-internal (implementation layer): full Nostr crypto

The shell handles:
- Deriving and holding the delegated session keypair (HMAC-SHA256 of shellSecret + dTag + hash)
- Signing IPC messages (if still using NIP-01 events internally — or not, if using verb arrays)
- Signing the AUTH kind 22242 event (shim does this using IDENTITY-provided privkey; napplet code does not)
- Verifying event signatures before storing in NOSTRDB
- Scoping storage keys by composite (dTag, aggregateHash)
- Routing pipe messages between iframe Windows by `MessageEvent.source`

### The NUB-SIGNER question

NUB-SIGNER exposes `window.nostr.signEvent(template)` to napplets. This is intentional and stays — it lets napplets publish events as the *user's real identity* (not the session key), for things like posting notes. What changes: napplets should not need to call `window.nostr.signEvent()` *before* calling `relay.publish()`. Those should be decoupled. `relay.publish(template)` should handle signing internally (using the session key for internal events, or delegating to the user signer for user-identity events). The napplet should not need to compose the signing chain manually.

---

## NIP-01 Message Format Mapping

How NIP-01 relay verbs map to simplified napplet-facing calls:

| NIP-01 Verb | Current wire (shell) | Napplet API (spec layer) | Notes |
|-------------|---------------------|--------------------------|-------|
| `REQ` | `["REQ", subId, ...filters]` | `subscribe(filters, onEvent, onEose)` | Shim generates subId; napplet never sees it |
| `EVENT` (c→r) | `["EVENT", signedEvent]` | `publish(template)` → returns `NostrEvent` | Template = `{kind, tags, content}`; shell signs |
| `CLOSE` | `["CLOSE", subId]` | `sub.close()` | Shim manages subId lifecycle |
| `EVENT` (r→c) | `["EVENT", subId, event]` | `onEvent(event: NostrEvent)` callback | Delivered to correct callback by subId |
| `EOSE` | `["EOSE", subId]` | `onEose()` callback | Passed through |
| `OK` | `["OK", eventId, bool, msg]` | Promise resolve/reject on `publish()` | Shim converts to async |
| `CLOSED` | `["CLOSED", subId, msg]` | Subscription handle emits close event | Shim maps back to handler |
| `COUNT` | `["COUNT", subId, filter]` | `query(filters)` + counting | Or `nostrdb.count(filters)` |

Verb arrays that exist in NUB-PIPES wire format (not NIP-01) are the right model for IFC, STORAGE, and NOSTRDB too:

```
# NUB-IFC simplified (no NIP-01 event wrapper needed)
["IFC_EMIT", "<topic>", <payload_json>]         # napplet → shell
["IFC_EVENT", "<topic>", <payload_json>]         # shell → napplet

# NUB-STORAGE simplified
["STORAGE_GET", "<id>", "<key>"]                 # napplet → shell
["STORAGE_SET", "<id>", "<key>", "<value>"]      # napplet → shell
["STORAGE_RESULT", "<id>", "<value|null>", <ok>] # shell → napplet

# NUB-NOSTRDB simplified (keep NIP-01 semantics for query/subscribe)
# query and subscribe already use REQ/EOSE/CLOSE — keep those
# add() can be a simple request/response
["DB_ADD", "<id>", <event_json>]                 # napplet → shell
["DB_RESULT", "<id>", <bool>]                    # shell → napplet
```

---

## Recommended Conventions for the Rewrite

### 1. Use verb arrays throughout

`["VERB", ...args]` is the established NIP-5D idiom. All new message types in the rewrite follow this. Never introduce JSON-RPC objects into the wire format.

### 2. Correlation IDs are positional, not named

For request/response pairs, the correlation ID is always the second element:
```
["REQUEST_VERB", "<uuid>", ...params]
["RESPONSE_VERB", "<uuid>", ...result]
```

### 3. EventTemplate is the napplet-facing publish type

A publish request is `{ kind: number, tags: string[][], content: string }`. No `pubkey`, no `id`, no `sig`. The shell computes and adds these. This is the NIP-07 pattern for `signEvent()` — it already accepts templates — extended to the relay proxy.

### 4. NostrEvent is the napplet-facing receive type

Napplets receive fully formed `NostrEvent` objects from `subscribe()` and `query()`. They are read-only data. The napplet never constructs one; it only consumes them.

### 5. IFC replaces IPC (per CLAUDE.md naming: NUB-IFC)

The rename from IPC to IFC is in CLAUDE.md. The wire format for IFC drops the NIP-01 event wrapper (kind 29003 with signed event). IFC messages are lightweight verb arrays. The shell still uses `MessageEvent.source` for sender identity — this is the security model, not per-message signatures.

### 6. Shell behavior sections document the crypto

Each NUB spec has two audiences: napplet developers (spec layer — no crypto) and shell implementors (implementation layer — full crypto). Shell Behavior sections document what the runtime must do. These sections are where signing, key derivation, signature verification, and ACL enforcement live.

---

## Anti-Patterns to Avoid in the Rewrite

| Anti-Pattern | Why Wrong | What Instead |
|--------------|-----------|--------------|
| Napplet calls `window.nostr.signEvent()` before `relay.publish()` | NUB-RELAY should be self-contained; napplet should not need to sign before publishing | `relay.publish(template)` handles signing internally |
| IFC messages wrapped in kind 29003 signed events | Per-message Schnorr signing is redundant with `MessageEvent.source` identity from AUTH | Lightweight verb arrays, shell uses source for identity |
| Exposing `pubkey` in PipeHandle for napplet security decisions | Napplet should not make trust decisions based on raw pubkeys; shell mediates trust | Shell-opaque `peerId` or `dTag` only |
| NUB-SIGNER spec describing `window.nostr.signEvent()` as napplet-initiated crypto | Napplets CAN use the signer but should not NEED to for routine operations | Signer is for user-identity publishing; relay operations use session key internally |
| NIP-26 delegation references | NIP-26 is deprecated ("unnecessary burden for little gain") | NIP-5D's HMAC-derived session key model is the right delegation approach |
| Spec exposing `privkey` to napplet (IDENTITY payload issue) | NIP-5D currently sends privkey in IDENTITY; this is the spec's own security gap to address | IDENTITY should only send pubkey; shim uses it to construct AUTH event internally |

---

## Sources

- NIP-5D spec: `/home/sandwich/Develop/napplet/specs/NIP-5D.md` — transport, AUTH handshake, wire format, security model (HIGH confidence)
- NIP-01: `https://github.com/nostr-protocol/nips/blob/master/01.md` — REQ/EVENT/CLOSE/EOSE/OK/CLOSED/NOTICE message shapes (HIGH confidence)
- NIP-42: `https://nips.nostr.com/42` — AUTH challenge-response, kind 22242 (HIGH confidence)
- NIP-46: `https://github.com/nostr-protocol/nips/blob/master/46.md` — remote signer protocol, EventTemplate pattern, app-facing vs signer-internal split (HIGH confidence)
- NIP-26 deprecation: `https://nips.nostr.com/26` — "unrecommended: adds unnecessary burden for little gain" (HIGH confidence)
- MCP Apps specification: `https://modelcontextprotocol.io/extensions/apps/overview` — host-delegated tool calling from sandboxed iframe, JSON-RPC over postMessage, security model (HIGH confidence — live production spec, Jan 2026)
- MCP Apps ext-apps spec: `https://github.com/modelcontextprotocol/ext-apps/blob/main/specification/2026-01-26/apps.mdx` — exact wire format, initialization handshake, tool delegation details (HIGH confidence)
- Mercari Web Worker signing proxy: `https://engineering.mercari.com/en/blog/entry/20220930-building-secure-apps-using-web-workers/` — UUID correlation pattern, app-layer zero-crypto pattern (MEDIUM confidence)
- Figma Plugin API: `https://www.figma.com/plugin-docs/api/properties/figma-ui-postmessage/` — message queuing on startup, minimal envelope pattern (MEDIUM confidence)
- krakenjs/zoid: `https://github.com/krakenjs/zoid` — xprops identity delegation, cross-domain component pattern (MEDIUM confidence)
- MDN iframe sandbox: `https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe` — opaque origin behavior, storage access restrictions (HIGH confidence)

---

*Stack research for: NUB spec rewrite — zero-crypto napplet API with host-delegated identity*
*Researched: 2026-04-07*
