NUB-CONFIG
==========

Per-Napplet Declarative Configuration
-------------------------------------

`draft`

**NUB ID:** NUB-CONFIG
**Namespace:** `window.napplet.config`
**Discovery:** `shell.supports("config")`
**Parent:** NIP-5D

## Description

NUB-CONFIG provides per-napplet declarative configuration. A napplet declares a JSON Schema (draft-07 or later) describing its user-facing settings surface; the shell renders a settings UI, validates user input against the schema, persists values in a napplet-scoped store, and delivers live values back to the napplet via snapshot + push.

The shell is the sole writer of configuration values. Napplets cannot mutate configuration over the wire -- they may only read (`config.get`), subscribe to live updates (`config.subscribe`), register a schema (`config.registerSchema`), and request that the shell open its settings UI (`config.openSettings`). All values a napplet sees have been validated by the shell against the currently-registered schema, with declared defaults applied.

## API Surface

```typescript
interface NappletConfig {
  /** Runtime escape hatch -- prefer manifest-declared schema. */
  registerSchema(schema: ConfigSchema, version?: number): void;

  /** One-shot snapshot of current validated + defaulted values. */
  get(): Promise<ConfigValues>;

  /** Subscribe to live updates; first delivery is an immediate snapshot. */
  subscribe(callback: (values: ConfigValues) => void): Subscription;

  /** Listen for schema-registration errors. */
  onSchemaError(callback: (err: ConfigSchemaError) => void): Subscription;

  /** Request the shell open its settings UI, optionally deep-linking by section. */
  openSettings(options?: { section?: string }): void;

  /** Read-access to the currently-registered schema (from manifest or registerSchema). */
  readonly schema: ConfigSchema | null;
}

/**
 * JSON Schema (draft-07+) describing a napplet's configuration surface.
 * Restricted to the Core Subset defined in Schema Contract below.
 */
type ConfigSchema = Record<string, unknown>;

/**
 * Shell-validated configuration values, shaped by the registered schema.
 * Keys correspond to schema properties; values are JSON-serializable.
 */
type ConfigValues = Record<string, unknown>;

interface ConfigSchemaError {
  code: 'invalid-schema' | 'version-conflict' | 'unsupported-draft'
      | 'ref-not-allowed' | 'pattern-not-allowed' | 'secret-with-default';
  message: string;
}

interface Subscription {
  close(): void;
}
```

**`registerSchema(schema, version?)`** -- Runtime escape hatch for declaring a configuration schema. The preferred path is manifest-declared schema at napplet build time; `registerSchema` is for napplets whose schema genuinely cannot be static. The optional `version` parameter is the `$version` migration hint. Shell responds asynchronously via a positive-ACK (`config.registerSchema.result`); errors surface via the `onSchemaError` subscription.

**`get()`** -- Returns a one-shot snapshot of the current configuration values. Returned values are always validated by the shell and defaulted per the registered schema. Resolves with a `ConfigValues` object.

**`subscribe(callback)`** -- Subscribes to live configuration updates. The shell MUST deliver an immediate initial `config.values` push on subscription (snapshot delivery), followed by pushes whenever the shell's settings UI commits a change. Returns a `Subscription` with a `close()` method that detaches the callback. The reference shim implementation SHOULD ref-count local subscribers and only send a wire-level `config.unsubscribe` when the last local subscriber detaches.

**`onSchemaError(callback)`** -- Subscribes to schema-registration errors. The shell pushes `config.schemaError` messages when it rejects a `registerSchema` call or cannot parse a manifest-declared schema. Returns a `Subscription` with `close()`.

**`openSettings(options?)`** -- Requests that the shell open its settings UI for this napplet. The optional `options.section` string deep-links the UI to a named section; the section MUST be declared via the `x-napplet-section` extension somewhere in the registered schema. Fire-and-forget -- the shell decides how to render the UI (modal, panel, new tab) and whether to honor the request.

**`schema`** -- Read-only accessor for the currently-registered schema. Returns the schema last registered (via manifest or `registerSchema`), or `null` if none has been declared. Useful for napplets that render capability-dependent UI based on which settings exist.

## Wire Protocol

`config.*` messages use the NIP-5D wire format (`{ "type": "domain.action", ...payload }`).

| Type | Direction | Payload fields |
|------|-----------|----------------|
| `config.registerSchema` | napplet -> shell | `id`, `schema`, `version?` |
| `config.registerSchema.result` | shell -> napplet | `id`, `ok`, `error?`, `code?` |
| `config.get` | napplet -> shell | `id` |
| `config.subscribe` | napplet -> shell | (none) |
| `config.unsubscribe` | napplet -> shell | (none) |
| `config.openSettings` | napplet -> shell | `section?` |
| `config.values` | shell -> napplet | `id?`, `values` |
| `config.schemaError` | shell -> napplet | `error`, `code` |

Key design notes:
- All correlation uses `id`.
- `config.values` is dual-use: with `id` it answers `config.get`; without `id` it is a subscription push.
- `config.registerSchema` uses positive-ACK -- every call receives a `config.registerSchema.result` with `ok: true`, or `ok: false` accompanied by `code` and `error` describing the rejection reason. It is never error-only.
- `config.subscribe` MUST produce an immediate initial `config.values` push (snapshot delivery) before any value-change pushes.
- `config.unsubscribe` is the clean-teardown path; iframe unmount also implicitly unsubscribes the source.

### Examples

**Register schema (success):**
```
-> { "type": "config.registerSchema", "id": "r1", "schema": { "$schema": "http://json-schema.org/draft-07/schema#", "type": "object", "properties": { "theme": { "type": "string", "enum": ["light", "dark"], "default": "dark" } } }, "version": 1 }
<- { "type": "config.registerSchema.result", "id": "r1", "ok": true }
```

**Register schema (rejection):**
```
-> { "type": "config.registerSchema", "id": "r2", "schema": { "type": "object", "properties": { "username": { "type": "string", "pattern": "^[a-z]+$" } } } }
<- { "type": "config.registerSchema.result", "id": "r2", "ok": false, "code": "pattern-not-allowed", "error": "JSON Schema `pattern` keyword is not permitted in the Core Subset" }
```

**Get values:**
```
-> { "type": "config.get", "id": "g1" }
<- { "type": "config.values", "id": "g1", "values": { "theme": "dark" } }
```

**Subscribe + initial snapshot:**
```
-> { "type": "config.subscribe" }
<- { "type": "config.values", "values": { "theme": "dark" } }
```

**Push update after user changes a value:**
```
<- { "type": "config.values", "values": { "theme": "light" } }
```

**Unsubscribe:**
```
-> { "type": "config.unsubscribe" }
```

**Open settings with section deep-link:**
```
-> { "type": "config.openSettings", "section": "notifications" }
```

**Schema error push (background rejection):**
```
<- { "type": "config.schemaError", "code": "invalid-schema", "error": "schema root must be of type `object`" }
```
