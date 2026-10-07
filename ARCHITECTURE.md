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
4. **Level 20 cap, scaled across the 1-60 zone range.** A pet's own level caps at 20 regardless of player level, but a pet's effective strength in a battle scales with where it is - a level-20 pet means something different in a level-5 zone than a level-55 one. The formula takes a zone's own character level range (as Wowhead Classic's zone pages state it, e.g. Desolace's "Level: 30 - 39") and divides each end by 3, rounded to a whole number, independently - not an average-plus-spread: `critter_min = max(1, round(zone_char_min ÷ 3))`, `critter_max = round(zone_char_max ÷ 3)`. Two real calibration points:
   - Durotar (character levels 1-10) → 1÷3 and 10÷3, rounded → critter levels 1-3
   - Desolace (character levels 30-39) → 30÷3 and 39÷3 → critter levels 10-13

   This is internally consistent with decision 4's own premise: a level-60 zone's character range divides to right around 20, so the cap and the top of the curve meet exactly where they should.

Capture-then-learn extends the existing Teach/Summon pattern: a successful capture is this system's "Teach" step, and the pet's three-ability kit plus its Summon-equivalent are what gets permanently learned, character-bound, exactly like a companion pet today.

## Battleable critters

Not every critter can fight - roughly 10% are flagged battleable at the data level (a property of the creature template, decided once when a critter is authored, not rolled live). A battleable critter shows an overhead indicator, the same anchor mechanism vanilla already uses for skinnable/lootable creatures, but rendered as two existing low-level sword weapon models crossed in an X over the creature's head - a real 3D model pair, not a 2D icon, and not the MoP paw. Any battleable critter is also capturable - there is no separate capturable flag.

## The two taught spells

Both are real, permanently-learned player spells (spellbook-visible, like any other), taught by a stable master through a **new, separate gossip option** ("Learn Critter Combat") alongside the existing stable/pet-management interface - not merged into it, and usable from character level 1 for a modest gold cost:

- **Engage Critter Combat** (name placeholder) - cast on a targeted battleable critter to start a battle, pitting your currently-summoned pet against it.
- **Capture** (name placeholder) - costs more to learn than Engage. Usable mid-battle; capture chance depends on the enemy pet's level relative to yours, the same shape as MoP's own formula (harder the further above your pet's level the target is).

## Battle flow

No camera change, unlike MoP - the two pets simply position themselves facing each other in the world, and a battle action bar appears, the same UI pattern as a hunter's or warlock's existing pet action bar, just carrying the active pet's three abilities instead of pet commands. Combat is turn-based: the higher-level pet acts first each round; a tie on level is broken by current HP (not max HP) - not speed or any other stat.

## Pet health & death

A pet's HP is persistent, character-bound state - it is not reset to full between battles or on summon. A pet that ends a fight at 10 HP is still at 10 HP the next time it comes out, carrying the consequence of a bad fight forward rather than resetting it for free. A pet that reaches 0 HP dies: it cannot be summoned at all until healed. Two ways to heal a pet:

- **The stable master**, for a fee (same NPC the two taught spells come from).
- **A Pet Bandage item**, usable by the character directly, crafted via the First Aid profession. Tiered the same way regular bandages are (Linen, Wool, Silk, Mageweave, Runecloth, ...), but skipping the "Heavy" variant at each tier - one bandage per cloth rank, not two.

## Still open (deliberately deferred)

- What happens to the player's pet specifically on a *loss* (does it just end the fight at low/zero HP and die normally, or is there a separate loss penalty beyond the health system above) - not yet distinguished from an ordinary damaging fight.
- The specific vanilla sword model to reuse for the overhead X indicator, and the specific icon/asset choices for the ability action bar.
- Ability content itself: the general families of effects (damage, heal, buff/debuff, ...) will take inspiration from MoP's own, and each critter species (Hare, Adder, ...) gets its own unique three - but authoring the actual list is a deliberately separate, later pass, not part of this design.
- The stable master's exact gold costs (learning each spell, resurrecting a dead pet) and the Pet Bandage recipes' exact tier costs/materials.
- Whether battling ever extends beyond wild critters (PvP pet battles) - nothing here assumes it will.

## Build order

Not yet planned - this file will grow a "Milestone 1" section once the combat engine and capture mechanic are designed enough to build a pilot against, the same way benilla-pets-mounts-tab started from a single piloted companion before the bulk migration.
