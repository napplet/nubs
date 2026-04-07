# Testing Patterns

**Analysis Date:** 2026-04-07

## Overview

The NUBs repository is a specification/documentation repository, not a code repository. There are no automated unit tests, test frameworks, or CI pipelines. Instead, the "testing" process is a **community review and validation workflow** for specifications.

## Specification Review Process

**Governance Model:**
- NIP-style informal process (based on Nostr Improvement Proposals)
- Community-driven, maintainer-gated

**Stages:**

1. **Proposal (Draft)**
   - Author forks repo, adds spec file (NUB-WORD.md or NUB-NN.md)
   - Author opens Pull Request with:
     - PR title: `NUB-{ID}: {Human Title}` (e.g., `NUB-RELAY: NIP-01 relay proxy interface`)
     - PR body includes: one-line summary, namespace, status, link to NIP-5D
   - Spec starts as `draft` status

2. **Community Review**
   - Reviewers discuss spec via PR comments
   - Focus areas: clarity, correctness, scope, security implications
   - Changes made via commits to the PR branch (no amend pattern observed)

3. **Implementation Requirement**
   - Spec must have at least one implementation before merge
   - Implementations linked in `## Implementations` section
   - Examples: `[@napplet/shim](https://github.com/sandwichfarm/napplet)`, `[@kehto/shell](https://github.com/sandwichfarm/kehto)`
   - Implementation files referenced with full paths: `packages/shim/src/relay-shim.ts`

4. **Maintainer Merge**
   - Maintainer (dskvr) evaluates spec completeness and community consensus
   - If spec makes sense and has implementation, maintainer merges PR
   - No formal voting or committee required
   - NUB-NN specs assigned sequential numbers on merge

5. **Stability**
   - Once merged, spec is canonical
   - Further revisions made via new commits/PRs to main branch
   - Status may evolve: `draft` → `stable` (not observed yet in this repo)

## Specification Validation Criteria

**What is Reviewed:**

1. **Clarity**
   - Description explains what problem the spec solves
   - API surface clearly defined (method names, parameter types, return types)
   - All sections present and complete

2. **Correctness**
   - Wire format is valid (NIP-01 events or minimal JSON arrays)
   - Event kinds are correct (valid Nostr kind numbers)
   - Tag structures follow Nostr conventions
   - Security considerations address threat model

3. **Scope**
   - Spec adheres to boundary rule: interface is shell-provided + API surface, protocol is napplet-agreed + event semantics
   - Implementation details excluded (focus on contracts, not internals)
   - References to other specs use links, not embedded copies

4. **Security**
   - ACL model is explicit (capabilities listed: `relay:read`, `storage:write`, etc.)
   - Auth flow explained (e.g., relies on NIP-5D AUTH handshake)
   - Threat vectors identified (e.g., signature verification, quota enforcement, sender exclusion)

5. **Implementation**
   - At least one implementation must exist
   - Implementation files referenced with full paths
   - SDK/shim and shell-side implementations typically required for interfaces

## PR Review History Example

**Example: NUB-RELAY spec**

Commit: `f3ec181c33d91c1338d33a494cc3b3507e43867e`

```
NUB-RELAY: NIP-01 relay proxy interface

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
```

Through subsequent commits, refinements made:

- `aae7138` — `fix: restore event kinds table, frame as NIP-07 proxy transport`
- Multiple `fix:` commits iteratively removed out-of-scope implementation details
- Multiple `fmt:` commits normalized markdown line wrapping
- Rename commit: `0a00269` — renamed NUB-IPC to NUB-IFC (Inter-Frame Communication)

This pattern shows:
- Spec approved and merged with implementation reference
- Community feedback (implicit in subsequent fixes) addressed via commits
- Format normalization applied across all specs
- Naming corrections made systematically

## Template-Driven Specification

**TEMPLATE-WORD.md** (for `TEMPLATE-WORD.md`):

```
NUB-{NAME}
==========

{Title}
-------

`draft`

**NUB ID:** NUB-{NAME}
**Namespace:** `window.napplet.{name}` (or `window.nostr`, `window.nostrdb`)
**Discovery:** `shell.supports("NUB-{NAME}")`

## Description
## API Surface
## Shell Behavior
## Event Kinds
## Security Considerations
## Implementations
```

**TEMPLATE-NN.md** (for `TEMPLATE-NN.md`):

```
NUB-{NN}
========

{Title}
-------

`draft`

**NUB ID:** NUB-{NN}
**Domain:** {e.g., feed rendering, chat, collaborative editing}
**Requires:** {NUB-WORD interfaces needed, e.g., NUB-RELAY, NUB-IPC}
**Discovery:** `shell.supports("NUB-{WORD}", "NUB-{NN}")`

## Description
## Event Semantics
## Negotiation
## Implementations
```

**Purpose:**
- Templates ensure consistency across specs
- Checklist for spec authors before submitting PR
- Guide reviewers on what sections to evaluate

## Registry as Canonical Reference

**File:** `README.md`

**Table: NUB-WORD (Interface Specs)**

```
| NUB ID | Namespace | Description | Status |
|--------|-----------|-------------|--------|
| [NUB-RELAY](https://github.com/napplet/nubs/pull/2) | `window.napplet.relay` | NIP-01 relay proxy | Draft |
| [NUB-STORAGE](https://github.com/napplet/nubs/pull/3) | `window.napplet.storage` | Scoped key-value storage | Draft |
```

**Key Patterns:**
- NUB ID links to GitHub PR (not file path)
- Namespace shown in backticks
- Status badge: `Draft` (only status observed)
- Registry is the authoritative catalog

## Validation Against Referenced Implementations

**Implementation Reference Pattern:**

Example from `NUB-RELAY.md`:

```markdown
## Implementations

- [@napplet/shim](https://github.com/sandwichfarm/napplet) (napplet-side: `packages/shim/src/relay-shim.ts`)
- [@kehto/shell](https://github.com/sandwichfarm/kehto) (shell-side: `packages/runtime/src/shell-bridge.ts`)
```

**Validation Approach:**
- Maintainer checks that referenced files exist in linked repos
- Reviewers can inspect implementations to verify spec correctness
- Implementations serve as executable spec (running code validates wire format)

## Testing at Implementation Level

This repo contains **no test code**, but **referenced implementations** have tests:

- `@napplet/shim` — Client-side shim library with unit and integration tests
- `@kehto/shell` — Shell implementation with test coverage
- Tests validate that wire format (kind 29001, 29003, 29006, etc.) is correctly implemented

**Test Scope (from linked implementations):**
- Event kind serialization/deserialization
- Method call→event translation and back
- Subscription lifecycle (open, deliver events, close)
- Error handling (timeout, invalid format, ACL denial)
- Mock shell behavior for napplet testing

## Common Validation Patterns

**Before Merge, Reviewers Check:**

1. **Metadata Completeness**
   - `**NUB ID:**` field matches filename
   - `**Namespace:**` or `**Domain:**` defined
   - `**Discovery:**` clause present
   - `draft` status badge included

2. **Section Completeness**
   - All required sections present (Description, API Surface, Shell Behavior, Event Kinds, Security, Implementations)
   - No placeholder text (`{...}`)
   - Prose is clear and non-redundant

3. **Wire Format Correctness**
   - If spec uses events: kind numbers are valid Nostr kinds (not reserved, typically 29000+ range for experimental)
   - Event tags follow NIP-01 conventions: `[["tag-name", "value"], ...]`
   - Tag structures documented in tables
   - Direction notation (→) consistent with shell mediation model

4. **Security Explicit**
   - ACL capabilities named: `relay:read`, `storage:write`, etc.
   - Threat model addressed: signature verification, quota enforcement, sender validation
   - Trust boundaries clear: where napplet runs vs. shell runs

5. **Implementation Links Valid**
   - Links point to real GitHub repos (sandwichfarm, napplet orgs)
   - File paths include full package path: `packages/*/src/...`
   - No broken links

6. **Scope Boundaries Respected**
   - No implementation details (e.g., how shell serializes data internally)
   - API contract focus, not internals
   - References to other specs use links, not copies

## Continuous Improvement Pattern

**Observed Evolution:**
- Initial NUB specs merged with implementation references
- Subsequent commits apply systematic changes:
  - `fmt:` commits unwrap hard line breaks (style consistency across all specs)
  - `fix:` commits remove out-of-scope details (implementation internals filtered out)
  - Rename commits standardize naming (IPC→IFC, IPC-PEER→IFC_PEER)

**This suggests:**
- Specs are treated as living documents
- Corrections and refinements made via incremental commits
- Community feedback drives improvements
- No formal versioning; latest commit on master is canonical

## PR and Issue Workflow

**GitHub PR Pattern:**

From recent git history:

- `178eb8b` — `link registry table to PRs instead of files`
  - README updated to link to PR URLs (`/pull/2`, `/pull/3`) instead of file paths
  - Reflects GitHub's PR-centric review workflow

**Implication:**
- Community reviews happen in GitHub PRs
- Comments and suggestions in PR discussion threads
- Once merged, spec file moves to master branch
- Registry table maintains PR link for traceability

## No Local Testing Tools

**Absent:**
- `package.json` — No npm scripts for linting markdown
- `.eslintrc`, `.prettierrc` — No code formatters (style enforced via manual review + `fmt:` commits)
- `jest.config.js`, `vitest.config.js` — No test runners (this is documentation)
- CI/CD pipelines (`.github/workflows/`, `gitlab-ci.yml`, etc.) — No automated checks

**Validation is Manual:**
- Maintainer + community review on GitHub PRs
- Referenced implementations serve as executable spec validation

---

*Testing analysis: 2026-04-07*
