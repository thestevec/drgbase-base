# Boss NPC Example

A complete boss encounter with multiple attack phases, special abilities, and challenging mechanics.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the Boss](#testing-the-boss)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)
- [See Also](#see-also)

## Overview

This example creates a challenging boss NPC that features:
- High health pool (1000 HP)
- Three distinct combat phases based on remaining health
- **Phase 1 (100%-50%)**: Standard melee attacks
- **Phase 2 (50%-25%)**: Enraged state - summons minion zombies, faster attacks
- **Phase 3 (25%-0%)**: Desperate state - uses devastating area attacks
- Dynamic behavior that adapts to damage taken
- Visual and audio feedback for phase transitions
- Can be possessed by players for testing

**Difficulty:** Intermediate
**File Location:** `lua/entities/npc_drg_boss_zombie.lua`
**Base:** `drgbase_nextbot`

## Complete Code

Create a file named `npc_drg_boss_zombie.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot" -- DO NOT TOUCH (obviously)

-- Misc --
ENT.PrintName = "Zombie Boss"
ENT.Category = "DrGBase"
ENT.Models = {"models/Zombie/Poison.mdl"}
ENT.BloodColor = BLOOD_COLOR_GREEN
ENT.CollisionBounds = Vector(40, 40, 90)

-- Sounds --
ENT.OnDamageSounds = {"NPC_PoisonZombie.Pain"}
ENT.OnDeathSounds = {"NPC_PoisonZombie.Die"}

-- Stats --
ENT.SpawnHealth = 1000
ENT.ArmorEnabled = true
ENT.SpawnArmor = 200

-- AI --
ENT.RangeAttackRange = 0
ENT.MeleeAttackRange = 80
ENT.ReachEnemyRange = 80
ENT.AvoidEnemyRange = 0

-- Relationships --
ENT.Factions = {FACTION_ZOMBIES}

-- Movements/animations --
ENT.UseWalkframes = true
ENT.WalkSpeed = 40
ENT.RunSpeed = 80

-- Detection --
ENT.EyeBone = "ValveBiped.Bip01_Spine4"
ENT.EyeOffset = Vector(7.5, 0, 5)
ENT.SightFOV = 140
ENT.SightRange = 3000

-- Possession --
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
	{
		offset = Vector(0, 50, 40),
		distance = 200
	},
	{
		offset = Vector(7.5, 0, 0),
		distance = 0,
		eyepos = true
	}
}
ENT.PossessionBinds = {
	[IN_ATTACK] = {{
		coroutine = true,
		onkeydown = function(self)
			if self:GetCurrentPhase() == 3 then
				self:AreaAttack()
			else
				self:BossAttack()
			end
		end
	}},
	[IN_ATTACK2] = {{
		coroutine = true,
		onkeydown = function(self)
			if self:GetCurrentPhase() >= 2 then
				self:SummonMinions()
			end
		end
	}}
}

if SERVER then

	-- Boss Data --

	function ENT:GetCurrentPhase()
		local healthPercent = self:Health() / self:GetMaxHealth()
		if healthPercent > 0.5 then
			return 1
		elseif healthPercent > 0.25 then
			return 2
		else
			return 3
		end
	end

	function ENT:GetPhaseColor()
		local phase = self:GetCurrentPhase()
		if phase == 1 then
			return Color(100, 255, 100) -- Green
		elseif phase == 2 then
			return Color(255, 200, 50) -- Orange
		else
			return Color(255, 50, 50) -- Red
		end
	end

	-- Init/Think --

	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_HT)
		self:SetBodygroup(1, 0)

		-- Boss tracking variables
		self.LastPhase = 1
		self.NextMinionSpawn = 0
		self.AttackComboCount = 0
		self.IsEnraged = false

		-- Create health bar effect
		self:Timer(0.1, function()
			local sprite = ents.Create("env_sprite")
			if IsValid(sprite) then
				sprite:SetKeyValue("model", "sprites/glow01.spr")
				sprite:SetKeyValue("scale", "0.5")
				sprite:SetKeyValue("rendermode", "5")
				sprite:SetKeyValue("rendercolor", "100 255 100")
				sprite:SetKeyValue("renderamt", "255")
				sprite:SetPos(self:GetPos() + Vector(0, 0, 100))
				sprite:SetParent(self)
				sprite:Spawn()
				sprite:Activate()
				self.HealthSprite = sprite
			end
		end)
	end

	function ENT:CustomThink()
		-- Check for phase transitions
		local currentPhase = self:GetCurrentPhase()
		if currentPhase ~= self.LastPhase then
			self:OnPhaseChange(self.LastPhase, currentPhase)
			self.LastPhase = currentPhase
		end

		-- Update health indicator sprite color
		if IsValid(self.HealthSprite) then
			local color = self:GetPhaseColor()
			self.HealthSprite:SetKeyValue("rendercolor",
				string.format("%d %d %d", color.r, color.g, color.b))
		end

		-- Phase 2: Periodic minion spawning during combat
		if currentPhase >= 2 and self:HasEnemy() then
			if CurTime() >= self.NextMinionSpawn then
				self:SummonMinions()
				self.NextMinionSpawn = CurTime() + math.random(15, 25)
			end
		end
	end

	-- Boss Abilities --

	function ENT:OnPhaseChange(oldPhase, newPhase)
		-- Play transition effects
		self:EmitSound("NPC_PoisonZombie.Alert")

		local effectData = EffectData()
		effectData:SetOrigin(self:GetPos() + Vector(0, 0, 40))
		effectData:SetMagnitude(3)
		effectData:SetScale(2)
		util.Effect("ElectricSpark", effectData)

		if newPhase == 2 then
			-- Enter Phase 2: Enrage
			self.IsEnraged = true
			self.WalkSpeed = 60
			self.RunSpeed = 120
			self.MeleeAttackRange = 100

			-- Spawn initial minions
			self:SummonMinions(3)

			-- Screen shake
			util.ScreenShake(self:GetPos(), 10, 5, 2, 1000)

		elseif newPhase == 3 then
			-- Enter Phase 3: Desperation
			self.WalkSpeed = 80
			self.RunSpeed = 150
			self.MeleeAttackRange = 120

			-- Bigger screen shake
			util.ScreenShake(self:GetPos(), 15, 5, 3, 1500)
		end
	end

	function ENT:BossAttack()
		local phase = self:GetCurrentPhase()

		if phase == 1 then
			-- Phase 1: Single heavy attack
			self:EmitSound("NPC_PoisonZombie.Attack")
			self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
		elseif phase >= 2 then
			-- Phase 2+: Combo attacks
			self.AttackComboCount = (self.AttackComboCount or 0) + 1

			if self.AttackComboCount >= 3 then
				-- Third attack in combo: Strong finishing blow
				self:EmitSound("NPC_PoisonZombie.Attack")
				self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 0.8, self.FaceEnemy)
				self.AttackComboCount = 0
			else
				-- Quick combo attacks
				self:EmitSound("NPC_PoisonZombie.Attack")
				self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1.2, self.FaceEnemy)
			end
		end
	end

	function ENT:SummonMinions(count)
		count = count or 2

		self:EmitSound("NPC_PoisonZombie.ThrowWarn")
		self:PlaySequence("throw")

		-- Find valid spawn positions around the boss
		for i = 1, count do
			local angle = (360 / count) * i
			local offset = Vector(math.cos(math.rad(angle)), math.sin(math.rad(angle)), 0) * 100
			local spawnPos = self:GetPos() + offset

			-- Spawn effect
			local effectData = EffectData()
			effectData:SetOrigin(spawnPos + Vector(0, 0, 10))
			effectData:SetMagnitude(1)
			util.Effect("ElectricSpark", effectData)

			-- Create minion
			self:Timer(0.5, function()
				local minion = ents.Create("npc_drg_zombie")
				if IsValid(minion) then
					minion:SetPos(spawnPos)
					minion:SetAngles(self:GetAngles())
					minion:Spawn()

					-- Scale down minion
					minion:SetModelScale(0.8)
					minion.SpawnHealth = 50
					minion:SetHealth(50)

					-- Set enemy to boss's enemy
					if IsValid(self:GetEnemy()) then
						minion:AddEntityRelationship(self:GetEnemy(), D_HT, 99)
					end

					-- Add to undo
					if IsValid(self:GetCreator()) then
						self:GetCreator():DrG_AddUndo(minion, "NPC", "Undone Minion")
					end
				end
			end)
		end

		self:PauseCoroutine(1.5)
	end

	function ENT:AreaAttack()
		self:EmitSound("NPC_PoisonZombie.Attack")
		self:PlaySequence("throw")

		self:PauseCoroutine(0.5)

		-- Create shockwave effect
		local effectData = EffectData()
		effectData:SetOrigin(self:GetPos())
		effectData:SetScale(300)
		util.Effect("ThumperDust", effectData)

		-- Screen shake
		util.ScreenShake(self:GetPos(), 20, 5, 1, 500)

		-- Damage all enemies in radius
		for _, ent in pairs(ents.FindInSphere(self:GetPos(), 300)) do
			if IsValid(ent) and (ent:IsPlayer() or ent:IsNPC()) and ent ~= self then
				if self:Disposition(ent) == D_HT then
					local dmg = DamageInfo()
					dmg:SetDamage(50)
					dmg:SetAttacker(self)
					dmg:SetInflictor(self)
					dmg:SetDamageType(DMG_BLAST)
					dmg:SetDamageForce((ent:GetPos() - self:GetPos()):GetNormalized() * 50000)
					ent:TakeDamageInfo(dmg)

					-- Ragdoll effect
					if ent:IsPlayer() then
						ent:ViewPunch(Angle(20, math.random(-15, 15), 0))
					end
				end
			end
		end

		self:PauseCoroutine(1)
	end

	-- AI --

	function ENT:OnMeleeAttack(enemy)
		self:BossAttack()
	end

	function ENT:OnReachedPatrol()
		self:Wait(math.random(3, 7))
	end

	function ENT:OnIdle()
		self:AddPatrolPos(self:RandomPos(1500))
	end

	-- Damage --

	function ENT:OnDeath(dmg, delay, hitgroup)
		-- Epic death effects
		for i = 1, 5 do
			self:Timer(i * 0.2, function()
				local effectData = EffectData()
				effectData:SetOrigin(self:GetPos() + Vector(
					math.random(-20, 20),
					math.random(-20, 20),
					math.random(10, 60)
				))
				effectData:SetMagnitude(2)
				util.Effect("ElectricSpark", effectData)
			end)
		end

		-- Big explosion effect
		self:Timer(1, function()
			local effectData = EffectData()
			effectData:SetOrigin(self:GetPos() + Vector(0, 0, 40))
			effectData:SetScale(3)
			util.Effect("Explosion", effectData)
			util.ScreenShake(self:GetPos(), 25, 5, 2, 2000)
		end)

		-- Remove health sprite
		if IsValid(self.HealthSprite) then
			self.HealthSprite:Remove()
		end
	end

	-- Animations/Sounds --

	function ENT:OnNewEnemy()
		self:EmitSound("NPC_PoisonZombie.Alert")
	end

	function ENT:OnAnimEvent()
		if self:IsAttacking() and self:GetCycle() > 0.3 then
			local phase = self:GetCurrentPhase()
			local damage = 25

			-- Increase damage based on phase
			if phase == 2 then
				damage = 40
			elseif phase == 3 then
				damage = 60
			end

			-- Check if this is the finishing blow in combo
			if self.AttackComboCount == 0 and phase >= 2 then
				damage = damage * 1.5
			end

			self:Attack({
				damage = damage,
				type = DMG_SLASH,
				viewpunch = Angle(30, math.random(-15, 15), 0)
			}, function(self, hit)
				if #hit > 0 then
					self:EmitSound("NPC_PoisonZombie.Hit")

					-- Apply poison effect in Phase 3
					if phase >= 3 then
						for _, victim in pairs(hit) do
							if IsValid(victim) and victim:IsPlayer() then
								-- Poison damage over time
								local poisonDuration = 5
								local poisonTicks = 0
								self:Timer(1, function()
									if IsValid(victim) and victim:Alive() and poisonTicks < poisonDuration then
										poisonTicks = poisonTicks + 1
										local dmg = DamageInfo()
										dmg:SetDamage(5)
										dmg:SetAttacker(self)
										dmg:SetInflictor(self)
										dmg:SetDamageType(DMG_POISON)
										victim:TakeDamageInfo(dmg)
										return true -- Continue timer
									end
									return false -- Stop timer
								end)
							end
						end
					end
				else
					self:EmitSound("NPC_PoisonZombie.Miss")
				end
			end)
		end
	end

	function ENT:OnRemove()
		if IsValid(self.HealthSprite) then
			self.HealthSprite:Remove()
		end
	end

end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### Boss Stats

```lua
ENT.SpawnHealth = 1000
ENT.ArmorEnabled = true
ENT.SpawnArmor = 200
```

**Purpose:** Boss has significantly more health than normal NPCs, plus armor for extra survivability.
- 1000 HP makes it a real challenge
- 200 armor provides additional damage mitigation
- Total effective health is much higher than standard enemies

### Phase System

```lua
function ENT:GetCurrentPhase()
	local healthPercent = self:Health() / self:GetMaxHealth()
	if healthPercent > 0.5 then
		return 1
	elseif healthPercent > 0.25 then
		return 2
	else
		return 3
	end
end
```

**Purpose:** Dynamically determine boss behavior based on health.
- **Phase 1**: > 50% health - Normal attacks
- **Phase 2**: 25-50% health - Enraged, summons minions
- **Phase 3**: < 25% health - Desperate, area attacks

The phase system is checked in `CustomThink()` to detect transitions:

```lua
local currentPhase = self:GetCurrentPhase()
if currentPhase ~= self.LastPhase then
	self:OnPhaseChange(self.LastPhase, currentPhase)
	self.LastPhase = currentPhase
end
```

### Phase Transitions

```lua
function ENT:OnPhaseChange(oldPhase, newPhase)
	self:EmitSound("NPC_PoisonZombie.Alert")

	if newPhase == 2 then
		self.IsEnraged = true
		self.WalkSpeed = 60
		self.RunSpeed = 120
		self:SummonMinions(3)
		util.ScreenShake(self:GetPos(), 10, 5, 2, 1000)
	elseif newPhase == 3 then
		self.WalkSpeed = 80
		self.RunSpeed = 150
		util.ScreenShake(self:GetPos(), 15, 5, 3, 1500)
	end
end
```

**Purpose:** Handle dramatic changes when entering new phases.
- Plays alert sound and visual effects
- Increases movement speed as boss gets more desperate
- Spawns minions when entering Phase 2
- Creates screen shake for dramatic effect

### Health Indicator

```lua
function ENT:CustomInitialize()
	local sprite = ents.Create("env_sprite")
	sprite:SetKeyValue("model", "sprites/glow01.spr")
	sprite:SetPos(self:GetPos() + Vector(0, 0, 100))
	sprite:SetParent(self)
	sprite:Spawn()
	self.HealthSprite = sprite
end
```

**Purpose:** Visual indicator of boss health above its head.
- Color changes based on phase (green -> orange -> red)
- Updated every frame in `CustomThink()`
- Parented to boss so it follows

### Ability 1: Summoning Minions

```lua
function ENT:SummonMinions(count)
	count = count or 2

	self:PlaySequence("throw")

	for i = 1, count do
		local angle = (360 / count) * i
		local offset = Vector(math.cos(math.rad(angle)), math.sin(math.rad(angle)), 0) * 100
		local spawnPos = self:GetPos() + offset

		local minion = ents.Create("npc_drg_zombie")
		minion:SetPos(spawnPos)
		minion:Spawn()
		minion:SetModelScale(0.8)
		minion:SetHealth(50)

		if IsValid(self:GetEnemy()) then
			minion:AddEntityRelationship(self:GetEnemy(), D_HT, 99)
		end
	end
end
```

**Purpose:** Summon weaker zombie minions to assist in combat.
- Spawns minions in a circle around the boss
- Minions are scaled down (0.8x) and weaker (50 HP)
- Automatically target the boss's current enemy
- Used in Phase 2 and Phase 3

### Ability 2: Area Attack

```lua
function ENT:AreaAttack()
	self:PlaySequence("throw")

	util.ScreenShake(self:GetPos(), 20, 5, 1, 500)

	for _, ent in pairs(ents.FindInSphere(self:GetPos(), 300)) do
		if IsValid(ent) and (ent:IsPlayer() or ent:IsNPC()) then
			if self:Disposition(ent) == D_HT then
				local dmg = DamageInfo()
				dmg:SetDamage(50)
				dmg:SetDamageType(DMG_BLAST)
				dmg:SetDamageForce((ent:GetPos() - self:GetPos()):GetNormalized() * 50000)
				ent:TakeDamageInfo(dmg)
			end
		end
	end
end
```

**Purpose:** Devastating area-of-effect attack in Phase 3.
- Damages all enemies within 300 units
- Deals 50 damage and knocks back targets
- Creates dramatic screen shake
- Only available in Phase 3

### Combo Attack System

```lua
function ENT:BossAttack()
	local phase = self:GetCurrentPhase()

	if phase >= 2 then
		self.AttackComboCount = (self.AttackComboCount or 0) + 1

		if self.AttackComboCount >= 3 then
			-- Finishing blow: Slower but stronger
			self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 0.8, self.FaceEnemy)
			self.AttackComboCount = 0
		else
			-- Quick combo attacks
			self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1.2, self.FaceEnemy)
		end
	end
end
```

**Purpose:** In Phase 2+, boss performs 3-hit combos.
- First two hits are fast (1.2x speed)
- Third hit is slower (0.8x speed) but deals 1.5x damage
- Resets after finishing blow

### Dynamic Damage Scaling

```lua
function ENT:OnAnimEvent()
	if self:IsAttacking() then
		local phase = self:GetCurrentPhase()
		local damage = 25

		if phase == 2 then
			damage = 40
		elseif phase == 3 then
			damage = 60
		end

		if self.AttackComboCount == 0 and phase >= 2 then
			damage = damage * 1.5
		end

		self:Attack({damage = damage, type = DMG_SLASH})
	end
end
```

**Purpose:** Boss damage increases with each phase.
- Phase 1: 25 damage
- Phase 2: 40 damage
- Phase 3: 60 damage
- Finishing blow: 1.5x damage multiplier

### Poison Effect (Phase 3)

```lua
if phase >= 3 then
	for _, victim in pairs(hit) do
		if IsValid(victim) and victim:IsPlayer() then
			local poisonDuration = 5
			local poisonTicks = 0
			self:Timer(1, function()
				if IsValid(victim) and victim:Alive() and poisonTicks < poisonDuration then
					poisonTicks = poisonTicks + 1
					victim:TakeDamage(5)
					return true -- Continue timer
				end
				return false -- Stop timer
			end)
		end
	end
end
```

**Purpose:** In Phase 3, melee attacks apply poison damage over time.
- Deals 5 damage per second for 5 seconds
- Only affects players
- Stacks with multiple hits

## Testing the Boss

### Spawning

1. Start Garry's Mod and load a large, open map (like `gm_flatgrass`)
2. Open spawn menu (Q)
3. Navigate to "DrGBase" tab under NPCs
4. Click "Zombie Boss"

### Recommended Testing Setup

```lua
-- In console, spawn some allies to help fight the boss:
npc_create npc_citizen
npc_create npc_alyx

-- Or possess the boss to test abilities:
-- Walk up to boss and press E (USE), then R to change camera
```

### Behavior to Observe

**Phase 1 (100%-50% HP)**:
- Boss performs standard melee attacks
- Deals 25 damage per hit
- Normal movement speed

**Phase 2 (50%-25% HP)**:
- Boss becomes enraged (visual effects, screen shake)
- Movement speed increases
- Summons 3 minions immediately
- Continues summoning minions every 15-25 seconds
- Performs 3-hit combos (40 damage, finishing blow deals 60)

**Phase 3 (25%-0% HP)**:
- Even faster movement
- Can perform devastating area attacks (50 damage, 300 unit radius)
- Melee attacks deal 60 damage plus poison (5 damage/sec for 5 sec)
- Health indicator turns red

**Death**:
- Multiple spark effects
- Final explosion effect
- Large screen shake

### Testing Tips

- Bring backup - this boss is tough!
- Watch the health indicator sprite to see phase changes
- Listen for audio cues during phase transitions
- Test possession to use boss abilities manually
- Try fighting in different environments

## Customization Ideas

### Adjust Difficulty

```lua
-- Easier boss
ENT.SpawnHealth = 500
ENT.SpawnArmor = 100

-- Harder boss
ENT.SpawnHealth = 2000
ENT.SpawnArmor = 500
```

### More Phases

```lua
function ENT:GetCurrentPhase()
	local hp = self:Health() / self:GetMaxHealth()
	if hp > 0.75 then return 1
	elseif hp > 0.5 then return 2
	elseif hp > 0.25 then return 3
	else return 4 end
end

-- Then add Phase 4 behavior in OnPhaseChange
```

### Different Minions

```lua
function ENT:SummonMinions(count)
	-- Mix of zombie types
	local minionTypes = {
		"npc_drg_zombie",
		"npc_drg_headcrab",
		"npc_drg_zombie" -- More zombies than headcrabs
	}

	for i = 1, count do
		local minion = ents.Create(table.Random(minionTypes))
		-- ... rest of spawn code
	end
end
```

### Ranged Attacks

```lua
-- Add ranged capability
ENT.RangeAttackRange = 500

function ENT:OnRangeAttack(enemy)
	if self:GetCurrentPhase() >= 2 then
		-- Throw projectile
		self:PlaySequence("throw")
		self:PauseCoroutine(0.3)

		local proj = ents.Create("obj_drg_grenade")
		proj:SetPos(self:EyePos())
		proj:Spawn()
		proj:GetPhysicsObject():SetVelocity(
			(enemy:EyePos() - self:EyePos()):GetNormalized() * 1000
		)
	end
end
```

### Regeneration

```lua
function ENT:CustomThink()
	-- ... existing code ...

	-- Phase 2: Regenerate health slowly
	if self:GetCurrentPhase() == 2 then
		if self:Health() < self:GetMaxHealth() then
			self:SetHealth(math.min(
				self:Health() + 1,
				self:GetMaxHealth()
			))
		end
	end
end
```

### Teleportation

```lua
function ENT:OnTakeDamage(dmg)
	-- Teleport away when taking heavy damage
	if dmg:GetDamage() > 100 and math.random(3) == 1 then
		local pos = self:RandomPos(500)
		if pos then
			self:SetPos(pos)

			-- Teleport effect
			local effectData = EffectData()
			effectData:SetOrigin(self:GetPos())
			util.Effect("ElectricSpark", effectData)
		end
	end
end
```

## Troubleshooting

### Boss spawns but doesn't change phases

**Problem:** Phase detection isn't working.

**Solutions:**
- Add debug prints: `print("Phase:", self:GetCurrentPhase(), "HP:", self:Health())`
- Make sure `CustomThink()` is being called
- Check that `self.LastPhase` is initialized in `CustomInitialize()`
- Verify health is actually decreasing when damaged

### Minions don't spawn

**Problem:** Entity creation or positioning issue.

**Solutions:**
- Check console for errors about `npc_drg_zombie` not existing
- Verify DrGBase is installed and `npc_drg_zombie` exists
- Make sure spawn position is valid (not in a wall)
- Add debug: `print("Spawning minion at", spawnPos)`

### Area attack doesn't damage anything

**Problem:** Range or relationship issue.

**Solutions:**
- Check that entities are within 300 units
- Verify `self:Disposition(ent) == D_HT` returns true for enemies
- Make sure damage is being applied: Add `print("Hit:", ent)`
- Test with debug sphere: `debugoverlay.Sphere(self:GetPos(), 300, 1, Color(255,0,0), true)`

### Boss is too easy/hard

**Problem:** Balance issues.

**Solutions:**
- Adjust `SpawnHealth` and `SpawnArmor`
- Modify damage values in phase calculations
- Change phase thresholds (currently 0.5 and 0.25)
- Adjust minion spawn frequency
- Modify area attack radius and damage

### Performance issues with many minions

**Problem:** Too many entities.

**Solutions:**
- Reduce minion count in `SummonMinions()`
- Increase time between spawns (currently 15-25 seconds)
- Limit total minion count:
```lua
function ENT:CountMinions()
	local count = 0
	for _, ent in pairs(ents.FindByClass("npc_drg_zombie")) do
		if IsValid(ent) and ent != self then count = count + 1 end
	end
	return count
end

function ENT:SummonMinions(count)
	if self:CountMinions() >= 10 then return end -- Max 10 minions
	-- ... rest of code
end
```

### Health sprite doesn't appear

**Problem:** Sprite entity issue.

**Solutions:**
- Check that timer is creating sprite after NPC fully initializes
- Verify sprite model path: `sprites/glow01.spr`
- Make sure sprite is being parented: `sprite:SetParent(self)`
- Add debug: `print("Sprite created:", IsValid(self.HealthSprite))`

## See Also

- [Simple Melee NPC Example](simple-melee-npc.md) - Basic NPC creation
- [Ranged NPC Example](ranged-npc.md) - Adding ranged attacks
- [Advanced AI Example](advanced-ai.md) - Complex AI behaviors
- [Creating NPCs Guide](../guides/creating-npcs.md) - Complete NPC tutorial
- [Combat API](../api/nextbot/weapons.md) - Combat system reference
- [Animation API](../api/nextbot/animation.md) - Animation system
- [Relationship API](../api/relationships.md) - Entity relationships
