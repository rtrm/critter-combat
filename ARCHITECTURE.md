# Benilla Pet Battles — Architecture

A client + server mod for vanilla World of Warcraft 1.12, adding a pet battle system on top of [benilla-pets-mounts-tab](https://github.com/rtrm/benilla-pets-mounts-tab)'s companion pet collection: capture a wild critter, fight with whatever pet is currently following you, and learn captured pets permanently, the same one-time-learn, character-bound pattern the companion collection already uses.

## Lineage, not a fresh fork

`client/` and `server/` continue `rtrm/benilla`'s and `rtrm/core`'s own history (the pets-mounts-tab forks), not a fresh fork of upstream benilla/vMaNGOS. The companion pet Teach/Summon two-spell pattern, the synthetic Pets tab, and all 69 migrated companion items are the foundation this builds on, not something to reimplement.

## Clean-room policy

Same as benilla-pets-mounts-tab's: this project does not read, clone, reference, or derive any code from the leaked Turtle WoW source some third parties mirror. Where a real game's publicly-documented pet-battle design (MoP's Pet Journal/battle system) inspired the concept, the implementation here is designed fresh from vanilla's own mechanics - and, per the design decisions below, deliberately diverges from that inspiration rather than reproducing it.

## Scope-defining design decisions

These four decisions were made explicitly, as departures from the original "MoP pet battles ported to vanilla" pitch:

1. **Vanilla-only assets, no MoP client.** Every battle UI, icon and effect uses assets already reachable from a real 1.12.1 client - no MoP-era data is bundled or required, matching benilla-pets-mounts-tab's own asset policy.
2. **One active pet, not a team of three.** You battle with whatever companion pet is currently summoned and following you, not a bench of three you pick before a fight. This matches the companion collection's own one-pet-out-at-a-time model (`_SetMiniPet`) instead of introducing a separate team-management concept.
3. **A fixed kit of three abilities per pet.** Every pet (companion or capturable wild critter) has its own unique set of exactly three abilities - not a shared/generic movepool, and not player-customizable. The abilities belong to the pet, the same way a companion's Summon spell already belongs to that specific companion.
4. **Level 20 cap, scaled across the 1-60 zone range.** A pet's own level caps at 20 regardless of player level, but a pet's effective strength in a battle scales with where it is - a level-20 pet means something different in a level-5 zone than a level-55 one. The exact scaling curve is not yet designed (see "Open design questions" below).

Capture-then-learn extends the existing Teach/Summon pattern: a successful capture is this system's "Teach" step, and the pet's three-ability kit plus its Summon-equivalent are what gets permanently learned, character-bound, exactly like a companion pet today.

## Open design questions (not yet designed in detail)

Deliberately left open until the combat system itself is designed:

- The capture mechanic: what makes an attempt succeed or fail, and what the player does to attempt one.
- The battle engine: turn order, how abilities resolve, win/loss conditions, what happens to the player's own pet on a loss.
- The level-scaling curve from decision 4: how a capped-at-20 pet's effective combat stats move across zones leveled 1 through 60.
- Which existing critters become capturable, and how their three-ability kits get authored (169 companion pets already exist from milestone 1 - wild non-companion critters are a separate, much larger set).
- Whether battling is PvE-only (wild pets) or extends to other players' active pets later.

## Build order

Not yet planned - this file will grow a "Milestone 1" section once the combat engine and capture mechanic are designed enough to build a pilot against, the same way benilla-pets-mounts-tab started from a single piloted companion before the bulk migration.
