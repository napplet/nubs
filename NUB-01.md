NUB-01
======

Profile Topics
--------------

`draft`

**NUB ID:** NUB-01

**Domain:** profile topic coordination

**Requires:** NUB-IFC

**Discovery:** `shell.supports("ifc", "NUB-01")`

## Description

This protocol defines the `profile:*` topic family for napplets that coordinate
profile-related actions through NUB-IFC. The current revision defines
`profile:open`. Future compatible revisions may define additional `profile:*`
topics without requiring a separate numbered NUB for every small profile action.

This protocol specifies topic strings, payload shapes, producer behavior, and
consumer behavior. It does not redefine NUB-IFC transport methods.

## Message Protocol

Napplets coordinate through NUB-IFC. The IFC event topic is carried by the
event's `t` tag and the payload is carried as JSON in the event content. This
NUB defines the meaning of selected `profile:*` topics; `ifc.emit`,
`ifc.subscribe`, `ifc.unsubscribe`, and `ifc.event` remain generic NUB-IFC
transport.

Topics in this family MUST use the `profile:` prefix and MUST describe profile
identity, profile display, or profile navigation behavior. A topic that belongs
to another product domain, even when it references a public key, should use its
own domain family instead of being placed here.

### `profile:open`

Producers send `profile:open` when a user action selects a profile identity,
for example from an author name, avatar, mention, contact list, or similar
profile affordance.

Consumers receive `profile:open` when they can open, focus, or otherwise
present a profile view for the requested public key.

Payload:

```json
{
  "pubkey": "<hex-pubkey>"
}
```

Fields:

| Field | Type | Required | Semantics |
|-------|------|----------|-----------|
| `pubkey` | string | yes | 64-character lowercase hexadecimal Nostr public key for the profile to open. |

Producer requirements:

- Producers MUST include `pubkey`.
- Producers SHOULD use canonical lowercase hexadecimal public keys.
- Producers MUST NOT require a specific shell windowing policy or profile
  napplet implementation.

Consumer requirements:

- Consumers MUST ignore payloads without a valid `pubkey`.
- Consumers SHOULD open or focus a profile view for `pubkey`.
- Consumers MAY use local profile caches or relay lookups to render metadata.
- Consumers MAY ignore the message when profile navigation is unsupported in
  the current context.

## Negotiation

Napplets discover support for this protocol with:

```ts
window.napplet.shell.supports("ifc", "NUB-01");
```

A napplet that requires IFC transport declares the interface dependency in its
manifest:

```
["requires", "ifc"]
```

The manifest declares only the interface dependency. The numbered protocol is
negotiated at runtime with `shell.supports()`.

## Non-goals

- Defining generic IFC transport.
- Defining shell window placement, focus, or routing policy.
- Defining profile metadata fetching, caching, or rendering.
- Defining direct-message, chat, relay, or stream coordination topics.

## Implementations

- (none yet)
