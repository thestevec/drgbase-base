# Custom Projectile Example

A complete guide to creating custom projectiles using DrGBase's projectile system, covering both physics-based and non-physics projectiles.

## Table of Contents

- [Overview](#overview)
- [Example 1: Physics Grenade](#example-1-physics-grenade)
- [Example 2: Energy Projectile](#example-2-energy-projectile)
- [Example 3: Seeking Missile](#example-3-seeking-missile)
- [Code Breakdown](#code-breakdown)
- [Testing Projectiles](#testing-projectiles)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)
- [See Also](#see-also)

## Overview

DrGBase provides a flexible projectile system through the `proj_drg_default` base class. You can create two main types of projectiles:

1. **Physics-Based** (like grenades, rockets): Affected by gravity, bounce off surfaces
2. **Non-Physics** (like plasma, lasers): Fly straight, ignore gravity

This guide shows complete examples of both types and how to customize them for your needs.

**Difficulty:** Intermediate
**Base:** `proj_drg_default`

## Example 1: Physics Grenade

A bouncing grenade that explodes after a delay or on impact.

### Complete Code

Create a file named `proj_drg_mygrenade.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "proj_drg_default"
ENT.IsDrGGrenade = true

-- Misc --
ENT.PrintName = "My Grenade"
ENT.Category = "DrGBase"
ENT.Models = {"models/weapons/w_eq_fraggrenade.mdl"}
ENT.Spawnable = true

-- Physics --
ENT.Physgun = true
ENT.Gravgun = true

-- Grenade Properties --
ENT.Bounce = 0.75  -- Bounciness (0 = no bounce, 1 = perfect bounce)
ENT.OnBounceSounds = {"weapons/hegrenade/he_bounce-1.wav"}

-- Explosion Settings
ENT.ExplosionDamage = 250
ENT.ExplosionRange = 250
ENT.ExplosionDelay = 3  -- Seconds until detonation

-- Get damage for explosion
function ENT:GetDamage()
	return math.Clamp(self:GetNW2Float("DrGBaseDamage", self.ExplosionDamage), 0, math.huge)
end

-- Get explosion range
function ENT:GetRange()
	return math.Clamp(self:GetNW2Float("DrGBaseRange", self.ExplosionRange), 0, math.huge)
end

-- Get detonation delay
function ENT:GetDelay()
	return math.Clamp(self:GetNW2Float("DrGBaseDelay", self.ExplosionDelay), 0, math.huge)
end

-- Check if grenade pin has been pulled
function ENT:IsUnpined()
	return self:GetNW2Bool("DrGBaseUnpined")
end

-- Check if grenade has exploded
function ENT:HasDetonated()
	return self:GetNW2Bool("DrGBaseDetonated")
end

if SERVER then
	AddCSLuaFile()

	function ENT:_BaseInitialize()
		self._DrGBaseBounceSoundDelay = 0
	end

	-- Handle bouncing
	function ENT:OnContact(ent)
		if self:GetVelocity():IsZero() then return end

		-- Apply bounce physics
		self:Timer(0, function()
			self:SetVelocity(self:GetVelocity() * self.Bounce)
		end)

		-- Play bounce sound
		if CurTime() < self._DrGBaseBounceSoundDelay then return end
		if istable(self.OnBounceSounds) and #self.OnBounceSounds > 0 then
			self._DrGBaseBounceSoundDelay = CurTime() + 0.25
			self:EmitSound(self.OnBounceSounds[math.random(#self.OnBounceSounds)])
		end
	end

	-- Pull pin when picked up
	function ENT:Use()
		self:Unpin()
	end

	-- Explode when damaged
	function ENT:OnTakeDamage()
		self:Timer(0.1, self.Detonate)
	end

	-- Grenade Functions --

	function ENT:SetDamage(damage)
		self:SetNW2Float("DrGBaseDamage", damage)
	end

	function ENT:SetRange(range)
		self:SetNW2Float("DrGBaseRange", range)
	end

	function ENT:SetDelay(delay)
		self:SetNW2Float("DrGBaseDelay", delay)
	end

	function ENT:Unpin(mute)
		if self:IsUnpined() then return end
		if not mute then self:EmitSound("weapons/pinpull.wav") end
		self:SetNW2Bool("DrGBaseUnpined", true)
		self:Timer(self:GetDelay(), self.Detonate)
	end

	function ENT:Detonate()
		if self:HasDetonated() then return end
		self:SetNW2Bool("DrGBaseDetonated", true)
		if not self:OnDetonate() then self:Remove() end
	end

	-- Called when grenade explodes
	function ENT:OnDetonate()
		self:Explosion(self:GetDamage(), self:GetRange())
	end
end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Example 2: Energy Projectile

A fast-moving energy projectile that ignores gravity and dissolves targets.

### Complete Code

Create a file named `proj_drg_myplasma.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "proj_drg_default"

-- Misc --
ENT.PrintName = "My Plasma Ball"
ENT.Category = "DrGBase"
ENT.Spawnable = true

-- Physics --
ENT.Gravity = false  -- Ignore gravity
ENT.Physgun = false
ENT.Gravgun = true

-- Contact --
ENT.OnContactDecals = {"Scorch"}

-- Sounds --
ENT.LoopSounds = {}
ENT.OnContactSounds = {"weapons/stunstick/stunstick_fleshhit1.wav"}
ENT.OnRemoveSounds = {}

-- Effects --
ENT.AttachEffects = {"drg_plasma_ball"}
ENT.OnContactEffects = {}
ENT.OnRemoveEffects = {}

if SERVER then
	AddCSLuaFile()

	function ENT:CustomInitialize()
		-- Create glowing light effect
		self:DynamicLight(Color(150, 255, 0), 300, 0.1)
	end

	function ENT:CustomThink()
		-- Maintain constant velocity if not affected by gravity
		if not self:GetPhysicsObject():IsGravityEnabled() then
			local velocity = self:GetVelocity()
			self:SetVelocity(velocity:GetNormalized() * 500)
		end
	end

	function ENT:OnContact(ent)
		-- Plasma balls destroy each other on contact
		if ent:GetClass() == self:GetClass() then
			self:Remove()
			ent:Remove()
		else
			-- Dissolve target
			self:DealDamage(ent, ent:Health(), DMG_SHOCK + DMG_DISSOLVE)
		end

		-- Remove plasma ball after hitting something
		if self:GetPhysicsObject():IsGravityEnabled() then
			self:Remove()
		end
	end
end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Example 3: Seeking Missile

A missile that homes in on targets.

### Complete Code

Create a file named `proj_drg_mymissile.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "proj_drg_default"

-- Misc --
ENT.PrintName = "Seeking Missile"
ENT.Category = "DrGBase"
ENT.Models = {"models/weapons/w_missile_closed.mdl"}
ENT.Spawnable = true

-- Physics --
ENT.Gravity = false
ENT.Physgun = false
ENT.Gravgun = false

-- Missile Properties --
ENT.Speed = 800
ENT.TurnRate = 3  -- How quickly it can turn
ENT.SeekRange = 2000  -- Range to detect targets

-- Effects --
ENT.AttachEffects = {"smoke_trail"}

-- Sounds --
ENT.LoopSounds = {"weapons/rpg/rocket1.wav"}
ENT.OnContactSounds = {}

if SERVER then
	AddCSLuaFile()

	function ENT:CustomInitialize()
		-- Create light and trail
		self:DynamicLight(Color(255, 200, 100), 400, 0.1)

		-- Start rocket sound
		if istable(self.LoopSounds) and #self.LoopSounds > 0 then
			self:StartLoopingSound(self.LoopSounds[1])
		end
	end

	function ENT:CustomThink()
		local target = self:GetTarget()

		if IsValid(target) then
			-- Calculate direction to target
			local targetPos = target:WorldSpaceCenter()
			local direction = (targetPos - self:GetPos()):GetNormalized()

			-- Smoothly rotate toward target
			local currentDir = self:GetForward()
			local newDir = LerpVector(self.TurnRate * FrameTime(), currentDir, direction)

			-- Set new velocity
			self:SetAngles(newDir:Angle())
			self:SetVelocity(newDir * self.Speed)
		else
			-- No target, maintain current direction
			self:SetVelocity(self:GetForward() * self.Speed)
		end
	end

	function ENT:GetTarget()
		-- Return stored target if still valid
		if IsValid(self.Target) and self:GetPos():Distance(self.Target:GetPos()) < self.SeekRange then
			return self.Target
		end

		-- Find new target
		self.Target = nil
		local owner = self:GetOwner()
		local bestTarget = nil
		local bestDist = self.SeekRange

		-- Find closest enemy in range
		for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.SeekRange)) do
			if IsValid(ent) and (ent:IsNPC() or ent:IsPlayer()) and ent ~= owner then
				local dist = self:GetPos():Distance(ent:GetPos())
				if dist < bestDist then
					-- Check line of sight
					local tr = util.TraceLine({
						start = self:GetPos(),
						endpos = ent:WorldSpaceCenter(),
						filter = {self, owner}
					})

					if not tr.Hit or tr.Entity == ent then
						bestTarget = ent
						bestDist = dist
					end
				end
			end
		end

		self.Target = bestTarget
		return self.Target
	end

	function ENT:OnContact(ent)
		-- Create explosion
		self:Explosion(150, 300)

		-- Visual effect
		local effectData = EffectData()
		effectData:SetOrigin(self:GetPos())
		effectData:SetScale(2)
		util.Effect("Explosion", effectData)

		-- Remove missile
		self:Remove()
	end

	function ENT:OnRemove()
		-- Stop looping sound
		if self.LoopSoundID then
			self:StopLoopingSound(self.LoopSoundID)
		end
	end
end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### Base Configuration

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"
```

**Purpose:** Inherit from DrGBase projectile system.
- All projectiles extend `proj_drg_default`
- Provides common functionality (physics, collision, effects, etc.)

### Physics Properties

```lua
-- For physics-based projectiles (grenades, rockets):
ENT.Gravity = true  -- (default)
ENT.Physgun = true
ENT.Gravgun = true

-- For energy projectiles (plasma, lasers):
ENT.Gravity = false  -- Ignore gravity
ENT.Physgun = false
```

**Purpose:** Control how projectile moves.
- `Gravity`: Whether affected by gravity
- `Physgun/Gravgun`: Whether players can pick up with tools

### Bounce System (Physics Projectiles)

```lua
ENT.Bounce = 0.75
ENT.OnBounceSounds = {"weapons/hegrenade/he_bounce-1.wav"}

function ENT:OnContact(ent)
	self:SetVelocity(self:GetVelocity() * self.Bounce)
end
```

**Purpose:** Make projectiles bounce realistically.
- `Bounce`: Velocity multiplier on bounce (0.75 = loses 25% speed)
- `OnBounceSounds`: Sounds played when bouncing
- Lower values = less bouncy, higher = more bouncy

### Detonation System (Grenades)

```lua
function ENT:Unpin(mute)
	if self:IsUnpined() then return end
	self:SetNW2Bool("DrGBaseUnpined", true)
	self:Timer(self:GetDelay(), self.Detonate)
end

function ENT:Detonate()
	if self:HasDetonated() then return end
	self:SetNW2Bool("DrGBaseDetonated", true)
	self:OnDetonate()
end

function ENT:OnDetonate()
	self:Explosion(self:GetDamage(), self:GetRange())
end
```

**Purpose:** Timed explosion mechanic.
- `Unpin()`: Starts countdown timer
- `Detonate()`: Triggers explosion
- `OnDetonate()`: Override this for custom explosion behavior

### Energy Projectile Velocity

```lua
function ENT:CustomThink()
	if not self:GetPhysicsObject():IsGravityEnabled() then
		local velocity = self:GetVelocity()
		self:SetVelocity(velocity:GetNormalized() * 500)
	end
end
```

**Purpose:** Maintain constant speed for energy projectiles.
- Normalizes velocity vector to maintain fixed speed
- Only applies when gravity is disabled
- Change `500` to adjust speed

### Seeking Behavior (Missiles)

```lua
function ENT:GetTarget()
	-- Find closest enemy in range
	for _, ent in pairs(ents.FindInSphere(self:GetPos(), self.SeekRange)) do
		if IsValid(ent) and (ent:IsNPC() or ent:IsPlayer()) then
			-- Check line of sight
			-- Store and return best target
		end
	end
end

function ENT:CustomThink()
	local target = self:GetTarget()
	if IsValid(target) then
		-- Calculate direction to target
		local direction = (target:WorldSpaceCenter() - self:GetPos()):GetNormalized()
		-- Smoothly rotate toward target
		local newDir = LerpVector(self.TurnRate * FrameTime(), currentDir, direction)
		self:SetVelocity(newDir * self.Speed)
	end
end
```

**Purpose:** Home in on targets.
- Scans for targets within range
- Checks line of sight
- Smoothly turns toward target using lerp
- `TurnRate`: How quickly it can turn (higher = tighter turns)

### Contact Behavior

```lua
function ENT:OnContact(ent)
	-- Called when projectile hits something
	-- ent = entity that was hit
end
```

**Purpose:** Handle what happens on impact.
- Override this to customize impact behavior
- Can deal damage, create explosions, apply effects, etc.

### Visual Effects

```lua
ENT.AttachEffects = {"drg_plasma_ball"}  -- Particle effect attached to projectile
ENT.OnContactEffects = {}  -- Effects on impact
ENT.OnContactDecals = {"Scorch"}  -- Decals left on surfaces

function ENT:CustomInitialize()
	self:DynamicLight(Color(150, 255, 0), 300, 0.1)  -- Glowing light
end
```

**Purpose:** Add visual polish.
- `AttachEffects`: Particle effects that follow projectile
- `OnContactDecals`: Marks left on impact
- `DynamicLight()`: Creates glowing light around projectile

### Audio Effects

```lua
ENT.LoopSounds = {"weapons/rpg/rocket1.wav"}  -- Continuous sound
ENT.OnContactSounds = {"explosion.wav"}  -- Sound on impact
ENT.OnBounceSounds = {"bounce.wav"}  -- Sound when bouncing
```

**Purpose:** Audio feedback.
- `LoopSounds`: Played continuously while projectile exists
- `OnContactSounds`: Played on impact
- `OnBounceSounds`: Played when bouncing (physics projectiles only)

## Testing Projectiles

### Spawning

1. Open spawn menu (Q)
2. Go to Entities tab
3. Find "DrGBase" category
4. Click your projectile name

### Console Commands

```lua
-- Spawn directly
lua_run ents.Create("proj_drg_mygrenade"):Spawn()

-- Launch toward where you're looking
lua_run local p = ents.Create("proj_drg_mygrenade") p:SetPos(Entity(1):GetShootPos()) p:Spawn() p:GetPhysicsObject():SetVelocity(Entity(1):GetAimVector() * 1000)
```

### Testing Tips

- **Grenades**: Spawn and watch them bounce and explode
- **Plasma**: Spawn and watch flight path (should be straight)
- **Missiles**: Spawn near NPCs to test seeking behavior
- Use `noclip` to follow projectiles
- Spawn multiple to test collisions
- Check console for errors

## Customization Ideas

### Sticky Grenades

```lua
ENT.Bounce = 0  -- No bounce

function ENT:OnContact(ent)
	if IsValid(ent) and not self.Stuck then
		-- Stick to entity
		self:SetParent(ent)
		self.Stuck = true

		-- Remove physics
		local phys = self:GetPhysicsObject()
		if IsValid(phys) then
			phys:EnableMotion(false)
		end
	end
end
```

### Impact Grenades

```lua
-- Explode immediately on contact
function ENT:OnContact(ent)
	if IsValid(ent) then
		self:Detonate()
	end
end
```

### Cluster Grenades

```lua
function ENT:OnDetonate()
	-- Main explosion
	self:Explosion(self:GetDamage(), self:GetRange())

	-- Spawn cluster bomblets
	for i = 1, 8 do
		local angle = (360 / 8) * i
		local dir = Vector(
			math.cos(math.rad(angle)),
			math.sin(math.rad(angle)),
			0.5
		):GetNormalized()

		local bomblet = ents.Create("proj_drg_mygrenade")
		bomblet:SetPos(self:GetPos())
		bomblet:Spawn()
		bomblet:SetDamage(50)  -- Less damage
		bomblet:SetRange(100)  -- Smaller radius
		bomblet:SetDelay(1)  -- Quick explosion
		bomblet:Unpin(true)  -- Auto-activate

		local phys = bomblet:GetPhysicsObject()
		if IsValid(phys) then
			phys:SetVelocity(dir * 500)
		end
	end
end
```

### Napalm/Fire Projectile

```lua
function ENT:OnContact(ent)
	-- Create fire at impact point
	for i = 1, 10 do
		local fire = ents.Create("env_fire")
		fire:SetPos(self:GetPos() + Vector(
			math.random(-100, 100),
			math.random(-100, 100),
			0
		))
		fire:SetKeyValue("health", "30")
		fire:SetKeyValue("firesize", "64")
		fire:SetKeyValue("fireattack", "10")
		fire:SetKeyValue("damagescale", "1.0")
		fire:Spawn()
		fire:Activate()
	end

	self:Remove()
end
```

### Freeze Projectile

```lua
function ENT:OnContact(ent)
	if IsValid(ent) and (ent:IsNPC() or ent:IsPlayer()) then
		-- Freeze target
		if ent:IsPlayer() then
			ent:Freeze(true)
			timer.Simple(5, function()
				if IsValid(ent) then
					ent:Freeze(false)
				end
			end)
		elseif ent:IsNPC() then
			local phys = ent:GetPhysicsObject()
			if IsValid(phys) then
				phys:EnableMotion(false)
				timer.Simple(5, function()
					if IsValid(phys) then
						phys:EnableMotion(true)
					end
				end)
			end
		end

		-- Visual effect
		local effectData = EffectData()
		effectData:SetOrigin(ent:GetPos())
		util.Effect("ManhackSparks", effectData)
	end

	self:Remove()
end
```

### Bouncing Betty (Proximity Mine)

```lua
ENT.Gravity = false

function ENT:CustomInitialize()
	-- Hover at spawn height
	local pos = self:GetPos()
	self.HoverHeight = pos.z
end

function ENT:CustomThink()
	-- Maintain hover height
	local pos = self:GetPos()
	pos.z = self.HoverHeight
	self:SetPos(pos)

	-- Check for nearby enemies
	for _, ent in pairs(ents.FindInSphere(self:GetPos(), 150)) do
		if IsValid(ent) and (ent:IsNPC() or ent:IsPlayer()) and ent ~= self:GetOwner() then
			-- Explode!
			self:Detonate()
			break
		end
	end
end
```

### Chain Lightning Projectile

```lua
function ENT:OnContact(ent)
	if IsValid(ent) and (ent:IsNPC() or ent:IsPlayer()) then
		-- Damage primary target
		self:DealDamage(ent, 50, DMG_SHOCK)

		-- Chain to nearby targets
		local chained = {ent}
		for i = 1, 3 do  -- Chain up to 3 times
			local lastTarget = chained[#chained]
			local nextTarget = nil
			local bestDist = 300

			-- Find closest unchained target
			for _, target in pairs(ents.FindInSphere(lastTarget:GetPos(), 300)) do
				if IsValid(target) and (target:IsNPC() or target:IsPlayer()) then
					if not table.HasValue(chained, target) then
						local dist = lastTarget:GetPos():Distance(target:GetPos())
						if dist < bestDist then
							nextTarget = target
							bestDist = dist
						end
					end
				end
			end

			if nextTarget then
				table.insert(chained, nextTarget)
				self:DealDamage(nextTarget, 50, DMG_SHOCK)

				-- Visual lightning arc
				local effectData = EffectData()
				effectData:SetOrigin(lastTarget:GetPos())
				effectData:SetStart(nextTarget:GetPos())
				util.Effect("TeslaHitBoxes", effectData)
			else
				break
			end
		end
	end

	self:Remove()
end
```

## Troubleshooting

### Projectile spawns but doesn't move

**Problem:** Physics or velocity not set.

**Solutions:**
- Make sure projectile has physics model
- Verify velocity is being set on spawn
- Check `ENT.Gravity` is configured correctly
- For energy projectiles, ensure `CustomThink()` is setting velocity

### Projectile falls through floor

**Problem:** Physics object invalid or collision issue.

**Solutions:**
- Verify model has collision mesh
- Check `self:GetPhysicsObject():IsValid()`
- Make sure model exists: `models/weapons/w_eq_fraggrenade.mdl`
- Try a different model

### Grenade doesn't explode

**Problem:** Timer or detonation issue.

**Solutions:**
- Check that `Unpin()` is being called
- Verify `GetDelay()` returns valid number
- Make sure `OnDetonate()` is defined
- Add debug: `print("Detonating in", self:GetDelay(), "seconds")`

### No explosion damage

**Problem:** Explosion function or damage configuration.

**Solutions:**
- Check `self:GetDamage()` returns valid number
- Verify entities are within explosion range
- Make sure `self:Explosion()` is being called
- Test with different damage values

### Missile doesn't seek targets

**Problem:** Target detection or turning issue.

**Solutions:**
- Check `GetTarget()` is finding valid entities
- Verify `SeekRange` is large enough
- Make sure `TurnRate` isn't too low
- Add debug: `print("Target:", self:GetTarget())`
- Check that targets aren't filtered out (owner check)

### Projectile doesn't bounce

**Problem:** Bounce calculation or OnContact issue.

**Solutions:**
- Verify `Bounce` value is between 0 and 1
- Check `OnContact()` is being called
- Make sure physics object is valid
- Ensure surface can be bounced on

### No sound effects

**Problem:** Sound path or playback issue.

**Solutions:**
- Verify sound paths are correct
- Test sound in console: `play weapons/rpg/rocket1.wav`
- Check sound tables aren't empty
- Make sure ` LoopSounds` are being started: `self:StartLoopingSound()`

### Projectile lags/stutters

**Problem:** Performance issue in CustomThink.

**Solutions:**
- Avoid expensive operations in `CustomThink()`
- Cache frequently accessed values
- Use timers for periodic checks instead of every frame
- Optimize target finding (don't scan too frequently)

## See Also

- [Custom Weapon Example](custom-weapon.md) - Create weapons that fire projectiles
- [Ranged NPC Example](ranged-npc.md) - NPCs that use projectiles
- [Boss NPC Example](boss-npc.md) - Bosses with special attacks
- [DrGBase Projectile Base](../../lua/entities/proj_drg_default.lua) - Base projectile code
- [Effects API](../api/effects.md) - Creating visual effects
