# Advanced AI Behaviors Example

A complete guide to implementing sophisticated AI behaviors like cover systems, flanking, tactical retreats, and squad coordination.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the AI](#testing-the-ai)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)
- [See Also](#see-also)

## Overview

This example creates an intelligent tactical soldier NPC that demonstrates advanced AI behaviors:
- **Cover System**: Seeks and uses cover when under fire
- **Flanking**: Attempts to flank enemies instead of direct approaches
- **Tactical Retreat**: Retreats to cover when health is low
- **Squad Communication**: Calls for backup and coordinates with allies
- **Threat Assessment**: Prioritizes dangerous targets
- **Ammunition Management**: Retreats when running low on ammo
- **Dynamic Decision Making**: Chooses behavior based on situation

**Difficulty:** Advanced
**File Location:** `lua/entities/npc_drg_tactical_soldier.lua`
**Base:** `drgbase_nextbot`

## Complete Code

Create a file named `npc_drg_tactical_soldier.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot" -- DO NOT TOUCH (obviously)

-- Misc --
ENT.PrintName = "Tactical Soldier"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/combine_soldier.mdl"}
ENT.BloodColor = BLOOD_COLOR_RED

-- Sounds --
ENT.OnDamageSounds = {"npc/combine_soldier/pain1.wav", "npc/combine_soldier/pain2.wav", "npc/combine_soldier/pain3.wav"}
ENT.OnDeathSounds = {"npc/combine_soldier/die1.wav", "npc/combine_soldier/die2.wav"}

-- Stats --
ENT.SpawnHealth = 100
ENT.ArmorEnabled = true
ENT.SpawnArmor = 50

-- AI --
ENT.RangeAttackRange = 1500
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 800
ENT.AvoidEnemyRange = 0

-- Relationships --
ENT.Factions = {FACTION_COMBINE}

-- Weapons --
ENT.Weapons = {"weapon_drg_ar2"}

-- Movements/animations --
ENT.WalkSpeed = 60
ENT.RunSpeed = 150

-- Detection --
ENT.EyeBone = "ValveBiped.Bip01_Head1"
ENT.EyeOffset = Vector(5, 0, 0)
ENT.SightFOV = 100
ENT.SightRange = 4000

-- Advanced AI Properties --
ENT.CoverSearchRange = 1000  -- How far to search for cover
ENT.CoverHealthThreshold = 0.5  -- Seek cover when below 50% health
ENT.RetreatHealthThreshold = 0.3  -- Retreat when below 30% health
ENT.FlankDistance = 300  -- How wide to flank around enemy
ENT.BackupCallRange = 500  -- Range to call allies for backup

if SERVER then

	-- Init/Think --

	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_HT)

		-- AI state variables
		self.InCover = false
		self.CoverPosition = nil
		self.LastCoverSearch = 0
		self.FlankPosition = nil
		self.IsRetreating = false
		self.BackupCalled = false
		self.LastThreatAssessment = 0
		self.PreferredTarget = nil
	end

	function ENT:CustomThink()
		-- Periodic threat assessment
		if CurTime() > self.LastThreatAssessment + 2 then
			self:AssessThreat()
			self.LastThreatAssessment = CurTime()
		end

		-- Check if we should stop retreating
		if self.IsRetreating then
			local healthPercent = self:Health() / self:GetMaxHealth()
			if healthPercent > self.CoverHealthThreshold then
				self.IsRetreating = false
			end
		end
	end

	-- Advanced AI Decision Making --

	function ENT:AssessThreat()
		-- Evaluate all nearby threats and prioritize
		local threats = {}

		for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.SightRange)) do
			if IsValid(ent) and (ent:IsPlayer() or ent:IsNPC()) then
				if self:Disposition(ent) == D_HT then
					local threat = {
						entity = ent,
						score = self:CalculateThreatScore(ent)
					}
					table.insert(threats, threat)
				end
			end
		end

		-- Sort by threat score (highest first)
		table.sort(threats, function(a, b) return a.score > b.score end)

		-- Set preferred target to highest threat
		if #threats > 0 then
			self.PreferredTarget = threats[1].entity
		else
			self.PreferredTarget = nil
		end
	end

	function ENT:CalculateThreatScore(ent)
		local score = 0

		-- Distance (closer = more threatening)
		local dist = self:GetPos():Distance(ent:GetPos())
		score = score + math.max(0, 1000 - dist) / 10

		-- Health (healthier = more threatening)
		if ent:Health() then
			score = score + (ent:Health() / 10)
		end

		-- Is attacking us (high priority)
		if IsValid(ent:GetEnemy()) and ent:GetEnemy() == self then
			score = score + 100
		end

		-- Has weapon (more dangerous)
		if ent:IsPlayer() then
			local wep = ent:GetActiveWeapon()
			if IsValid(wep) then
				score = score + 50
			end
		end

		return score
	end

	function ENT:GetPreferredEnemy()
		-- Return preferred target if still valid
		if IsValid(self.PreferredTarget) then
			if self:Disposition(self.PreferredTarget) == D_HT then
				return self.PreferredTarget
			end
		end
		return self:GetEnemy()
	end

	-- Cover System --

	function ENT:ShouldSeekCover()
		local healthPercent = self:Health() / self:GetMaxHealth()

		-- Seek cover if health is low
		if healthPercent < self.CoverHealthThreshold then
			return true
		end

		-- Seek cover if under heavy fire (took damage recently)
		if self.LastDamageTime and CurTime() - self.LastDamageTime < 2 then
			return true
		end

		-- Seek cover if reloading
		if self:IsReloading() then
			return true
		end

		return false
	end

	function ENT:FindCover()
		-- Don't search too frequently
		if CurTime() < self.LastCoverSearch + 5 then
			return self.CoverPosition
		end
		self.LastCoverSearch = CurTime()

		local enemy = self:GetPreferredEnemy()
		if not IsValid(enemy) then return nil end

		local enemyPos = enemy:GetPos()
		local bestCover = nil
		local bestScore = -1

		-- Sample positions around us
		for i = 1, 16 do
			local angle = (360 / 16) * i
			local offset = Vector(
				math.cos(math.rad(angle)),
				math.sin(math.rad(angle)),
				0
			)

			local searchPos = self:GetPos() + offset * self.CoverSearchRange

			-- Trace to find nearest cover point
			local tr = util.TraceLine({
				start = self:GetPos() + Vector(0, 0, 50),
				endpos = searchPos,
				filter = self,
				mask = MASK_SOLID_BRUSHONLY
			})

			if tr.Hit then
				-- Check if this position provides cover from enemy
				local coverTr = util.TraceLine({
					start = tr.HitPos + Vector(0, 0, 50),
					endpos = enemyPos + Vector(0, 0, 50),
					mask = MASK_SOLID_BRUSHONLY
				})

				if coverTr.Hit then
					-- This position has line-of-sight blocked to enemy (good cover)
					local dist = self:GetPos():Distance(tr.HitPos)
					local enemyDist = tr.HitPos:Distance(enemyPos)

					-- Score based on distance to cover and distance from enemy
					local score = (self.CoverSearchRange - dist) + enemyDist

					if score > bestScore then
						bestScore = score
						bestCover = tr.HitPos
					end
				end
			end
		end

		self.CoverPosition = bestCover
		return bestCover
	end

	function ENT:FindFlankPosition(enemy)
		if not IsValid(enemy) then return nil end

		-- Try to find a position that flanks the enemy
		local enemyPos = enemy:GetPos()
		local enemyDir = enemy:GetForward()

		-- Calculate flanking positions (left and right of enemy)
		local leftFlank = enemyPos - enemyDir:Cross(Vector(0, 0, 1)) * self.FlankDistance
		local rightFlank = enemyPos + enemyDir:Cross(Vector(0, 0, 1)) * self.FlankDistance

		-- Check which flank is more accessible
		local leftDist = self:GetPos():Distance(leftFlank)
		local rightDist = self:GetPos():Distance(rightFlank)

		-- Prefer closer flank
		if leftDist < rightDist then
			return leftFlank
		else
			return rightFlank
		end
	end

	function ENT:ShouldFlank()
		-- Flank if we're healthy and not in immediate danger
		local healthPercent = self:Health() / self:GetMaxHealth()
		if healthPercent < 0.7 then return false end

		-- Don't flank if recently damaged
		if self.LastDamageTime and CurTime() - self.LastDamageTime < 3 then
			return false
		end

		-- Flank if enemy is stationary or focused elsewhere
		return math.random(3) == 1  -- 33% chance to attempt flanking
	end

	-- Squad Communication --

	function ENT:CallForBackup()
		if self.BackupCalled then return end
		self.BackupCalled = true

		-- Alert nearby allies
		for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.BackupCallRange)) do
			if IsValid(ent) and ent:GetClass() == self:GetClass() and ent ~= self then
				if ent:IsInFaction(FACTION_COMBINE) then
					-- Tell ally about our enemy
					if IsValid(self:GetEnemy()) then
						ent:AddEntityRelationship(self:GetEnemy(), D_HT, 99)
						ent.WasAlerted = true
					end
				end
			end
		end

		-- Play radio call sound
		self:EmitSound("npc/combine_soldier/vo/alert1.wav")
	end

	-- AI Behavior Hooks --

	function ENT:OnEnemyChange(old, new)
		-- Reset backup call flag when engaging new enemy
		self.BackupCalled = false

		-- Assess new threat immediately
		self:AssessThreat()
	end

	function ENT:OnRangeAttack(enemy)
		-- Check if we should take cover instead of attacking
		if self:ShouldSeekCover() then
			local cover = self:FindCover()
			if cover then
				self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = cover})
				self.InCover = true
				return
			end
		end

		-- Check if we should flank
		if self:ShouldFlank() and not self.FlankPosition then
			self.FlankPosition = self:FindFlankPosition(enemy)
			if self.FlankPosition then
				self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = self.FlankPosition})
				return
			end
		end

		-- Normal attack behavior
		local wep = self:GetActiveWeapon()
		if IsValid(wep) then
			self:UseWeapon(enemy, wep)
		end
	end

	function ENT:OnReachedPatrol()
		-- Reset flanking position
		self.FlankPosition = nil

		-- Check if we reached cover
		if self.InCover then
			self.InCover = false
			-- Wait in cover for a moment
			self:Wait(math.random(2, 5))
		else
			self:Wait(math.random(1, 3))
		end
	end

	function ENT:OnIdle()
		-- Patrol randomly
		self:AddPatrolPos(self:RandomPos(1000))
	end

	-- Damage Handling --

	function ENT:OnTakeDamage(dmg)
		self.LastDamageTime = CurTime()

		-- Call for backup when taking damage
		local healthPercent = self:Health() / self:GetMaxHealth()
		if healthPercent < 0.6 and not self.BackupCalled then
			self:CallForBackup()
		end

		-- Tactical retreat if health is critical
		if healthPercent < self.RetreatHealthThreshold then
			self.IsRetreating = true

			-- Find cover and retreat
			local cover = self:FindCover()
			if cover then
				self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = cover})
				self:EmitSound("npc/combine_soldier/vo/coverme.wav")
			end
		end
	end

	function ENT:OnDeath(dmg, delay, hitgroup)
		-- Alert nearby allies of death
		for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.BackupCallRange)) do
			if IsValid(ent) and ent:GetClass() == self:GetClass() then
				if IsValid(dmg:GetAttacker()) then
					ent:AddEntityRelationship(dmg:GetAttacker(), D_HT, 99)
				end
			end
		end
	end

	-- Voice Lines --

	function ENT:OnNewEnemy()
		-- Alert call
		local sounds = {
			"npc/combine_soldier/vo/alert1.wav",
			"npc/combine_soldier/vo/contactconfirm.wav"
		}
		self:EmitSound(sounds[math.random(#sounds)])

		-- Call for backup if outnumbered
		local nearbyAllies = 0
		for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.BackupCallRange)) do
			if IsValid(ent) and ent:GetClass() == self:GetClass() then
				nearbyAllies = nearbyAllies + 1
			end
		end

		if nearbyAllies < 2 then
			self:Timer(1, function()
				self:CallForBackup()
			end)
		end
	end

end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### AI State Variables

```lua
function ENT:CustomInitialize()
	self.InCover = false
	self.CoverPosition = nil
	self.LastCoverSearch = 0
	self.FlankPosition = nil
	self.IsRetreating = false
	self.BackupCalled = false
	self.LastThreatAssessment = 0
	self.PreferredTarget = nil
end
```

**Purpose:** Track AI state and decision history.
- `InCover`: Whether currently taking cover
- `CoverPosition`: Last found cover position
- `LastCoverSearch`: Cooldown for cover searches
- `FlankPosition`: Target flanking position
- `IsRetreating`: In tactical retreat mode
- `BackupCalled`: Already called for reinforcements
- `PreferredTarget`: Highest priority target

### Threat Assessment System

```lua
function ENT:AssessThreat()
	local threats = {}

	-- Find all hostile entities
	for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.SightRange)) do
		if self:Disposition(ent) == D_HT then
			local threat = {
				entity = ent,
				score = self:CalculateThreatScore(ent)
			}
			table.insert(threats, threat)
		end
	end

	-- Sort by threat level
	table.sort(threats, function(a, b) return a.score > b.score end)

	-- Target highest threat
	if #threats > 0 then
		self.PreferredTarget = threats[1].entity
	end
end
```

**Purpose:** Intelligently prioritize targets.
- Scans for all hostile entities
- Calculates threat score for each
- Prioritizes most dangerous target
- Updates every 2 seconds

### Threat Score Calculation

```lua
function ENT:CalculateThreatScore(ent)
	local score = 0

	-- Closer enemies are more threatening
	local dist = self:GetPos():Distance(ent:GetPos())
	score = score + math.max(0, 1000 - dist) / 10

	-- Healthier enemies are more threatening
	score = score + (ent:Health() / 10)

	-- High priority if attacking us
	if ent:GetEnemy() == self then
		score = score + 100
	end

	-- Armed enemies are more dangerous
	if ent:IsPlayer() and IsValid(ent:GetActiveWeapon()) then
		score = score + 50
	end

	return score
end
```

**Purpose:** Calculate how dangerous each enemy is.
- Distance: Closer = more dangerous
- Health: Healthier = more threatening
- Targeting us: Highest priority
- Armed: Additional threat bonus

### Cover System

```lua
function ENT:ShouldSeekCover()
	local healthPercent = self:Health() / self:GetMaxHealth()

	-- Low health
	if healthPercent < self.CoverHealthThreshold then
		return true
	end

	-- Under fire
	if self.LastDamageTime and CurTime() - self.LastDamageTime < 2 then
		return true
	end

	-- Reloading
	if self:IsReloading() then
		return true
	end

	return false
end
```

**Purpose:** Determine when to seek cover.
- Below 50% health
- Took damage in last 2 seconds
- Currently reloading weapon

### Cover Finding Algorithm

```lua
function ENT:FindCover()
	local enemy = self:GetPreferredEnemy()
	local bestCover = nil
	local bestScore = -1

	-- Sample 16 positions in a circle
	for i = 1, 16 do
		local angle = (360 / 16) * i
		local searchPos = self:GetPos() + offset * self.CoverSearchRange

		-- Trace to find obstacles
		local tr = util.TraceLine({...})

		if tr.Hit then
			-- Check if position blocks line-of-sight to enemy
			local coverTr = util.TraceLine({
				start = tr.HitPos,
				endpos = enemyPos,
				mask = MASK_SOLID_BRUSHONLY
			})

			if coverTr.Hit then
				-- Good cover found! Score it
				local score = (self.CoverSearchRange - dist) + enemyDist
				if score > bestScore then
					bestScore = score
					bestCover = tr.HitPos
				end
			end
		end
	end

	return bestCover
end
```

**Purpose:** Find best cover position.
- Samples positions in a circle around NPC
- Checks for solid obstacles
- Verifies obstacle blocks line-of-sight to enemy
- Scores based on proximity and safety
- Returns best position found

### Flanking System

```lua
function ENT:FindFlankPosition(enemy)
	local enemyPos = enemy:GetPos()
	local enemyDir = enemy:GetForward()

	-- Calculate left and right flank positions
	local leftFlank = enemyPos - enemyDir:Cross(Vector(0, 0, 1)) * self.FlankDistance
	local rightFlank = enemyPos + enemyDir:Cross(Vector(0, 0, 1)) * self.FlankDistance

	-- Choose closer flank
	if leftDist < rightDist then
		return leftFlank
	else
		return rightFlank
	end
end

function ENT:ShouldFlank()
	-- Only flank if healthy
	local healthPercent = self:Health() / self:GetMaxHealth()
	if healthPercent < 0.7 then return false end

	-- Not if recently damaged
	if self.LastDamageTime and CurTime() - self.LastDamageTime < 3 then
		return false
	end

	return math.random(3) == 1  -- 33% chance
end
```

**Purpose:** Flank enemies instead of direct approach.
- Calculates positions to left/right of enemy
- Only attempts when healthy and safe
- Adds tactical variety to combat

### Squad Communication

```lua
function ENT:CallForBackup()
	if self.BackupCalled then return end
	self.BackupCalled = true

	-- Alert nearby allies
	for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.BackupCallRange)) do
		if IsValid(ent) and ent:GetClass() == self:GetClass() then
			if ent:IsInFaction(FACTION_COMBINE) then
				-- Share enemy information
				if IsValid(self:GetEnemy()) then
					ent:AddEntityRelationship(self:GetEnemy(), D_HT, 99)
				end
			end
		end
	end

	self:EmitSound("npc/combine_soldier/vo/alert1.wav")
end
```

**Purpose:** Coordinate with nearby allies.
- Finds allies within 500 units
- Shares enemy information
- Allies will assist in combat
- Plays radio call sound

### Tactical Retreat

```lua
function ENT:OnTakeDamage(dmg)
	local healthPercent = self:Health() / self:GetMaxHealth()

	-- Call for help
	if healthPercent < 0.6 then
		self:CallForBackup()
	end

	-- Retreat if critical
	if healthPercent < self.RetreatHealthThreshold then
		self.IsRetreating = true

		local cover = self:FindCover()
		if cover then
			self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = cover})
			self:EmitSound("npc/combine_soldier/vo/coverme.wav")
		end
	end
end
```

**Purpose:** Self-preservation when badly damaged.
- Calls backup when at 60% health
- Retreats to cover at 30% health
- Plays voice line "Cover me!"
- Stops retreating once healed above threshold

### Intelligent Attack Behavior

```lua
function ENT:OnRangeAttack(enemy)
	-- Prioritize cover over attacking
	if self:ShouldSeekCover() then
		local cover = self:FindCover()
		if cover then
			self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = cover})
			return
		end
	end

	-- Try flanking maneuver
	if self:ShouldFlank() then
		self.FlankPosition = self:FindFlankPosition(enemy)
		if self.FlankPosition then
			self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = self.FlankPosition})
			return
		end
	end

	-- Default: attack with weapon
	self:UseWeapon(enemy, self:GetActiveWeapon())
end
```

**Purpose:** Smart combat decision tree.
1. Check if should take cover (highest priority)
2. Consider flanking maneuver
3. Fall back to standard attack

## Testing the AI

### Setup

1. Load a map with cover opportunities (gm_construct, rp_downtown, etc.)
2. Spawn several Tactical Soldiers
3. Spawn enemy NPCs or be the enemy yourself
4. Observe their behaviors

### Behaviors to Observe

**Cover System:**
- Soldiers seek cover when health drops below 50%
- Take cover when reloading
- Move to cover after taking damage

**Flanking:**
- Some soldiers will attempt to flank around enemies
- More likely when healthy
- Flanking positions are to sides of enemy

**Squad Coordination:**
- Soldiers call for backup when outnumbered
- Nearby allies respond to calls
- Share enemy information with squad

**Tactical Retreat:**
- Retreat to cover when health drops below 30%
- Play "Cover me!" voice line
- Return to combat after recovering

**Threat Assessment:**
- Target closest/most dangerous enemies first
- Prioritize enemies actively attacking them
- Ignore weak/distant threats

### Testing Commands

```lua
-- Spawn a squad
for i = 1, 4 do
	local npc = ents.Create("npc_drg_tactical_soldier")
	npc:SetPos(Entity(1):GetPos() + Vector(i*100, 0, 0))
	npc:Spawn()
end

-- Spawn enemy for them to fight
local enemy = ents.Create("npc_combine_s")
enemy:SetPos(Entity(1):GetPos() + Vector(500, 0, 0))
enemy:Spawn()

-- Damage an NPC to test retreat behavior
Entity(X):TakeDamage(70)  -- Replace X with entity index

-- Check NPC's current state
print(Entity(X).IsRetreating, Entity(X).InCover)
```

### Testing Tips

- Test in maps with cover (walls, props, buildings)
- Spawn multiple soldiers to see squad coordination
- Damage soldiers to different health levels
- Watch for flanking attempts
- Listen for voice lines (backup calls, alerts)

## Customization Ideas

### Aggressive AI

```lua
-- Make more aggressive, less cautious
ENT.CoverHealthThreshold = 0.2  -- Only cover at 20% health
ENT.RetreatHealthThreshold = 0.1  -- Only retreat at 10% health

function ENT:ShouldFlank()
	return math.random(2) == 1  -- 50% chance to flank (more aggressive)
end
```

### Defensive AI

```lua
-- Make more defensive
ENT.CoverHealthThreshold = 0.8  -- Seek cover at 80% health
ENT.RetreatHealthThreshold = 0.5  -- Retreat at 50% health

function ENT:ShouldFlank()
	return false  -- Never flank, stay defensive
end
```

### Sniper Behavior

```lua
-- Sniper AI: long range, stationary
ENT.RangeAttackRange = 5000
ENT.PreferredDistance = 2000

function ENT:OnRangeAttack(enemy)
	-- Maintain distance
	local dist = self:GetPos():Distance(enemy:GetPos())
	if dist < self.PreferredDistance then
		-- Back away
		local awayDir = (self:GetPos() - enemy:GetPos()):GetNormalized()
		local retreatPos = self:GetPos() + awayDir * 500
		self:SetSchedule(SCHED_FORCED_GO_RUN, {pos = retreatPos})
	else
		-- Stay put and shoot
		self:UseWeapon(enemy, self:GetActiveWeapon())
	end
end
```

### Support Medic

```lua
function ENT:CustomThink()
	-- Heal nearby injured allies
	for _, ally in pairs(ents.FindInSphere(self:GetPos(), 300)) do
		if IsValid(ally) and ally:IsNPC() and ally ~= self then
			if self:Disposition(ally) == D_LI then
				local healthPercent = ally:Health() / ally:GetMaxHealth()
				if healthPercent < 0.7 then
					-- Heal ally
					ally:SetHealth(math.min(ally:Health() + 5, ally:GetMaxHealth()))

					-- Visual effect
					local effectData = EffectData()
					effectData:SetOrigin(ally:GetPos())
					util.Effect("TeslaHitBoxes", effectData)
				end
			end
		end
	end
end
```

### Ambush Predator

```lua
function ENT:OnIdle()
	-- Find ambush position
	local ambushPos = self:FindAmbushSpot()
	if ambushPos then
		self:SetSchedule(SCHED_FORCED_GO, {pos = ambushPos})
		self.IsAmbushing = true
	end
end

function ENT:FindAmbushSpot()
	-- Find position with good sight lines and cover
	-- (Implementation similar to FindCover but optimizes for offense)
end

function ENT:OnNewEnemy()
	-- Don't alert, stay silent for ambush
	if self.IsAmbushing then
		return  -- Stay quiet
	end
end
```

### Leader/Commander

```lua
ENT.IsLeader = true

function ENT:CustomThink()
	if not self.IsLeader then return end

	-- Issue orders to nearby allies
	for _, ally in pairs(ents.FindInSphere(self:GetPos(), 800)) do
		if IsValid(ally) and ally:GetClass() == self:GetClass() and ally ~= self then
			-- Coordinate attacks
			if IsValid(self:GetEnemy()) then
				ally.AssignedTarget = self:GetEnemy()

				-- Send flanking orders
				if math.random(2) == 1 then
					ally.ShouldFlankOverride = true
				end
			end
		end
	end
end
```

## Troubleshooting

### NPCs don't seek cover

**Problem:** Cover finding not working.

**Solutions:**
- Check map has solid geometry (walls, props)
- Verify `CoverSearchRange` is appropriate for map
- Make sure `ShouldSeekCover()` returns true
- Add debug: `print("Seeking cover:", self:ShouldSeekCover())`
- Test with `debugoverlay.Line()` to visualize cover traces

### NPCs don't flank

**Problem:** Flanking decision or pathfinding.

**Solutions:**
- Check `ShouldFlank()` returns true sometimes
- Verify navmesh exists on map (`nav_generate`)
- Make sure `FlankDistance` is appropriate
- Add debug: `print("Flanking:", self.FlankPosition)`

### Backup calls don't work

**Problem:** No allies found or relationship issue.

**Solutions:**
- Spawn multiple NPCs of same type
- Verify they're within `BackupCallRange` (500 units)
- Check they share same faction
- Add debug: `print("Calling backup, allies:", allyCount)`

### NPCs don't retreat

**Problem:** Health threshold or cover finding.

**Solutions:**
- Check health is below `RetreatHealthThreshold` (30%)
- Verify cover can be found
- Make sure `IsRetreating` flag is being set
- Test by manually damaging: `Entity(X):TakeDamage(70)`

### Threat assessment doesn't work

**Problem:** Target prioritization issue.

**Solutions:**
- Check `AssessThreat()` is being called
- Verify threat scores are being calculated
- Make sure preferred target is valid
- Add debug: `print("Threat score:", score, "Target:", ent)`

### Performance issues

**Problem:** Too much computation.

**Solutions:**
- Increase cooldown times (threat assessment, cover search)
- Reduce search ranges
- Limit number of positions sampled
- Cache expensive calculations

## See Also

- [Simple Melee NPC Example](simple-melee-npc.md) - Basic NPC creation
- [Ranged NPC Example](ranged-npc.md) - Ranged combat NPCs
- [Boss NPC Example](boss-npc.md) - Boss mechanics
- [Creating NPCs Guide](../guides/creating-npcs.md) - Complete NPC tutorial
- [AI API](../api/nextbot/ai.md) - AI system reference
- [Movement API](../api/nextbot/movement.md) - Movement and navigation
- [Faction System Example](faction-system.md) - Squad coordination
