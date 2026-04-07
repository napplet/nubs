NUB-IFC
=======

Inter-Frame Communication
-------------------------

`draft`

**NUB ID:** NUB-IFC
**Namespace:** `window.napplet.ifc`
**Discovery:** `shell.supports("ifc")`

## Description

NUB-IFC provides topic-based publish/subscribe for communication between napplets. Sandboxed iframes cannot communicate directly because the `allow-same-origin` sandbox token is absent — they have opaque origins with no shared context. The shell routes messages between napplets using typed `ifc.*` messages over the NIP-5D wire format. This is loose coupling: the sender does not know who (if anyone) receives the message. The shell also uses IFC topics for internal coordination (state operations, service commands, configuration).

## API Surface

```typescript
interface NappletIfc {
  emit(topic: string, payload?: unknown): void;         // via ifc.emit
  on(topic: string, callback: (event: IfcEvent) => void): Subscription;  // via ifc.subscribe
}

interface IfcEvent {
  topic: string;
  sender: string;    // sender dTag (napplet type identifier, per SPEC.md)
  payload: unknown;
}

interface Subscription {
  close(): void;     // via ifc.unsubscribe
}
```

**`emit(topic, payload?)`** — Broadcasts a message to all napplets subscribed to the given topic. Fire-and-forget — there is no delivery confirmation. The shell identifies the sender via `MessageEvent.source` (per SPEC.md) and includes the sender's `dTag` in delivered events.

**`on(topic, callback)`** — Subscribes to messages on a topic. The callback receives an `IfcEvent` with the topic, sender `dTag`, and payload. Returns a `Subscription` handle with a `close()` method to unsubscribe. Multiple subscriptions to the same topic are independent.

## Wire Protocol

IFC operations use the NIP-5D wire format (`{ "type": "domain.action", ...payload }`).

| Type | Direction | Payload fields |
|------|-----------|----------------|
| `ifc.emit` | napplet -> shell | `topic`, `payload`? |
| `ifc.subscribe` | napplet -> shell | `id`, `topic` |
| `ifc.subscribe.result` | shell -> napplet | `id` |
| `ifc.unsubscribe` | napplet -> shell | `id`, `topic` |
| `ifc.event` | shell -> napplet | `topic`, `sender` (dTag), `payload`? |

Key design notes:

- `ifc.emit` has no `id` field — it is fire-and-forget with no acknowledgment.
- `ifc.subscribe` uses `id` for correlation so the shim can confirm the subscription was registered.
- `ifc.subscribe.result` confirms registration by echoing the `id`.
- `ifc.unsubscribe` uses `id` for correlation.
- `ifc.event` has no `id` field — it is a shell-initiated delivery identified by `topic` and `sender` (the emitting napplet's `dTag` string per SPEC.md).

### Examples

**Emit:**
```
-> { "type": "ifc.emit", "topic": "profile:open", "payload": { "pubkey": "abc123..." } }
```
No response — fire-and-forget.

**Subscribe:**
```
-> { "type": "ifc.subscribe", "id": "a1", "topic": "profile:open" }
<- { "type": "ifc.subscribe.result", "id": "a1" }
```

**Event delivery:**
```
<- { "type": "ifc.event", "topic": "profile:open", "sender": "social-feed", "payload": { "pubkey": "abc123..." } }
```

**Unsubscribe:**
```
-> { "type": "ifc.unsubscribe", "id": "b2", "topic": "profile:open" }
```

### Error Handling

`ifc.subscribe.result` MAY include an `error` field (string) if the shell rejects the subscription. When `error` is present, the subscription was not registered.

```
<- { "type": "ifc.subscribe.result", "id": "a1", "error": "topic rejected by ACL" }
```

## Topic Conventions

Topics use a prefix convention to signal direction and scope:

| Prefix | Direction | Meaning |
|--------|-----------|---------|
| `shell:*` | napplet -> shell | Commands sent by a napplet to the shell (e.g., `shell:state-get`) |
| `napplet:*` | shell -> napplet | Responses/notifications from shell to napplet (e.g., `napplet:state-response`) |
| `{domain}:*` | bidirectional | Domain-scoped messages between napplets (e.g., `profile:open`, `chat:open-dm`) |

These conventions are advisory. The shell routes by topic match, not by prefix parsing. A napplet can subscribe to any topic regardless of prefix.

## Shell Behavior

- The shell MUST route `ifc.emit` messages to all napplets subscribed to the matching topic.
- The shell MUST identify the sender via `MessageEvent.source` and include the sender's `dTag` in delivered `ifc.event` messages (per SPEC.md identity model).
- The shell MUST NOT deliver `ifc.event` back to the emitting napplet (sender exclusion).
- The shell MUST respond to `ifc.subscribe` with `ifc.subscribe.result` carrying the same `id`.
- The shell MUST honor `ifc.unsubscribe` by removing the subscription for that topic.
- The shell MAY intercept specific topic prefixes (e.g., `shell:*`) for internal command handling rather than routing them to other napplets.
- The shell MAY enforce ACL checks on IFC capabilities and reject subscriptions or emits that violate shell policy.

## Security Considerations

Sender identity is shell-enforced via `MessageEvent.source` mapping to napplet identity (per SPEC.md). There is no per-message signing — the shell's sender identification is the trust boundary. `MessageEvent.source` is unforgeable within the same browsing context.

- Topic namespaces are not enforced — any napplet can emit on any topic. The shell MAY restrict topics via ACL.
- Payloads are opaque to the shell. Receiving napplets are responsible for validating payload content.
- Sender exclusion prevents echo loops but does not prevent a napplet from emitting messages on any topic. Receivers should check the `sender` dTag if sender identity matters for their use case.
- For high-frequency point-to-point communication where per-message overhead matters, use NUB-PIPES instead.

## Relationship to NUB-PIPES

NUB-IFC is for loose-coupled, topic-based pub/sub. NUB-PIPES is for tight-coupled, point-to-point connections. Both coexist in the napplet protocol: IFC for infrequent coordination (UI commands, state sync, configuration), pipes for sustained data streams (real-time collaboration, media).
