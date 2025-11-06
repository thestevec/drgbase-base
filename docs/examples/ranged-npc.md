# Ranged Combat NPC Example

A complete, working example of a leaping/ranged attack NPC using DrGBase - modeled after Half-Life 2's headcrab.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the NPC](#testing-the-npc)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)

## Overview

This example creates a headcrab NPC that:
- Performs leaping attacks from range (instead of traditional melee)
- Broadcasts before leaping (wind-up animation)
- Deals damage on contact while in mid-leap
- Uses ranged attack AI behavior
- Avoids getting too close when not attacking
- Has patrol behavior when idle
- Can be possessed by players with custom leap controls
- Takes double damage from crowbar (easter egg)

**Difficulty:** Intermediate
**File Location:** `lua/entities/npc_drg_headcrab.lua`
**Base:** `drgbase_nextbot`
**Key Concept:** Ranged attacks using leap mechanics

## Complete Code

Create a file named `npc_drg_headcrab.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot" -- DO NOT TOUCH (obviously)

-- Misc --
ENT.PrintName = "Headcrab"
ENT.Category = "DrGBase"
ENT.Models = {"models/headcrabclassic.mdl"}
ENT.CollisionBounds = Vector(12, 12, 24)
ENT.BloodColor = BLOOD_COLOR_GREEN
ENT.VJ_ID_Headcrab = true

-- Stats --
ENT.SpawnHealth = 40

-- Sounds --
ENT.OnIdleSounds = {"NPC_HeadCrab.Idle"}
ENT.OnDamageSounds = {"NPC_HeadCrab.Pain"}
ENT.OnDeathSounds = {"NPC_HeadCrab.Die"}

-- AI --
ENT.RangeAttackRange = 150
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 125
ENT.AvoidEnemyRange = 100

-- Relationships --
ENT.Factions = {FACTION_ZOMBIES}

-- Animations --
ENT.WalkAnimation = ACT_RUN
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE
ENT.JumpAnimation = ACT_IDLE

-- Movements --
ENT.UseWalkframes = true

-- Detection --
ENT.EyeBone = "HeadcrabClassic.SpineControl"
ENT.EyeOffset = Vector(4, 0, 0)

-- Possession --
ENT.PossessionEnabled = true
ENT.PossessionCrosshair = true
ENT.PossessionViews = {
	{
		offset = Vector(0, 10, 10),
		distance = 50
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
			self:HeadcrabLeap(self:PossessorTrace().HitPos)
		end
	}}
}

if SERVER then

	-- Headcrab --

	function ENT:HeadcrabLeap(pos)
		self:FaceTo(pos)
		self.OnIdleSounds = {}
		self:PlaySequence("jumpattack_broadcast")
		self:PauseCoroutine(0.5)
		self.CanBite = true
		self:EmitSound("NPC_Headcrab.Attack")
		self:Leap(pos, 400)
		self.CanBite = false
		self.OnIdleSounds = {"NPC_HeadCrab.Idle"}
	end

	-- Init/Think --

	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_HT)
	end
	function ENT:CustomThink() end

	-- AI --

	function ENT:OnRangeAttack(enemy)
		self:HeadcrabLeap(enemy:EyePos()-Vector(0, 0, 10))
	end
	function ENT:OnContact(ent)
		if not IsValid(ent) then return end
		if self.CanBite and
		(self:IsPossessed() or ent == self:GetEnemy()) then
			self:EmitSound("NPC_HeadCrab.Bite")
			self.CanBite = false
			local dmg = DamageInfo()
			dmg:SetDamage(20)
			dmg:SetAttacker(self)
			dmg:SetInflictor(self)
			dmg:SetDamageType(DMG_SLASH)
			ent:TakeDamageInfo(dmg)
		end
	end

	function ENT:OnReachedPatrol(pos)
		self:Wait(math.random(3, 7))
	end
	function ENT:OnIdle()
		self:AddPatrolPos(self:RandomPos(1500))
	end

	-- Damage --

	function ENT:OnTakeDamage(dmg)
		local attacker = dmg:GetAttacker()
		if IsValid(attacker) and attacker:IsPlayer() then
			local weapon = attacker:GetActiveWeapon()
			if IsValid(weapon) and weapon:GetClass() == "weapon_crowbar" then
				return 2
			end
		end
	end

	-- Sounds --

	function ENT:OnNewEnemy()
		self:EmitSound("NPC_HeadCrab.Alert")
	end

end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### Understanding Ranged vs Melee NPCs

The key difference between ranged and melee NPCs is which attack range is configured:

```lua
ENT.RangeAttackRange = 150  -- This NPC uses ranged attacks
ENT.MeleeAttackRange = 0    -- No melee attacks
```

When `RangeAttackRange` is greater than 0, DrGBase will call `OnRangeAttack()` instead of `OnMeleeAttack()` when within range.

### Collision Bounds

```lua
ENT.CollisionBounds = Vector(12, 12, 24)
```

**Purpose:** Define custom collision hull size.
- Vector(width, width, height) in units
- Headcrab is small, so we use a smaller collision box
- Default collision would be too large for this tiny creature
- Affects pathfinding and collision detection

### AI Configuration - The Ranged Attack Pattern

```lua
ENT.RangeAttackRange = 150
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 125
ENT.AvoidEnemyRange = 100
```

**Purpose:** Create a "keep distance and leap" behavior.
- `RangeAttackRange = 150`: Attack when enemy is within 150 units
- `MeleeAttackRange = 0`: Disable melee attacks completely
- `ReachEnemyRange = 125`: Consider enemy "reached" at 125 units (so it doesn't try to get closer)
- `AvoidEnemyRange = 100`: Stay at least 100 units away (maintains distance)

**How it works together:**
1. Headcrab approaches enemy until reaching 125 units (ReachEnemyRange)
2. Stays at least 100 units away (AvoidEnemyRange)
3. When enemy is within 150 units, performs ranged attack (leaps)
4. This creates a "dance" where it maintains distance and leaps periodically

### Animation Setup

```lua
ENT.WalkAnimation = ACT_RUN
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE
ENT.JumpAnimation = ACT_IDLE
```

**Purpose:** Override default animations.
- Headcrab always "runs" (scuttles) even when walking
- Uses idle animation when jumping (because leap is sequence-based)
- Model has custom sequences for attacks

### The Leap Attack Function

```lua
function ENT:HeadcrabLeap(pos)
	self:FaceTo(pos)
	self.OnIdleSounds = {}
	self:PlaySequence("jumpattack_broadcast")
	self:PauseCoroutine(0.5)
	self.CanBite = true
	self:EmitSound("NPC_Headcrab.Attack")
	self:Leap(pos, 400)
	self.CanBite = false
	self.OnIdleSounds = {"NPC_HeadCrab.Idle"}
end
```

**Purpose:** Custom leap attack with telegraph and damage window.

**Step-by-step breakdown:**
1. `self:FaceTo(pos)` - Turn toward target
2. `self.OnIdleSounds = {}` - Temporarily disable idle sounds (prevents sound spam)
3. `self:PlaySequence("jumpattack_broadcast")` - Play wind-up animation (telegraph)
4. `self:PauseCoroutine(0.5)` - Wait 0.5 seconds (gives enemy time to react)
5. `self.CanBite = true` - Enable damage flag
6. `self:EmitSound("NPC_Headcrab.Attack")` - Play attack scream
7. `self:Leap(pos, 400)` - Leap toward position at 400 units/second
8. `self.CanBite = false` - Disable damage flag (only damages during flight)
9. `self.OnIdleSounds = {"NPC_HeadCrab.Idle"}` - Re-enable idle sounds

**Why this pattern?**
- Telegraph (broadcast animation) gives fair warning to players
- Damage window (CanBite flag) ensures only mid-air contact deals damage
- Sounds give audio feedback for each phase of attack

### Ranged Attack AI Hook

```lua
function ENT:OnRangeAttack(enemy)
	self:HeadcrabLeap(enemy:EyePos()-Vector(0, 0, 10))
end
```

**Purpose:** Called by DrGBase AI when in range to perform ranged attack.
- Leaps toward enemy's head position
- Subtracts 10 units on Z axis (aims slightly below head, at chest level)
- Makes leap more likely to connect with player's hitbox

### Contact Damage System

```lua
function ENT:OnContact(ent)
	if not IsValid(ent) then return end
	if self.CanBite and
	(self:IsPossessed() or ent == self:GetEnemy()) then
		self:EmitSound("NPC_HeadCrab.Bite")
		self.CanBite = false
		local dmg = DamageInfo()
		dmg:SetDamage(20)
		dmg:SetAttacker(self)
		dmg:SetInflictor(self)
		dmg:SetDamageType(DMG_SLASH)
		ent:TakeDamageInfo(dmg)
	end
end
```

**Purpose:** Deal damage when touching enemies during leap.

**How it works:**
- `OnContact` is called whenever the NPC touches another entity
- Only deals damage if `CanBite` is true (during leap)
- Only damages current enemy or when possessed (prevents accidental damage)
- Uses DamageInfo for precise damage control
- Immediately sets `CanBite = false` to prevent multiple hits from one leap

**Why DamageInfo instead of Attack()?**
- `Attack()` requires a position and direction (for area attacks)
- Contact damage needs to work on the entity we're already touching
- DamageInfo gives precise control for this specific use case

### Crowbar Weakness

```lua
function ENT:OnTakeDamage(dmg)
	local attacker = dmg:GetAttacker()
	if IsValid(attacker) and attacker:IsPlayer() then
		local weapon = attacker:GetActiveWeapon()
		if IsValid(weapon) and weapon:GetClass() == "weapon_crowbar" then
			return 2
		end
	end
end
```

**Purpose:** Easter egg - headcrabs take double damage from crowbar.
- Checks if damage is from a player
- Checks if player is using crowbar
- Returns 2 (200% damage multiplier)
- Faithful to Half-Life 2 mechanics

### Possession Controls

```lua
ENT.PossessionBinds = {
	[IN_ATTACK] = {{
		coroutine = true,
		onkeydown = function(self)
			self:HeadcrabLeap(self:PossessorTrace().HitPos)
		end
	}}
}
```

**Purpose:** Let players perform leap attacks when possessing.
- `coroutine = true`: Function can pause/wait (required for HeadcrabLeap)
- `PossessorTrace()`: Raycast from possessor's camera
- `.HitPos`: Where the camera is looking
- Reuses the same leap function AI uses

### Patrol Behavior

```lua
function ENT:OnReachedPatrol(pos)
	self:Wait(math.random(3, 7))
end
function ENT:OnIdle()
	self:AddPatrolPos(self:RandomPos(1500))
end
```

**Purpose:** Wandering behavior when not in combat (same as zombie example).

## Testing the NPC

### Spawning

1. Start Garry's Mod and load a map
2. Open spawn menu (Q)
3. Navigate to the "DrGBase" tab under NPCs
4. Click "Headcrab" to spawn

### Behavior to Observe

- **Idle:** Headcrab wanders randomly, occasionally making idle sounds
- **Detection:** Approaches enemy but maintains distance (doesn't get too close)
- **Attack:** Plays broadcast animation, pauses, then leaps at enemy
- **Contact:** Deals damage and bites when making contact mid-leap
- **Possession:** Use (E) to possess, then LMB to leap toward crosshair

### Testing Tips

- **Test the telegraph:** Watch the wind-up animation before leap
- **Test crowbar:** Spawn a headcrab and hit it with crowbar vs other weapons
- **Test range AI:** Notice how it keeps distance instead of rushing in
- **Test contact damage:** Observe that it only damages on contact during leap
- **Test possession:** Possess and leap at targets - very satisfying!

### Specific Scenarios

1. **Spawn multiple headcrabs** - They'll work together (same faction)
2. **Spawn on elevated platform** - They can leap down at you
3. **Spawn with zombies** - They're allies (FACTION_ZOMBIES)
4. **Possess and leap around** - Test the leap controls

## Customization Ideas

### Adjust Leap Power and Range

```lua
-- In HeadcrabLeap function:
self:Leap(pos, 600) -- Faster, more powerful leap

-- AI configuration:
ENT.RangeAttackRange = 300 -- Attack from farther away
ENT.AvoidEnemyRange = 150 -- Maintain more distance
```

### Add Different Headcrab Types

```lua
-- Fast headcrab (aggressive, low HP)
ENT.PrintName = "Fast Headcrab"
ENT.Models = {"models/headcrabclassic.mdl"}
ENT.SpawnHealth = 20
ENT.RangeAttackRange = 250
ENT.AvoidEnemyRange = 50 -- Gets closer

function ENT:HeadcrabLeap(pos)
	self:FaceTo(pos)
	self:PlaySequence("jumpattack_broadcast")
	self:PauseCoroutine(0.3) -- Shorter telegraph
	self.CanBite = true
	self:EmitSound("NPC_HeadCrab.Attack")
	self:Leap(pos, 600) -- Faster leap
	self.CanBite = false
end

-- Poison headcrab (slow, high damage)
ENT.PrintName = "Poison Headcrab"
ENT.Models = {"models/headcrabblack.mdl"}
ENT.SpawnHealth = 100
ENT.BloodColor = BLOOD_COLOR_GREEN

-- In OnContact, change damage:
dmg:SetDamage(50) -- Much more damage!
```

### Multiple Leaps Before Landing

```lua
function ENT:HeadcrabLeap(pos)
	self:FaceTo(pos)
	self:PlaySequence("jumpattack_broadcast")
	self:PauseCoroutine(0.5)

	-- Do 3 mini-leaps!
	for i = 1, 3 do
		self.CanBite = true
		self:EmitSound("NPC_HeadCrab.Attack")
		self:Leap(pos, 300)
		self:PauseCoroutine(0.3)
		self.CanBite = false
	end
end
```

### Poison Effect on Hit

```lua
function ENT:OnContact(ent)
	if not IsValid(ent) then return end
	if self.CanBite and (self:IsPossessed() or ent == self:GetEnemy()) then
		self:EmitSound("NPC_HeadCrab.Bite")
		self.CanBite = false

		local dmg = DamageInfo()
		dmg:SetDamage(20)
		dmg:SetAttacker(self)
		dmg:SetInflictor(self)
		dmg:SetDamageType(DMG_POISON) -- Poison damage type!
		ent:TakeDamageInfo(dmg)

		-- Reduce player health to 1 over time (like HL2 poison headcrab)
		if ent:IsPlayer() then
			timer.Simple(0.5, function()
				if IsValid(ent) and ent:Health() > 1 then
					ent:SetHealth(math.max(1, ent:Health() - 10))
				end
			end)
		end
	end
end
```

### Add Aerial Maneuverability

```lua
-- At the top with other stats:
ENT.LeapControlSpeed = 50 -- How fast can adjust direction mid-leap

function ENT:CustomThink()
	-- If in air and has enemy, slightly adjust toward them
	if not self:IsOnGround() and IsValid(self:GetEnemy()) then
		local toEnemy = (self:GetEnemy():GetPos() - self:GetPos()):GetNormalized()
		local currentVel = self:GetVelocity()
		local newVel = currentVel + (toEnemy * self.LeapControlSpeed)
		self:SetVelocity(newVel)
	end
end
```

### Different Models/Skins

```lua
-- Random headcrab color variation
function ENT:CustomInitialize()
	self:SetDefaultRelationship(D_HT)
	self:SetColor(Color(
		math.random(200, 255),
		math.random(150, 200),
		math.random(100, 150)
	))
end
```

## Troubleshooting

### Headcrab doesn't leap, just stands there

**Problem:** AI not triggering ranged attack.

**Solutions:**
- Verify `RangeAttackRange = 150` (not 0)
- Check that `MeleeAttackRange = 0` (should be disabled)
- Ensure `ReachEnemyRange` is less than `RangeAttackRange`
- Spawn an enemy NPC to test combat behavior
- Check console for errors in HeadcrabLeap function

### Leap is too short/doesn't reach target

**Problem:** Leap power too low or being blocked.

**Solutions:**
- Increase leap speed: `self:Leap(pos, 600)` instead of 400
- Check if something is blocking the path
- Ensure target position is valid (not inside geometry)
- Try adjusting the target height offset in OnRangeAttack

### Deals damage even when not leaping

**Problem:** CanBite flag not being managed correctly.

**Solutions:**
- Make sure `CanBite = false` after dealing damage
- Check that CanBite is only set to true during HeadcrabLeap
- Add debug prints: `print("CanBite:", self.CanBite)` in OnContact
- Ensure only one leap is happening at a time

### Damages allies or random entities

**Problem:** OnContact hitting wrong targets.

**Solutions:**
- Check the condition: `ent == self:GetEnemy()`
- Verify faction relationships are set correctly
- Add additional checks for entity class or type
- Consider adding a small cooldown timer

### Headcrab gets stuck after leaping

**Problem:** Landing in bad position or on props.

**Solutions:**
- DrGBase handles this automatically, but if stuck persists:
- Add `self:DropToFloor()` after leap
- Check collision bounds aren't too large
- Ensure navmesh exists on the map (some maps don't have it)

### Multiple damage instances from one leap

**Problem:** OnContact called multiple times per collision.

**Solutions:**
- The `CanBite = false` inside OnContact should prevent this
- If still happening, add a timer cooldown:
```lua
if self.CanBite and not self.DamageOnCooldown then
	self.DamageOnCooldown = true
	-- damage code here
	timer.Simple(0.5, function()
		if IsValid(self) then
			self.DamageOnCooldown = false
		end
	end)
end
```

### Possessed leap doesn't work

**Problem:** Possession keybind or coroutine issue.

**Solutions:**
- Verify `coroutine = true` in PossessionBinds
- Check that `PossessorTrace()` returns valid data
- Make sure HeadcrabLeap function can pause (uses PauseCoroutine)
- Test with different keys to rule out input issues

### Crowbar doesn't do double damage

**Problem:** OnTakeDamage not returning multiplier correctly.

**Solutions:**
- Ensure function returns 2 (the multiplier)
- Check player actually has crowbar equipped
- Verify weapon class is exactly "weapon_crowbar"
- Add debug print to see if condition is met:
```lua
print("Weapon:", weapon:GetClass())
```

## See Also

- [Creating NPCs Guide](../guides/creating-npcs.md) - Comprehensive NPC creation tutorial
- [Simple Melee NPC Example](simple-melee-npc.md) - Basic melee combat
- [Flying NPC Example](flying-npc.md) - Advanced aerial movement with leaping
- [Boss NPC Example](boss-npc.md) - Create tougher boss NPCs
- [Movement API](../api/nextbot/movement.md) - Leap and movement functions
- [Combat API](../api/nextbot/weapons.md) - Attack and damage systems
- [AI API](../api/nextbot/ai.md) - AI behavior and attack ranges
