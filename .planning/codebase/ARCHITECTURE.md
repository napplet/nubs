# Architecture

**Analysis Date:** 2026-04-07

## Pattern Overview

**Overall:** Specification Registry

This is a specification repository that implements a two-track proposal system extending NIP-5D (Nostr Web Applets). The codebase is not executable code but rather a curated set of interface and protocol specifications managed through version control and GitHub pull requests.

**Key Characteristics:**
- Two-track governance model: NUB-WORD (interface specs) and NUB-NN (message protocol specs)
- Markdown-based specification documents with machine-readable registry
- Template-driven structure to ensure consistency across proposals
- NIP-style informal process with community review via PR comments
- Hierarchical relationship to NIP-5D core protocol

## Layers

**Governance & Standards:**
- Purpose: Define the rules and processes for adding new specs to the registry
- Location: `README.md`, `CLAUDE.md`
- Contains: Governance rules, boundary rules, track definitions, PR review guidelines
- Depends on: Nothing (top-level policy)
- Used by: All specification writers and maintainers

**Template Layer:**
- Purpose: Ensure consistent structure across all proposals
- Location: `TEMPLATE-WORD.md`, `TEMPLATE-NN.md`, `specs/TEMPLATE-WORD.md`, `specs/TEMPLATE-NN.md`
- Contains: Markdown templates with required sections (Description, API Surface, Shell Behavior, Security Considerations, Implementations)
- Depends on: Governance rules for section definitions
- Used by: Specification writers creating new NUB proposals

**Interface Specifications (NUB-WORD):**
- Purpose: Define shell-provided API contracts and namespaces
- Location: `specs/NUB-*.md` (NUB-RELAY, NUB-STORAGE, NUB-SIGNER, NUB-NOSTRDB, NUB-IPC, NUB-PIPES)
- Contains: TypeScript interface signatures, method documentation, wire format definitions, shell requirements
- Depends on: NIP-5D core protocol (postMessage transport, authentication flow)
- Used by: Napplet developers consuming shell-provided APIs

**Protocol Specifications (NUB-NN):**
- Purpose: Define napplet-agreed event semantics and negotiation patterns
- Location: `specs/NUB-NN.md` (reserved for numbered protocols)
- Contains: Event kind definitions, tag structures, content formats, producer/consumer requirements
- Depends on: NIP-01 event structure, one or more interface specs for transport
- Used by: Napplets coordinating with each other via agreed protocols

**Registry:**
- Purpose: Central index of all approved specs with status and links
- Location: `README.md` (lines 18-25)
- Contains: Table mapping NUB IDs to namespaces, descriptions, and PR links
- Depends on: All interface specifications
- Used by: Developers discovering available interfaces

## Data Flow

**Specification Proposal Flow:**

1. **Author forks repository** and creates a new branch (e.g., `nub-relay`)
2. **Author writes spec file** using appropriate template (`TEMPLATE-WORD.md` or `TEMPLATE-NN.md`)
3. **Author updates registry** in `README.md` with new spec row (NUB ID, namespace, description, status link to PR)
4. **Author opens PR** with title format: `{NUB-ID}: {Description}` and PR body including namespace and status
5. **Community reviews** via PR comments (no formal review committee)
6. **Maintainer merges** when spec is accepted and has at least one implementation
7. **Merged spec** becomes canonical reference for that interface or protocol

**Runtime Discovery Flow (Napplet Perspective):**

1. Napplet calls `shell.supports("NUB-RELAY")` to discover available interfaces
2. If available, napplet accesses `window.napplet.relay` (or appropriate namespace)
3. Napplet calls interface methods which invoke shell-provided implementations
4. Shell routes requests to actual implementations (relays, storage backends, signers)
5. Shell returns results back through postMessage

**State Management:**

- **Static Registry State**: Managed in `README.md` registry table — updates via PR merge only
- **Specification State**: Each spec file (`specs/NUB-*.md`) contains version history in git; no runtime state
- **Status Labels**: `draft`, `unimplemented`, `stable` — maintained as badges in spec frontmatter

## Key Abstractions

**Interface Specification (NUB-WORD):**
- Purpose: Represent a shell-provided API contract
- Examples: `specs/NUB-RELAY.md`, `specs/NUB-STORAGE.md`, `specs/NUB-SIGNER.md`
- Pattern: TypeScript interface signature + NIP-01 event kinds table (for wire format) + security requirements
- Canonical identity: Single name (no competing specs)
- Discovery: `shell.supports("NUB-NAME")`

**Protocol Specification (NUB-NN):**
- Purpose: Represent event semantics agreed upon between napplets
- Examples: Reserved but not yet allocated in this repo
- Pattern: Event kind definitions + tag structures + negotiation mechanism
- Canonical identity: Sequential number (multiple protocols per domain allowed)
- Discovery: `shell.supports("NUB-WORD", "NUB-NN")`

**Topic Convention (in NUB-IPC):**
- Purpose: Prefix-based routing hint for inter-napplet messages
- Examples: `shell:state-get` (napplet→shell), `napplet:state-response` (shell→napplet), `auth:identity-changed` (shell→napplet)
- Pattern: `{prefix}:{operation}` where prefix signals direction/scope
- Used by: NUB-IPC implementation to hint at message intent (not enforced)

**Event Kind Alias:**
- Purpose: Named identifiers for custom Nostr event kinds used by interfaces
- Examples: Kind 29001 (signer requests), Kind 29003 (IPC peer messages), Kind 29006-29007 (DB requests/responses)
- Pattern: Reserved range [29000-29999] for napplet protocol extensions
- Used by: All wire-format specifications for postMessage encoding

## Entry Points

**Specification Author:**
- Location: User forks repo and creates PR from local branch
- Triggers: Manual — author writes spec and opens PR
- Responsibilities: Follow template structure, provide detailed API/protocol definition, cite implementations, run through maintainer approval

**Maintainer Review:**
- Location: `README.md` (governance section), PR merge operation
- Triggers: When a PR is opened with a spec
- Responsibilities: Verify boundary rule applies (interface = shell + API surface, protocol = napplet agreement + event semantics), check for implementations, merge when ready

**Registry Lookup:**
- Location: `README.md` registry table
- Triggers: Developer visiting repo or searching for available NUBs
- Responsibilities: None — registry is read-only reference

**Napplet Runtime Discovery:**
- Location: Shell implementation's `supports()` method (defined in NIP-5D, not in this repo)
- Triggers: Napplet code calls `shell.supports("NUB-NAME")`
- Responsibilities: Shell returns `true/false` based on loaded interfaces

## Error Handling

**Strategy:** Prevention through governance rules

**Patterns:**

- **Boundary Confusion**: Prevented by explicit boundary rule (interface = shell-provided API, protocol = napplet-agreed semantics). Maintainer judges edge cases pragmatically.
- **Duplicate Names**: Prevented by first-come-first-served rule for NUB-WORD names with maintainer approval. NUB-NN names assigned sequentially on merge.
- **Implementation Gap**: Spec must have at least one implementation before merge. Links to implementations required in Implementations section of each spec.
- **Specification Incompleteness**: Enforced by template structure — missing sections indicate incomplete proposals.

## Cross-Cutting Concerns

**Naming Consistency:** 
- Interface specs use single uppercase word (NUB-RELAY, NUB-STORAGE, NUB-SIGNER, NUB-NOSTRDB, NUB-IPC, NUB-PIPES)
- Protocol specs use sequential numbers (NUB-01, NUB-02, etc.)
- Topic conventions established in NUB-IPC (shell:*, napplet:*, domain:*)

**Security Requirements:**
- All interface specs include dedicated "Security Considerations" section
- All wire-format specs reference Schnorr signature verification
- Scope isolation enforced at shell level (e.g., NUB-STORAGE composite key (dTag, aggregateHash))

**Protocol Dependencies:**
- All interfaces depend on NIP-5D core (postMessage, AUTH flow, session keys)
- NUB-IPC depends on NUB-RELAY for subscription routing
- NUB-PIPES designed as high-frequency alternative to NUB-IPC
- NUB-NOSTRDB provides local cache backing for NUB-RELAY queries

**Documentation Layers:**
- `CLAUDE.md`: High-level project goals and contributing instructions
- `README.md`: Governance, registry, two-track overview
- `TEMPLATE-WORD.md`, `TEMPLATE-NN.md`: Required section checklist
- Individual spec files: Authoritative definitions

---

*Architecture analysis: 2026-04-07*
