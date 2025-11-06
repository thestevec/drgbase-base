# Faction System Guide

A comprehensive guide to using DrGBase's faction and relationship system to create complex NPC interactions.

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding Dispositions](#understanding-dispositions)
3. [Setting Up NPC Factions](#setting-up-npc-factions)
4. [Faction Relationships](#faction-relationships)
5. [Player Factions](#player-factions)
6. [Relationship Priorities](#relationship-priorities)
7. [Damage Tolerance](#damage-tolerance)
8. [Dynamic Faction Changes](#dynamic-faction-changes)
9. [Complex Scenarios](#complex-scenarios)
10. [Advanced Techniques](#advanced-techniques)

---

## Introduction

DrGBase provides a powerful relationship system that determines how NPCs interact with players, other NPCs, and entities. The faction system allows you to create complex alliances, rivalries, and dynamic interactions without manually coding each relationship.

### Key Concepts

- **Factions**: Groups that NPCs can belong to (e.g., FACTION_REBELS, FACTION_COMBINE)
- **Dispositions**: How an NPC feels about another entity (like, hate, fear, neutral)
- **Priorities**: Which relationship takes precedence when multiple rules conflict
- **Damage Tolerance**: How much damage before relationships change

### How It Works

1. NPCs join one or more factions
2. Factions define relationships with other factions
3. The relationship system resolves priorities to determine final disposition
4. Damage can change relationships dynamically

---

## Understanding Dispositions

Dispositions define how an NPC behaves toward another entity.

### Disposition Types

```lua
D_LI  -- Like/Love: Ally with this entity
D_HT  -- Hate: Attack this entity on sight
D_FR  -- Fear: Flee from this entity
D_NU  -- Neutral: Ignore this entity
D_ER  -- Error: Invalid (internal use only)
```

### Disposition Behaviors

| Disposition | Behavior | AI Response |
|-------------|----------|-------------|
| `D_LI` | Friendly ally | Defends, follows, assists |
| `D_HT` | Active enemy | Attacks, chases, targets |
| `D_FR` | Feared entity | Flees, avoids, panics |
| `D_NU` | Neutral entity | Ignores, no interaction |

### Example: Basic Disposition Setup

```lua
-- Hostile zombie that attacks everything
ENT.DefaultRelationship = D_HT

-- Friendly companion that likes everyone
ENT.DefaultRelationship = D_LI

-- Passive creature that ignores others
ENT.DefaultRelationship = D_NU

-- Cowardly NPC that fears everything
ENT.DefaultRelationship = D_FR
```

---

## Setting Up NPC Factions

### Built-in Factions

DrGBase includes factions for Half-Life 2 and Half-Life 1 NPCs:

**Half-Life 2 Factions:**
```lua
FACTION_REBELS      -- "FACTION_HL2_REBELS"
FACTION_COMBINE     -- "FACTION_HL2_COMBINE"
FACTION_ZOMBIES     -- "FACTION_HL2_ZOMBIES"
FACTION_ANTLIONS    -- "FACTION_HL2_ANTLIONS"
FACTION_ANIMALS     -- "FACTION_HL2_ANIMALS"
FACTION_GMAN        -- "FACTION_HL2_GMAN"
FACTION_BARNACLES   -- "FACTION_HL2_BARNACLES"
```

**Half-Life 1 Factions:**
```lua
FACTION_XEN_ARMY     -- "FACTION_HL_XEN_ARMY"
FACTION_XEN_WILDLIFE -- "FACTION_HL_XEN_WILDLIFE"
FACTION_HECU         -- "FACTION_HL_HECU"
```

**Special Factions:**
```lua
FACTION_SANIC       -- "FACTION_SHITTY_SANIC_CLONES"
```

### Basic Faction Setup

```lua
-- Single faction
ENT.Factions = {FACTION_ZOMBIES}

-- Multiple factions
ENT.Factions = {FACTION_REBELS, FACTION_ANIMALS}

-- No faction (lone wolf)
ENT.Factions = {}
```

### Example: Combine Soldier

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Combine Soldier"

ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_COMBINE}

function ENT:CustomInitialize()
    -- Combine soldiers automatically ally with other FACTION_COMBINE members
    -- and inherit faction relationships
end
```

### Example: Multi-Faction NPC

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Vortigaunt Ally"

ENT.DefaultRelationship = D_NU
ENT.Factions = {FACTION_REBELS, FACTION_XEN_ARMY}

-- This NPC is allied with BOTH rebels and Xen aliens
-- Useful for creating bridge characters or reformed enemies
```

### Creating Custom Factions

Create custom factions by defining string constants:

```lua
-- In shared.lua or autorun file
FACTION_CUSTOM_ROBOTS = "FACTION_CUSTOM_ROBOTS"
FACTION_CUSTOM_WILDLIFE = "FACTION_CUSTOM_WILDLIFE"
FACTION_CUSTOM_BANDITS = "FACTION_CUSTOM_BANDITS"

-- Use in your NPC
ENT.Factions = {FACTION_CUSTOM_ROBOTS}
```

**Important:** Faction names are case-insensitive and automatically converted to uppercase.

---

## Faction Relationships

### Understanding Faction Relationships

When an NPC joins a faction, it automatically:
1. Becomes friendly (D_LI) with other members of that faction
2. Can set relationships with other factions
3. Inherits faction-wide relationships

### Setting Faction Relationships

There are three ways to set faction relationships:

#### Method 1: SetFactionRelationship (Replaces)

```lua
function ENT:CustomInitialize()
    -- Set relationship to another faction
    self:SetFactionRelationship(FACTION_COMBINE, D_HT)
    self:SetFactionRelationship(FACTION_REBELS, D_LI)
    self:SetFactionRelationship(FACTION_ANIMALS, D_NU)
end
```

#### Method 2: AddFactionRelationship (Higher Priority Wins)

```lua
function ENT:CustomInitialize()
    -- Only sets if priority is higher than existing
    self:AddFactionRelationship(FACTION_ZOMBIES, D_HT, 10)
    self:AddFactionRelationship(FACTION_ANTLIONS, D_FR, 5)
end
```

#### Method 3: Configure in Entity Table

```lua
ENT.Factions = {FACTION_REBELS}

function ENT:CustomInitialize()
    -- Rebels hate Combine
    self:SetFactionRelationship(FACTION_COMBINE, D_HT)

    -- Rebels like other friendly factions
    self:SetFactionRelationship(FACTION_ANIMALS, D_LI)

    -- Rebels fear zombies
    self:SetFactionRelationship(FACTION_ZOMBIES, D_FR)
end
```

### Example: Faction War

```lua
-- Rebel NPC
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Rebel Fighter"
ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_REBELS}

function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_COMBINE, D_HT)  -- Hate Combine
    self:SetFactionRelationship(FACTION_ZOMBIES, D_HT)  -- Hate Zombies
    self:SetFactionRelationship(FACTION_ANIMALS, D_NU)  -- Ignore animals
end
```

```lua
-- Combine NPC
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Combine Soldier"
ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_COMBINE}

function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_REBELS, D_HT)   -- Hate Rebels
    self:SetFactionRelationship(FACTION_ZOMBIES, D_HT)  -- Hate Zombies
    self:SetFactionRelationship(FACTION_ANIMALS, D_NU)  -- Ignore animals
end
```

**Result:** Rebels and Combine fight each other on sight, both attack zombies, both ignore animals.

### Faction Membership Functions

```lua
-- Join faction at runtime
self:JoinFaction(FACTION_ZOMBIES)

-- Join multiple factions
self:JoinFactions({FACTION_REBELS, FACTION_ANIMALS})

-- Leave a faction
self:LeaveFaction(FACTION_COMBINE)

-- Leave multiple factions
self:LeaveFactions({FACTION_REBELS, FACTION_ANIMALS})

-- Leave all factions
self:LeaveAllFactions()

-- Check faction membership
if self:IsInFaction(FACTION_ZOMBIES) then
    print("I'm a zombie!")
end

-- Get all factions
local factions = self:GetFactions()
for _, faction in ipairs(factions) do
    print("Member of:", faction)
end
```

---

## Player Factions

Players can join factions to interact with DrGBase NPCs.

### Player Faction Functions

```lua
-- SERVER ONLY

-- Join faction
player:DrG_JoinFaction(FACTION_REBELS)

-- Join multiple factions
player:DrG_JoinFactions({FACTION_REBELS, FACTION_ANIMALS})

-- Leave faction
player:DrG_LeaveFaction(FACTION_REBELS)

-- Leave multiple factions
player:DrG_LeaveFactions({FACTION_REBELS, FACTION_ANIMALS})

-- Leave all factions
player:DrG_LeaveAllFactions()

-- Check membership
if player:DrG_IsInFaction(FACTION_REBELS) then
    print("Player is a rebel!")
end

-- Get all factions
local factions = player:DrG_GetFactions()
```

### Example: Player Faction System

```lua
-- gamemode/init.lua or autorun/server

hook.Add("PlayerSpawn", "AssignPlayerFaction", function(ply)
    -- All players start as rebels
    ply:DrG_JoinFaction(FACTION_REBELS)
end)

-- Console command to change faction
concommand.Add("faction_join", function(ply, cmd, args)
    if not args[1] then return end

    local faction = args[1]
    if faction == "rebels" then
        ply:DrG_LeaveAllFactions()
        ply:DrG_JoinFaction(FACTION_REBELS)
        ply:ChatPrint("Joined the Rebels!")
    elseif faction == "combine" then
        ply:DrG_LeaveAllFactions()
        ply:DrG_JoinFaction(FACTION_COMBINE)
        ply:ChatPrint("Joined the Combine!")
    end
end)
```

### Example: Dynamic Player-NPC Relationships

```lua
-- When player joins rebels, rebels like them
hook.Add("PlayerSpawn", "RebelAlliance", function(ply)
    ply:DrG_JoinFaction(FACTION_REBELS)

    -- Now all rebel NPCs will be friendly to this player
    for _, npc in ipairs(ents.FindByClass("npc_drg_rebel")) do
        if npc.IsDrGNextbot then
            npc:UpdateRelationshipWith(ply)
        end
    end
end)
```

### Example: Team-Based Factions

```lua
-- Assign factions based on team
hook.Add("PlayerSpawn", "TeamFactions", function(ply)
    if ply:Team() == TEAM_REBELS then
        ply:DrG_LeaveAllFactions()
        ply:DrG_JoinFaction(FACTION_REBELS)
    elseif ply:Team() == TEAM_COMBINE then
        ply:DrG_LeaveAllFactions()
        ply:DrG_JoinFaction(FACTION_COMBINE)
    end
end)
```

---

## Relationship Priorities

When multiple relationship rules conflict, DrGBase uses a priority system to resolve the final disposition.

### Priority Hierarchy

From **highest** to **lowest** priority:

1. **Entity-specific** - Direct relationship with specific entity
2. **Class-specific** - Relationship with entity class
3. **Model-specific** - Relationship with entity model
4. **Faction** - Relationship based on faction membership
5. **Custom** - CustomRelationship() hook result
6. **Default** - DefaultRelationship property

### Priority Values

Each relationship can have a numerical priority (default is 1):

```lua
-- Higher priority = takes precedence
self:SetEntityRelationship(entity, D_HT, 100)  -- Very high priority
self:SetEntityRelationship(entity, D_LI, 1)    -- Normal priority
```

### Example: Priority Resolution

```lua
function ENT:CustomInitialize()
    -- Default: Hate everyone (priority 1)
    self.DefaultRelationship = D_HT

    -- Faction: Like all rebels (priority 1)
    self:SetFactionRelationship(FACTION_REBELS, D_LI)

    -- Class: Hate all players (priority 1)
    self:SetClassRelationship("player", D_HT)

    -- Entity: Like this specific player (priority 10)
    local specificPlayer = Entity(1)
    self:SetEntityRelationship(specificPlayer, D_LI, 10)
end
```

**Result:**
- Most players: D_HT (class relationship)
- Entity(1): D_LI (entity relationship overrides class)
- Rebel NPCs: D_LI (faction relationship)
- Everything else: D_HT (default)

### Setting Relationship Priority

```lua
-- Entity relationships with priority
self:SetEntityRelationship(ent, D_LI, 50)
self:AddEntityRelationship(ent, D_HT, 100)  -- Only sets if priority > current

-- Class relationships with priority
self:SetClassRelationship("npc_zombie", D_HT, 25)
self:AddClassRelationship("npc_zombie", D_FR, 10)  -- Won't override (10 < 25)

-- Model relationships with priority
self:SetModelRelationship("models/zombie.mdl", D_HT, 15)
self:AddModelRelationship("models/zombie.mdl", D_NU, 5)  -- Won't override

-- Faction relationships with priority
self:SetFactionRelationship(FACTION_COMBINE, D_HT, 30)
self:AddFactionRelationship(FACTION_COMBINE, D_LI, 20)  -- Won't override
```

### Example: Complex Priority System

```lua
function ENT:CustomInitialize()
    -- Base: Neutral to everyone
    self.DefaultRelationship = D_NU

    -- Faction rule: Like rebels
    self:SetFactionRelationship(FACTION_REBELS, D_LI, 10)

    -- Class rule: Hate players (overrides faction)
    self:SetClassRelationship("player", D_HT, 20)

    -- Specific exception: Like admin player (overrides class)
    local admin = player.GetByID(1)
    if IsValid(admin) then
        self:SetEntityRelationship(admin, D_LI, 100)
    end
end
```

**Result:**
- Admin player (ID 1): D_LI (priority 100)
- Other players: D_HT (priority 20)
- Rebel NPCs: D_LI (priority 10)
- Everything else: D_NU (default)

---

## Damage Tolerance

Damage tolerance controls how relationships change when an NPC takes damage from another entity.

### Tolerance Properties

```lua
ENT.AllyDamageTolerance = 0.33     -- Forgiveness from allies (default 33%)
ENT.AfraidDamageTolerance = 0.33   -- Tolerance while afraid (default 33%)
ENT.NeutralDamageTolerance = 0.33  -- Provocation threshold (default 33%)
```

### How Damage Tolerance Works

1. NPC is damaged by an entity
2. System checks relationship type (ally, afraid, or neutral)
3. Adds damage to running total for that entity
4. If total exceeds tolerance, relationship changes to D_HT (hate)

**Formula:**
```lua
-- Damage accumulates as priority points
toleranceAccumulated = toleranceAccumulated + ENT.DamageTolerance
self:AddEntityRelationship(attacker, D_HT, toleranceAccumulated)
```

### Tolerance Values

```lua
-- Very forgiving (ignores most damage)
ENT.AllyDamageTolerance = 0.9      -- 90% health before becoming hostile
ENT.NeutralDamageTolerance = 0.8   -- 80% health before attacking

-- Quick to anger (aggressive)
ENT.AllyDamageTolerance = 0.1      -- 10% health = instant betrayal
ENT.NeutralDamageTolerance = 0.05  -- 5% health = provoked

-- Cowardly (becomes hostile when afraid)
ENT.AfraidDamageTolerance = 0.1    -- Quickly fights back when cornered

-- Absolutely loyal (never retaliates against allies)
ENT.AllyDamageTolerance = 999      -- Effectively infinite tolerance
```

### Example: Forgiving Companion

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Loyal Companion"

ENT.DefaultRelationship = D_LI
ENT.Factions = {FACTION_REBELS}

-- Very forgiving to allies
ENT.AllyDamageTolerance = 0.9      -- Takes 90% damage before turning
ENT.NeutralDamageTolerance = 0.5   -- Still somewhat patient with neutrals
```

**Result:**
- Ally shoots you: Forgives unless massive damage
- Neutral entity hits you: Tolerates moderate damage
- Enemy attacks: Immediate hostile response

### Example: Paranoid NPC

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Paranoid Stalker"

ENT.DefaultRelationship = D_NU
ENT.Factions = {}

-- Extremely sensitive to any damage
ENT.AllyDamageTolerance = 0.01     -- 1% damage = betrayal
ENT.NeutralDamageTolerance = 0.01  -- 1% damage = attack
ENT.AfraidDamageTolerance = 0.01   -- Fights back immediately when scared
```

**Result:**
- Any damage immediately turns entity hostile
- Perfect for jumpy, paranoid NPCs
- Creates tension in gameplay

### Example: Protective Ally

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Protective Guard"

ENT.DefaultRelationship = D_LI
ENT.Factions = {FACTION_REBELS}

-- Extremely loyal to allies
ENT.AllyDamageTolerance = 999      -- Never turns on allies
ENT.NeutralDamageTolerance = 0.1   -- Quick to defend against threats
```

**Result:**
- Will never attack allies, even if shot repeatedly
- Immediately retaliates against hostile neutral entities
- Perfect for bodyguard NPCs

### Damage Tolerance Scenarios

```lua
-- Example: Player damages a neutral NPC
-- NPC health: 100
-- NeutralDamageTolerance: 0.33 (33%)

-- Player shoots for 20 damage
-- 20 damage = 20% of health
-- 20% < 33% tolerance
-- Result: NPC still neutral

-- Player shoots for another 20 damage
-- Total: 40 damage = 40% of health
-- 40% > 33% tolerance
-- Result: NPC becomes hostile (D_HT)
```

---

## Dynamic Faction Changes

Change faction memberships and relationships at runtime based on game events.

### Joining Factions Dynamically

```lua
function ENT:CustomInitialize()
    -- Start with no faction
    self.Factions = {}

    -- Join faction after 5 seconds
    timer.Simple(5, function()
        if IsValid(self) then
            self:JoinFaction(FACTION_ZOMBIES)
            self:EmitSound("npc/zombie/zombie_alert.wav")
        end
    end)
end
```

### Event-Based Faction Changes

```lua
function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()

    -- If rebels attack us, leave rebel faction and join combine
    if IsValid(attacker) and attacker.IsDrGNextbot then
        if attacker:IsInFaction(FACTION_REBELS) then
            self:LeaveFaction(FACTION_REBELS)
            self:JoinFaction(FACTION_COMBINE)
            self:EmitSound("npc/combine_soldier/vo/alert1.wav")
        end
    end
end
```

### Health-Based Faction Changes

```lua
function ENT:CustomThink()
    local healthPercent = self:Health() / self:GetMaxHealth()

    if healthPercent < 0.25 and not self._hasBetrayed then
        -- Below 25% health: Betray allies and go rogue
        self:LeaveAllFactions()
        self:SetDefaultRelationship(D_HT)
        self._hasBetrayed = true

        self:EmitSound("npc/combine_soldier/vo/bouncerbouncer.wav")
    end
end
```

### Time-Based Faction Changes

```lua
function ENT:CustomInitialize()
    -- Start neutral
    self.DefaultRelationship = D_NU
    self.Factions = {}

    -- Every 30 seconds, randomly change faction
    timer.Create("FactionChange_" .. self:GetCreationID(), 30, 0, function()
        if not IsValid(self) then return end

        local factions = {
            FACTION_REBELS,
            FACTION_COMBINE,
            FACTION_ZOMBIES,
            FACTION_ANTLIONS
        }

        self:LeaveAllFactions()
        self:JoinFaction(factions[math.random(#factions)])

        print(self, "Changed to faction:", factions[math.random(#factions)])
    end)
end
```

### Updating NPC Relationships

After changing factions, update relationships:

```lua
-- Update relationships with all entities
self:UpdateRelationships()

-- Update relationship with specific entity
self:UpdateRelationshipWith(entity)

-- Update relationships for all NPCs toward this one
for _, npc in ipairs(DrGBase.GetNextbots()) do
    npc:UpdateRelationshipWith(self)
end
```

---

## Complex Scenarios

### Scenario 1: Three-Way Faction War

Create a complex battle between three factions:

```lua
-- Rebels
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Rebel Fighter"
ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_REBELS}

function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_COMBINE, D_HT)
    self:SetFactionRelationship(FACTION_ZOMBIES, D_HT)
end
```

```lua
-- Combine
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Combine Soldier"
ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_COMBINE}

function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_REBELS, D_HT)
    self:SetFactionRelationship(FACTION_ZOMBIES, D_HT)
end
```

```lua
-- Zombies
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Zombie"
ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_ZOMBIES}

function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_REBELS, D_HT)
    self:SetFactionRelationship(FACTION_COMBINE, D_HT)
end
```

**Result:** All three factions fight each other on sight.

### Scenario 2: Progressive Infection System

NPCs become infected and join the zombie faction:

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Infectible Citizen"

ENT.DefaultRelationship = D_LI
ENT.Factions = {FACTION_REBELS}
ENT.NeutralDamageTolerance = 0.5

function ENT:CustomInitialize()
    self._infected = false
    self._infectionTime = 0
end

function ENT:OnTakeDamage(dmg, hitgroup)
    local attacker = dmg:GetAttacker()

    -- Get infected by zombies
    if IsValid(attacker) and attacker.IsDrGNextbot then
        if attacker:IsInFaction(FACTION_ZOMBIES) and not self._infected then
            self._infected = true
            self._infectionTime = CurTime()

            self:EmitSound("npc/barnacle/barnacle_gulp2.wav")
        end
    end
end

function ENT:CustomThink()
    -- Turn into zombie after 10 seconds
    if self._infected and CurTime() > self._infectionTime + 10 then
        self:LeaveAllFactions()
        self:JoinFaction(FACTION_ZOMBIES)
        self:SetDefaultRelationship(D_HT)

        -- Change appearance
        self:SetColor(Color(100, 255, 100))
        self:EmitSound("npc/zombie/zombie_voice_idle1.wav")

        self._infected = false  -- Prevent re-triggering
    end
end
```

**Result:** Citizens become zombies when infected, turning on their former allies.

### Scenario 3: Faction Reputation System

Track player actions to determine faction standing:

```lua
-- In gamemode or autorun/server

-- Initialize reputation
local playerRep = {}

hook.Add("PlayerInitialSpawn", "InitReputation", function(ply)
    playerRep[ply] = {
        [FACTION_REBELS] = 0,
        [FACTION_COMBINE] = 0,
        [FACTION_ZOMBIES] = -100
    }
end)

-- Track NPC kills
hook.Add("OnNPCKilled", "ReputationKills", function(npc, attacker, inflictor)
    if not IsValid(attacker) or not attacker:IsPlayer() then return end
    if not npc.IsDrGNextbot then return end

    local rep = playerRep[attacker]
    if not rep then return end

    -- Lose reputation with NPC's factions
    for _, faction in ipairs(npc:GetFactions()) do
        rep[faction] = rep[faction] - 10
    end

    -- Check if player should be removed from factions
    for faction, score in pairs(rep) do
        if score < -50 then
            attacker:DrG_LeaveFaction(faction)
        elseif score > 50 then
            attacker:DrG_JoinFaction(faction)
        end
    end
end)

-- Track healing/helping
hook.Add("PlayerHealedNPC", "ReputationHealing", function(ply, npc, amount)
    if not npc.IsDrGNextbot then return end

    local rep = playerRep[ply]
    if not rep then return end

    -- Gain reputation with NPC's factions
    for _, faction in ipairs(npc:GetFactions()) do
        rep[faction] = rep[faction] + 5
    end
end)
```

### Scenario 4: Neutral Trader NPCs

NPCs that remain neutral unless attacked:

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Trader"

ENT.DefaultRelationship = D_NU
ENT.Factions = {}

-- Very high tolerance for accidental damage
ENT.NeutralDamageTolerance = 0.8
ENT.AllyDamageTolerance = 0.9

function ENT:CustomInitialize()
    self._totalDamageTaken = 0

    -- Ignore all factions
    self:SetFactionRelationship(FACTION_REBELS, D_NU)
    self:SetFactionRelationship(FACTION_COMBINE, D_NU)
    self:SetFactionRelationship(FACTION_ZOMBIES, D_NU)
end

function ENT:OnTakeDamage(dmg)
    self._totalDamageTaken = self._totalDamageTaken + dmg:GetDamage()

    -- Warn player before becoming hostile
    local attacker = dmg:GetAttacker()
    if IsValid(attacker) and attacker:IsPlayer() then
        local healthPercent = self._totalDamageTaken / self:GetMaxHealth()

        if healthPercent > 0.3 and healthPercent < 0.4 then
            attacker:ChatPrint("The trader warns you to stop!")
        elseif healthPercent > 0.7 then
            attacker:ChatPrint("The trader becomes hostile!")
        end
    end
end
```

### Scenario 5: Pack Behavior with Faction Awareness

NPCs coordinate with faction members:

```lua
function ENT:CustomThink()
    local allies = self:GetAllies()

    -- Check if any allies have enemies
    for ally in self:AllyIterator() do
        local allyEnemy = ally:GetEnemy()

        -- If ally is fighting, help them
        if IsValid(allyEnemy) and not self:HasEnemy() then
            if self:Visible(allyEnemy) then
                self:SetEnemy(allyEnemy)
                self:EmitSound("npc/combine_soldier/vo/cover.wav")
                break
            end
        end
    end
end

function ENT:OnNewEnemy(enemy)
    -- Alert nearby faction members
    for ally in self:AllyIterator() do
        if self:GetRangeTo(ally) < 500 then
            -- Ally becomes aware of enemy
            ally:SetEnemy(enemy)
            ally:EmitSound("npc/combine_soldier/vo/confirmed.wav")
        end
    end
end
```

---

## Advanced Techniques

### Custom Relationship Resolution

Override default relationship logic:

```lua
function ENT:CustomRelationship(ent)
    -- Return disposition and priority

    -- Example: Hate entities holding crowbars
    if IsValid(ent) and ent:IsPlayer() then
        local wep = ent:GetActiveWeapon()
        if IsValid(wep) and wep:GetClass() == "weapon_crowbar" then
            return D_HT, 100  -- High priority hate
        end
    end

    -- Example: Like entities with low health (prey)
    if IsValid(ent) and ent:Health() < 20 then
        return D_LI, 50
    end

    -- Return nil to use default logic
    return nil, nil
end
```

### Conditional Entity Ignoring

Ignore entities based on custom conditions:

```lua
function ENT:ShouldIgnore(ent)
    -- Ignore entities that are too far away
    if self:GetRangeTo(ent) > 2000 then
        return true
    end

    -- Ignore entities behind walls
    if not self:Visible(ent) then
        return true
    end

    -- Ignore friendly fire targets
    if ent:IsPlayer() and ent:GetNW2Bool("IsFriendly") then
        return true
    end

    return false
end
```

### Relationship Change Notifications

Hook into relationship changes:

```lua
function ENT:OnRelationshipChange(ent, oldDisp, newDisp)
    -- Called when relationship changes

    if oldDisp == D_LI and newDisp == D_HT then
        -- Former ally became enemy
        self:EmitSound("npc/metropolice/vo/betrayed.wav")
        print(self, "was betrayed by", ent)
    end

    if oldDisp == D_HT and newDisp == D_LI then
        -- Enemy became friend
        self:EmitSound("npc/alyx/al_yes01.wav")
        print(self, "made peace with", ent)
    end
end
```

### Using Teams Instead of Factions

Use Source Engine teams for relationships:

```lua
function ENT:CustomInitialize()
    -- Set team
    self:SetTeam(TEAM_COMBINE)
end

-- NPCs on the same team are automatically allies
-- Players on same team as NPC are allies
```

### Frightening NPCs

Make NPCs scary to others:

```lua
ENT.Frightening = true  -- Other NPCs fear this NPC

function ENT:CustomInitialize()
    -- When Frightening = true:
    -- - NPCs with D_NU relationship become D_FR (fear)
    -- - NPCs with D_LI relationship may become D_FR
    -- - Creates intimidating boss-like NPCs
end

-- Change frightening status at runtime
self:SetFrightening(true)   -- Become scary
self:SetFrightening(false)  -- No longer scary
```

### Class and Model Relationships

Set relationships based on class or model:

```lua
function ENT:CustomInitialize()
    -- Class relationships
    self:SetClassRelationship("npc_zombie", D_HT)
    self:SetClassRelationship("player", D_LI)

    -- Model relationships
    self:SetModelRelationship("models/zombie.mdl", D_HT)

    -- Self-class relationship (hate others of same type)
    self:SetSelfClassRelationship(D_HT)

    -- Self-model relationship
    self:SetSelfModelRelationship(D_HT)

    -- Player relationship shortcut
    self:SetPlayersRelationship(D_LI)  -- Like all players
end
```

### Querying Entities by Relationship

Efficiently find entities by relationship:

```lua
function ENT:CustomThink()
    -- Iterate over enemies
    for enemy in self:EnemyIterator() do
        print("Enemy:", enemy)
    end

    -- Iterate over spotted enemies only
    for enemy in self:EnemyIterator(true) do
        print("Spotted enemy:", enemy)
    end

    -- Get all allies
    local allies = self:GetAllies()

    -- Get spotted hostiles
    local hostiles = self:GetHostiles(true)

    -- Get closest enemy
    local closest = self:GetClosestEnemy()

    -- Count remaining enemies
    local count = self:EnemiesLeft()
end
```

### Debug Relationship Information

```lua
-- Enable relationship debugging
-- Console: drgbase_debug_relationships 1

function ENT:CustomThink()
    -- Print relationship info for debugging
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        local disp = self:GetRelationship(enemy)
        local prio = self:GetPriority(enemy)

        print("Enemy:", enemy)
        print("Disposition:", disp)
        print("Priority:", prio)
        print("IsEnemy:", self:IsEnemy(enemy))
        print("IsAlly:", self:IsAlly(enemy))
    end
end
```

---

## Best Practices

### 1. Plan Your Faction Structure

Before coding, map out faction relationships:

```
FACTION_REBELS
  ├─ Allies: FACTION_ANIMALS, Players
  ├─ Enemies: FACTION_COMBINE, FACTION_ZOMBIES
  └─ Neutral: FACTION_ANTLIONS

FACTION_COMBINE
  ├─ Allies: None
  ├─ Enemies: FACTION_REBELS, FACTION_ZOMBIES, Players
  └─ Neutral: FACTION_ANIMALS
```

### 2. Use Consistent Naming

```lua
-- Good: Clear, consistent naming
FACTION_CUSTOM_MILITARY = "FACTION_CUSTOM_MILITARY"
FACTION_CUSTOM_BANDITS = "FACTION_CUSTOM_BANDITS"

-- Bad: Inconsistent, unclear naming
MY_FACTION = "somefaction"
faction2 = "ANOTHER_ONE"
```

### 3. Set Default Relationship Appropriately

```lua
-- Hostile NPC: Default D_HT, then add ally exceptions
ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_COMBINE}

-- Friendly NPC: Default D_LI, then add enemy exceptions
ENT.DefaultRelationship = D_LI
ENT.Factions = {FACTION_REBELS}

-- Passive NPC: Default D_NU, let damage tolerance handle threats
ENT.DefaultRelationship = D_NU
ENT.NeutralDamageTolerance = 0.3
```

### 4. Balance Damage Tolerance

```lua
-- Combat NPC: Low tolerance, quick to fight
ENT.NeutralDamageTolerance = 0.1
ENT.AllyDamageTolerance = 0.2

-- Civilian NPC: High tolerance, avoids conflict
ENT.NeutralDamageTolerance = 0.8
ENT.AllyDamageTolerance = 0.9

-- Companion NPC: Infinite ally tolerance
ENT.AllyDamageTolerance = 999
ENT.NeutralDamageTolerance = 0.2
```

### 5. Update Relationships After Changes

```lua
-- Always update after changing factions
self:JoinFaction(FACTION_ZOMBIES)
self:UpdateRelationships()

-- Update other NPCs' relationships toward this one
for _, npc in ipairs(DrGBase.GetNextbots()) do
    npc:UpdateRelationshipWith(self)
end
```

### 6. Document Custom Factions

```lua
--[[
    Custom Faction Structure:

    FACTION_CUSTOM_WILDLIFE
    - Passive creatures
    - Neutral to everyone
    - Flees when attacked

    FACTION_CUSTOM_BANDITS
    - Hostile to players and rebels
    - Allies with each other
    - Attacks on sight
]]--

FACTION_CUSTOM_WILDLIFE = "FACTION_CUSTOM_WILDLIFE"
FACTION_CUSTOM_BANDITS = "FACTION_CUSTOM_BANDITS"
```

### 7. Test Faction Interactions

```lua
-- Spawn test NPCs to verify relationships
concommand.Add("test_factions", function()
    -- Spawn rebel
    local rebel = ents.Create("npc_drg_rebel")
    rebel:SetPos(Vector(0, 0, 100))
    rebel:Spawn()

    -- Spawn combine
    local combine = ents.Create("npc_drg_combine")
    combine:SetPos(Vector(100, 0, 100))
    combine:Spawn()

    -- Spawn zombie
    local zombie = ents.Create("npc_drg_zombie")
    zombie:SetPos(Vector(50, 100, 100))
    zombie:Spawn()

    -- Verify they fight each other
    print("Rebel vs Combine:", rebel:GetRelationship(combine))
    print("Rebel vs Zombie:", rebel:GetRelationship(zombie))
    print("Combine vs Zombie:", combine:GetRelationship(zombie))
end)
```

---

## Troubleshooting

### NPCs Don't Attack Each Other

**Problem:** NPCs with different factions remain neutral.

**Solution:** Set `DefaultRelationship = D_HT` and configure faction relationships:

```lua
ENT.DefaultRelationship = D_HT
function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_ENEMY, D_HT)
end
```

### NPCs Attack Same Faction

**Problem:** NPCs in same faction fight each other.

**Solution:** Verify faction names match exactly:

```lua
-- Both must use same faction constant
ENT.Factions = {FACTION_ZOMBIES}  -- Not "zombies" or "ZOMBIES"
```

### Relationships Don't Update

**Problem:** Changing factions doesn't affect relationships.

**Solution:** Call update functions:

```lua
self:JoinFaction(FACTION_REBELS)
self:UpdateRelationships()  -- Update this NPC's relationships

-- Update all NPCs' relationships toward this one
for _, npc in ipairs(DrGBase.GetNextbots()) do
    npc:UpdateRelationshipWith(self)
end
```

### Priority Conflicts

**Problem:** Wrong relationship wins in priority conflicts.

**Solution:** Check priority values and hierarchy:

```lua
-- Higher priority wins
self:SetEntityRelationship(ent, D_HT, 100)  -- This wins
self:SetClassRelationship("player", D_LI, 10)  -- This loses
```

### Player Factions Don't Work

**Problem:** Player joins faction but NPCs don't react.

**Solution:** Update NPC relationships after player joins:

```lua
ply:DrG_JoinFaction(FACTION_REBELS)

-- Update all NPC relationships
for _, npc in ipairs(DrGBase.GetNextbots()) do
    npc:UpdateRelationshipWith(ply)
end
```

---

## See Also

- **[API: Relationships](../api/nextbot/relationships.md)** - Complete API reference
- **[Getting Started: Relationships & Factions](../getting-started/04-relationships-factions.md)** - Quick start guide
- **[API: Enumerations](../api/enumerations.md)** - Faction constants and disposition values
- **[Systems: Relationships](../systems/relationships/README.md)** - System architecture

---

**Next Steps:**
1. Experiment with different faction combinations
2. Create custom faction systems for your gamemode
3. Build complex AI behaviors based on relationships
4. Test three-way faction wars and dynamic alliances
