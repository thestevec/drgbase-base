# Simple Melee NPC Example

A complete, working example of a basic melee combat NPC using DrGBase - modeled after Half-Life 2's zombie.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the NPC](#testing-the-npc)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)

## Overview

This example creates a simple zombie NPC that:
- Walks toward enemies and performs melee attacks
- Plays attack animations during combat
- Has patrol behavior when idle
- Spawns a headcrab when killed (if not headshot)
- Can be possessed by players
- Has proper sounds and visual effects

**Difficulty:** Beginner
**File Location:** `lua/entities/npc_drg_myzombie.lua`
**Base:** `drgbase_nextbot`

## Complete Code

Create a file named `npc_drg_myzombie.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot" -- DO NOT TOUCH (obviously)

-- Misc --
ENT.PrintName = "My Zombie"
ENT.Category = "DrGBase"
ENT.Models = {"models/Zombie/Classic.mdl"}
ENT.BloodColor = BLOOD_COLOR_GREEN

-- Sounds --
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- Stats --
ENT.SpawnHealth = 100

-- AI --
ENT.RangeAttackRange = 0
ENT.MeleeAttackRange = 30
ENT.ReachEnemyRange = 30
ENT.AvoidEnemyRange = 0

-- Relationships --
ENT.Factions = {FACTION_ZOMBIES}

-- Movements/animations --
ENT.UseWalkframes = true
ENT.RunAnimation = ACT_WALK

-- Detection --
ENT.EyeBone = "ValveBiped.Bip01_Spine4"
ENT.EyeOffset = Vector(7.5, 0, 5)

-- Possession --
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
	{
		offset = Vector(0, 30, 20),
		distance = 100
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
			self:EmitSound("Zombie.Attack")
			self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.PossessionFaceForward)
		end
	}}
}

if SERVER then

	-- Init/Think --

	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_HT)
		self:SetBodygroup(1, 1)
	end

	-- AI --

	function ENT:OnMeleeAttack(enemy)
		self:EmitSound("Zombie.Attack")
		self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
	end

	function ENT:OnReachedPatrol()
		self:Wait(math.random(3, 7))
	end
	function ENT:OnIdle()
		self:AddPatrolPos(self:RandomPos(1500))
	end

	-- Damage --

	function ENT:OnDeath(dmg, delay, hitgroup)
		if hitgroup ~= HITGROUP_HEAD then
			self:SetBodygroup(1, 0)
			local headcrab = ents.Create("npc_drg_headcrab")
			if not IsValid(headcrab) then return end
			headcrab:SetPos(self:EyePos())
			headcrab:SetAngles(self:GetAngles())
			headcrab:Spawn()
			if IsValid(self:GetCreator()) then
				self:GetCreator():DrG_AddUndo(headcrab, "NPC", "Undone Headcrab")
			end
		end
	end

	-- Animations/Sounds --

	function ENT:OnNewEnemy()
		self:EmitSound("Zombie.Alert")
	end

	function ENT:OnAnimEvent()
		if self:IsAttacking() and self:GetCycle() > 0.3 then
			self:Attack({
				damage = 10,
				type = DMG_SLASH,
				viewpunch = Angle(20, math.random(-10, 10), 0)
			}, function(self, hit)
				if #hit > 0 then
					self:EmitSound("Zombie.AttackHit")
				else self:EmitSound("Zombie.AttackMiss") end
			end)
		elseif math.random(2) == 1 then
			self:EmitSound("Zombie.FootstepLeft")
		else self:EmitSound("Zombie.FootstepRight") end
	end

end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### File Header

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot" -- DO NOT TOUCH (obviously)
```

**Purpose:** Safety check and base class definition.
- First line prevents errors if DrGBase isn't installed
- Second line sets the base class to inherit all DrGBase functionality

### Basic Properties

```lua
ENT.PrintName = "My Zombie"
ENT.Category = "DrGBase"
ENT.Models = {"models/Zombie/Classic.mdl"}
ENT.BloodColor = BLOOD_COLOR_GREEN
```

**Purpose:** Define the NPC's appearance and spawn menu info.
- `PrintName`: Name shown in spawn menu
- `Category`: Which tab it appears under
- `Models`: Table of models to randomly choose from (we only have one)
- `BloodColor`: Blood particle color when damaged

### Sound Configuration

```lua
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}
```

**Purpose:** Sounds automatically played on specific events.
- `OnDamageSounds`: Played when taking damage
- `OnDeathSounds`: Played when dying
- Can be a table of multiple sounds for variety

### Stats

```lua
ENT.SpawnHealth = 100
```

**Purpose:** Health the NPC spawns with. Default is 100 if not specified.

### AI Configuration

```lua
ENT.RangeAttackRange = 0
ENT.MeleeAttackRange = 30
ENT.ReachEnemyRange = 30
ENT.AvoidEnemyRange = 0
```

**Purpose:** Define combat behavior ranges.
- `RangeAttackRange = 0`: No ranged attacks (melee only)
- `MeleeAttackRange = 30`: Perform melee attacks within 30 units
- `ReachEnemyRange = 30`: Consider enemy "reached" at 30 units
- `AvoidEnemyRange = 0`: Never retreat from enemies

### Relationships

```lua
ENT.Factions = {FACTION_ZOMBIES}
```

**Purpose:** Define which faction(s) this NPC belongs to.
- Zombies hate players and most NPCs by default
- NPCs in the same faction are friendly to each other
- See [Faction System Example](faction-system.md) for more details

### Movement & Animation

```lua
ENT.UseWalkframes = true
ENT.RunAnimation = ACT_WALK
```

**Purpose:** Control movement behavior.
- `UseWalkframes`: Use model's animation speed to control movement speed
- `RunAnimation`: Override run animation (zombie shambles instead of runs)

### Detection

```lua
ENT.EyeBone = "ValveBiped.Bip01_Spine4"
ENT.EyeOffset = Vector(7.5, 0, 5)
```

**Purpose:** Define where the NPC "sees" from.
- `EyeBone`: Bone to attach eye position to (chest bone)
- `EyeOffset`: Offset from that bone (forward 7.5, up 5)
- Affects vision and line-of-sight detection

### Possession System

```lua
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
	{
		offset = Vector(0, 30, 20),
		distance = 100
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
			self:EmitSound("Zombie.Attack")
			self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.PossessionFaceForward)
		end
	}}
}
```

**Purpose:** Allow players to control the NPC.
- `PossessionEnabled`: Enable/disable possession
- `PossessionMovement`: 8-directional movement (WASD + diagonals)
- `PossessionViews`: Camera positions (third-person and first-person)
- `PossessionBinds`: Attack key plays attack animation

### Server-Side Code

All gameplay logic runs server-side (inside `if SERVER then`):

#### Initialization

```lua
function ENT:CustomInitialize()
	self:SetDefaultRelationship(D_HT)
	self:SetBodygroup(1, 1)
end
```

**Purpose:** Setup called when NPC spawns.
- `SetDefaultRelationship(D_HT)`: Hate everything by default
- `SetBodygroup(1, 1)`: Show the headcrab on top (bodygroup 1)

#### Melee Attack

```lua
function ENT:OnMeleeAttack(enemy)
	self:EmitSound("Zombie.Attack")
	self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

**Purpose:** Called when AI decides to melee attack.
- Play attack sound
- Play attack animation while continuing to face enemy
- Animation is synchronized with movement

#### Patrol Behavior

```lua
function ENT:OnReachedPatrol()
	self:Wait(math.random(3, 7))
end
function ENT:OnIdle()
	self:AddPatrolPos(self:RandomPos(1500))
end
```

**Purpose:** Wandering behavior when not in combat.
- `OnReachedPatrol`: Wait 3-7 seconds when reaching patrol point
- `OnIdle`: When bored, pick a random position within 1500 units to walk to

#### Death Behavior

```lua
function ENT:OnDeath(dmg, delay, hitgroup)
	if hitgroup ~= HITGROUP_HEAD then
		self:SetBodygroup(1, 0)
		local headcrab = ents.Create("npc_drg_headcrab")
		if not IsValid(headcrab) then return end
		headcrab:SetPos(self:EyePos())
		headcrab:SetAngles(self:GetAngles())
		headcrab:Spawn()
		if IsValid(self:GetCreator()) then
			self:GetCreator():DrG_AddUndo(headcrab, "NPC", "Undone Headcrab")
		end
	end
end
```

**Purpose:** Spawn headcrab on death (unless killed by headshot).
- Check if death wasn't from headshot
- Hide headcrab bodygroup
- Create and spawn headcrab entity
- Add to undo list if spawned by a player

#### Alert Sound

```lua
function ENT:OnNewEnemy()
	self:EmitSound("Zombie.Alert")
end
```

**Purpose:** Play alert sound when spotting a new enemy.

#### Animation Events

```lua
function ENT:OnAnimEvent()
	if self:IsAttacking() and self:GetCycle() > 0.3 then
		self:Attack({
			damage = 10,
			type = DMG_SLASH,
			viewpunch = Angle(20, math.random(-10, 10), 0)
		}, function(self, hit)
			if #hit > 0 then
				self:EmitSound("Zombie.AttackHit")
			else self:EmitSound("Zombie.AttackMiss") end
		end)
	elseif math.random(2) == 1 then
		self:EmitSound("Zombie.FootstepLeft")
	else self:EmitSound("Zombie.FootstepRight") end
end
```

**Purpose:** Called on every animation frame event.
- During attack animation (at 30% through): Deal damage
- Attack deals 10 slash damage with screen shake
- Play hit/miss sounds based on callback
- On footstep events: Play footstep sounds

### File Footer

```lua
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Purpose:** Register the NPC with the system.
- `AddCSLuaFile()`: Send this file to clients
- `DrGBase.AddNextbot(ENT)`: Register with DrGBase framework

## Testing the NPC

### Spawning

1. Start Garry's Mod and load a map
2. Open spawn menu (Q)
3. Navigate to the "DrGBase" tab under NPCs
4. Click "My Zombie" to spawn

### Behavior to Observe

- **Idle:** Zombie wanders randomly
- **Combat:** Approaches and attacks enemies with melee
- **Sounds:** Growls, alert sounds, attack sounds, footsteps
- **Death:** Spawns headcrab when killed (except headshots)
- **Possession:** Press USE (default E) near zombie, then press R to possess

### Testing Tips

- Spawn multiple zombies to see faction behavior
- Spawn enemies (NPCs or rebels) to test combat
- Try killing with headshots vs body shots
- Test possession controls (WASD to move, LMB to attack)

## Customization Ideas

### Adjust Stats

```lua
ENT.SpawnHealth = 200 -- Tougher zombie
ENT.MeleeAttackRange = 50 -- Longer reach
```

### Change Appearance

```lua
ENT.Models = {
	"models/Zombie/Classic.mdl",
	"models/Zombie/Fast.mdl",
	"models/Zombie/Poison.mdl"
} -- Random zombie types

ENT.Skins = {0, 1, 2} -- Random skins
```

### Modify Attack

```lua
function ENT:OnMeleeAttack(enemy)
	self:EmitSound("Zombie.Attack")
	-- Use different attack animations
	local attacks = {ACT_MELEE_ATTACK1, ACT_MELEE_ATTACK2}
	self:PlayActivityAndMove(table.Random(attacks), 1, self.FaceEnemy)
end

-- In OnAnimEvent, change damage:
self:Attack({
	damage = 25, -- More damage!
	type = DMG_SLASH,
	viewpunch = Angle(30, math.random(-20, 20), 0) -- More screen shake
})
```

### Add Speed Variation

```lua
function ENT:CustomInitialize()
	self:SetDefaultRelationship(D_HT)
	self:SetBodygroup(1, 1)

	-- Random speed: slow or fast zombie
	if math.random(2) == 1 then
		self.RunSpeed = 80 -- Fast zombie
		self.WalkSpeed = 60
		self:SetModel("models/Zombie/Fast.mdl")
	else
		self.RunSpeed = 40 -- Slow zombie
		self.WalkSpeed = 30
	end
end
```

### Remove Headcrab on Death

```lua
function ENT:OnDeath(dmg, delay, hitgroup)
	-- Comment out or remove the entire function to disable headcrab spawning
end
```

## Troubleshooting

### NPC doesn't appear in spawn menu

**Solution:** Make sure the file is in the correct location:
- `garrysmod/lua/entities/npc_drg_myzombie.lua`
- Restart Garry's Mod after creating the file

### NPC spawns but doesn't move

**Problem:** Animation or movement configuration issue.

**Solutions:**
- Check console for errors
- Verify the model path is correct
- Make sure `ENT.Models` contains valid model paths
- Try removing `UseWalkframes = true`

### NPC doesn't attack

**Problem:** Range configuration or AI issue.

**Solutions:**
- Verify `MeleeAttackRange` is set (not 0)
- Check that `ReachEnemyRange` matches `MeleeAttackRange`
- Ensure faction relationships are hostile (see factions)
- Spawn an enemy NPC to test combat

### Attack doesn't deal damage

**Problem:** `OnAnimEvent` timing or `Attack()` function issue.

**Solutions:**
- Check that `GetCycle() > 0.3` timing works for your animation
- Try different timing values (0.2, 0.4, 0.5)
- Add `print()` statements to debug when damage is triggered
- Verify the attack range matches your animation reach

### Headcrab doesn't spawn on death

**Problem:** Entity name or spawn issue.

**Solutions:**
- Make sure `npc_drg_headcrab` exists (comes with DrGBase)
- Check console for errors about entity creation
- Verify you're not killing with headshots (intentional behavior)

### Possession doesn't work

**Problem:** Possession configuration or input issue.

**Solutions:**
- Ensure `PossessionEnabled = true`
- Check that you're pressing USE (E) to possess
- Press R after possessing to cycle camera views
- Verify possession keybinds are configured correctly

## See Also

- [Creating NPCs Guide](../guides/creating-npcs.md) - Comprehensive NPC creation tutorial
- [Ranged NPC Example](ranged-npc.md) - Add ranged attacks
- [Boss NPC Example](boss-npc.md) - Create tougher boss NPCs
- [Faction System Example](faction-system.md) - Complex faction relationships
- [Animation API](../api/nextbot/animation.md) - Animation system reference
- [Combat API](../api/nextbot/weapons.md) - Combat system reference
