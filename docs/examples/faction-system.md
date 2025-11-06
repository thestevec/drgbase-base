# Faction System Example

A comprehensive guide to creating custom factions, managing complex relationships, and coordinating multiple NPC types.

## Table of Contents

- [Overview](#overview)
- [Example 1: Custom Faction](#example-1-custom-faction)
- [Example 2: Multi-Faction System](#example-2-multi-faction-system)
- [Example 3: Dynamic Faction Changes](#example-3-dynamic-faction-changes)
- [Code Breakdown](#code-breakdown)
- [Testing Factions](#testing-factions)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)
- [See Also](#see-also)

## Overview

DrGBase provides a powerful faction system that allows you to:
- Create custom factions for your NPCs
- Define complex relationship networks between factions
- Allow NPCs to join/leave factions dynamically
- Create alliance systems and faction warfare
- Implement player faction membership

This guide demonstrates how to build complete faction systems with multiple NPC types working together.

**Difficulty:** Intermediate
**Concepts Covered:** Factions, relationships, alliances, dynamic changes

## Example 1: Custom Faction

Creating a simple custom faction with NPCs that work together.

### Step 1: Define Custom Factions

Create a file named `autorun/my_factions.lua` in `garrysmod/lua/`:

```lua
-- Define custom factions
FACTION_BANDITS = "FACTION_CUSTOM_BANDITS"
FACTION_GUARDS = "FACTION_CUSTOM_GUARDS"
FACTION_WILDLIFE = "FACTION_CUSTOM_WILDLIFE"

-- Set up faction relationships (called after DrGBase loads)
hook.Add("DrGBaseInitialized", "SetupCustomFactions", function()
	-- Bandits hate everyone
	DrGBase.SetFactionRelationship(FACTION_BANDITS, FACTION_GUARDS, D_HT)
	DrGBase.SetFactionRelationship(FACTION_BANDITS, FACTION_PLAYERS, D_HT)
	DrGBase.SetFactionRelationship(FACTION_BANDITS, FACTION_REBELS, D_HT)

	-- Guards protect civilians
	DrGBase.SetFactionRelationship(FACTION_GUARDS, FACTION_BANDITS, D_HT)
	DrGBase.SetFactionRelationship(FACTION_GUARDS, FACTION_PLAYERS, D_LI)
	DrGBase.SetFactionRelationship(FACTION_GUARDS, FACTION_REBELS, D_LI)

	-- Wildlife is neutral to everyone except bandits
	DrGBase.SetFactionRelationship(FACTION_WILDLIFE, FACTION_BANDITS, D_HT)
	DrGBase.SetFactionRelationship(FACTION_WILDLIFE, FACTION_GUARDS, D_NU)
	DrGBase.SetFactionRelationship(FACTION_WILDLIFE, FACTION_PLAYERS, D_NU)

	print("Custom factions initialized!")
end)
```

### Step 2: Create Faction NPCs

Create `npc_drg_bandit.lua`:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Bandit"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/phoenix.mdl"}
ENT.SpawnHealth = 80
ENT.Weapons = {"weapon_drg_ar2"}

-- Join bandit faction
ENT.Factions = {FACTION_BANDITS}

if SERVER then
	function ENT:CustomInitialize()
		-- Hostile to everyone by default (inherited from faction)
		self:SetDefaultRelationship(D_HT)
	end

	function ENT:OnNewEnemy()
		self:EmitSound("vo/npc/male01/pain0" .. math.random(1, 9) .. ".wav")
	end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

Create `npc_drg_guard.lua`:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Guard"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/urban.mdl"}
ENT.SpawnHealth = 100
ENT.Weapons = {"weapon_drg_ar2"}

-- Join guard faction
ENT.Factions = {FACTION_GUARDS}

if SERVER then
	function ENT:CustomInitialize()
		-- Friendly to players (inherited from faction)
		self:SetDefaultRelationship(D_NU)
	end

	function ENT:OnNewEnemy()
		self:EmitSound("npc/metropolice/vo/chuckle.wav")
	end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Example 2: Multi-Faction System

A complete three-way faction war with alliances and betrayals.

### Complete Code

Create `autorun/faction_war.lua`:

```lua
-- Define three warring factions
FACTION_EMPIRE = "FACTION_CUSTOM_EMPIRE"
FACTION_REBELS = "FACTION_CUSTOM_REBELS"
FACTION_MERCENARIES = "FACTION_CUSTOM_MERCENARIES"

hook.Add("DrGBaseInitialized", "SetupFactionWar", function()
	-- Empire vs Rebels (eternal enemies)
	DrGBase.SetFactionRelationship(FACTION_EMPIRE, FACTION_REBELS, D_HT)
	DrGBase.SetFactionRelationship(FACTION_REBELS, FACTION_EMPIRE, D_HT)

	-- Mercenaries initially neutral
	DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_EMPIRE, D_NU)
	DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_REBELS, D_NU)
	DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_PLAYERS, D_NU)

	-- Empire friendly to players initially
	DrGBase.SetFactionRelationship(FACTION_EMPIRE, FACTION_PLAYERS, D_LI)

	-- Rebels friendly to players initially
	DrGBase.SetFactionRelationship(FACTION_REBELS, FACTION_PLAYERS, D_LI)

	print("Faction War system initialized!")
end)

-- Track faction scores
if SERVER then
	FACTION_SCORES = FACTION_SCORES or {
		[FACTION_EMPIRE] = 0,
		[FACTION_REBELS] = 0,
		[FACTION_MERCENARIES] = 0
	}

	-- When faction member dies, adjust relationships
	hook.Add("OnNPCKilled", "FactionWarDeath", function(npc, attacker, inflictor)
		if not npc.Factions then return end

		-- Get victim's faction
		local victimFaction = npc.Factions[1]
		if not victimFaction then return end

		-- Decrease faction score
		FACTION_SCORES[victimFaction] = (FACTION_SCORES[victimFaction] or 0) - 1

		-- If mercenaries see who killed their neutral target, they may take sides
		if victimFaction == FACTION_MERCENARIES then
			if IsValid(attacker) and attacker.Factions then
				local attackerFaction = attacker.Factions[1]

				-- Mercenaries now hate the attacker's faction
				DrGBase.SetFactionRelationship(FACTION_MERCENARIES, attackerFaction, D_HT)

				PrintMessage(HUD_PRINTTALK, "Mercenaries are now hostile to " .. attackerFaction)
			end
		end

		-- Check for faction dominance
		if FACTION_SCORES[FACTION_EMPIRE] < -10 then
			-- Empire is losing badly, mercenaries join rebels
			DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_REBELS, D_LI)
			DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_EMPIRE, D_HT)
			PrintMessage(HUD_PRINTTALK, "Mercenaries have joined the Rebels!")
			FACTION_SCORES[FACTION_EMPIRE] = 0  -- Reset
		elseif FACTION_SCORES[FACTION_REBELS] < -10 then
			-- Rebels are losing badly, mercenaries join empire
			DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_EMPIRE, D_LI)
			DrGBase.SetFactionRelationship(FACTION_MERCENARIES, FACTION_REBELS, D_HT)
			PrintMessage(HUD_PRINTTALK, "Mercenaries have joined the Empire!")
			FACTION_SCORES[FACTION_REBELS] = 0  -- Reset
		end
	end)

	-- Console command to check faction scores
	concommand.Add("faction_scores", function(ply)
		print("=== Faction Scores ===")
		for faction, score in pairs(FACTION_SCORES) do
			print(faction .. ": " .. score)
		end
	end)
end
```

### Create NPCs for Each Faction

Empire Soldier (`npc_drg_empire_soldier.lua`):

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Empire Soldier"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/combine_soldier.mdl"}
ENT.SpawnHealth = 100
ENT.Weapons = {"weapon_drg_ar2"}
ENT.Factions = {FACTION_EMPIRE}

if SERVER then
	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_NU)
	end

	function ENT:OnDeath(dmg)
		-- Alert nearby allies
		for _, ent in pairs(ents.FindInSphere(self:GetPos(), 500)) do
			if IsValid(ent) and ent:IsNPC() and ent:IsInFaction(FACTION_EMPIRE) then
				if IsValid(dmg:GetAttacker()) then
					ent:AddEntityRelationship(dmg:GetAttacker(), D_HT, 99)
				end
			end
		end
	end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

Rebel Fighter (`npc_drg_rebel_fighter.lua`):

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Rebel Fighter"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/group03/male_04.mdl"}
ENT.SpawnHealth = 80
ENT.Weapons = {"weapon_drg_ar2"}
ENT.Factions = {FACTION_REBELS}

if SERVER then
	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_NU)
	end

	function ENT:OnNewEnemy()
		-- Call for backup
		for _, ally in pairs(ents.FindInSphere(self:GetPos(), 800)) do
			if IsValid(ally) and ally:IsNPC() and ally:IsInFaction(FACTION_REBELS) then
				if IsValid(self:GetEnemy()) then
					ally:AddEntityRelationship(self:GetEnemy(), D_HT, 99)
				end
			end
		end
	end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

Mercenary (`npc_drg_mercenary.lua`):

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Mercenary"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/riot.mdl"}
ENT.SpawnHealth = 120
ENT.SpawnArmor = 50
ENT.Weapons = {"weapon_drg_ar2"}
ENT.Factions = {FACTION_MERCENARIES}

if SERVER then
	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_NU)  -- Neutral by default
	end

	function ENT:OnTakeDamage(dmg)
		-- Mercenaries remember who attacks them
		local attacker = dmg:GetAttacker()
		if IsValid(attacker) then
			self:AddEntityRelationship(attacker, D_HT, 99)

			-- Share info with nearby mercenaries
			for _, ally in pairs(ents.FindInSphere(self:GetPos(), 500)) do
				if IsValid(ally) and ally:IsNPC() and ally:IsInFaction(FACTION_MERCENARIES) then
					ally:AddEntityRelationship(attacker, D_HT, 99)
				end
			end
		end
	end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Example 3: Dynamic Faction Changes

NPCs that change factions based on conditions.

### Complete Code

Defecting Soldier (`npc_drg_defector.lua`):

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Defector"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/combine_soldier_prisonguard.mdl"}
ENT.SpawnHealth = 100
ENT.Weapons = {"weapon_drg_ar2"}

-- Start as Empire soldier
ENT.Factions = {FACTION_EMPIRE}

if SERVER then
	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_NU)
		self.DefectionThreshold = 30  -- Defect when health drops below this
		self.HasDefected = false
	end

	function ENT:CustomThink()
		-- Check if should defect
		if not self.HasDefected and self:Health() < self.DefectionThreshold then
			self:DefectToRebels()
		end
	end

	function ENT:DefectToRebels()
		if self.HasDefected then return end
		self.HasDefected = true

		-- Leave empire faction
		self:LeaveFaction(FACTION_EMPIRE)

		-- Join rebel faction
		self:JoinFaction(FACTION_REBELS)

		-- Update relationships
		self:SetFactionRelationship(FACTION_EMPIRE, D_HT)
		self:SetFactionRelationship(FACTION_REBELS, D_LI)
		self:SetFactionRelationship(FACTION_PLAYERS, D_LI)

		-- Visual and audio feedback
		self:EmitSound("vo/npc/male01/yeah02.wav")

		-- Effect
		local effectData = EffectData()
		effectData:SetOrigin(self:GetPos() + Vector(0, 0, 40))
		effectData:SetMagnitude(1)
		util.Effect("ManhackSparks", effectData)

		-- Change model
		self:SetModel("models/player/group03/male_07.mdl")

		-- Alert nearby rebels
		for _, rebel in pairs(ents.FindInSphere(self:GetPos(), 1000)) do
			if IsValid(rebel) and rebel:IsNPC() and rebel:IsInFaction(FACTION_REBELS) then
				-- Rebels now friendly to defector
				rebel:AddEntityRelationship(self, D_LI, 99)
			end
		end

		-- Alert nearby empire soldiers
		for _, soldier in pairs(ents.FindInSphere(self:GetPos(), 1000)) do
			if IsValid(soldier) and soldier:IsNPC() and soldier:IsInFaction(FACTION_EMPIRE) then
				-- Empire now hates defector
				soldier:AddEntityRelationship(self, D_HT, 99)
			end
		end

		PrintMessage(HUD_PRINTTALK, "A soldier has defected to the Rebels!")
	end

	function ENT:OnDeath(dmg, hitgroup)
		if not self.HasDefected then
			-- Died before defecting, standard death
			return
		end

		-- Special death message for defectors
		PrintMessage(HUD_PRINTTALK, "A defector has been killed!")
	end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### Defining Custom Factions

```lua
FACTION_BANDITS = "FACTION_CUSTOM_BANDITS"
FACTION_GUARDS = "FACTION_CUSTOM_GUARDS"
```

**Purpose:** Create unique faction identifiers.
- Use descriptive names prefixed with `FACTION_CUSTOM_`
- Store as global variables for easy access
- Must be unique strings

### Setting Faction Relationships

```lua
hook.Add("DrGBaseInitialized", "SetupCustomFactions", function()
	DrGBase.SetFactionRelationship(FACTION_BANDITS, FACTION_GUARDS, D_HT)
	DrGBase.SetFactionRelationship(FACTION_GUARDS, FACTION_PLAYERS, D_LI)
end)
```

**Purpose:** Define how factions interact.
- Use `DrGBaseInitialized` hook to ensure DrGBase is loaded
- `D_HT` = Hate (hostile)
- `D_LI` = Like (friendly)
- `D_NU` = Neutral
- `D_FR` = Fear

### NPC Faction Membership

```lua
ENT.Factions = {FACTION_BANDITS}
```

**Purpose:** Assign NPC to faction at spawn.
- NPCs can belong to multiple factions: `{FACTION_A, FACTION_B}`
- Inherits faction relationships automatically
- Can be changed at runtime with `:JoinFaction()` and `:LeaveFaction()`

### Dynamic Faction Changes

```lua
function ENT:DefectToRebels()
	-- Leave old faction
	self:LeaveFaction(FACTION_EMPIRE)

	-- Join new faction
	self:JoinFaction(FACTION_REBELS)

	-- Update relationships
	self:SetFactionRelationship(FACTION_EMPIRE, D_HT)
	self:SetFactionRelationship(FACTION_REBELS, D_LI)
end
```

**Purpose:** Change NPC's faction at runtime.
- Remove from old faction
- Add to new faction
- Update relationships accordingly
- Notify nearby NPCs of change

### Faction Score System

```lua
FACTION_SCORES = {
	[FACTION_EMPIRE] = 0,
	[FACTION_REBELS] = 0
}

hook.Add("OnNPCKilled", "FactionWarDeath", function(npc, attacker)
	local victimFaction = npc.Factions[1]
	FACTION_SCORES[victimFaction] = FACTION_SCORES[victimFaction] - 1

	-- Check for faction dominance
	if FACTION_SCORES[FACTION_EMPIRE] < -10 then
		-- Empire losing, trigger events
	end
end)
```

**Purpose:** Track faction performance.
- Decrease score when faction members die
- Trigger events based on score thresholds
- Can influence faction relationships dynamically

### Squad Coordination

```lua
function ENT:OnNewEnemy()
	-- Alert nearby allies
	for _, ally in pairs(ents.FindInSphere(self:GetPos(), 800)) do
		if IsValid(ally) and ally:IsNPC() and ally:IsInFaction(FACTION_REBELS) then
			if IsValid(self:GetEnemy()) then
				ally:AddEntityRelationship(self:GetEnemy(), D_HT, 99)
			end
		end
	end
end
```

**Purpose:** Coordinate faction members.
- Find allies within range
- Share enemy information
- Create squad cohesion
- Improves tactical effectiveness

## Testing Factions

### Basic Testing

```lua
-- Spawn NPCs from different factions
local bandit = ents.Create("npc_drg_bandit")
bandit:SetPos(Vector(0, 0, 0))
bandit:Spawn()

local guard = ents.Create("npc_drg_guard")
guard:SetPos(Vector(500, 0, 0))
guard:Spawn()

-- They should automatically be hostile to each other
```

### Testing Relationships

```lua
-- Check faction relationship
local npc = Entity(X)  -- Replace X with entity index
print("Factions:", npc:GetFactions())
print("Disposition to player:", npc:Disposition(Entity(1)))

-- Change relationship
npc:SetFactionRelationship(FACTION_PLAYERS, D_HT)  -- Now hates players
```

### Testing Faction Changes

```lua
-- Force an NPC to change factions
local npc = Entity(X)
npc:LeaveFaction(FACTION_EMPIRE)
npc:JoinFaction(FACTION_REBELS)
```

### Testing Squad Behavior

```lua
-- Spawn a squad of same faction
for i = 1, 5 do
	local npc = ents.Create("npc_drg_rebel_fighter")
	npc:SetPos(Entity(1):GetPos() + Vector(i*100, 0, 0))
	npc:Spawn()
end

-- Spawn an enemy and watch them coordinate
local enemy = ents.Create("npc_drg_empire_soldier")
enemy:SetPos(Entity(1):GetPos() + Vector(500, 0, 0))
enemy:Spawn()
```

## Customization Ideas

### Reputation System

```lua
-- Track player reputation with each faction
PLAYER_FACTION_REP = {}

hook.Add("OnNPCKilled", "FactionReputation", function(npc, attacker)
	if not attacker:IsPlayer() or not npc.Factions then return end

	local victimFaction = npc.Factions[1]

	-- Initialize reputation table
	PLAYER_FACTION_REP[attacker] = PLAYER_FACTION_REP[attacker] or {}
	PLAYER_FACTION_REP[attacker][victimFaction] = (PLAYER_FACTION_REP[attacker][victimFaction] or 0) - 10

	-- Check reputation thresholds
	if PLAYER_FACTION_REP[attacker][victimFaction] < -50 then
		-- Player is now hostile to this faction
		attacker:DrG_LeaveFaction(victimFaction)
		PrintMessage(HUD_PRINTTALK, "You are now hostile to " .. victimFaction)
	end
end)
```

### Alliance System

```lua
-- Create formal alliances between factions
FACTION_ALLIANCES = {
	[FACTION_REBELS] = {FACTION_GUARDS, FACTION_WILDLIFE},
	[FACTION_GUARDS] = {FACTION_REBELS},
}

function AreFactionAllies(faction1, faction2)
	local allies = FACTION_ALLIANCES[faction1]
	if allies and table.HasValue(allies, faction2) then
		return true
	end
	return false
end
```

### Faction Resources

```lua
-- Factions can have resources
FACTION_RESOURCES = {
	[FACTION_EMPIRE] = {
		credits = 1000,
		soldiers = 10,
		territory = 5
	},
	[FACTION_REBELS] = {
		credits = 500,
		soldiers = 15,
		territory = 3
	}
}

-- NPCs can be more/less aggressive based on resources
function ENT:CustomThink()
	local resources = FACTION_RESOURCES[self.Factions[1]]
	if resources.soldiers < 5 then
		-- Faction is weak, be more defensive
		self.AvoidEnemyRange = 200
	end
end
```

### Territory Control

```lua
-- Define territories
FACTION_TERRITORIES = {}

-- Claim territory
function ClaimTerritory(pos, radius, faction)
	table.insert(FACTION_TERRITORIES, {
		pos = pos,
		radius = radius,
		faction = faction
	})
end

-- Check if position is in faction territory
function GetTerritoryFaction(pos)
	for _, territory in pairs(FACTION_TERRITORIES) do
		if pos:Distance(territory.pos) < territory.radius then
			return territory.faction
		end
	end
	return nil
end

-- NPCs stronger in their territory
function ENT:CustomInitialize()
	local territory = GetTerritoryFaction(self:GetPos())
	if territory == self.Factions[1] then
		-- In friendly territory, boost stats
		self:SetHealth(self:Health() * 1.5)
	end
end
```

### Faction Victory Conditions

```lua
-- Check for faction victory
hook.Add("Think", "CheckFactionVictory", function()
	-- Count living members of each faction
	local factionCounts = {}

	for _, npc in pairs(ents.GetAll()) do
		if IsValid(npc) and npc:IsNPC() and npc.Factions then
			local faction = npc.Factions[1]
			factionCounts[faction] = (factionCounts[faction] or 0) + 1
		end
	end

	-- Check for victory
	local livingFactions = 0
	local winnerFaction = nil

	for faction, count in pairs(factionCounts) do
		if count > 0 then
			livingFactions = livingFactions + 1
			winnerFaction = faction
		end
	end

	if livingFactions == 1 then
		PrintMessage(HUD_PRINTCENTER, winnerFaction .. " has won!")
		-- Reset or end game
	end
end)
```

## Troubleshooting

### Factions not hostile to each other

**Problem:** Relationship not set correctly.

**Solutions:**
- Check `DrGBaseInitialized` hook is firing
- Verify `DrGBase.SetFactionRelationship()` is called
- Make sure faction names match exactly
- Test with: `print(npc:Disposition(otherNpc))`
- Should return `D_HT` for hostile, `D_LI` for friendly

### NPCs attacking allies

**Problem:** Faction membership issue.

**Solutions:**
- Check NPC has correct `ENT.Factions` set
- Verify `npc:IsInFaction(FACTION_X)` returns true
- Make sure faction relationships are mutual
- Check for conflicting entity relationships

### Faction changes don't work

**Problem:** Dynamic faction change not updating.

**Solutions:**
- Verify `:LeaveFaction()` and `:JoinFaction()` are called
- Update relationships after changing: `:SetFactionRelationship()`
- Notify nearby NPCs manually
- Check faction variable is correct string

### Squad coordination not working

**Problem:** NPCs not sharing enemy information.

**Solutions:**
- Check `ents.FindInSphere()` range is appropriate
- Verify NPCs share same faction
- Make sure enemy is being passed correctly
- Test with print statements: `print("Alerting allies")`

### Performance issues with many factions

**Problem:** Too many relationship checks.

**Solutions:**
- Limit number of active factions
- Use cooldowns for expensive operations
- Cache faction lookups
- Reduce squad communication range

## See Also

- [Simple Melee NPC Example](simple-melee-npc.md) - Basic NPC creation
- [Advanced AI Example](advanced-ai.md) - Squad coordination
- [Boss NPC Example](boss-npc.md) - Complex NPC behaviors
- [Creating NPCs Guide](../guides/creating-npcs.md) - Complete NPC tutorial
- [Faction Guide](../guides/factions.md) - Comprehensive faction system guide
- [Relationship API](../api/relationships.md) - Relationship system reference
