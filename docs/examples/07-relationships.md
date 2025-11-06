# Relationship and Faction System

## Overview

DrGBase includes a powerful relationship system that controls how NPCs interact with each other, players, and other entities. NPCs can be organized into factions with complex disposition rules.

## Basic Relationships

### Disposition Types

```lua
D_HT  -- Hate (attack on sight)
D_FR  -- Fear (run away)
D_LI  -- Like (friendly, will protect)
D_NU  -- Neutral (ignore)
```

### Setting Relationships

```lua
function ENT:CustomInitialize()
    -- Hate everything by default
    self:SetDefaultRelationship(D_HT)

    -- Like players
    self:SetPlayersRelationship(D_LI)

    -- Fear zombies
    self:SetClassRelationship("npc_drg_zombie", D_FR)

    -- Like NPCs with same model
    self:SetSelfModelRelationship(D_LI)

    -- Like NPCs of same class
    self:SetSelfClassRelationship(D_LI)
end
```

### Checking Relationships

```lua
local disposition = self:GetRelationship(entity)

if disposition == D_HT then
    print("Enemy!")
    self:AttackEntity(entity)
elseif disposition == D_FR then
    print("Scary!")
    self:FleeFrom(entity)
elseif disposition == D_LI then
    print("Friend!")
elseif disposition == D_NU then
    print("Don't care")
end
```

## Faction System

### Built-in Factions

```lua
FACTION_PLAYERS     -- All players
FACTION_ZOMBIES     -- Zombie faction
FACTION_ANTLIONS    -- Antlion faction
FACTION_COMBINE     -- Combine soldiers
FACTION_REBELS      -- Rebel forces
FACTION_PASSIVE     -- Peaceful NPCs
```

### Joining Factions

```lua
ENT.Factions = {FACTION_ZOMBIES} -- Set in ENT table

-- Or in code:
function ENT:CustomInitialize()
    self:JoinFaction(FACTION_ZOMBIES)

    -- Or multiple factions:
    self:JoinFactions({FACTION_ZOMBIES, FACTION_PASSIVE})
end
```

### Creating Custom Factions

```lua
-- In your addon's initialization:
FACTION_BANDITS = DrGBase.CreateFaction("Bandits")
FACTION_GUARDS = DrGBase.CreateFaction("Guards")
FACTION_TRADERS = DrGBase.CreateFaction("Traders")

-- Then use in NPC:
ENT.Factions = {FACTION_BANDITS}

function ENT:CustomInitialize()
    -- Set faction relationships
    self:SetFactionRelationship(FACTION_GUARDS, D_HT) -- Hate guards
    self:SetFactionRelationship(FACTION_TRADERS, D_LI) -- Like traders
end
```

## Advanced Relationship Examples

### Model-Based Teams

```lua
ENT.Models = {
    "models/player/kleiner.mdl",
    "models/player/magnusson.mdl"
}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)

    -- Create model-specific factions
    if self:GetModel() == "models/player/kleiner.mdl" then
        self:JoinFaction("FACTION_KLEINER")
        -- Kleiners like other Kleiners
        self:SetModelRelationship("models/player/kleiner.mdl", D_LI)
        -- Kleiners hate Magnussons
        self:SetModelRelationship("models/player/magnusson.mdl", D_HT)
    elseif self:GetModel() == "models/player/magnusson.mdl" then
        self:JoinFaction("FACTION_MAGNUSSON")
        self:SetModelRelationship("models/player/magnusson.mdl", D_LI)
        self:SetModelRelationship("models/player/kleiner.mdl", D_HT)
    end
end
```

### Dynamic Relationships

```lua
function ENT:CustomInitialize()
    self.FriendlyFire = 0
end

function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()

    if IsValid(attacker) and self:GetRelationship(attacker) == D_LI then
        -- Friendly fire!
        self.FriendlyFire = self.FriendlyFire + 1

        if self.FriendlyFire >= 3 then
            -- Turn hostile after 3 friendly fire incidents
            self:SetEntityRelationship(attacker, D_HT)
            self:SetEnemy(attacker)
            self:EmitSound("Betrayal.Sound")
        end
    end
end

-- Forgive over time
function ENT:CustomThink()
    if self.FriendlyFire > 0 then
        self.FriendlyFire = math.max(0, self.FriendlyFire - 0.001)
    end
end
```

### Intimidation System

```lua
ENT.Frightening = false -- Set to true for scary NPCs

function ENT:CustomInitialize()
    if self.IsBoss then
        self.Frightening = true
    end
end

-- NPCs with D_FR will flee from frightening entities
ENT.Factions = {FACTION_CIVILIANS}

function ENT:CustomInitialize()
    -- Fear frightening entities
    self:SetDefaultRelationship(D_NU)

    -- Check for scary entities periodically
    self:LoopTimer(1, function(self)
        for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
            if IsValid(ent) and ent.Frightening and ent:GetRelationship(self) == D_HT then
                self:SetEntityRelationship(ent, D_FR)
                self:EmitSound("Scream.Sound")
            end
        end
    end)
end
```

### Reputation System

```lua
-- Global reputation table
DrGReputation = DrGReputation or {}

function ENT:CustomInitialize()
    self:LoopTimer(2, function(self)
        -- Check player reputation
        for _, ply in ipairs(player.GetAll()) then
            local rep = DrGReputation[ply:SteamID()] or 0

            if rep > 50 then
                self:SetEntityRelationship(ply, D_LI)
            elseif rep < -50 then
                self:SetEntityRelationship(ply, D_HT)
            else
                self:SetEntityRelationship(ply, D_NU)
            end
        end
    end)
end

-- Modify reputation when player kills NPC
function ENT:OnDeath(dmg)
    local attacker = dmg:GetAttacker()

    if IsValid(attacker) and attacker:IsPlayer() then
        local steamid = attacker:SteamID()
        DrGReputation[steamid] = (DrGReputation[steamid] or 0) - 10
    end
end

-- Reward reputation for helping
function ENT:OnEnemyKilled(enemy, killer)
    if IsValid(killer) and killer:IsPlayer() then
        -- Player helped kill our enemy
        local steamid = killer:SteamID()
        DrGReputation[steamid] = (DrGReputation[steamid] or 0) + 5
    end
end
```

### Class-Based Alliances

```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)

    -- Ally with all "friendly" classes
    local friendlyClasses = {
        "npc_drg_medic",
        "npc_drg_guard",
        "npc_drg_trader"
    }

    for _, class in ipairs(friendlyClasses) do
        self:SetClassRelationship(class, D_LI)
    end

    -- Enemy to all "hostile" classes
    local hostileClasses = {
        "npc_drg_bandit",
        "npc_drg_raider",
        "npc_drg_zombie"
    }

    for _, class in ipairs(hostileClasses) do
        self:SetClassRelationship(class, D_HT)
    end
end
```

## Complex Faction Systems

### Three-Way Faction War

```lua
-- Define factions
FACTION_RED_TEAM = DrGBase.CreateFaction("Red Team")
FACTION_BLUE_TEAM = DrGBase.CreateFaction("Blue Team")
FACTION_GREEN_TEAM = DrGBase.CreateFaction("Green Team")

-- Red Team NPC
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Red Soldier"
ENT.Factions = {FACTION_RED_TEAM}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_NU)
    self:SetFactionRelationship(FACTION_RED_TEAM, D_LI)
    self:SetFactionRelationship(FACTION_BLUE_TEAM, D_HT)
    self:SetFactionRelationship(FACTION_GREEN_TEAM, D_HT)
    self:SetColor(Color(255, 0, 0))
end

-- Blue Team NPC (similar setup)
-- Green Team NPC (similar setup)
```

### Temporary Alliances

```lua
function ENT:CustomInitialize()
    self.AlliedFactions = {}
    self.AllianceExpiry = {}
end

function ENT:FormAlliance(faction, duration)
    self.AlliedFactions[faction] = true
    self.AllianceExpiry[faction] = CurTime() + duration

    self:SetFactionRelationship(faction, D_LI)
end

function ENT:CustomThink()
    -- Check for expired alliances
    for faction, expiry in pairs(self.AllianceExpiry) do
        if CurTime() > expiry then
            self:BreakAlliance(faction)
        end
    end
end

function ENT:BreakAlliance(faction)
    self.AlliedFactions[faction] = nil
    self.AllianceExpiry[faction] = nil
    self:SetFactionRelationship(faction, D_HT)
end

-- Example: Form alliance when threatened
function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()

    if IsValid(attacker) then
        -- Ask nearby NPCs to help
        for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
            if ent.IsNextbot and ent != self then
                -- Form temporary alliance
                if not self.AlliedFactions[ent.Factions[1]] then
                    self:FormAlliance(ent.Factions[1], 60) -- 60 second alliance
                    ent:SetEntityRelationship(attacker, D_HT)
                end
            end
        end
    end
end
```

### Priority System

```lua
function ENT:CustomInitialize()
    -- Relationship priorities (higher = more important)
    self.RelationshipPriorities = {
        entity = 100,    -- Specific entity relationships (highest)
        model = 75,      -- Model-based relationships
        class = 50,      -- Class-based relationships
        faction = 25,    -- Faction relationships
        default = 0      -- Default relationship (lowest)
    }
end

function ENT:SetEntityRelationship(ent, disposition, priority)
    priority = priority or self.RelationshipPriorities.entity
    -- Store with priority
    self.EntityRelationships = self.EntityRelationships or {}
    self.EntityRelationships[ent] = {
        disposition = disposition,
        priority = priority
    }
end

function ENT:GetRelationship(ent)
    -- Check in priority order
    local rel = self.EntityRelationships and self.EntityRelationships[ent]
    if rel then return rel.disposition end

    -- Check model relationships
    rel = self.ModelRelationships and self.ModelRelationships[ent:GetModel()]
    if rel then return rel end

    -- Check class relationships
    rel = self.ClassRelationships and self.ClassRelationships[ent:GetClass()]
    if rel then return rel end

    -- Check faction relationships
    if ent.Factions then
        for _, faction in ipairs(ent.Factions) do
            rel = self.FactionRelationships and self.FactionRelationships[faction]
            if rel then return rel end
        end
    end

    -- Return default
    return self.DefaultRelationship or D_NU
end
```

## Practical Examples

### Guards Protecting NPCs

```lua
-- Trader NPC (protected)
ENT.PrintName = "Trader"
ENT.Factions = {FACTION_TRADERS}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_NU)
    self:SetPlayersRelationship(D_LI)
end

-- Guard NPC (protector)
ENT.PrintName = "Guard"
ENT.Factions = {FACTION_GUARDS}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_NU)
    self:SetFactionRelationship(FACTION_TRADERS, D_LI) -- Protect traders
    self:SetPlayersRelationship(D_LI)
end

function ENT:CustomThink()
    -- Protect nearby traders
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
        if table.HasValue(ent.Factions or {}, FACTION_TRADERS) then
            local enemy = ent:GetEnemy()
            if IsValid(enemy) and not IsValid(self:GetEnemy()) then
                -- Trader is under attack, defend them
                self:SetEnemy(enemy)
                self:SetEntityRelationship(enemy, D_HT)
            end
        end
    end
end
```

### Pack Animals

```lua
ENT.PrintName = "Wolf"
ENT.Factions = {FACTION_WOLVES}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)
    self:SetSelfClassRelationship(D_LI) -- Like other wolves
    self.PackMembers = {}
    self:FindPack()
end

function ENT:FindPack()
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
        if ent:GetClass() == self:GetClass() and ent != self then
            table.insert(self.PackMembers, ent)
            table.insert(ent.PackMembers, self)
        end
    end
end

function ENT:OnNewEnemy(enemy)
    -- Alert pack members
    for _, member in ipairs(self.PackMembers) do
        if IsValid(member) and not IsValid(member:GetEnemy()) then
            member:SetEnemy(enemy)
            member:EmitSound("Wolf.Howl")
        end
    end
end
```

## Troubleshooting

### Relationships not working
- Check relationship is set in `CustomInitialize()`
- Verify faction names are correct
- Use `print(self:GetRelationship(ent))` to debug
- Check if entity has factions: `PrintTable(ent.Factions)`

### NPCs attacking allies
- Ensure faction relationships are set correctly
- Check for conflicting relationship rules
- Verify entities are in correct factions

### NPCs ignoring enemies
- Check default relationship is set
- Verify enemy detection is working
- Ensure AI is running properly

## Source Reference

Relationships: `lua/entities/drgbase_nextbot/relationships.lua` (831 lines)
Factions: `lua/drgbase/enumerations.lua`

## Related Documentation

- [Faction System](../reference/factions.md)
- [AI Behavior](06-custom-ai.md)
- [Detection System](../reference/detection.md)
