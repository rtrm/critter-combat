# Critter Combat — Architecture

A client + server mod for vanilla World of Warcraft 1.12, adding a critter combat system on top of [benilla-pets-mounts-tab](https://github.com/rtrm/benilla-pets-mounts-tab)'s companion pet collection: capture a wild critter, fight with whatever pet is currently following you, and learn captured pets permanently, the same one-time-learn, character-bound pattern the companion collection already uses. Named apart from "pet battles" since the design below departs from MoP's own system in several real ways, not just a vanilla reskin of it.

## Lineage, not a fresh fork

`client/` and `server/` continue `rtrm/benilla`'s and `rtrm/core`'s own history (the pets-mounts-tab forks), not a fresh fork of upstream benilla/vMaNGOS. The companion pet Teach/Summon two-spell pattern, the synthetic Pets tab, and all 69 migrated companion items are the foundation this builds on, not something to reimplement.

## Clean-room policy

Same as benilla-pets-mounts-tab's: this project does not read, clone, reference, or derive any code from the leaked Turtle WoW source some third parties mirror. Where a real game's publicly-documented pet-battle design (MoP's Pet Journal/battle system) inspired the concept, the implementation here is designed fresh from vanilla's own mechanics - and, per the design decisions below, deliberately diverges from that inspiration rather than reproducing it, which is also why this project is named Critter Combat and not Pet Battles.

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

Not every critter can fight - roughly 10% are flagged battleable at the data level (a property of the creature template, decided once when a critter is authored, not rolled live). A battleable critter shows an overhead indicator, the same anchor mechanism vanilla already uses for skinnable/lootable creatures, but rendered as a crossed X of two real weapon models rather than a 2D icon or the MoP paw - picked for being the plainest, smallest basic shortsword model in the data, with no ornate variant to look out of place on a low-level critter: `Item\ObjectComponents\Weapon\Sword_1H_Short_A_01.m2`. Any battleable critter is also capturable - there is no separate capturable flag.

## The two taught spells

Both are real, permanently-learned player spells (spellbook-visible, like any other), taught by a stable master through a **new, separate gossip option** ("Learn Critter Combat") alongside the existing stable/pet-management interface - not merged into it, and usable from character level 1 for a modest gold cost:

- **Engage Critter Combat** (name placeholder) - cast on a targeted battleable critter to start a battle, pitting your currently-summoned pet against it.
- **Capture** (name placeholder) - costs more to learn than Engage. Usable mid-battle; capture chance depends on the enemy pet's level relative to yours, the same shape as MoP's own formula (harder the further above your pet's level the target is).

## Battle flow

No camera change, unlike MoP - the two pets simply position themselves facing each other in the world, and a battle action bar appears, the same UI pattern as a hunter's or warlock's existing pet action bar, just carrying the active pet's three abilities instead of pet commands. Combat is turn-based: the higher-level pet acts first each round; a tie on level is broken by current HP (not max HP) - not speed or any other stat.

## Pet health & death

A pet's HP is persistent, character-bound state - it is not reset to full between battles or on summon. A pet that ends a fight at 10 HP is still at 10 HP the next time it comes out, carrying the consequence of a bad fight forward rather than resetting it for free. A pet that reaches 0 HP dies: it cannot be summoned at all until healed. Losing a battle *is* this - there is no separate loss penalty, a loss is simply the fight that ended in the pet's HP hitting 0. Two ways to heal a pet:

- **The stable master**, for a fee: resurrecting every dead pet at once costs `1 silver × that pet's level`, summed across however many are dead.
- **A Pet Bandage item**, usable by the character directly, crafted via the First Aid profession. Tiered the same way regular bandages are (Linen, Wool, Silk, Mageweave, Runecloth, ...), but skipping the "Heavy" variant at each tier - one bandage per cloth rank, not two. Each tier's recipe cost will land around the same as its equivalent normal-bandage recipe; exact values are a later pass.

## Abilities don't change with level

A pet's three abilities are fixed from the moment it's learned (level 1) for its whole life - there is no separate ability unlock as it levels, the way a class gains new spells. Only the *values* an ability already has (damage, healing, duration, ...) scale with the pet's current level; the kit itself never grows or changes.

## Still open (deliberately deferred)

- The specific icon/asset choices for the ability action bar (depends on the abilities themselves, which are a later pass).
- Ability content itself: the general families of effects (damage, heal, buff/debuff, ...) will take inspiration from MoP's own, and each critter species (Hare, Adder, ...) gets its own unique three - but authoring the actual list is a deliberately separate, later pass, not part of this design.
- The stable master's exact gold cost for learning each spell, and the Pet Bandage recipes' exact tier costs/materials (principle set above, numbers later).
- PvP pet battles - confirmed as a later milestone, not out of scope permanently, just not part of this one.

## Milestone 1: pilot the whole loop on one critter

The **Prairie Dog** is the pilot, the same role Black Tabby Cat played for the companion collection: prove the full pattern end to end on one critter before authoring the rest. Bulk-flagging the ~10% battleable population and authoring every other species' three abilities is explicitly a later, separate pass - this milestone is about the mechanics working at all, not content breadth.

1. **Done.** Flag the Prairie Dog battleable at the data level, and give it its three abilities for real (this is where ability-content work actually starts, even though full authoring across every species is deferred) - enough to drive a real fight, not placeholders.

   Pet abilities are their own new table (`pet_battle_ability`/`pet_battle_abilities`, custom id space `63000+`), not `spell_template` rows: turn-based 1v1 pet combat has no cast time, GCD, mana or line-of-sight, which is most of what that table's columns exist for - reusing it would fight the schema more than use it. A small fixed `effect_type` enum (1 DAMAGE, 2 HIT_CHANCE_DEBUFF, 3 DAMAGE_TAKEN_SHIELD) is what the battle engine will switch on once it exists. Battleable-and-therefore-capturable is `pet_battle_wild`, presence-only, no separate flag column. Prairie Dog needed two different creature_template rows carrying the same three abilities - 2620 "Prairie Dog" (the wild critter, flagged battleable) and 14421 "Brown Prairie Dog" (milestone 1's existing companion mini-pet creature, Summon spell 60023) - so a successful Capture against the wild one just needs to grant that already-existing Summon spell, no new Summon-equivalent needed for this pilot. Persistent pet HP/level lives in a new `character_pet_battle` table, keyed by `summon_spell_id` - the same id that already uniquely identifies a learned companion. All three new tables and the pilot's rows are in `20261007150600_world.sql` and `20261007150700_characters.sql`, verified live against the running DB.

   The three abilities themselves, picked for this pilot (values are placeholders, not balanced): **Nibble** (direct damage), **Dust Cloud** (a hit-chance debuff on the enemy), **Burrow** (a damage-taken shield on self) - one damage, one debuff, one defensive, each scaling linearly with the pet's level.

   Server C++ now loads all three tables at startup (`ObjectMgr::LoadPetBattleAbilities`/`LoadPetBattleSpeciesAbilities`/`LoadPetBattleWild`, called from `World.cpp` right after `LoadFactionChangeMounts()`), modeled on `LoadFactionChangeReputations()`'s own pattern. Not yet done: the client still doesn't read any of this - that's step 2.
2. Add the client-side overhead indicator: the crossed `Sword_1H_Short_A_01.m2` pair over a battleable critter's head.
3. **Done.** Add the stable master's new, separate "Learn Critter Combat" gossip option, teaching Engage Critter Combat and Capture for gold, from character level 1.

   Pilot NPC only: Erma, the Stormwind stable master (entry 6749), via a new `critter_combat_stablemaster` script (`src/scripts/custom/custom_creatures.cpp`). All 91 stable masters currently have an empty `script_name` (verified live), so bulk rollout to the rest is a safe, mechanical follow-up once the pilot is confirmed working, not attempted here. The two spells themselves (64000 Engage Critter Combat, 64001 Capture, in `20261007155319_world.sql`) are `SPELL_EFFECT_DUMMY` placeholders - learning them already works end to end through the gossip menu, but what casting them actually *does* is step 4/5.
4. Build the battle flow: casting Engage Critter Combat on a targeted battleable critter positions the two pets, raises the pet-style action bar carrying the active pet's three abilities, and runs turn-based combat (higher level first, current-HP tiebreak).
5. Wire up Capture: usable mid-battle, chance keyed off the enemy pet's level relative to yours; a success teaches the Prairie Dog permanently, extending the existing Teach/Summon pattern (its own ability kit and Summon-equivalent land in `character_spell`, the same as a companion pet today).
6. Add persistent pet HP: carried across summons and battles, a pet at 0 HP becomes unsummonable, and both heal paths work - the stable master's `1 silver × level` resurrect-all, and a Pet Bandage item from First Aid.
7. Verify live: learn both spells, find a wild Prairie Dog, see the sword-X indicator, fight it with a companion pet, lose on purpose (confirm death + unsummonable), resurrect at the stable master, fight and win, capture it, confirm it's now a permanently known pet with its own three abilities.
