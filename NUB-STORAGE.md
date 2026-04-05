NUB-STORAGE
===========

Scoped Key-Value Storage
------------------------

`draft`

**NUB ID:** NUB-STORAGE
**Namespace:** `window.napplet.storage`
**Discovery:** `shell.supports("storage")`

## Description

NUB-STORAGE provides napplets with an async localStorage-like API. Without
`allow-same-origin`, iframes have opaque origins and cannot access localStorage
directly. This interface routes storage operations through the shell via
postMessage, which scopes data by napplet identity -- a composite key of
`(dTag, aggregateHash)` -- to enforce isolation between napplets. Different
napplet types and different versions of the same napplet have completely
separate storage namespaces.

## API Surface

```typescript
interface NappletStorage {
  getItem(key: string): Promise<string | null>;
  setItem(key: string, value: string): Promise<void>;
  removeItem(key: string): Promise<void>;
  keys(): Promise<string[]>;
}
```

**`getItem(key)`** -- Retrieves a stored value by key. Returns `null` if the
key does not exist, matching localStorage semantics.

**`setItem(key, value)`** -- Stores a key-value pair. Throws if the napplet
exceeds its storage quota.

**`removeItem(key)`** -- Deletes a stored key. No-op if the key does not exist.

**`keys()`** -- Lists all keys stored by this napplet.

All methods are async because they cross the postMessage boundary. Each request
includes a correlation ID; the shell responds with a matching ID so the shim
can resolve the correct Promise.

## Shell Behavior

- The shell MUST scope storage by composite key `(dTag, aggregateHash)`.
  Different napplet types and different versions of the same napplet MUST have
  isolated storage.
- The shell MUST enforce a per-napplet storage quota. A recommended default
  is 512 KB measured in UTF-8 byte count.
- The shell MUST persist storage to the host's localStorage (or equivalent)
  so data survives page reloads.
- The shell MUST respond to storage requests within a reasonable timeout.
- The shell MUST return structured error responses for quota violations
  (`"quota exceeded: ..."`) and invalid requests (`"missing key tag"`).
- The shell MUST validate storage key prefixes to prevent scope escape (a
  napplet MUST NOT be able to read or write another napplet's data).
- The shell SHOULD support backward-compatible migration across storage key
  format changes.

## Transport

Storage operations are transported via postMessage between the napplet iframe
and the shell. Each request includes a correlation ID; the shell responds with
a matching ID so the napplet can resolve the correct Promise. The internal
message format is an implementation detail of the shell.

## Security Considerations

- Storage is scoped by composite key `(dTag, aggregateHash)`. A napplet MUST
  NOT be able to read or write another napplet's data.
- The shell MUST validate storage key prefixes to prevent scope escape.
- Storage quota enforcement prevents a single napplet from consuming
  unbounded host storage.
- Storage values are strings only. The shell SHOULD NOT attempt to parse or
  execute stored content.
- The shell MAY enforce ACL checks on `storage:read` and `storage:write`
  capabilities before processing storage requests.
- Storage keys and values are visible to the shell. Napplets that need
  confidential storage should encrypt values before storing.

