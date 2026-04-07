# Coding Conventions

**Analysis Date:** 2026-04-07

## Overview

This is a specification repository for NUBs (Napplet Unified Blueprints) — interface and message protocol proposals for the napplet ecosystem. All content is markdown documentation. There is no executable code. Conventions focus on:

- Markdown formatting and structure
- Specification document layout
- Naming patterns for NUBs and identifiers
- Cross-references and linking

## Markdown Formatting

**Style:**
- Soft line wrapping (lines break at editor margins, not hardcoded)
- Git history shows pattern: `fmt: unwrap hard line breaks to markdown auto-wrap style`
- Use semantic markdown (not HTML)

**Headers:**
- H1 (`#`) for spec title: `NUB-RELAY`, `NUB-STORAGE`, etc.
- H2 (`##`) for section names: `Description`, `API Surface`, `Shell Behavior`, etc.
- H3 (`###`) for subsections: `Event: {name}`, `Request Tags`, `Built-in Topics`, etc.

**Code Blocks:**
- TypeScript/JavaScript interfaces: Use triple backticks with `typescript` language hint
- JSON examples: Use triple backticks with `json` language hint
- Pseudocode/wire format: Use triple backticks with no language hint (plain text)
- Type signatures in method documentation can use inline code

**Tables:**
- Use markdown table syntax (pipe separators)
- Consistent column count per table
- Examples: `NUB ID | Namespace | Description | Status` (NIP registry), `Kind | Name | Direction | Description` (event kinds)

## Naming Patterns

**NUB Identifiers:**

**NUB-WORD (Interface Specs):**
- Format: `NUB-{UPPERCASE_WORD}`
- Examples: `NUB-RELAY`, `NUB-STORAGE`, `NUB-SIGNER`, `NUB-NOSTRDB`, `NUB-IPC`, `NUB-PIPES`
- One canonical spec per name — no competing interface specs
- Names are first-come-first-served but must be approved by maintainer

**NUB-NN (Message Protocol Specs):**
- Format: `NUB-{NUMBER}` (sequential: `NUB-01`, `NUB-02`, etc.)
- Numbers assigned sequentially on merge
- Multiple competing specs allowed per domain

**Namespace Naming:**

Interface specs define JavaScript namespaces on `window`:

- `window.napplet.{lowercase}` — standard napplet interfaces (e.g., `window.napplet.relay`, `window.napplet.storage`)
- `window.nostr` — NIP-07 signer proxy (special case, not `window.napplet.nostr`)
- `window.nostrdb` — Nostr database (special case, not `window.napplet.nostrdb`)

## Specification Document Structure

**Required Sections (in order):**

1. **Title and Status**
   ```
   NUB-{ID}
   ========
   
   {Human-readable title}
   ----------------------
   
   `draft` (or `draft` + `unimplemented`)
   ```

2. **Metadata Header**
   ```
   **NUB ID:** NUB-{ID}
   **Namespace:** `window.napplet.{name}` (or equivalent)
   **Discovery:** `shell.supports("NUB-{ID}")`
   ```
   
   For protocol specs (NUB-NN), replace with:
   ```
   **NUB ID:** NUB-{NN}
   **Domain:** {domain description}
   **Requires:** {NUB-WORD interfaces needed}
   **Discovery:** `shell.supports("NUB-{WORD}", "NUB-{NN}")`
   ```

3. **Description** — One paragraph explaining what the spec provides and why it exists

4. **API Surface** — Method signatures with types (interface blocks in TypeScript)

5. **Shell Behavior** — Mandatory (MUST) and optional (MAY/SHOULD) shell behaviors

6. **Event Kinds** — If spec uses postMessage bus kinds:
   - Table format: `Kind | Name | Direction | Description`
   - Include tag structure and semantics if applicable

7. **Security Considerations** — Interface-specific security requirements

8. **Implementations** — Links to reference implementations with file paths

**Optional Sections:**

- For protocols: `Event Semantics`, `Negotiation`, `Event Structure`
- For complex specs: `Comparison with [Other Spec]`, `Wire Format`, `Lifecycle` (diagrams/state machines)
- For future work: `Future Extensions`

## Comment and Documentation Patterns

**Status Badges:**
- `draft` — Initial, community feedback stage
- `draft` + `unimplemented` — Draft that has no implementations yet (e.g., `NUB-PIPES`)

**Direction Notation:**
- `napplet -> shell` — napplet sends, shell receives
- `shell -> napplet` — shell sends, napplet receives
- `bidirectional` — both directions (usually routed through shell)

**Requirement Language:**
- **MUST** — Mandatory implementation requirement (RFC 2119)
- **MAY** — Optional extension point
- **SHOULD** — Recommended but not required
- When marking prose sections: **bold** for MUST, MAY, SHOULD keywords

**Examples in Prose:**
- Prose descriptions precede or follow code blocks
- Explain: what the method does, what it returns, when to use it
- Include error conditions and edge cases
- Reference related specs (e.g., "following NIP-01 semantics", "per NIP-07")

## Cross-References and Links

**Internal References:**
- Link to specs via github PR URLs: `[NUB-RELAY](https://github.com/napplet/nubs/pull/2)`
- Link to templates in governance docs: `[TEMPLATE-WORD.md](TEMPLATE-WORD.md)`
- Reference core spec: `[NIP-5D](../NIP-5D.md)` or GitHub link

**External References:**
- NIP links: `[NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md)`
- GitHub code: `[@napplet/shim](https://github.com/sandwichfarm/napplet) (napplet-side: `packages/shim/src/relay-shim.ts`)`
- Use full file paths in implementation references

**Code Examples:**
- Use backticks for identifiers: `getPublicKey()`, `window.nostr`, `kind 29003`
- Use backticks for event kinds: `kind 29001`, `kind 29003`, `kind 29006`
- Use backticks for tags: `#t` tag, `['t', topic]`, `id` tag

## Commit Message Conventions

**Format:**
- Prefix with type: `docs:`, `fix:`, `fmt:`
- Imperative mood: "add spec", "fix typo", "unwrap lines" (not "added", "fixed", "unwrapped")
- Summary line only (no body) for simple changes

**Examples:**
- `docs: receive nub specs from napplet repo`
- `fix: remove implementation references from spec`
- `fmt: unwrap hard line breaks to markdown auto-wrap style`
- `fix: restore event kinds table, frame as NIP-07 proxy transport`

**Prefixes in Use:**
- `docs:` — Documentation content, new specs, spec updates
- `fix:` — Corrections, removals of out-of-scope details
- `fmt:` — Formatting/style changes (line breaks, whitespace)
- (No `feat:` prefix observed; specs are treated as documentation updates)

**Co-authored Commits:**
- When generated/assisted, include trailer:
  ```
  Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
  ```

## Interface vs. Protocol Distinction

**NUB-WORD (Interface):**
- Shell-provided AND defines an API surface (methods/properties on `window` namespaces)
- Template: `TEMPLATE-WORD.md`
- Sections: API Surface (methods), Shell Behavior, Event Kinds (if wire format), Security
- Example: `NUB-RELAY` provides `window.napplet.relay.subscribe()`, `publish()`, `query()`

**NUB-NN (Protocol):**
- Napplet-agreed AND defines event semantics
- Template: `TEMPLATE-NN.md`
- Sections: Event Semantics (kinds, tags, content), Negotiation, Implementations
- Example: A feed rendering protocol (hypothetical NUB-15) defines event kinds and tag structures for feed events

**Boundary Rule** (from README):
> An interface (NUB-WORD) is **shell-provided** AND defines an **API surface**. A protocol (NUB-NN) is **napplet-agreed** AND defines **event semantics**. Both criteria must apply. Edge cases are judged pragmatically by the maintainer.

## Type Notation

**TypeScript Interfaces:**
- Use standard TS syntax in code blocks
- Include `interface`, method signatures, parameter names and types
- Use `Promise<T>` for async operations
- Use `Record<K, V>` for object maps
- Use `... | ...` for union types
- Optional parameters: `param?: Type`
- Union: `NostrFilter | NostrFilter[]` (or array of one)

**Event Structure (JSON):**
- Show full event JSON with representative values
- Include all required fields: `kind`, `tags`, `content`
- Use `{...}` for complex nested structures
- Tag structure as array: `["tag-name", "value1", ...]`

**Wire Format (Pseudocode):**
- Use JSON array format: `["VERB", id, params]`
- Show pipes/verbs: `PIPE_OPEN`, `PIPE_ACK`, `PIPE_CLOSE`, `PIPE_CLOSED`
- Use angle brackets for placeholders: `<pipe_id>`, `<dTag>`

## Enum/Constant Patterns

**Event Kinds:**
- Named by domain: `IPC_PEER` (kind 29003), `NIPDB_REQUEST` (kind 29006)
- Use UPPER_SNAKE_CASE for kind names in prose
- Use numeric kind value in tables: `Kind: 29003`

**Topics:**
- Prefix convention: `shell:*` (napplet → shell commands), `napplet:*` (shell → napplet responses), `{domain}:*` (peer-to-peer)
- Examples: `shell:state-get`, `napplet:state-response`, `auth:identity-changed`, `stream:channel-switch`
- Use backticks in prose: `` `shell:state-get` ``

**Capabilities (ACL):**
- Format: `{domain}:{action}` (colon-separated)
- Examples: `relay:read`, `relay:write`, `storage:read`, `storage:write`, `sign:event`, `sign:nip04`, `sign:nip44`
- Use backticks in prose: `` `relay:read` ``

---

*Convention analysis: 2026-04-07*
