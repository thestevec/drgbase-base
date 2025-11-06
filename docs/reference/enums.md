# Enumerations Reference

Complete reference of DrGBase enumerations and constants.

## Faction Enumerations

Factions define which NPCs are allies and enemies.

### Built-in Factions

#### FACTION_REBELS
**Value**: `"FACTION_HL2_REBELS"`  
**Description**: Half-Life 2 resistance fighters.

```lua
ENT.Factions = {FACTION_REBELS}

-- Allied with: Citizens, Vortigaunts
-- Enemy with: Combine, Zombies
```

#### FACTION_COMBINE
**Value**: `"FACTION_HL2_COMBINE"`  
**Description**: Half-Life 2 Combine forces.

```lua
ENT.Factions = {FACTION_COMBINE}

-- Allied with: Manhacks, Scanners, Soldiers
-- Enemy with: Rebels, Citizens, Antlions
```

#### FACTION_ZOMBIES
**Value**: `"FACTION_HL2_ZOMBIES"`  
**Description**: Half-Life 2 zombies and headcrabs.

```lua
ENT.Factions = {FACTION_ZOMBIES}

-- Allied with: Other zombies, headcrabs
-- Enemy with: Everyone else
```

#### FACTION_ANTLIONS
**Value**: `"FACTION_HL2_ANTLIONS"`  
**Description**: Half-Life 2 antlions.

```lua
ENT.Factions = {FACTION_ANTLIONS}

-- Allied with: Other antlions
-- Enemy with: Combine, Rebels
```

#### FACTION_ANIMALS
**Value**: `"FACTION_HL2_ANIMALS"`  
**Description**: Wildlife and passive creatures.

```lua
ENT.Factions = {FACTION_ANIMALS}

-- Neutral to most factions
-- Examples: Crows, pigeons, seagulls
```

#### FACTION_BARNACLES
**Value**: `"FACTION_HL2_BARNACLES"`  
**Description**: Ceiling-dwelling barnacles.

```lua
ENT.Factions = {FACTION_BARNACLES}

-- Hostile to everything that comes near
```

#### FACTION_GMAN
**Value**: `"FACTION_HL2_GMAN"`  
**Description**: G-Man faction.

```lua
ENT.Factions = {FACTION_GMAN}

-- Neutral to all
```

#### FACTION_XEN_ARMY
**Value**: `"FACTION_HL_XEN_ARMY"`  
**Description**: Half-Life 1 Xen military forces.

```lua
ENT.Factions = {FACTION_XEN_ARMY}

-- Allied with: Alien Grunts, Controllers
-- Enemy with: HECU, Scientists
```

#### FACTION_XEN_WILDLIFE
**Value**: `"FACTION_HL_XEN_WILDLIFE"`  
**Description**: Half-Life 1 Xen wildlife.

```lua
ENT.Factions = {FACTION_XEN_WILDLIFE}

-- Allied with: Houndeyes, Bullsquids
-- Enemy with: HECU
```

#### FACTION_HECU
**Value**: `"FACTION_HL_HECU"`  
**Description**: Half-Life 1 Hazardous Environment Combat Unit.

```lua
ENT.Factions = {FACTION_HECU}

-- Allied with: Military, Black Ops
-- Enemy with: Xen creatures, Scientists
```

#### FACTION_SANIC
**Value**: `"FACTION_SHITTY_SANIC_CLONES"`  
**Description**: Example custom faction.

```lua
ENT.Factions = {FACTION_SANIC}

-- Demonstrates custom faction creation
```

### Custom Factions

Create your own factions:

```lua
-- Define custom faction
FACTION_BANDITS = "FACTION_CUSTOM_BANDITS"
FACTION_TRADERS = "FACTION_CUSTOM_TRADERS"

-- Use in NPC
ENT.Factions = {FACTION_BANDITS}

function ENT:CustomInitialize()
    -- Set hostility to traders
    DrGBase.SetFactionRelationship(FACTION_BANDITS, FACTION_TRADERS, D_HT)
end
```

## Relationship Dispositions

Dispositions define how entities feel about each other.

### D_HT (Hate)
**Value**: `1`  
**Description**: Hostile - will attack on sight.

```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)  -- Hate all
end
```

### D_FR (Fear)
**Value**: `2`  
**Description**: Afraid - will flee from target.

```lua
self:SetEntityRelationship(bigScaryEnemy, D_FR)  -- Run away!
```

### D_LI (Like)
**Value**: `3`  
**Description**: Friendly - will not attack.

```lua
function ENT:CustomInitialize()
    self:SetSelfModelRelationship(D_LI)  -- Friendly to same model
end
```

### D_NU (Neutral)
**Value**: `4`  
**Description**: Neutral - will ignore unless provoked.

```lua
self:SetEntityRelationship(civilian, D_NU)  -- Ignore them
```

### D_ER (Error)
**Value**: `0`  
**Description**: Error state - invalid relationship.

```lua
-- Don't use D_ER directly
if relationship == D_ER then
    print("Invalid relationship!")
end
```

## Node Types

Node types for DrGBase navigation system.

### NODE_TYPE_GROUND
**Value**: `2`  
**Description**: Ground-level pathfinding nodes.

```lua
if node.type == NODE_TYPE_GROUND then
    // Normal ground node
end
```

### NODE_TYPE_AIR
**Value**: `3`  
**Description**: Airborne pathfinding nodes.

```lua
ENT.NodeType = NODE_TYPE_AIR  -- For flying NPCs
```

### NODE_TYPE_CLIMB
**Value**: `4`  
**Description**: Climbing/ladder nodes.

```lua
if node.type == NODE_TYPE_CLIMB then
    self:Climb(node.pos)
end
```

### NODE_TYPE_WATER
**Value**: `5`  
**Description**: Underwater nodes.

```lua
ENT.NodeType = NODE_TYPE_WATER  -- For swimming NPCs
```

## AI Behavior Types

Behavior modes for NPCs.

### AI_BEHAV_CUSTOM
**Value**: `0`  
**Description**: Fully custom AI behavior.

```lua
ENT.AIBehaviour = AI_BEHAV_CUSTOM

function ENT:CustomThink()
    // Implement your own AI
end
```

### AI_BEHAV_BASE
**Value**: `1`  
**Description**: Standard DrGBase nextbot behavior.

```lua
ENT.AIBehaviour = AI_BEHAV_BASE  -- Default
```

### AI_BEHAV_HUMAN
**Value**: `2`  
**Description**: Human NPC behavior with weapons.

```lua
ENT.AIBehaviour = AI_BEHAV_HUMAN
ENT.Base = "drgbase_nextbot_human"
```

## Possession Movement Types

Movement modes for possessed NPCs.

### POSSESSION_MOVE_CUSTOM
**Value**: `0`  
**Description**: Custom movement implementation.

```lua
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM
```

### POSSESSION_MOVE_8DIR (NSEW, COMPASS)
**Value**: `1`  
**Description**: 8-directional movement (N, NE, E, SE, S, SW, W, NW).

```lua
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
-- Aliases: POSSESSION_MOVE_NSEW, POSSESSION_MOVE_COMPASS
```

### POSSESSION_MOVE_1DIR (FORWARD)
**Value**: `2`  
**Description**: Forward-only movement (like driving).

```lua
ENT.PossessionMovement = POSSESSION_MOVE_1DIR
-- Alias: POSSESSION_MOVE_FORWARD
```

### POSSESSION_MOVE_4DIR
**Value**: `3`  
**Description**: 4-directional movement (forward, back, left, right).

```lua
ENT.PossessionMovement = POSSESSION_MOVE_4DIR
```

## Patrol Types

Patrol point priorities.

### PATROL_POS
**Value**: `0`  
**Description**: Normal patrol position.

```lua
self:AddPatrolPos(Vector(100, 200, 0), PATROL_POS)
```

### PATROL_SEARCH
**Value**: `100`  
**Description**: Search/investigation point (higher priority).

```lua
function ENT:OnLostEnemy(enemy)
    -- Investigate last known position
    self:AddPatrolPos(enemy:GetPos(), PATROL_SEARCH)
end
```

### PATROL_SOUND
**Value**: `200`  
**Description**: Sound investigation point (highest priority).

```lua
function ENT:OnHearSound(pos, sound)
    -- Investigate sound
    self:AddPatrolPos(pos, PATROL_SOUND)
end
```

## Input Constants

### IN_ATTACK3
**Value**: `33554432`  
**Description**: Third attack button input flag.

```lua
function ENT:OnPossessed(ply, cmd)
    if cmd:KeyDown(IN_ATTACK3) then
        self:SpecialAbility()
    end
end
```

## Damage Type Enumerations

Standard Source engine damage types (not DrGBase-specific but commonly used).

```lua
DMG_GENERIC      -- Generic damage
DMG_CRUSH        -- Crushing damage
DMG_BULLET       -- Bullet damage
DMG_SLASH        -- Sharp/slashing damage
DMG_BURN         -- Fire damage
DMG_VEHICLE      -- Vehicle impact
DMG_FALL         -- Fall damage
DMG_BLAST        -- Explosive damage
DMG_CLUB         -- Blunt damage
DMG_SHOCK        -- Electric damage
DMG_SONIC        -- Sound damage
DMG_ENERGYBEAM   -- Energy beam
DMG_NEVERGIB     -- Don't create gibs
DMG_ALWAYSGIB    -- Always create gibs
DMG_DROWN        -- Drowning
DMG_PARALYZE     -- Neurotoxin/paralysis
DMG_NERVEGAS     -- Nerve gas
DMG_POISON       -- Poison damage
DMG_RADIATION    -- Radiation
DMG_DROWNRECOVER -- Drowning recovery
DMG_ACID         -- Acid damage
DMG_SLOWBURN     -- Damage over time (fire)
DMG_REMOVENORAGDOLL -- Don't create ragdoll
DMG_PHYSGUN      -- Physics gun damage
DMG_PLASMA       -- Plasma damage
DMG_AIRBOAT      -- Airboat gun
DMG_DISSOLVE     -- Dissolve effect
DMG_BLAST_SURFACE -- Surface-only blast
DMG_DIRECT       -- Direct damage (ignores armor)
DMG_BUCKSHOT     -- Shotgun pellets
```

### Usage Example
```lua
function ENT:OnMeleeAttack(enemy)
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({
            damage = 25,
            type = DMG_SLASH,  -- Sharp weapon
            -- or DMG_CLUB for blunt weapon
            -- or DMG_BURN for fire attack
        })
    end
end
```

## Hitgroup Enumerations

Body part constants for damage localization.

```lua
HITGROUP_GENERIC  -- 0 - Generic/unknown
HITGROUP_HEAD     -- 1 - Head
HITGROUP_CHEST    -- 2 - Chest
HITGROUP_STOMACH  -- 3 - Stomach
HITGROUP_LEFTARM  -- 4 - Left arm
HITGROUP_RIGHTARM -- 5 - Right arm
HITGROUP_LEFTLEG  -- 6 - Left leg
HITGROUP_RIGHTLEG -- 7 - Right leg
HITGROUP_GEAR     -- 10 - Gear/equipment
```

### Usage Example
```lua
function ENT:OnTakeDamage(dmginfo, hitgroup)
    if hitgroup == HITGROUP_HEAD then
        -- Headshot: triple damage
        dmginfo:ScaleDamage(3.0)
        self:EmitSound("Headshot.Sound")
    end
    return true
end
```

## Complete Examples

### Faction Setup
```lua
ENT.Factions = {FACTION_COMBINE}

function ENT:CustomInitialize()
    -- Hate all non-Combine
    self:SetDefaultRelationship(D_HT)
    
    -- Like other Combine
    self:SetFactionRelationship(FACTION_COMBINE, D_LI)
    
    -- Fear antlion guards
    self:SetEntityRelationship(antlionGuard, D_FR)
end
```

### Patrol System
```lua
function ENT:OnIdle()
    -- Add normal patrol point
    self:AddPatrolPos(self:RandomPos(1000), PATROL_POS)
end

function ENT:OnLostEnemy(enemy)
    -- Investigate last known position (high priority)
    self:AddPatrolPos(enemy:GetPos(), PATROL_SEARCH)
end

function ENT:OnHearSound(pos)
    -- Investigate sound (highest priority)
    self:AddPatrolPos(pos, PATROL_SOUND)
end
```

### Possession Movement
```lua
-- Flying NPC with 8-directional movement
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.NodeType = NODE_TYPE_AIR

-- Ground vehicle with forward-only movement
ENT.PossessionMovement = POSSESSION_MOVE_FORWARD

-- Standard NPC with 4-directional movement
ENT.PossessionMovement = POSSESSION_MOVE_4DIR
```

### Damage Types
```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        local attackType = self.CurrentAttack
        
        if attackType == "slash" then
            self:Attack({damage = 20, type = DMG_SLASH})
        elseif attackType == "crush" then
            self:Attack({damage = 35, type = DMG_CLUB})
        elseif attackType == "poison" then
            self:Attack({damage = 5, type = DMG_POISON})
        end
    end
end
```

For usage examples, see [Faction System Examples](../examples/faction-system.md) and [Advanced AI Examples](../examples/advanced-ai.md).
