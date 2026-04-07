# Codebase Concerns

**Analysis Date:** 2026-04-07

## Naming and Consistency Issues

**NUB-IPC vs NUB-IFC inconsistency:**
- Issue: The spec file is named `NUB-IPC.md` with `NUB ID: NUB-IPC` and namespace `window.napplet.ipc`, but the `README.md` registry table lists this interface as `NUB-IFC` with namespace `window.napplet.ifc` and description "Inter-frame communication"
- Files: `README.md` (line 24), `specs/NUB-IPC.md` (lines 1, 9, 10), `specs/README.md` (line 24)
- Impact: Developers, shell implementers, and napplet developers may use conflicting names when referencing this interface in code and PRs. This breaks discovery via `shell.supports()` if callers disagree on the name. The git history shows recent renames (commits `bebcace`, `0a00269`) suggesting unresolved confusion.
- Fix approach: Choose one canonical name (`NUB-IPC` or `NUB-IFC`) and `ipc` or `ifc` namespace. Audit all specs and templates for references. Update both `README.md` files and the spec file consistently. Document the decision in CLAUDE.md rationale.

## Missing NUB-NN (Message Protocol) Specifications

**No message protocol specs have been written:**
- Issue: The spec repository defines the two-track system (NUB-WORD for interfaces, NUB-NN for message protocols) in governance, but no NUB-01, NUB-02, etc. specs exist yet. The `TEMPLATE-NN.md` exists as guidance, but no domains have been formalized.
- Files: `README.md` (lines 27-32), `specs/README.md` (lines 27-32), `TEMPLATE-NN.md`, `specs/TEMPLATE-NN.md`
- Impact: This blocks the second major use case of the NUB system — inter-napplet event semantics and negotiation. Shell implementers and napplet teams cannot propose or implement feed rendering, chat, collaborative editing, or other message protocol domains. The CLAUDE.md explicitly mentions "The 6 initial interface specs" but makes no mention of planned message protocols.
- Fix approach: Define 1-2 initial message protocol specs (e.g., a basic chat or feed rendering protocol) to flesh out NUB-NN patterns. Document how `shell.supports("NUB-RELAY", "NUB-01")` actually works in practice. Clarify numbering assignment in the governance section — does the maintainer pre-assign numbers, or do contributors propose them?

## Incomplete Implementation Status

**NUB-PIPES marked as unimplemented but included as core interface:**
- Issue: `NUB-PIPES.md` is marked `draft` `unimplemented` (line 7) with implementation section stating "No implementations exist" (line 368). However, the README registry table lists it alongside implemented interfaces with the same draft status, and the architectural description treats pipes as a core capability for "data streams, real-time sync, and command channels."
- Files: `specs/NUB-PIPES.md` (lines 7, 368), `README.md` (line 25), `CLAUDE.md` (line 34)
- Impact: Shell and napplet developers may plan integration expecting pipes to be available, only to discover late that there is no reference implementation. The 369-line spec is highly detailed with wire formats, state machines, and edge cases, creating expectation of maturity despite being unimplemented.
- Fix approach: Move NUB-PIPES to a separate "Future" or "Proposed" section in the registry until at least one implementation exists. Alternatively, add a reference implementation requirement to the governance ("Maintainer merges when the spec makes sense and has at least one implementation") and enforce it consistently. Update status markers to be machine-readable (`status: draft-unimplemented` vs `status: draft-implemented`).

## Incomplete Reference Documentation

**NIP-5D reference is local but not committed:**
- Issue: Both `README.md` files reference `[NIP-5D](../NIP-5D.md)` but this file does not exist in the repository. The CLAUDE.md mentions it lives in `nostr-protocol/nips` on GitHub, but the relative path suggests it should be local.
- Files: `README.md` (line 4, 62), `specs/README.md` (line 4, 61), `CLAUDE.md` (line 4)
- Impact: Readers following the link get a 404. The specs define security models, authentication flows (REGISTER/IDENTITY/AUTH), and message formats that depend on NIP-5D — without access, reviewers and implementers cannot validate the specs against the core protocol.
- Fix approach: Either commit `NIP-5D.md` to the repo (with appropriate license notice), or change all relative links to `https://github.com/nostr-protocol/nips/blob/master/5D.md`. Update CLAUDE.md to clarify the relationship.

## Ambiguous Spec Maturity and Governance

**"Draft" status undefined and changeover criteria unclear:**
- Issue: All six interface specs are marked `draft`, but there is no definition of what draft means or when a spec graduates to a later stage (e.g., `proposed`, `stable`, `deprecated`). The governance section says "Maintainer merges when the spec makes sense and has at least one implementation," but does not define how the status field changes post-merge.
- Files: `specs/NUB-RELAY.md` (line 7), `specs/NUB-STORAGE.md` (line 7), `specs/NUB-SIGNER.md` (line 7), `specs/NUB-NOSTRDB.md` (line 7), `specs/NUB-IPC.md` (line 7), `specs/NUB-PIPES.md` (line 7), `README.md` (lines 20-25)
- Impact: Developers cannot determine which specs are stable enough to rely on or which are expected to change. The PR links in the registry suggest specs are in active review, but the specs themselves were committed as finalized files. It is unclear if specs are versioned or if they can be modified post-merge.
- Fix approach: Define maturity levels in governance: `draft` (under review), `proposed` (merged, seeking implementation), `stable` (has 2+ independent implementations, frozen interface), `extended` (additive changes allowed), `deprecated`. Update specs to include version history or "Last Updated" dates. Clarify whether PRs show the review process before merge or if they are just documentation links.

## Cross-Spec Interface Dependencies Not Formally Tracked

**Specs reference other specs informally but no dependency matrix exists:**
- Issue: `NUB-RELAY.md` references NUB-SIGNER (line 56: "Signs the event template via `window.nostr.signEvent()`"). `NUB-IPC.md` references `NUB-RELAY` implicitly (line 36: "opens a NIP-01 subscription"). No formal dependency graph is documented. The template `TEMPLATE-NN.md` includes a `Requires:` field (line 11) for message protocols, but interface specs lack equivalent dependency tracking.
- Files: `specs/NUB-RELAY.md` (line 56), `specs/NUB-IPC.md` (line 36), `specs/NUB-PIPES.md` (line 96), `TEMPLATE-NN.md` (line 11)
- Impact: Implementers may build an incomplete shell without realizing they need to implement multiple specs together. No way to query "if I implement NUB-IPC, what else do I need?" Spec reviews cannot detect if a new NUB-NN breaks existing assumptions in interface specs.
- Fix approach: Add a `Depends on:` field to `TEMPLATE-WORD.md` (e.g., "NUB-RELAY depends on NUB-SIGNER for event signing"). Create a dependency matrix in `README.md` showing which specs require which. When accepting a new spec, audit dependencies against the entire spec set.

## Security Considerations Not Uniformly Comprehensive

**Security sections vary widely in depth and coverage:**
- Issue: `NUB-RELAY.md` security section is 13 lines (lines 98-111). `NUB-STORAGE.md` is 12 lines (lines 89-102). `NUB-SIGNER.md` is 15 lines (lines 65-80). `NUB-IPC.md` is 7 lines (lines 90-96). `NUB-PIPES.md` is 32 lines (lines 297-327). No consistent template or checklist.
- Files: All six spec files in `specs/`
- Impact: Reviews cannot ensure security considerations are addressed uniformly. A critical threat model might be missing from one spec because the author focused on a different angle. New NUB proposals may omit key categories.
- Fix approach: Expand `TEMPLATE-WORD.md` to include a detailed security checklist: sandboxing assumptions, principal isolation, capability boundaries, input validation, timing attacks, side channels, dependencies on other specs' security properties. Provide examples (e.g., from NUB-RELAY) in the template.

## Implementation Links Not Validated

**Reference implementations may be outdated or deleted:**
- Issue: Each spec lists implementations at the bottom (e.g., `[@napplet/shim](https://github.com/sandwichfarm/napplet)`, `[@kehto/shell](https://github.com/sandwichfarm/kehto)`). These are GitHub URLs to external repos. No verification that links are valid, that the repos still exist, or that the implementation actually matches the spec.
- Files: `specs/NUB-RELAY.md` (lines 113-115), `specs/NUB-STORAGE.md` (lines 103-106), `specs/NUB-SIGNER.md` (lines 81-84), `specs/NUB-NOSTRDB.md` (lines 90-93), `specs/NUB-IPC.md` (lines 102-105), `specs/NUB-PIPES.md` (lines 366-369)
- Impact: A user implementing NUB-RELAY may visit the linked implementation and find it is stale, refactored, or deleted. No way to know if the spec and implementation are in sync. Over time, as repos evolve, specs become orphaned.
- Fix approach: Document in CLAUDE.md that implementation links are maintained manually. Create a `.github/workflows/check-implementations.yml` that periodically fetches the repos and reports if implementation files mentioned in specs still exist. Add a "Last verified" date to each spec.

## Repo Structure Duplication

**Duplicate templates and specs in root and specs/ directory:**
- Issue: `TEMPLATE-WORD.md` and `TEMPLATE-NN.md` exist in both `/home/sandwich/Develop/nubs/` (root) and `/home/sandwich/Develop/nubs/specs/` directory. Similarly, `README.md` exists in both locations with nearly identical content.
- Files: `TEMPLATE-WORD.md`, `TEMPLATE-NN.md`, `README.md` (root and specs/)
- Impact: Developers are unsure which file is canonical. PRs may update one but not the other, creating confusion. The `specs/` directory is where actual specs live, but templates and governance docs in root create ambiguity about whether specs should live there or in a subdirectory.
- Fix approach: Consolidate: keep governance docs (templates, `README.md`) in root only. Move all NUB specs into `specs/` and update relative links. If the maintainer wants specs to live in root as they are proposed via PR, document this explicitly in CLAUDE.md and remove the duplicate `specs/` directory or use it for reference only.

## No Versioning or Changelog Mechanism

**Specs cannot be versioned; breaking changes are untracked:**
- Issue: Specs are plain markdown files with no version field, changelog, or amendment process. If a spec author updates the file to add a breaking change (e.g., changing method signatures in NUB-RELAY), there is no way to know which version existing implementations support or to communicate the change to dependent projects.
- Files: All spec files in `specs/`
- Impact: As the ecosystem grows, shell and napplet implementations diverge. An app may ship against NUB-RELAY v1 semantics, but the repo's version is now v2 with breaking changes. No deprecation period, no semver, no migration guide.
- Fix approach: Add a version field to spec frontmatter (e.g., `Version: 1.0.0` + `Last Updated: YYYY-MM-DD`). Document the amendment process: minor clarifications do not require a version bump; breaking API changes require a new version (e.g., `NUB-RELAY-v2.md`) and a deprecation notice on the old spec. Or, adopt a formal NIP-like numbered proposal system (already done for NUB-NN; apply the same to NUB-WORD).

## Test Coverage and Spec Validation

**No formal validation or test suite for specs:**
- Issue: The specs define wire formats (e.g., NUB-PIPES message structures in lines 112-194), API signatures (e.g., NUB-RELAY subscribe method), and event kinds (e.g., kind 29001, 29003, 29006, 29007). There is no automated test that verifies examples in specs are syntactically correct or that wire format examples match the described protocol.
- Files: All spec files; especially `specs/NUB-PIPES.md` (wire format section), `specs/NUB-NOSTRDB.md` (event kinds)
- Impact: A spec author may publish a JSON example that is malformed. Wire format descriptions may omit required fields. Implementations that follow the spec literally will fail in the field. Reviewers must manually audit every code example.
- Fix approach: Create a `schema/` directory with JSON Schema or TypeScript definitions for wire formats and API contracts. Create a test file that parses and validates all code examples in specs. Add a CI check that runs before accepting PRs.

## Incomplete Boundary Between NUB-WORD and NUB-NN

**Specs do not consistently clarify interface vs. protocol role:**
- Issue: The governance section defines boundary rule (lines 34-38 in `specs/README.md`): "An interface (NUB-WORD) is **shell-provided** AND defines an **API surface**. A protocol (NUB-NN) is **napplet-agreed** AND defines **event semantics**. Both criteria must apply." However, many specs blur this line. For example, NUB-IPC defines both an API surface (`emit()`, `on()`) and event semantics (kind 29003, topic tags). NUB-PIPES defines API surface and control message types but explicitly does not use NIP-01 event kinds.
- Files: `specs/README.md` (lines 34-38), `specs/NUB-IPC.md` (entire spec), `specs/NUB-PIPES.md` (lines 278-295)
- Impact: Proposers cannot easily determine which track to use for a new spec. Reviewers have no clear criterion for rejecting an out-of-scope proposal. The description "judged pragmatically by the maintainer" suggests the boundary is subjective.
- Fix approach: Clarify in governance: NUB-WORD is shell API (what the shell exposes as callable functions or objects). NUB-NN is napplet-to-napplet event semantics (what events and formats napplets agree to exchange). Shell-only coordination (e.g., storage ops via kind 29003) that napplets do not use can be internal and undocumented. If a spec requires both API and event semantics, it should be NUB-WORD with event kinds defined as an implementation detail. Provide 2-3 examples to illustrate the boundary.

## Missing Capability/ACL Framework Definition

**Specs reference "ACL capabilities" but the system is not defined:**
- Issue: Nearly every spec mentions ACL capabilities: `relay:read`, `relay:write` (NUB-RELAY, line 74), `storage:read`, `storage:write` (NUB-STORAGE, line 98), `sign:event`, `sign:nip04`, `sign:nip44` (NUB-SIGNER, line 53), `pipe:connect` (NUB-PIPES, line 263). The napplet AUTH handshake (NIP-5D) is referenced as the source of these capabilities, but the actual framework — how capabilities are granted, checked, revoked — is not explained in any NUB spec.
- Files: `specs/NUB-RELAY.md` (line 74), `specs/NUB-STORAGE.md` (line 98), `specs/NUB-SIGNER.md` (line 53), `specs/NUB-PIPES.md` (line 263), `CLAUDE.md` (lines 4-7 reference NIP-5D)
- Impact: Shell implementers must guess how to check capabilities. Napplet developers do not know how to request specific capabilities. The governance authority (maintainer or NIP process) for capability definitions is unclear.
- Fix approach: Create a `NUB-CAPABILITIES.md` spec that defines the capability model: how they are encoded in the AUTH response, how the shell checks them, and how napplets request them. Or, clarify in CLAUDE.md that this is defined in NIP-5D and add explicit examples. Create a canonical list of all capabilities across all specs in one place.

---

*Concerns audit: 2026-04-07*
