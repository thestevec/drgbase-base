# Relationship & Faction System

Faction management and entity relationships.

**File:** `relationships.lua` (831 lines)

## Overview

The relationship system manages how NPCs perceive and interact with other entities through dispositions (like/hate/fear/neutral), factions (groups of allies), and priority levels. It automatically handles NPC-to-NPC relationships and integrates with Garry's Mod's native NPC disposition system.

Key features:
- **Factions** - Pre-defined groups (FACTION_REBELS, FACTION_COMBINE, etc.)
- **Dispositions** - Four relationship types (D_LI, D_HT, D_FR, D_NU)
- **Relationship Priority** - Numeric priority for enemy selection
- **Damage Tolerance** - Forgiveness system for friendly fire
- **Automatic NPC Integration** - Works with vanilla NPCs

## Components

### Factions

Factions are predefined groups that share relationships. DrGBase includes factions for Half-Life 2 NPCs and supports custom factions.

**Built-in Factions:**
```lua
FACTION_NONE          -- No faction
FACTION_PLAYER        -- Players
FACTION_REBELS        -- Resistance (Alyx, Barney, Citizens, Vortigaunts)
FACTION_COMBINE       -- Combine forces (Soldiers, Metro Police, Scanners, etc.)
FACTION_ZOMBIES       -- Zombies and Headcrabs
FACTION_ANTLIONS      -- Antlions
FACTION_BARNACLES     -- Barnacles
FACTION_XEN_ARMY      -- Xen military (Alien Grunts, Controllers, Gargantua)
FACTION_XEN_WILDLIFE  -- Xen creatures (Houndeyes, Bullsquids, etc.)
FACTION_HECU          -- HECU Marines
FACTION_ANIMALS       -- Passive creatures (Crows, Pigeons, etc.)
FACTION_GMAN          -- G-Man
```

**Joining Factions:**
```lua
-- Join single faction
self:JoinFaction(FACTION_REBELS)

-- Join multiple factions
self:JoinFactions({FACTION_REBELS, FACTION_PLAYER})

-- Leave faction
self:LeaveFaction(FACTION_REBELS)
```

**Faction Queries:**
```lua
if self:IsInFaction(FACTION_REBELS) then end
local factions = self:GetFactions()
```

### Dispositions

Dispositions define the relationship type between entities.

**Disposition Types:**
- `D_LI` (Like) - Allies, will not attack
- `D_HT` (Hate) - Enemies, will attack
- `D_FR` (Fear) - Frightening, will flee from
- `D_NU` (Neutral) - Neutral, will ignore
- `D_ER` (Error) - Invalid relationship

**Checking Dispositions:**
```lua
if self:IsAlly(ent) then end      -- D_LI
if self:IsEnemy(ent) then end     -- D_HT
if self:IsAfraidOf(ent) then end  -- D_FR
if self:IsNeutral(ent) then end   -- D_NU
if self:IsHostile(ent) then end   -- D_HT or D_FR

local disp = self:GetRelationship(ent)
```

**Frightening Entities:**

Some entities can be frightening (cause D_FR):
```lua
ENT.Frightening = true  -- Make this NPC scary
self:SetFrightening(true)  -- Dynamic control

if ent:IsFrightening() then
    -- This entity scares other NPCs
end
```

### Relationship Priority

Priority determines which enemy is selected when multiple are available. Higher priority = more important target.

**Setting Priority:**
```lua
-- Set relationship with priority
self:SetRelationship(ent, D_HT, 5)  -- Enemy with priority 5

-- Default priority is 1
local prio = self:GetPriority(ent)
```

**Enemy Selection:**

When `FetchEnemy()` selects an enemy:
1. Entities with higher priority are preferred
2. If priority is equal, closer entities are preferred
3. Can be customized via `OnFetchEnemy(ent1, ent2)` hook

### Damage Tolerance

Allies can accidentally damage each other without becoming enemies. Damage tolerance controls how much damage is forgiven.

**Behavior:**
- When an ally deals damage, tolerance decreases
- When tolerance reaches 0, ally becomes enemy
- Tolerance regenerates over time
- Applies to allies (D_LI), neutral entities (D_NU), and afraid-of (D_FR)

**No direct API** - handled automatically by the system.

## Configuration

**Faction Properties:**
```lua
ENT.Factions = {FACTION_REBELS}  -- Starting factions
ENT.DefaultRelationship = D_NU   -- Default disposition to unknown entities
ENT.Frightening = false          -- Make other NPCs fear this one
```

**Relationship Definition:**

Set relationships to specific entities, classes, models, or factions:
```lua
function ENT:SetupDataTables()
    -- Hate all Combine
    self:SetRelationshipToFaction(FACTION_COMBINE, D_HT)

    -- Fear barnacles
    self:SetRelationshipToClass("npc_barnacle", D_FR)

    -- Like specific entity
    self:SetRelationship(someEntity, D_LI)

    -- Custom relationship table
    self.RelationshipTable = {
        [FACTION_REBELS] = D_LI,
        [FACTION_COMBINE] = D_HT,
        [FACTION_ZOMBIES] = D_HT,
    }
end
```

**Relationship Hooks:**
```lua
function ENT:OnRelationshipChange(ent, old, new)
    print("Relationship with " .. tostring(ent) .. " changed: " .. old .. " -> " .. new)
end

function ENT:ShouldIgnore(ent)
    -- Return true to ignore entity completely
    if ent:GetClass() == "prop_physics" then
        return true
    end
end
```

## Usage Examples

**Basic Faction Setup:**
```lua
function ENT:SetupDataTables()
    self:JoinFaction(FACTION_REBELS)
    self:SetRelationshipToFaction(FACTION_COMBINE, D_HT)
    self:SetRelationshipToFaction(FACTION_ZOMBIES, D_HT)
end
```

**Custom Relationship Logic:**
```lua
function ENT:SetupDataTables()
    -- Fear strong enemies
    for _, ent in ipairs(ents.GetAll()) do
        if ent:Health() > 500 then
            self:SetRelationship(ent, D_FR, 10)
        end
    end
end
```

**Dynamic Relationship Changes:**
```lua
function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()

    -- Become hostile to attacker
    if self:IsAlly(attacker) then
        self:SetRelationship(attacker, D_HT)
    end
end
```

**Relationship Table (Advanced):**
```lua
ENT.Factions = {FACTION_REBELS}
ENT.RelationshipTable = {
    [FACTION_REBELS] = D_LI,
    [FACTION_COMBINE] = D_HT,
    [FACTION_ZOMBIES] = D_HT,
    [FACTION_ANTLIONS] = D_HT,
}
```

**Iterators:**
```lua
-- Iterate through all enemies
for enemy in self:EnemyIterator() do
    print("Enemy: " .. tostring(enemy))
end

-- Iterate through all allies
for ally in self:AllyIterator() do
    print("Ally: " .. tostring(ally))
end

-- Iterate through spotted hostiles only
for hostile in self:HostileIterator(true) do
    print("Spotted hostile: " .. tostring(hostile))
end
```

**Ignore Entities:**
```lua
-- Temporarily ignore entity
self:SetIgnored(ent, true)

-- Check if ignored
if self:IsIgnored(ent) then end

-- Entities are automatically ignored if:
-- - Dead or dying
-- - Have FL_NOTARGET flag
-- - In NPC_STATE_DEAD or NPC_STATE_PLAYDEAD
```

**Default Relationship:**
```lua
-- Set default disposition to unknown entities
self:SetDefaultRelationship(D_HT)  -- Hostile to everything by default

local disp, prio = self:GetDefaultRelationship()
```

## API Reference

- [Relationship Functions](../../api/nextbot/relationships.md)
- [Enumerations](../../api/core/enumerations.md)

## Best Practices

**Faction Design:**
- Use existing factions when possible for compatibility with HL2 NPCs
- Create custom factions for unique groups (e.g., `FACTION_BANDITS = 100`)
- NPCs in the same faction are allies by default
- Use multiple factions to create complex alliances

**Relationship Management:**
- Set relationships in `SetupDataTables()` for initial state
- Use `OnRelationshipChange()` to react to relationship changes
- Prefer faction-based relationships over entity-specific ones
- Use priority to control target selection (bosses = high priority)

**Performance:**
- Relationships are cached for performance
- Iterators (EnemyIterator, AllyIterator) use caches
- Spotted entity caches separate from general caches
- Use `IsIgnored()` to exclude entities from AI consideration

**Disposition Guidelines:**
- `D_LI` - Will not attack, may help (allies)
- `D_HT` - Will actively pursue and attack (enemies)
- `D_FR` - Will flee from (frightening entities)
- `D_NU` - Will ignore unless provoked (neutral)

**Debugging:**
- Enable `drgbase_debug_relationships 1` to log relationship changes
- Relationships are networked to clients
- Check `GetRelationship(ent, true)` for absolute relationship (ignoring ignored status)

**Integration with Vanilla NPCs:**
- DrGBase automatically syncs relationships with Source engine NPCs
- Use `_UpdateNPCRelationship()` internally
- Vanilla NPCs will react to DrGBase NPCs appropriately
- Works with `ai_relationship` entities
