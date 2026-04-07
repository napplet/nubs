NUB-RELAY
=========

Relay Proxy
-----------

`draft`

**NUB ID:** NUB-RELAY
**Namespace:** `window.napplet.relay`
**Discovery:** `shell.supports("relay")`

## Description

NUB-RELAY provides napplets with relay access through the shell. Sandboxed iframes cannot open WebSocket connections directly because the `allow-same-origin` sandbox token is absent. The shell acts as a relay proxy, forwarding typed relay messages to its connected relay pool and delivering matching events back to the napplet. This is the most fundamental shell capability -- without it, napplets have no way to read or write Nostr events.

## API Surface

```typescript
interface NappletRelay {
  subscribe(                                         // via relay.subscribe
    filters: NostrFilter | NostrFilter[],
    options?: { relay?: string },
  ): Subscription;

  publish(event: NostrEvent): Promise<void>;         // via relay.publish

  query(                                             // via relay.query
    filters: NostrFilter | NostrFilter[],
  ): Promise<NostrEvent[]>;
}

interface Subscription {
  on(event: 'event', cb: (event: NostrEvent) => void): void;
  on(event: 'eose', cb: () => void): void;
  close(): void;                                     // via relay.close
}
```

**`subscribe(filters, options?)`** -- Opens a live subscription. The shell queries its relay pool, streaming matching events back as `relay.event` messages. Returns a handle to listen for events and EOSE, and to close the subscription. When `options.relay` is provided, the subscription targets a specific relay (e.g., for NIP-29 group relays) instead of the shared pool.

**`publish(event)`** -- Publishes a signed Nostr event to the shell's relay pool. The napplet MUST sign the event via `window.nostr.signEvent(template)` before calling publish. The shell forwards the signed event to relays and returns the result.

**`query(filters)`** -- Convenience wrapper: subscribes, collects events until EOSE, then closes the subscription and resolves the Promise with the collected events.

All methods are async because they cross the postMessage boundary. Requests include correlation IDs; the shell responds with matching IDs so the shim can resolve the correct Promise or route events to the correct subscription.

## Wire Protocol

Relay operations use the NIP-5D wire format. Requests include an `id` field for correlation. Subscription-scoped messages use a `subId` field to identify which subscription they belong to.

| Type | Direction | Payload fields |
|------|-----------|----------------|
| `relay.subscribe` | napplet -> shell | `id`, `subId`, `filters` (NostrFilter array), `relay`? (string) |
| `relay.close` | napplet -> shell | `id`, `subId` |
| `relay.publish` | napplet -> shell | `id`, `event` (signed NostrEvent) |
| `relay.query` | napplet -> shell | `id`, `filters` (NostrFilter array) |
| `relay.event` | shell -> napplet | `subId`, `event` (NostrEvent) |
| `relay.eose` | shell -> napplet | `subId` |
| `relay.closed` | shell -> napplet | `subId`, `reason`? (string) |
| `relay.publish.result` | shell -> napplet | `id`, `ok` (boolean), `eventId`? (string), `error`? (string) |
| `relay.query.result` | shell -> napplet | `id`, `events` (NostrEvent array) |

### NostrFilter

Filters follow the NIP-01 filter format:

```json
{ "kinds": [1], "authors": ["<hex>"], "#e": ["<hex>"], "#p": ["<hex>"], "since": 1234, "until": 5678, "limit": 50 }
```

All filter fields are optional. Multiple filters in the `filters` array are OR-combined.

### Examples

**Subscribe:**
```
-> { "type": "relay.subscribe", "id": "a1", "subId": "sub-1", "filters": [{ "kinds": [1], "limit": 50 }] }
<- { "type": "relay.event", "subId": "sub-1", "event": { "id": "abc...", "pubkey": "def...", "kind": 1, "content": "hello", "tags": [], "created_at": 1234567890, "sig": "..." } }
<- { "type": "relay.eose", "subId": "sub-1" }
```

**Publish (napplet signs first, then publishes):**
```javascript
// Step 1: Napplet signs the event via NIP-07
const signed = await window.nostr.signEvent({ kind: 1, content: "hello world", tags: [], created_at: Math.floor(Date.now()/1000) });

// Step 2: Napplet sends the signed event to the shell
-> { "type": "relay.publish", "id": "b2", "event": { "id": "abc...", "pubkey": "def...", "kind": 1, "content": "hello world", "tags": [], "created_at": 1234567890, "sig": "ghi..." } }
<- { "type": "relay.publish.result", "id": "b2", "ok": true, "eventId": "abc..." }
```

**Query:**
```
-> { "type": "relay.query", "id": "c3", "filters": [{ "authors": ["def..."], "kinds": [0], "limit": 1 }] }
<- { "type": "relay.query.result", "id": "c3", "events": [{ "id": "xyz...", "pubkey": "def...", "kind": 0, "content": "{...}", "tags": [], "created_at": 1234567890, "sig": "..." }] }
```

**Close subscription:**
```
-> { "type": "relay.close", "id": "d4", "subId": "sub-1" }
```

**Scoped relay:**
```
-> { "type": "relay.subscribe", "id": "e5", "subId": "sub-2", "filters": [{ "kinds": [9, 10, 11, 12] }], "relay": "wss://groups.example.com" }
```

**Error in publish:**
```
<- { "type": "relay.publish.result", "id": "b2", "ok": false, "error": "blocked: content policy violation" }
```

### Error Handling

`relay.publish.result` includes `ok: false` and an `error` string on failure. `relay.closed` indicates the shell terminated a subscription, with an optional `reason`. `relay.query.result` MAY include an `error` field instead of `events` if the query fails.

## Shell Behavior

- The shell MUST forward subscriptions to its relay pool and deliver matching events as `relay.event` messages to the subscribing napplet.
- The shell MUST send `relay.eose` when stored events for a subscription are exhausted.
- The shell MUST honor `relay.close` by tearing down the subscription and ceasing event delivery for that `subId`.
- The shell MUST forward signed events from `relay.publish` to its relay pool and respond with `relay.publish.result`.
- The shell MUST verify event signatures before forwarding to relays. Events with invalid signatures MUST be rejected with `relay.publish.result { ok: false, error: "invalid signature" }`.
- The shell MUST respond to `relay.query` by collecting events until EOSE and returning them in `relay.query.result`.
- The shell MUST respond to every request with a result or lifecycle message carrying the same `id` or `subId`.
- The shell MAY send `relay.closed` to indicate a subscription was terminated by the shell (e.g., due to an error, resource limit, or policy).
- The shell MAY enforce ACL checks on relay read and relay write capabilities before processing subscribe or publish messages.
- The shell MAY support scoped relay connections when the `relay` field is present, targeting a specific relay independently from the shared pool.
- The shell MAY manage a relay pool internally -- which relays to connect to, reconnection strategy, and load balancing are implementation details.

## Security Considerations

- The shell controls which relays the napplet can access. Napplets cannot open arbitrary WebSocket connections.
- The shell MUST verify event signatures before relay broadcast. Events with invalid signatures MUST be rejected.
- The shell MAY filter or reject subscriptions and publishes based on ACL capabilities.
- Scoped relay URLs SHOULD be validated by the shell to prevent SSRF-like abuse (e.g., napplets targeting internal network addresses via `wss://`).
- Subscription filters SHOULD be validated to prevent resource exhaustion (e.g., unbounded subscriptions without `limit`).
- The napplet signs events via `window.nostr.signEvent()` (NIP-07, provided by the shell per NIP-5D). The shell does not sign on behalf of the napplet for relay publishes -- napplets own their signing.
