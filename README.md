# Benilla Pet Battles

A client + server mod for vanilla World of Warcraft 1.12, adding a pet battle system on top of the companion pet collection from [benilla-pets-mounts-tab](https://github.com/rtrm/benilla-pets-mounts-tab).

See [ARCHITECTURE.md](ARCHITECTURE.md) for the design decisions and scope.

## Repo layout

- [`client/`](client) — [rtrm/benilla-pet-battles-client](https://github.com/rtrm/benilla-pet-battles-client), git submodule, branch `main`. Continues `rtrm/benilla`'s history (the pets-mounts-tab client) rather than forking upstream benilla fresh, so the companion pet Teach/Summon pattern and the Pets tab are the foundation, not something reimplemented.
- [`server/`](server) — [rtrm/core-pet-battles](https://github.com/rtrm/core-pet-battles), git submodule, branch `main`. Continues `rtrm/core`'s history (the pets-mounts-tab server, `development` branch upstream) for the same reason - the pet/mount SQL migrations are already in this lineage.

## Getting started

Clone with submodules:

```
git clone --recurse-submodules https://github.com/rtrm/benilla-pet-battles.git
```

Build instructions for the client and server live in their own READMEs ([client/README.md](client/README.md), [server/README.md](server/README.md)) — this repo only tracks the integration work on top of them.
