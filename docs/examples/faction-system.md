# Faction System Example

This example demonstrates creating and managing custom factions, relationships, and dynamic faction systems.

## Overview

DrGBase's faction system allows NPCs to:
- Belong to multiple factions
- Have complex relationship dynamics
- Change factions dynamically
- Create custom faction hierarchies

## Built-in Factions

```lua
FACTION_PLAYERS      -- Friendly to players
FACTION_REBELS       -- Half-Life 2 Rebels
FACTION_COMBINE      -- Half-Life 2 Combine
FACTION_ZOMBIES      -- Zombies and headcrabs
FACTION_ANTLIONS     -- Antlions
FACTION_ANIMALS      -- Wildlife
```

## Basic Faction Setup

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Rebel Soldier"
ENT.Factions = {FACTION_REBELS}  -- This NPC is a Rebel

if SERVER then
    function ENT:CustomInitialize()
        -- Hate everyone not in same faction
        self:SetDefaultRelationship(D_HT)

        -- Friendly to players
        self:SetPlayersRelationship(D_LI)
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Creating Custom Factions

```lua
-- In lua/autorun/server/my_factions.lua (or shared)
FACTION_BANDITS = "FACTION_CUSTOM_BANDITS"
FACTION_GUARDS = "FACTION_CUSTOM_GUARDS"
FACTION_TRADERS = "FACTION_CUSTOM_TRADERS"
```

Then use them:
```lua
ENT.Factions = {FACTION_BANDITS}
```

## Multiple Factions

```lua
-- NPC belongs to both factions
ENT.Factions = {FACTION_REBELS, FACTION_PLAYERS}

function ENT:CustomInitialize()
    -- This NPC is friendly to both Rebels and Players
    self:SetDefaultRelationship(D_HT)  -- Hostile to others
end
```

## Dynamic Faction Changes

```lua
function ENT:CustomInitialize()
    self.CurrentFaction = FACTION_REBELS
    self.Factions = {self.CurrentFaction}
    self:SetDefaultRelationship(D_HT)
end

function ENT:SwitchFaction(newFaction)
    -- Leave old faction
    self:LeaveFaction(self.CurrentFaction)

    -- Join new faction
    self:JoinFaction(newFaction)
    self.CurrentFaction = newFaction

    -- Update relationships
    self:SetDefaultRelationship(D_HT)

    -- Visual feedback
    self:EmitSound("items/suitchargeok1.wav")

    -- Announce
    print(self, "switched to faction", newFaction)
end

function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()

    -- Betrayed by own faction? Switch sides!
    if IsValid(attacker) and self:GetRelationship(attacker) == D_LI then
        if attacker.Factions and attacker.Factions[1] then
            -- Join attacker's faction
            self:SwitchFaction(attacker.Factions[1])
        end
    end

    return true
end
```

## Faction Reputation System

```lua
-- Shared reputation table
_G.FactionReputation = _G.FactionReputation or {}

function ENT:CustomInitialize()
    self.PlayerRep = {}  -- Reputation with each player

    -- Initialize reputation
    for _, ply in ipairs(player.GetAll()) do
        self.PlayerRep[ply] = _G.FactionReputation[ply] and
            _G.FactionReputation[ply][FACTION_BANDITS] or 0
    end

    self:UpdateRelationshipsByReputation()
end

function ENT:UpdateRelationshipsByReputation()
    for ply, rep in pairs(self.PlayerRep) do
        if not IsValid(ply) then continue end

        if rep >= 100 then
            self:AddEntityRelationship(ply, D_LI)  -- Friendly
        elseif rep <= -100 then
            self:AddEntityRelationship(ply, D_HT)  -- Hostile
        else
            self:AddEntityRelationship(ply, D_NU)  -- Neutral
        end
    end
end

function ENT:ModifyPlayerReputation(ply, amount)
    self.PlayerRep[ply] = (self.PlayerRep[ply] or 0) + amount

    -- Save to global reputation
    _G.FactionReputation[ply] = _G.FactionReputation[ply] or {}
    _G.FactionReputation[ply][FACTION_BANDITS] = self.PlayerRep[ply]

    self:UpdateRelationshipsByReputation()
end

function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()
    if IsValid(attacker) and attacker:IsPlayer() then
        -- Attacking decreases reputation
        self:ModifyPlayerReputation(attacker, -10)
    end
    return true
end
```

## Faction Alliances

```lua
-- Define faction relationships
_G.FactionAlliances = {
    [FACTION_REBELS] = {
        [FACTION_PLAYERS] = D_LI,  -- Allied
        [FACTION_COMBINE] = D_HT,  -- Enemies
        [FACTION_ZOMBIES] = D_HT   -- Enemies
    },
    [FACTION_COMBINE] = {
        [FACTION_REBELS] = D_HT,
        [FACTION_ZOMBIES] = D_HT,
        [FACTION_ANTLIONS] = D_HT
    }
}

function ENT:CustomInitialize()
    -- Apply faction alliances
    if _G.FactionAlliances and self.Factions then
        for _, myFaction in ipairs(self.Factions) do
            local allies = _G.FactionAlliances[myFaction]
            if allies then
                for targetFaction, relationship in pairs(allies) do
                    self:SetFactionRelationship(targetFaction, relationship)
                end
            end
        end
    end
end
```

## Faction-Based Spawning

```lua
-- Create faction wave spawner
function SpawnFactionWave(faction, count, position)
    local npcs = {
        [FACTION_REBELS] = {"npc_drg_rebel1", "npc_drg_rebel2"},
        [FACTION_COMBINE] = {"npc_drg_soldier", "npc_drg_elite"},
        [FACTION_ZOMBIES] = {"npc_drg_zombie", "npc_drg_headcrab"}
    }

    local npcClasses = npcs[faction]
    if not npcClasses then return end

    for i = 1, count do
        local npcClass = npcClasses[math.random(#npcClasses)]
        local npc = ents.Create(npcClass)

        if IsValid(npc) then
            local offset = VectorRand() * 200
            npc:SetPos(position + offset)
            npc:Spawn()
        end
    end
end

-- Usage
SpawnFactionWave(FACTION_REBELS, 5, Vector(0, 0, 0))
```

## Faction Hierarchy (Chain of Command)

```lua
ENT.FactionRank = 1  -- 1 = soldier, 2 = sergeant, 3 = commander

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)

    -- Find squad leader (highest rank in faction nearby)
    self.SquadLeader = self:FindSquadLeader()

    if self.FactionRank >= 3 then
        -- Commanders give orders
        self:BeginCommandMode()
    end
end

function ENT:FindSquadLeader()
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
        if ent.Base == self.Base and ent.FactionRank and ent.FactionRank > self.FactionRank then
            -- Check same faction
            if ent.Factions and table.HasValue(ent.Factions, self.Factions[1]) then
                return ent
            end
        end
    end
    return nil
end

function ENT:BeginCommandMode()
    -- Issue orders to nearby allies
    timer.Create("Commander_"..self:EntIndex(), 5, 0, function()
        if not IsValid(self) then return end

        for _, ally in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
            if ally.Base == self.Base and ally ~= self then
                if ally.Factions and table.HasValue(ally.Factions, self.Factions[1]) then
                    -- Give order
                    if IsValid(self:GetEnemy()) then
                        ally:AddTargetEntity(self:GetEnemy())
                    end
                end
            end
        end
    end)
end
```

## Faction Territory System

```lua
-- Define territories
_G.FactionTerritories = {
    {
        faction = FACTION_REBELS,
        center = Vector(0, 0, 0),
        radius = 1000
    },
    {
        faction = FACTION_COMBINE,
        center = Vector(2000, 0, 0),
        radius = 1000
    }
}

function ENT:CustomThink()
    local territory = self:GetCurrentTerritory()

    if territory and territory.faction ~= self.Factions[1] then
        -- In enemy territory
        self.InEnemyTerritory = true

        -- Increase alertness
        self.SightRange = 3000
        self.SightFOV = 180
    else
        self.InEnemyTerritory = false
        self.SightRange = 2000
        self.SightFOV = 120
    end
end

function ENT:GetCurrentTerritory()
    for _, territory in ipairs(_G.FactionTerritories or {}) do
        if self:GetPos():Distance(territory.center) <= territory.radius then
            return territory
        end
    end
    return nil
end
```

## Next Steps

- Combine with [Advanced AI](./advanced-ai.md) for smarter faction behavior
- Use with [Boss NPC](./boss-npc.md) for faction leaders
- Apply to [Custom Weapons](./custom-weapon.md) for faction-specific loadouts
