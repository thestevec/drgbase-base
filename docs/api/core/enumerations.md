# Enumerations

Constants and enumerations used throughout DrGBase.

**File:** `lua/drgbase/enumerations.lua`

## Factions

Factions are string identifiers used by the relationship system to group NPCs and determine alliances.

### Half-Life 2 Factions

#### FACTION_REBELS
- **Value:** `"FACTION_HL2_REBELS"`
- **Description:** Half-Life 2 rebel/resistance faction
- **Default Members:** npc_citizen, npc_barney, npc_alyx, npc_dog, npc_eli, npc_kleiner, npc_magnusson, npc_mossman, npc_monk, npc_fisherman, npc_odessa, npc_vortigaunt
- **Typical Enemies:** Combine, Zombies
- **Typical Allies:** Players (by default)

#### FACTION_COMBINE
- **Value:** `"FACTION_HL2_COMBINE"`
- **Description:** Half-Life 2 Combine forces
- **Default Members:** All npc_combine_*, npc_metropolice, npc_manhack, npc_strider, npc_turret_*, npc_hunter, npc_helicopter, npc_clawscanner, npc_cscanner, npc_stalker, npc_rollermine, npc_breen, npc_combine_camera
- **Typical Enemies:** Rebels, Players (by default)
- **Typical Allies:** Other Combine units

#### FACTION_ZOMBIES
- **Value:** `"FACTION_HL2_ZOMBIES"`
- **Description:** Zombie and headcrab faction
- **Default Members:** npc_zombie, npc_zombie_torso, npc_fastzombie, npc_fastzombie_torso, npc_poisonzombie, npc_zombine, npc_headcrab, npc_headcrab_fast, npc_headcrab_black, monster_babycrab, monster_headcrab, monster_zombie, monster_bigmomma
- **Typical Enemies:** Most living creatures
- **Typical Behavior:** Hostile to non-zombies

#### FACTION_ANTLIONS
- **Value:** `"FACTION_HL2_ANTLIONS"`
- **Description:** Antlion faction
- **Default Members:** npc_antlion, npc_antlionguard, npc_antlionguardian, npc_antlion_worker, npc_antlion_grub
- **Typical Behavior:** Hostile to most factions

#### FACTION_ANIMALS
- **Value:** `"FACTION_HL2_ANIMALS"`
- **Description:** Animal and passive creature faction
- **Default Members:** npc_crow, npc_pigeon, npc_seagull, monster_cockroach
- **Typical Behavior:** Neutral or defensive

#### FACTION_BARNACLES
- **Value:** `"FACTION_HL2_BARNACLES"`
- **Description:** Barnacle faction
- **Default Members:** npc_barnacle
- **Typical Behavior:** Hostile to all (ceiling trap)

#### FACTION_GMAN
- **Value:** `"FACTION_HL2_GMAN"`
- **Description:** G-Man faction (special)
- **Default Members:** npc_gman
- **Typical Behavior:** Special entity, typically neutral

### Half-Life 1 Factions

#### FACTION_XEN_ARMY
- **Value:** `"FACTION_HL_XEN_ARMY"`
- **Description:** Xen military units from Half-Life 1
- **Default Members:** monster_alien_grunt, monster_alien_slave, monster_alien_controller, monster_gargantua, monster_nihilanth

#### FACTION_XEN_WILDLIFE
- **Value:** `"FACTION_HL_XEN_WILDLIFE"`
- **Description:** Xen wildlife/creatures from Half-Life 1
- **Default Members:** monster_bullchicken, monster_houndeye, monster_snark, monster_tentacle

#### FACTION_HECU
- **Value:** `"FACTION_HL_HECU"`
- **Description:** Hazardous Environment Combat Unit (Marines) from Half-Life 1
- **Default Members:** monster_human_grunt, monster_human_assassin

### Special Factions

#### FACTION_SANIC
- **Value:** `"FACTION_SHITTY_SANIC_CLONES"`
- **Description:** Special faction for Sanic entities (easter egg/joke faction)

---

## Relationship Dispositions

Disposition constants define the type of relationship between entities. On the server, these use Garry's Mod's built-in disposition enums. On the client, DrGBase defines its own versions for networking.

### D_LI (Like)
- **Value:** `D_LI` (GMod constant on server, `3` on client)
- **Description:** Friendly/ally relationship
- **Behavior:** Will not attack, may assist or protect
- **Usage:** Set for allies and friends

### D_HT (Hate)
- **Value:** `D_HT` (GMod constant on server, `1` on client)
- **Description:** Hostile/enemy relationship
- **Behavior:** Will actively seek out and attack entity
- **Usage:** Set for enemies to attack on sight

### D_FR (Fear)
- **Value:** `D_FR` (GMod constant on server, `2` on client)
- **Description:** Fearful relationship
- **Behavior:** Will flee from entity, avoid confrontation
- **Usage:** Set for entities that frighten the NPC

### D_NU (Neutral)
- **Value:** `D_NU` (GMod constant on server, `4` on client)
- **Description:** Neutral/indifferent relationship (default)
- **Behavior:** Ignores entity unless provoked
- **Usage:** Default relationship for unknown entities

### D_ER (Error)
- **Value:** `0` on client
- **Description:** Error/undefined/invalid relationship
- **Behavior:** Used internally for invalid entity references
- **Usage:** Return value for error cases

---

## Possession Modes

Possession modes control how player input translates to NPC movement when possessing an NPC.

### POSSESSION_MOVE_8DIR
- **Value:** `1`
- **Aliases:** `POSSESSION_MOVE_NSEW`, `POSSESSION_MOVE_COMPASS`
- **Description:** 8-directional movement with diagonals (default)
- **Controls:** WASD keys with diagonal movement when pressing two keys
- **Usage:** Most common mode, allows full directional control

### POSSESSION_MOVE_1DIR
- **Value:** `2`
- **Aliases:** `POSSESSION_MOVE_FORWARD`
- **Description:** Forward-only movement
- **Controls:** W key moves forward, no strafing or backward movement
- **Usage:** For vehicles or forward-moving entities

### POSSESSION_MOVE_4DIR
- **Value:** `3`
- **Description:** 4-directional movement (no diagonals)
- **Controls:** WASD keys, but no diagonal movement (cardinal directions only)
- **Usage:** For grid-based or retro-style movement

### POSSESSION_MOVE_CUSTOM
- **Value:** `0`
- **Description:** Custom movement mode
- **Usage:** Implement your own movement logic in hooks

---

## AI Behavior Types

AI behavior types define which behavior system the NPC uses.

### AI_BEHAV_CUSTOM
- **Value:** `0`
- **Description:** Custom AI behavior
- **Usage:** Implement all AI behavior through hooks

### AI_BEHAV_BASE
- **Value:** `1`
- **Description:** Base AI behavior (default)
- **Usage:** Standard DrGBase AI for creatures and simple NPCs

### AI_BEHAV_HUMAN
- **Value:** `2`
- **Description:** Human/humanoid AI behavior
- **Usage:** Enhanced AI for humanoid NPCs with weapon support

---

## Patrol Types

Patrol types define how NPCs handle patrol point priorities.

### PATROL_POS
- **Value:** `0`
- **Description:** Position-based patrol (lowest priority)
- **Usage:** Regular patrol points

### PATROL_SEARCH
- **Value:** `100`
- **Description:** Search location (medium priority)
- **Usage:** Locations to investigate when searching

### PATROL_SOUND
- **Value:** `200`
- **Description:** Sound origin (highest priority)
- **Usage:** Locations where sounds were heard

---

## Node Types

Navigation node types used by the nav mesh system.

### NODE_TYPE_GROUND
- **Value:** `2`
- **Description:** Ground/walking node

### NODE_TYPE_AIR
- **Value:** `3`
- **Description:** Air/flying node

### NODE_TYPE_CLIMB
- **Value:** `4`
- **Description:** Climbing node

### NODE_TYPE_WATER
- **Value:** `5`
- **Description:** Water/swimming node

---

## Input Flags

Additional input flags defined by DrGBase.

### IN_ATTACK3
- **Value:** `33554432`
- **Description:** Third attack button (for possession system)
- **Usage:** Used for special attacks when possessing NPCs

---

## Notes

### States
DrGBase does not define custom AI state enumerations - it uses Garry's Mod's built-in `NPC_STATE_*` constants (NPC_STATE_IDLE, NPC_STATE_ALERT, NPC_STATE_COMBAT, etc.).

### Animation Events
DrGBase uses Garry's Mod's built-in animation event system and does not define custom animation event constants.

### Damage Types
DrGBase uses Garry's Mod's built-in `DMG_*` damage type constants (DMG_SLASH, DMG_BULLET, DMG_BLAST, etc.).

### Trace Masks
DrGBase uses Garry's Mod's built-in `MASK_*` trace mask constants (MASK_SOLID, MASK_SHOT, MASK_NPCSOLID, etc.).

---

## See Also

- [Relationship System](../../systems/relationships/README.md)
- [Faction Guide](../../guides/factions.md)
