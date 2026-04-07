# CLAUDE.md

## What This Is

**NUBs** (Napplet Unified Blueprints) — the extension proposal system for the napplet protocol. This repo hosts interface and message protocol specs that extend [NIP-5D](https://github.com/nostr-protocol/nips/blob/master/5D.md) (Nostr Web Applets).

NIP-5D defines the core: transport (`postMessage`), authentication (`REGISTER` → `IDENTITY` → `AUTH`), extension discovery (`shell.supports()`), and security model. Everything else — relay proxy, storage, signing, IFC, pipes, message protocols — is a NUB.

**Remote:** `git@github.com:napplet/nubs.git`

## Repo Structure

```
README.md           — Governance doc, dual-track overview, interface registry table
TEMPLATE-WORD.md    — Template for interface proposals (NUB-WORD)
TEMPLATE-NN.md      — Template for message protocol proposals (NUB-NN)
```

### What gets committed directly

- `README.md` — governance, registry table, track descriptions
- `TEMPLATE-WORD.md` — interface proposal template
- `TEMPLATE-NN.md` — message protocol proposal template

### What gets submitted as PRs

- `NUB-RELAY.md`, `NUB-STORAGE.md`, `NUB-IFC.md`, etc. — interface specs are **opened as PRs**, not committed to `master` directly. They follow the NIP-style informal process: fork, add spec, open PR, community reviews, maintainer merges.
- `NUB-01.md`, `NUB-02.md`, etc. — message protocol specs, same PR process.

## Two Tracks

### NUB-WORD (Interface Specs)

- Named by a single uppercase word: `NUB-RELAY`, `NUB-STORAGE`, `NUB-NOSTRDB`, `NUB-IFC`, `NUB-PIPES`
- One canonical spec per name — no competing interface specs
- Defines shell-provided API contracts on `window.napplet.*` namespaces
- Discovery: `shell.supports("NUB-RELAY")`
- Use `TEMPLATE-WORD.md` as the starting point

### NUB-NN (Message Protocol Specs)

- Numbered: `NUB-01`, `NUB-02`, etc.
- Multiple competing specs allowed per domain (e.g., two different feed protocols)
- Defines event semantics napplets agree on with each other
- Napplets negotiate via `shell.supports("NUB-RELAY", "NUB-02")`
- Use `TEMPLATE-NN.md` as the starting point

## Governance

NIP-style informal. Fork repo, add markdown spec, open PR. Community comments on PRs. Maintainer (dskvr) merges when it makes sense. No formal stages or review committee.

## Preparing PRs

Each NUB spec should be opened as its own PR:

1. Create a branch: `nub-relay`, `nub-storage`, etc.
2. Add the spec file: `NUB-RELAY.md`
3. Update the registry table in `README.md` (add the row with link to the spec)
4. Open PR with title: `NUB-RELAY: NIP-01 relay proxy interface`
5. PR body should include: one-line summary, namespace (`window.napplet.relay`), status (`draft`), and link to NIP-5D

The 6 initial interface specs to submit as PRs (source files in `~/Develop/napplet/specs/nubs/`):

| Spec | Source | Branch |
|------|--------|--------|
| NUB-RELAY | `~/Develop/napplet/specs/nubs/NUB-RELAY.md` | `nub-relay` |
| NUB-STORAGE | `~/Develop/napplet/specs/nubs/NUB-STORAGE.md` | `nub-storage` |
| NUB-NOSTRDB | `~/Develop/napplet/specs/nubs/NUB-NOSTRDB.md` | `nub-nostrdb` |
| NUB-IFC | `~/Develop/napplet/specs/nubs/NUB-IFC.md` | `nub-ifc` |
| NUB-PIPES | `~/Develop/napplet/specs/nubs/NUB-PIPES.md` | `nub-pipes` |

## Related Repos

- [`napplet/napplet`](https://github.com/sandwichfarm/napplet) — SDK monorepo (`@napplet/shim`, `@napplet/shell`, etc.)
- [`sandwichfarm/hyprgate`](https://github.com/sandwichfarm/hyprgate) — Reference shell implementation
- [`nostr-protocol/nips`](https://github.com/nostr-protocol/nips) — NIP-5D lives here
