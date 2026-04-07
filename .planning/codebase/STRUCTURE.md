# Codebase Structure

**Analysis Date:** 2026-04-07

## Directory Layout

```
nubs/
├── README.md               # Governance doc, registry table, track overview
├── CLAUDE.md               # Project instructions and context
├── TEMPLATE-WORD.md        # Template for interface specifications
├── TEMPLATE-NN.md          # Template for protocol specifications
├── specs/                  # Published interface and protocol specifications
│   ├── README.md           # Specs directory overview
│   ├── TEMPLATE-WORD.md    # Copy of interface template (reference)
│   ├── TEMPLATE-NN.md      # Copy of protocol template (reference)
│   ├── NUB-RELAY.md        # NIP-01 relay proxy interface
│   ├── NUB-STORAGE.md      # Scoped key-value storage interface
│   ├── NUB-SIGNER.md       # NIP-07 signer proxy interface
│   ├── NUB-NOSTRDB.md      # Local event database interface
│   ├── NUB-IPC.md          # Inter-napplet pub/sub interface
│   └── NUB-PIPES.md        # Point-to-point connection interface
└── .git/                   # Git repository metadata (not readable)
```

## Directory Purposes

**Root Directory:**
- Purpose: Top-level governance and project metadata
- Contains: Markdown specifications, governance policy, contributing instructions
- Key files: `README.md` (registry and governance), `CLAUDE.md` (project context), templates

**`specs/`:**
- Purpose: Canonical location for all published NUB specifications
- Contains: Individual markdown files for each approved spec (NUB-RELAY, NUB-STORAGE, NUB-SIGNER, NUB-NOSTRDB, NUB-IPC, NUB-PIPES)
- Key files: Individual `NUB-*.md` specs, mirrored templates for reference
- Committed to master: Only after maintainer merge

## Key File Locations

**Entry Points:**

- `README.md`: Start here for governance model and spec registry. Contains:
  - Two-track overview (NUB-WORD vs NUB-NN)
  - Registry table (lines 18-25) mapping NUB ID → namespace → description → PR link → status
  - Boundary rule for distinguishing interfaces from protocols
  - Governance rules (fork, PR, review, merge)
  - Template references

- `CLAUDE.md`: Project-specific instructions for contributors and agents
  - Governance summary
  - PR submission workflow
  - Links to source specs and branch names
  - Remote repository URL

**Configuration:**

- No `.eslintrc`, `tsconfig.json`, or build config files (specification repo, not executable code)
- Git hooks: None visible (informal NIP-style process)
- `.git/`: Standard git repository (not part of codebase content)

**Core Logic (Specifications):**

All specifications live in `specs/` directory:

- `specs/NUB-RELAY.md`: NIP-01 relay proxy (provides `window.napplet.relay` namespace)
  - Wire format: REQ/EVENT/CLOSE/EOSE/OK/CLOSED/NOTICE verbs
  - Event kinds: 29001 for scoped relay commands
  - Methods: `subscribe()`, `publish()`, `query()`

- `specs/NUB-STORAGE.md`: Scoped key-value storage (provides `window.napplet.storage` namespace)
  - Wire format: Kind 29003 IPC events with topic tags
  - Methods: `getItem()`, `setItem()`, `removeItem()`, `keys()`
  - Scope: Per-napplet composite key (dTag, aggregateHash)

- `specs/NUB-SIGNER.md`: NIP-07 signer proxy (provides `window.nostr` namespace)
  - Wire format: Kind 29001/29002 for request/response
  - Methods: Standard NIP-07 interface (getPublicKey, signEvent, nip04, nip44)
  - No extensions to NIP-07 spec

- `specs/NUB-NOSTRDB.md`: Local event database (provides `window.nostrdb` namespace)
  - Wire format: Kind 29006 (request) / 29007 (response)
  - Methods: `query()`, `add()`, `event()`, `replaceable()`, `count()`, `subscribe()`
  - No implementations yet in this repo (links to external)

- `specs/NUB-IPC.md`: Inter-napplet pub/sub (provides `window.napplet.ipc` namespace)
  - Wire format: Kind 29003 with topic tags
  - Methods: `emit()`, `on()`
  - Topic conventions: `shell:*`, `napplet:*`, `domain:*`

- `specs/NUB-PIPES.md`: Point-to-point connections (provides `window.napplet.pipes` namespace)
  - Status: Draft, unimplemented
  - Wire format: PIPE_OPEN / PIPE_ACK / PIPE_MESSAGE / PIPE_CLOSED
  - Methods: `open()`, `onOpen()`, `broadcast()`, plus PipeHandle methods

**Testing:**

- No test files in this repository (specification repo, not executable code)
- Testing occurs in external implementation repositories (napplet SDK, shell implementations)

## Naming Conventions

**Files:**

- `README.md`: Standard documentation in any directory
- `TEMPLATE-*.md`: Prefix indicates template purpose (TEMPLATE-WORD for interfaces, TEMPLATE-NN for protocols)
- `NUB-*.md`: Published specifications follow NUB ID exactly (NUB-RELAY, NUB-STORAGE, etc.)
- `CLAUDE.md`: Project-specific instructions (checked into root)
- No other file types (no code, no build artifacts, no config files)

**Directories:**

- All lowercase except `.` prefix for hidden directories
- `specs/`: Plural to indicate collection
- `.git/`, `.planning/`: Standard hidden directories (git repo and planning artifacts)

**Specification Naming:**

- NUB-WORD: Single uppercase word (RELAY, STORAGE, SIGNER, NOSTRDB, IPC, PIPES)
  - One canonical spec per word
  - Namespace pattern: `window.napplet.{word}` or `window.nostr` or `window.nostrdb`
  
- NUB-NN: Sequential number (01, 02, etc.)
  - Multiple competing specs allowed per domain
  - Reserved for future protocol proposals

## Where to Add New Code

**New Interface Specification (NUB-WORD):**

1. **Check existing specs** in `specs/` to confirm name not used
2. **Create branch**: `git checkout -b nub-{name}` (e.g., `nub-audio`)
3. **Copy template**: `cp specs/TEMPLATE-WORD.md specs/NUB-{NAME}.md`
4. **Fill template** with sections:
   - Description (one paragraph)
   - API Surface (TypeScript interface + method docs)
   - Shell Behavior (MUST/MAY/SHOULD)
   - Event Kinds (if using postMessage wire format)
   - Security Considerations
   - Implementations (links to external repos)
5. **Update registry** in `README.md` (add row with NUB ID, namespace, description, PR link, status)
6. **Open PR** with title: `NUB-{NAME}: {Description}`
   - Body should include: namespace, status (draft), NIP-5D reference
7. **Wait for review** and maintainer merge

**New Protocol Specification (NUB-NN):**

1. **Determine sequence number** (check highest number in CLAUDE.md or ask maintainer)
2. **Create branch**: `git checkout -b nub-{number}` (e.g., `nub-01`)
3. **Copy template**: `cp specs/TEMPLATE-NN.md specs/NUB-{NN}.md`
4. **Fill template** with sections:
   - Description (one paragraph)
   - Event Semantics (event kinds, tag structure, content format)
   - Negotiation (how napplets discover each other)
   - Implementations (links to example implementations)
5. **Registry update** in `README.md` (add row for new protocol when merged)
6. **Open PR** with title: `NUB-{NN}: {Description}`
   - Body should include: domain, dependent interfaces, status (draft)
7. **Wait for merge**; maintainer assigns final NUB-NN number on merge

**Updating Existing Spec:**

- Make changes directly to `specs/NUB-*.md`
- Document changes in PR with rationale
- Update registry status if moving from draft → stable
- Maintainer reviews and merges

**Contributing to Project Governance:**

- Edits to `README.md` governance section or `CLAUDE.md` go through PR review
- Changes to templates (`TEMPLATE-WORD.md`, `TEMPLATE-NN.md`) require explicit discussion

## Special Directories

**`.planning/codebase/`:**
- Purpose: GSD analysis output (codebase mapping)
- Generated: Yes (by GSD mapper agent)
- Committed: Yes (documents are checked in for reference)
- Contains: ARCHITECTURE.md, STRUCTURE.md (when mapped for `arch` focus)

**`.git/`:**
- Purpose: Git version control repository
- Generated: Yes (standard git)
- Committed: No (git metadata, not content)
- Contains: Commit history, branches, remotes

## Registry Details

The registry table in `README.md` (lines 18-25) is the single source of truth for approved specs:

| Column | Purpose | Example |
|--------|---------|---------|
| NUB ID | Spec identifier | `NUB-RELAY` |
| Namespace | JavaScript global path | `window.napplet.relay` |
| Description | One-line purpose | `NIP-01 relay proxy` |
| Status | Current maturity | `Draft`, `Stable`, etc. |
| PR Link | Link to approval PR | `https://github.com/napplet/nubs/pull/2` |

When submitting a new spec:
1. Author creates PR with spec file
2. Registry row added to PR description (or first comment)
3. Upon maintainer merge, row is committed to `README.md`

## Documentation Structure

**Per-Specification Documentation:**

Every spec in `specs/NUB-*.md` contains:

1. **Header** (Title, Status badge, NUB ID, Namespace, Discovery method)
2. **Description** (1 paragraph: what it provides and why)
3. **API Surface** (TypeScript interface + method signatures + behavior docs)
4. **Shell Behavior** (Requirements for shell implementation using MUST/MAY/SHOULD)
5. **Event Kinds** (Wire format definition, kind numbers, tag structures)
6. **Security Considerations** (Interface-specific security requirements)
7. **Implementations** (Links to napplet SDK and shell implementations)

---

*Structure analysis: 2026-04-07*
