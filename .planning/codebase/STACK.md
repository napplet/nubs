# Technology Stack

**Analysis Date:** 2026-04-07

## Languages

**Primary:**
- Markdown - Specification documents (all `.md` files)

**Documentation:**
- Markdown - Used exclusively for proposal specifications and governance documentation

## Runtime

**Environment:**
- Not applicable - This is a specification repository, not executable code

**Package Manager:**
- Not applicable - No dependencies or package management required

## Frameworks

**Core:**
- Not applicable - Pure documentation/specification repository

**Build/Dev:**
- Not applicable - No build system required

## Key Dependencies

**External References (Protocol-level, not code dependencies):**
- [NIP-5D](https://github.com/nostr-protocol/nips/blob/master/5D.md) - Core Nostr Web Applets protocol specification
- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) - Nostr relay protocol (referenced in NUB-RELAY)
- [NIP-07](https://github.com/nostr-protocol/nips/blob/master/07.md) - `window.nostr` browser extension standard (referenced in NUB-SIGNER)
- [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md) - Replaceable events (referenced in NUB-NOSTRDB)
- [NIP-45](https://github.com/nostr-protocol/nips/blob/master/45.md) - COUNT semantics (referenced in NUB-NOSTRDB)
- [Nostr Protocol](https://nostr.com/) - Event-based protocol foundation

## Configuration

**Repository Structure:**
- Root-level governance and templates
- `/specs/` directory - Specification proposals
- `.git/` - Git version control

**Version Control:**
- GitHub remote: `git@github.com:napplet/nubs.git`
- Branch strategy: Feature branches for individual NUB proposals, merged to `master` via PR

## Platform Requirements

**Development:**
- Git for version control
- Markdown editor (any text editor)
- GitHub account for contributing via PRs

**Platform for Deployment:**
- GitHub (for PR review and archival)
- Not deployed as executable code

---

*Stack analysis: 2026-04-07*
