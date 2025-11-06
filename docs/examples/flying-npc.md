# Flying NPC Example

A complete, working example of a flying/leaping NPC with aerial combat using DrGBase - modeled after Half-Life 2's antlion.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the NPC](#testing-the-npc)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)

## Overview

This example creates an antlion NPC that:
- Uses both ranged leaping and melee attacks
- Opens wings and plays sounds when airborne
- Closes wings when landing
- Has burrow ability (teleport with animations)
- Supports multiple attack animations
- Uses bodygroups to show/hide wings
- Can be possessed with jump, attack, and burrow controls
- Follows players when allied

**Difficulty:** Advanced
**File Location:** `lua/entities/npc_drg_antlion.lua`
**Base:** `drgbase_nextbot`
**Key Concepts:** Aerial movement, bodygroups, looping sounds, dual combat modes

## Complete Code

Create a file named `npc_drg_antlion.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot" -- DO NOT TOUCH (obviously)

-- Misc --
ENT.PrintName = "Antlion"
ENT.Category = "DrGBase"
ENT.Models = {"models/Antlion.mdl"}
ENT.Skins = {0, 1, 2, 3}
ENT.CollisionBounds = Vector(30, 30, 60)
ENT.BloodColor = BLOOD_COLOR_YELLOW

-- Sounds --
ENT.OnDamageSounds = {"NPC_Antlion.Pain"}

-- Stats --
ENT.SpawnHealth = 40

-- AI --
ENT.RangeAttackRange = 750
ENT.MeleeAttackRange = 50
ENT.ReachEnemyRange = 50
ENT.AvoidEnemyRange = 0
ENT.FollowPlayers = true

-- Relationships --
ENT.Factions = {FACTION_ANTLIONS}

-- Movements/animations --
ENT.UseWalkframes = true
ENT.JumpAnimation = ACT_GLIDE

-- Detection --
ENT.EyeBone = "Antlion.Head_Bone"
ENT.EyeOffset = Vector(7.5, 0, 5)
ENT.EyeAngle = Angle(0, 0, 0)

-- Possession --
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionCrosshair = true
ENT.PossessionViews = {
	{
		offset = Vector(0, 40, 0),
		distance = 125
	},
	{
		offset = Vector(7.5, 0, 10),
		distance = 0,
		eyepos = true
	}
}
ENT.PossessionBinds = {
	[IN_JUMP] = {{
		coroutine = false,
		onkeypressed = function(self)
			if not self:IsOnGround() then return end
			self:LeaveGround()
			self:SetVelocity(self:PossessorNormal()*1500)
		end
	}},
	[IN_ATTACK] = {{
		coroutine = true,
		onkeydown = function(self)
			self:PlaySequenceAndMove("attack"..math.random(6), 1, self.PossessionFaceForward)
		end
	}},
	[IN_ATTACK2] = {{
		coroutine = true,
		onkeydown = function(self)
			self:BurrowTo(self:PossessorTrace().HitPos)
		end
	}}
}

if SERVER then

	-- Antlion --

	function ENT:BurrowTo(pos)
		self:PlaySequenceAndMove("digin")
		if navmesh.IsLoaded() then
			pos = navmesh.GetNearestNavArea(pos):GetClosestPointOnArea(pos) or pos
		end
		self:SetPos(pos)
		self:DropToFloor()
		self:PlaySequenceAndMove("digout")
	end

	-- Init/Think --

	function ENT:CustomInitialize()
		self:SetDefaultRelationship(D_HT)
	end

	-- AI --

	function ENT:OnRangeAttack(enemy)
		if not self:IsInRange(enemy, 500) then
			self:Leap(enemy, 1000)
			self:PauseCoroutine(0.25)
		end
	end
	function ENT:OnMeleeAttack(enemy)
		self:PlaySequenceAndMove("attack"..math.random(6), 1, self.FaceEnemy)
	end

	function ENT:OnReachedPatrol()
		self:Wait(math.random(3, 7))
	end
	function ENT:OnIdle()
		self:AddPatrolPos(self:RandomPos(1500))
	end

	-- Animations/Sounds --

	function ENT:OnLeaveGround()
		self:SetBodygroup(1, 1)
		self:PlayActivity(ACT_JUMP)
		self.WingsOpen = self:StartLoopingSound("NPC_Antlion.WingsOpen")
	end
	function ENT:OnLandOnGround()
		self:SetBodygroup(1, 0)
		if isnumber(self.WingsOpen) then
			self:StopLoopingSound(self.WingsOpen)
			self:EmitSlotSound("Landing", 1, "NPC_Antlion.Land")
		end
	end
	function ENT:OnRemove()
		if isnumber(self.WingsOpen) then
			self:StopLoopingSound(self.WingsOpen)
		end
	end

	function ENT:OnAnimEvent()
		if self:IsAttacking() then
			if self:GetCycle() > 0.3 then
				self:Attack({
					damage = 5,
					range = 50,
					type = DMG_SLASH,
					viewpunch = Angle(10, 0, 0)
				}, function(self, hit)
					if #hit > 0 then self:EmitSound("NPC_Antlion.MeleeAttack") end
				end)
			else self:EmitSound("NPC_Antlion.MeleeAttackSingle") end
		elseif self:IsOnGround() then
			self:EmitSound("NPC_Antlion.Footstep")
		end
	end

end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### Understanding Flying/Aerial NPCs

This NPC demonstrates three key flying NPC concepts:
1. **Dual Combat Modes:** Both ranged (leap) and melee attacks
2. **Aerial State Management:** Different behavior when airborne vs grounded
3. **Visual/Audio Feedback:** Wings and sounds that respond to flight state

### Dual Combat System

```lua
ENT.RangeAttackRange = 750
ENT.MeleeAttackRange = 50
ENT.ReachEnemyRange = 50
ENT.AvoidEnemyRange = 0
```

**Purpose:** Enable both attack types at different ranges.
- `RangeAttackRange = 750`: Can leap from very far away
- `MeleeAttackRange = 50`: Can also do close-range attacks
- `ReachEnemyRange = 50`: Tries to get close to enemy
- `AvoidEnemyRange = 0`: Never retreats (aggressive)

**How it works together:**
1. If enemy is beyond 500 units: Leap toward them (ranged attack)
2. If enemy is within 50 units: Use melee attack
3. This creates dynamic combat - leap to close distance, then claw attacks

### Follow Players

```lua
ENT.FollowPlayers = true
```

**Purpose:** Makes antlion follow players when not hostile.
- Combined with faction settings, can create friendly/pet antlions
- Useful for escort missions or allied NPCs
- Automatically paths toward nearby players

### Collision Bounds

```lua
ENT.CollisionBounds = Vector(30, 30, 60)
```

**Purpose:** Larger collision hull for bigger creature.
- Antlions are bigger than headcrabs, need appropriate collision size
- Vector(width, width, height) in units
- Affects pathfinding, doorway navigation, and collision detection

### Jump Animation

```lua
ENT.JumpAnimation = ACT_GLIDE
```

**Purpose:** Use glide animation when airborne.
- Makes antlion look like it's flying/gliding instead of falling
- Default would use jump animation which looks wrong for extended flight
- Combines with wing bodygroup for authentic flying appearance

### The Burrow Function

```lua
function ENT:BurrowTo(pos)
	self:PlaySequenceAndMove("digin")
	if navmesh.IsLoaded() then
		pos = navmesh.GetNearestNavArea(pos):GetClosestPointOnArea(pos) or pos
	end
	self:SetPos(pos)
	self:DropToFloor()
	self:PlaySequenceAndMove("digout")
end
```

**Purpose:** Teleport with dig animations for cinematic underground movement.

**Step-by-step breakdown:**
1. `self:PlaySequenceAndMove("digin")` - Play dig-in animation (antlion burrows underground)
2. `if navmesh.IsLoaded()` - Check if navigation mesh exists
3. `GetNearestNavArea(pos)` - Find nearest valid navigation area
4. `GetClosestPointOnArea(pos)` - Snap position to walkable space
5. `self:SetPos(pos)` - Teleport to target position
6. `self:DropToFloor()` - Ensure on ground (not floating)
7. `self:PlaySequenceAndMove("digout")` - Play dig-out animation

**Why this pattern?**
- Prevents teleporting into walls or invalid spaces
- Navmesh ensures target position is walkable
- Animations make teleport feel natural (underground travel)
- Fallback to original pos if navmesh not available

### Dual Attack AI

```lua
function ENT:OnRangeAttack(enemy)
	if not self:IsInRange(enemy, 500) then
		self:Leap(enemy, 1000)
		self:PauseCoroutine(0.25)
	end
end
function ENT:OnMeleeAttack(enemy)
	self:PlaySequenceAndMove("attack"..math.random(6), 1, self.FaceEnemy)
end
```

**Purpose:** Different attacks based on distance.

**Ranged Attack (leap):**
- Only leaps if enemy is beyond 500 units
- Even though `RangeAttackRange = 750`, adds distance check
- This prevents leaping when already close
- `Leap(enemy, 1000)` - Leaps directly at enemy entity at high speed
- `PauseCoroutine(0.25)` - Brief pause after leap

**Melee Attack:**
- `"attack"..math.random(6)` - Picks random attack animation (1-6)
- Creates visual variety - each attack looks different
- `PlaySequenceAndMove` - Plays animation while moving
- `FaceEnemy` - Keeps facing enemy during animation

**Combined behavior:**
1. If far away: Leap to close distance
2. If close: Use claw attacks
3. Creates "pounce then maul" behavior pattern

### Wing Animation System

```lua
function ENT:OnLeaveGround()
	self:SetBodygroup(1, 1)
	self:PlayActivity(ACT_JUMP)
	self.WingsOpen = self:StartLoopingSound("NPC_Antlion.WingsOpen")
end
function ENT:OnLandOnGround()
	self:SetBodygroup(1, 0)
	if isnumber(self.WingsOpen) then
		self:StopLoopingSound(self.WingsOpen)
		self:EmitSlotSound("Landing", 1, "NPC_Antlion.Land")
	end
end
```

**Purpose:** Visual and audio feedback for flight state.

**When leaving ground:**
- `SetBodygroup(1, 1)` - Show wings (bodygroup 1, variant 1)
- `PlayActivity(ACT_JUMP)` - Play jump/glide animation
- `StartLoopingSound()` - Play continuous wing buzzing sound
- Stores sound ID in `self.WingsOpen` for later cleanup

**When landing:**
- `SetBodygroup(1, 0)` - Hide wings (bodygroup 1, variant 0)
- `isnumber(self.WingsOpen)` - Check if wing sound is playing
- `StopLoopingSound()` - Stop the wing sound
- `EmitSlotSound()` - Play landing impact sound

**Why looping sounds?**
- Normal `EmitSound()` plays once then stops
- Looping sounds continue until manually stopped
- Perfect for ongoing states like flying
- Must store ID to stop the specific sound instance

### Cleanup on Remove

```lua
function ENT:OnRemove()
	if isnumber(self.WingsOpen) then
		self:StopLoopingSound(self.WingsOpen)
	end
end
```

**Purpose:** Stop looping sounds when NPC is deleted.
- Prevents sounds from continuing after entity removed
- Important for preventing audio bugs
- Always clean up looping sounds in OnRemove

### Attack Animation Events

```lua
function ENT:OnAnimEvent()
	if self:IsAttacking() then
		if self:GetCycle() > 0.3 then
			self:Attack({
				damage = 5,
				range = 50,
				type = DMG_SLASH,
				viewpunch = Angle(10, 0, 0)
			}, function(self, hit)
				if #hit > 0 then self:EmitSound("NPC_Antlion.MeleeAttack") end
			end)
		else self:EmitSound("NPC_Antlion.MeleeAttackSingle") end
	elseif self:IsOnGround() then
		self:EmitSound("NPC_Antlion.Footstep")
	end
end
```

**Purpose:** Synchronized damage and sounds with animations.

**During attack animations:**
- If 30% through animation: Deal damage (timing matches claw swing)
- If hit something: Play melee hit sound (callback)
- Otherwise: Play attack windup sound

**During movement:**
- If on ground: Play footstep sound
- Creates authentic walking audio

### Possession Controls - Three Actions

```lua
ENT.PossessionBinds = {
	[IN_JUMP] = {{
		coroutine = false,
		onkeypressed = function(self)
			if not self:IsOnGround() then return end
			self:LeaveGround()
			self:SetVelocity(self:PossessorNormal()*1500)
		end
	}},
	[IN_ATTACK] = {{
		coroutine = true,
		onkeydown = function(self)
			self:PlaySequenceAndMove("attack"..math.random(6), 1, self.PossessionFaceForward)
		end
	}},
	[IN_ATTACK2] = {{
		coroutine = true,
		onkeydown = function(self)
			self:BurrowTo(self:PossessorTrace().HitPos)
		end
	}}
}
```

**Purpose:** Three different player controls.

**Jump (Space):**
- `onkeypressed` - Triggers once per keypress
- `coroutine = false` - Doesn't pause, instant action
- `if not self:IsOnGround() then return end` - Only jump when grounded
- `LeaveGround()` - Manually trigger OnLeaveGround hook
- `PossessorNormal()` - Direction player's camera is facing
- `SetVelocity(...*1500)` - Launch in that direction

**Attack (Left Click):**
- `onkeydown` - Triggers continuously while held
- `coroutine = true` - Can pause during animation
- Random attack animation (1-6)
- `PossessionFaceForward` - Continue facing forward during attack

**Burrow (Right Click):**
- `onkeydown` - Hold to activate
- `coroutine = true` - Required (burrow function pauses)
- `PossessorTrace().HitPos` - Where camera is looking
- Reuses the burrow function

**Why this is fun:**
- Space to fly in direction you're looking
- LMB for melee attacks
- RMB to burrow/teleport to crosshair
- Very dynamic and satisfying to control

## Testing the NPC

### Spawning

1. Start Garry's Mod and load a map
2. Open spawn menu (Q)
3. Navigate to the "DrGBase" tab under NPCs
4. Click "Antlion" to spawn

### Behavior to Observe

- **Idle:** Wanders randomly with random skin variations
- **Detection:** Rushes toward enemies
- **Far Range:** Leaps through air toward distant enemies
- **Close Range:** Uses claw attacks with varied animations
- **Flying:** Wings extend, buzzing sound plays
- **Landing:** Wings retract, landing thud sound
- **Possession:** Full control with jump, attack, and burrow

### Testing Tips

- **Test leaping:** Spawn far from enemy, watch it leap to close distance
- **Test melee:** Watch attack animation variety (6 different attacks)
- **Test wings:** Watch bodygroup change when jumping/landing
- **Test burrow:** Use possession and right-click to burrow around map
- **Test sounds:** Listen for wing buzz, landing sounds, footsteps

### Specific Scenarios

1. **Spawn on high platform** - Watch it leap down with wings extended
2. **Spawn multiple antlions** - They work together (same faction)
3. **Possess and fly around** - Most fun aspect, very satisfying controls
4. **Test burrow on different terrain** - See navmesh validation in action
5. **Listen to sound cleanup** - Delete while flying, wing sound should stop

## Customization Ideas

### Adjust Flight Speed and Distance

```lua
function ENT:OnRangeAttack(enemy)
	if not self:IsInRange(enemy, 800) then -- Leap from farther
		self:Leap(enemy, 1500) -- Faster leap
		self:PauseCoroutine(0.25)
	end
end

-- Possession:
self:SetVelocity(self:PossessorNormal()*2000) -- Faster flight
```

### Guardian Antlion (Bigger, Tougher)

```lua
ENT.PrintName = "Antlion Guardian"
ENT.SpawnHealth = 300
ENT.CollisionBounds = Vector(50, 50, 100) -- Bigger

-- Scale the model:
function ENT:CustomInitialize()
	self:SetDefaultRelationship(D_HT)
	self:SetModelScale(1.5) -- 50% bigger
end

-- In OnAnimEvent, increase damage:
self:Attack({
	damage = 15, -- More damage
	range = 75, -- Longer reach
	type = DMG_SLASH,
	viewpunch = Angle(20, math.random(-10, 10), 0) -- More screen shake
})
```

### Flying Swarm Behavior

```lua
-- Add to top:
ENT.SwarmHeight = 200 -- Preferred flying height

function ENT:CustomThink()
	-- Try to maintain height when near enemy
	if IsValid(self:GetEnemy()) and self:IsInRange(self:GetEnemy(), 500) then
		local heightDiff = self.SwarmHeight - (self:GetPos().z - self:GetEnemy():GetPos().z)
		if math.abs(heightDiff) > 50 then
			-- Adjust height
			local newVel = self:GetVelocity()
			newVel.z = heightDiff * 0.5
			self:SetVelocity(newVel)
		end
	end
end
```

### Acid Spit Ranged Attack

```lua
function ENT:OnRangeAttack(enemy)
	if not self:IsInRange(enemy, 500) then
		-- Long range: Leap
		self:Leap(enemy, 1000)
		self:PauseCoroutine(0.25)
	else
		-- Medium range: Spit acid
		self:FaceTowards(enemy:GetPos())
		self:PlaySequence("spit") -- If model has spit anim
		self:PauseCoroutine(0.3)

		local projectile = ents.Create("grenade_spit") -- Acid spit entity
		if IsValid(projectile) then
			projectile:SetPos(self:EyePos())
			projectile:SetOwner(self)
			projectile:Spawn()
			projectile:SetVelocity((enemy:EyePos() - self:EyePos()):GetNormalized() * 1000)
		end
	end
end
```

### Burrow Attack

```lua
-- Burrow toward enemy, emerge underneath them
function ENT:OnRangeAttack(enemy)
	if math.random(3) == 1 then -- 33% chance
		-- Burrow attack!
		self:BurrowTo(enemy:GetPos())
		-- Emerge and immediately attack
		self:PlaySequenceAndMove("attack"..math.random(6), 1, self.FaceEnemy)
	else
		-- Normal leap
		if not self:IsInRange(enemy, 500) then
			self:Leap(enemy, 1000)
			self:PauseCoroutine(0.25)
		end
	end
end
```

### Charge Attack

```lua
-- Add to top:
ENT.ChargeSpeed = 500

function ENT:ChargeAttack(enemy)
	self:FaceTo(enemy:GetPos())
	self:PlayActivity(ACT_RUN)
	self:EmitSound("NPC_Antlion.Distracted")

	local chargeTime = CurTime() + 2 -- Charge for 2 seconds
	while CurTime() < chargeTime and IsValid(enemy) do
		self:FaceTo(enemy:GetPos())
		self:SetVelocity(self:GetForward() * self.ChargeSpeed)
		self:PauseCoroutine(0.1)
	end
end

-- Use in combat:
function ENT:OnRangeAttack(enemy)
	if math.random(4) == 1 then
		self:ChargeAttack(enemy)
	elseif not self:IsInRange(enemy, 500) then
		self:Leap(enemy, 1000)
		self:PauseCoroutine(0.25)
	end
end
```

### Spawn Grubs on Death

```lua
function ENT:OnDeath(dmg, hitgroup)
	-- Spawn 3 antlion grubs
	for i = 1, 3 do
		local grub = ents.Create("item_grubnugget") -- Health item
		if IsValid(grub) then
			grub:SetPos(self:GetPos() + Vector(math.random(-20, 20), math.random(-20, 20), 10))
			grub:Spawn()
		end
	end
end
```

## Troubleshooting

### Wings don't show/hide properly

**Problem:** Bodygroup not changing or wrong bodygroup index.

**Solutions:**
- Check model in model viewer to verify bodygroup indices
- Try different bodygroup numbers: `self:SetBodygroup(0, 1)` or `self:SetBodygroup(2, 1)`
- Add debug: `print("Bodygroups:", self:GetNumBodyGroups())`
- Verify model is `models/Antlion.mdl` (other models may not have wings)

### Wing sound continues after landing

**Problem:** Looping sound not stopped correctly.

**Solutions:**
- Verify `isnumber(self.WingsOpen)` check is working
- Make sure `OnLandOnGround` is being called (add debug print)
- Check that sound ID is being stored correctly
- Try alternative: Use timer to check if grounded and stop sound

### Antlion doesn't leap, only melee attacks

**Problem:** Range check preventing leaps.

**Solutions:**
- Check `RangeAttackRange = 750` is set correctly
- Verify range check in OnRangeAttack: `if not self:IsInRange(enemy, 500)`
- Try increasing range: `if not self:IsInRange(enemy, 600)`
- Ensure enemy is far enough away to trigger leap

### Burrow teleports to invalid positions

**Problem:** Navmesh position validation failing.

**Solutions:**
- Some maps don't have navmesh - use `nav_generate` in console
- Add fallback position validation:
```lua
if not pos then pos = self:GetPos() end -- Stay in place if invalid
```
- Check console for Lua errors about navmesh
- Test on maps with known good navmesh (gm_construct, gm_flatgrass)

### Multiple damage hits from one attack

**Problem:** Attack() being called multiple times.

**Solutions:**
- Check `GetCycle() > 0.3` threshold
- Animation might have multiple anim events
- Add cooldown flag:
```lua
if not self.AttackCooldown then
	self.AttackCooldown = true
	self:Attack({...})
	timer.Simple(0.5, function()
		if IsValid(self) then self.AttackCooldown = false end
	end)
end
```

### Possessed jump doesn't work

**Problem:** Jump bind or velocity setting issue.

**Solutions:**
- Verify `IN_JUMP` is correct key
- Make sure on ground before jumping
- Check `PossessorNormal()` returns valid vector
- Try higher velocity multiplier: `*2000` instead of `*1500`
- Make sure `LeaveGround()` is being called

### Sound spam/overlapping sounds

**Problem:** Too many sound instances playing.

**Solutions:**
- Use `EmitSlotSound()` instead of `EmitSound()` for exclusive sounds
- Add sound cooldowns with timers
- Use `StopSound()` before playing new sound
- Example:
```lua
self:StopSound("NPC_Antlion.MeleeAttack")
self:EmitSound("NPC_Antlion.MeleeAttack")
```

### Antlion gets stuck after burrowing

**Problem:** Teleported into invalid position.

**Solutions:**
- Ensure `DropToFloor()` is called after SetPos
- Add unstuck logic:
```lua
function ENT:BurrowTo(pos)
	self:PlaySequenceAndMove("digin")
	if navmesh.IsLoaded() then
		pos = navmesh.GetNearestNavArea(pos):GetClosestPointOnArea(pos) or pos
	end
	self:SetPos(pos)
	self:DropToFloor()
	-- Add unstuck:
	if self:IsStuck() then
		self:SetPos(self:GetPos() + Vector(0, 0, 50)) -- Move up
	end
	self:PlaySequenceAndMove("digout")
end
```

## See Also

- [Creating NPCs Guide](../guides/creating-npcs.md) - Comprehensive NPC creation tutorial
- [Simple Melee NPC Example](simple-melee-npc.md) - Basic melee combat
- [Ranged NPC Example](ranged-npc.md) - Leaping attacks and ranged behavior
- [Boss NPC Example](boss-npc.md) - Create tougher boss NPCs
- [Movement API](../api/nextbot/movement.md) - Leap and movement functions
- [Animation API](../api/nextbot/animation.md) - Bodygroups and animations
- [Sound API](../api/nextbot/sound.md) - Looping sounds and audio
- [AI API](../api/nextbot/ai.md) - AI behavior and attack ranges
