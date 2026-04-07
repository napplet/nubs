# External Integrations

**Analysis Date:** 2026-04-07

## APIs & External Services

**Nostr Protocol Suite:**
- [Nostr Relays (NIP-01)](https://github.com/nostr-protocol/nips/blob/master/01.md) - WebSocket-based event relay protocol
  - Referenced in: `NUB-RELAY.md`
  - Purpose: Defines relay proxy semantics for napplet relay access
  
- [NIP-07 Browser Extension Standard](https://github.com/nostr-protocol/nips/blob/master/07.md) - `window.nostr` signing interface
  - Referenced in: `NUB-SIGNER.md`
  - Purpose: NUB-SIGNER implements this interface inside sandboxed iframes
  
- [NIP-29 Group Relay Events (kind 29001)](https://github.com/nostr-protocol/nips/blob/master/29.md) - Group relay and IPC events
  - Referenced in: `NUB-RELAY.md`, `NUB-SIGNER.md`, `NUB-IPC.md`, `NUB-PIPES.md`
  - Purpose: Used for inter-process communication between napplets and shell
  - Custom kinds used: 29001, 29002, 29003, 29006, 29007
  
- [NIP-33 Replaceable Events](https://github.com/nostr-protocol/nips/blob/master/33.md) - Versioned event semantics
  - Referenced in: `NUB-NOSTRDB.md`
  - Purpose: Defines replaceable event lookup in local database
  
- [NIP-45 COUNT Protocol](https://github.com/nostr-protocol/nips/blob/master/45.md) - Event counting
  - Referenced in: `NUB-NOSTRDB.md`
  - Purpose: COUNT semantics for database queries

## Data Storage

**Local Storage:**
- Browser `localStorage` - Host application stores napplet data
  - Implementation: `NUB-STORAGE.md` provides async key-value interface
  - Scoping: Per-napplet namespace by `(dTag, aggregateHash)` composite key
  - Quota: 512 KB per napplet (reference implementation)
  - Persistence: Survives page reloads

**Event Database:**
- OPFS (Origin Private File System) or IndexedDB - Local Nostr event cache
  - Implementation: `NUB-NOSTRDB.md` provides query interface
  - Purpose: Cached events from relays, searchable and subscribable
  - Access: Scoped by napplet ACLs, validated by shell

## Authentication & Identity

**Auth Provider:**
- NIP-07 Browser Extension (or NIP-46 Remote Signer) - User signing authority
  - Implementation: Shell proxies `window.nostr` calls to actual signer
  - Flow: User's real signer remains outside sandbox; shell controls access
  - Per-napplet ACLs: `sign:event`, `sign:nip04`, `sign:nip44` capabilities

**Core Authentication:**
- NIP-5D delegation flow - Session key authentication
  - Establishes sandboxed napplet identity via delegated ephemeral keys
  - Shell verifies delegation before granting capabilities

## Monitoring & Observability

**Error Tracking:**
- Not detected

**Logs:**
- Not explicitly defined in specs
- Shells may implement internal logging for debugging

## CI/CD & Deployment

**Hosting:**
- GitHub - Repository hosting and PR review platform

**CI Pipeline:**
- Not detected - This is a specification repository without automated testing

## Environment Configuration

**Not applicable:**
- This repository contains specifications only, no runtime environment needed
- No environment variables required for this documentation repo
- Implementations in `@napplet/shim` and `@kehto/shell` repos handle actual environment configuration

## Webhooks & Callbacks

**postMessage IPC:**
- Browser `postMessage()` API - Core transport between shell and napplet iframe
  - Used by all NUB interfaces for bidirectional communication
  - Correlation IDs match requests to responses

**Subscription Push Model:**
- Shell pushes new events to napplets via postMessage
  - NUB-RELAY: `["EVENT", subId, event]` pushes
  - NUB-NOSTRDB: `event-push` method responses with `sub-id` tag
  - NUB-IPC: Topic-based pub/sub routing

## Related Implementation Repositories

**Reference Implementations (external dependencies):**
- [`@napplet/shim`](https://github.com/sandwichfarm/napplet) - Client-side napplet SDK
  - Implements napplet-side stubs for all NUB interfaces
  - Files: `packages/shim/src/relay-shim.ts`, `packages/shim/src/state-shim.ts`, etc.

- [`@kehto/shell`](https://github.com/sandwichfarm/kehto) - Reference shell implementation
  - Implements shell-side handlers for all NUB interfaces
  - Files: `packages/runtime/src/shell-bridge.ts`, `packages/runtime/src/signer-handler.ts`, `packages/runtime/src/state-handler.ts`

- [`sandwichfarm/hyprgate`](https://github.com/sandwichfarm/hyprgate) - Alternative reference shell
  - Production reference shell implementation

- [`nostr-protocol/nips`](https://github.com/nostr-protocol/nips) - Official NIP standards
  - Contains NIP-5D core specification and all referenced NIPs

## Protocol Communication Patterns

**Request-Response via postMessage:**
- Each request includes correlation `id` tag
- Shell responds with matching `id` tag for async/await resolution
- Timeout enforcement: 5-30 seconds (varies by operation type)

**Event Kind Routing:**
- Kind 29001: Signer requests/responses (REQUEST/RESPONSE)
- Kind 29003: Storage operations (IPC-PEER bus)
- Kind 29006: Database requests (NIPDB_REQUEST)
- Kind 29007: Database responses (NIPDB_RESPONSE)
- Custom topic tags (`t` tag) route to correct handler

---

*Integration audit: 2026-04-07*
