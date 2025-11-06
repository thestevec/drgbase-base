# Sprite-Based NPC Example

A complete, working example of a 2D sprite-based NPC using DrGBase - perfect for retro-style or simplified NPCs.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the NPC](#testing-the-npc)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)

## Overview

This example creates a 2D sprite NPC that:
- Uses sprite sheets instead of 3D models
- Supports multiple sprite animations (walk, idle, jump, climb)
- Can climb ladders, ledges, and props
- Uses frame-based animations with customizable frame rates
- Has sprite animation events (for footsteps, etc.)
- Spawns with random colors
- Can be possessed with jump controls
- Demonstrates 2D billboard rendering in 3D space

**Difficulty:** Intermediate
**File Location:** `lua/entities/npc_drg_testsprite.lua`
**Base:** `drgbase_nextbot_sprite`
**Key Concepts:** Sprite animations, frame events, climbing system, billboard rendering

## Complete Code

Create a file named `npc_drg_testsprite.lua` in `garrysmod/lua/entities/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
ENT.Base = "drgbase_nextbot_sprite" -- DO NOT TOUCH (obviously)

-- Misc --
ENT.PrintName = "Test 2D Nextbot"
ENT.Category = "DrGBase"
ENT.CollisionBounds = Vector(10, 10, 100)

-- AI --
ENT.RangeAttackRange = 200
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 200
ENT.AvoidEnemyRange = 0

-- Animations --
ENT.SpriteFolder = "drgbase/stick_boi"
ENT.FramesPerSecond = 6
ENT.WalkAnimation = "walk"
ENT.WalkAnimRate = 0.5
ENT.RunAnimation = "walk"
ENT.IdleAnimation = "idle"
ENT.JumpAnimation = "jump"

-- Climbing --
ENT.ClimbLedges = true
ENT.ClimbProps = true
ENT.ClimbLadders = true
ENT.ClimbLaddersUp = true
ENT.ClimbLaddersDown = true
ENT.ClimbUpAnimation = "climb"
ENT.ClimbDownAnimation = "climb"
ENT.ClimbAnimRate = 0.5

-- Detection --
ENT.EyeOffset = Vector(0, 0, 30)

-- Possession --
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
	{
		offset = Vector(0, 30, 20),
		distance = 100
	},
	{
		offset = Vector(5, 0, 0),
		distance = 0,
		eyepos = true
	}
}
ENT.PossessionBinds = {
	[IN_JUMP] = {{
		coroutine = false,
		onkeypressed = function(self)
			if not self:IsOnGround() then return end
			self:EmitFootstep()
			self:Jump()
		end
	}}
}

if SERVER then

	-- Init/Think --

	function ENT:CustomInitialize()
		self:SetSelfClassRelationship(D_HT)
		self:SetDefaultRelationship(D_HT, 2)
		self:SetPlayersRelationship(D_HT, 3)
		self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
			self:EmitFootstep()
		end)
		self:SpriteAnimEvent("climb", {1, 2}, function(self, frame)
			if self:IsClimbingLadder() then self:EmitSound("player/footsteps/ladder"..math.random(4)..".wav") end
		end)
		self:SetColor(Color(math.random(0, 255), math.random(0, 255), math.random(0, 255)))
	end

	-- AI --

	function ENT:OnRangeAttack(enemy)
		self:PauseCoroutine(0.5)
		self:EmitFootstep()
		self:Jump()
	end
	function ENT:OnReachedPatrol()
		self:Wait(math.random(3, 7))
	end
	function ENT:OnIdle()
		self:AddPatrolPos(self:RandomPos(1500))
	end

	-- Sounds --

	function ENT:OnLandOnGround()
		self:EmitFootstep()
	end

end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Code Breakdown

### Understanding Sprite-Based NPCs

Sprite NPCs use a different base class and rendering system:

```lua
ENT.Base = "drgbase_nextbot_sprite"
```

**Key differences from 3D model NPCs:**
- No 3D model - uses 2D images instead
- Always faces camera (billboard rendering)
- Frame-based animations instead of skeletal
- Lighter on performance
- Great for retro games or simple NPCs

### Sprite Configuration

```lua
ENT.SpriteFolder = "drgbase/stick_boi"
ENT.FramesPerSecond = 6
```

**Purpose:** Define where sprites are located and animation speed.

**How sprite folders work:**
- Path is relative to `materials/` folder
- Each animation is a subfolder: `materials/drgbase/stick_boi/walk/`, `materials/drgbase/stick_boi/idle/`, etc.
- Frames are numbered: `1.png`, `2.png`, `3.png`, etc.
- DrGBase automatically loads all frames in each folder

**Frame rate:**
- `FramesPerSecond = 6`: Play 6 animation frames per second
- Lower = slower, choppier animation
- Higher = faster, smoother animation
- Typical range: 4-12 FPS for retro look

### Sprite Animations

```lua
ENT.WalkAnimation = "walk"
ENT.WalkAnimRate = 0.5
ENT.RunAnimation = "walk"
ENT.IdleAnimation = "idle"
ENT.JumpAnimation = "jump"
```

**Purpose:** Map behaviors to sprite animation folders.
- `WalkAnimation = "walk"` - Uses `materials/drgbase/stick_boi/walk/` folder
- `WalkAnimRate = 0.5` - Play walk animation at 50% speed (3 FPS instead of 6)
- `RunAnimation = "walk"` - Reuse walk animation for running
- `IdleAnimation = "idle"` - Uses idle folder
- `JumpAnimation = "jump"` - Uses jump folder

**Animation rate:**
- `1.0` = Normal speed (uses FramesPerSecond)
- `0.5` = Half speed
- `2.0` = Double speed
- Useful for making walk look slower than run

### Climbing System

```lua
ENT.ClimbLedges = true
ENT.ClimbProps = true
ENT.ClimbLadders = true
ENT.ClimbLaddersUp = true
ENT.ClimbLaddersDown = true
ENT.ClimbUpAnimation = "climb"
ENT.ClimbDownAnimation = "climb"
ENT.ClimbAnimRate = 0.5
```

**Purpose:** Enable advanced movement over obstacles.

**Climbing types:**
- `ClimbLedges = true` - Can climb up ledges/platforms
- `ClimbProps = true` - Can climb over physics props
- `ClimbLadders = true` - Can use func_ladder entities
- `ClimbLaddersUp = true` - Will climb ladders going up
- `ClimbLaddersDown = true` - Will climb ladders going down

**Why this is powerful:**
- NPCs can navigate complex environments
- Can pursue players up ladders
- Can climb over obstacles that would block normal pathfinding
- Uses climb animation folder during climbing

**Animation setup:**
- `ClimbUpAnimation = "climb"` - Uses `materials/drgbase/stick_boi/climb/`
- `ClimbDownAnimation = "climb"` - Reuses same animation
- `ClimbAnimRate = 0.5` - Slower climbing animation

### Collision Bounds

```lua
ENT.CollisionBounds = Vector(10, 10, 100)
```

**Purpose:** Define 3D collision box for 2D sprite.
- Vector(width, width, height) in units
- Even though sprite is 2D, still needs 3D collision
- Width should be small (sprite is thin)
- Height should match sprite image height
- This sprite is 100 units tall

### Eye Position for Sprites

```lua
ENT.EyeOffset = Vector(0, 0, 30)
```

**Purpose:** Where the sprite "sees" from.
- No bone system (sprites don't have bones)
- Offset from sprite's origin position
- `Vector(forward, right, up)`
- This puts eyes 30 units up from ground
- Affects vision and line-of-sight

### Sprite Animation Events

```lua
self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
	self:EmitFootstep()
end)
self:SpriteAnimEvent("climb", {1, 2}, function(self, frame)
	if self:IsClimbingLadder() then
		self:EmitSound("player/footsteps/ladder"..math.random(4)..".wav")
	end
end)
```

**Purpose:** Trigger code on specific animation frames.

**How it works:**
- `SpriteAnimEvent(animation, frames, callback)`
- `"walk"` - Which animation to watch
- `{1, 2}` - Trigger on frames 1 and 2
- `function(self, frame)` - Called when those frames display

**Example breakdown:**
1. When walk animation shows frame 1: Play footstep
2. When walk animation shows frame 2: Play footstep
3. Creates realistic footstep timing synced to animation

**Climb events:**
- Triggers on climb animation frames 1 and 2
- Only plays sound if actually on ladder (not climbing ledge)
- Uses random ladder sound (1-4) for variety

### Random Colors

```lua
self:SetColor(Color(math.random(0, 255), math.random(0, 255), math.random(0, 255)))
```

**Purpose:** Spawn each sprite with random color tint.
- `Color(red, green, blue)` - RGB values 0-255
- `math.random(0, 255)` - Random intensity
- Creates visual variety without needing different sprites
- Works because sprites are modulated (multiplied) by color

### Relationship Setup

```lua
self:SetSelfClassRelationship(D_HT)
self:SetDefaultRelationship(D_HT, 2)
self:SetPlayersRelationship(D_HT, 3)
```

**Purpose:** Make all sprite NPCs hostile to everything.
- `SetSelfClassRelationship(D_HT)` - Hate other sprites of same class
- `SetDefaultRelationship(D_HT, 2)` - Hate all entities (priority 2)
- `SetPlayersRelationship(D_HT, 3)` - Hate players most (priority 3)
- Creates chaotic "everyone fights everyone" behavior

### Ranged Attack - Jump Attack

```lua
function ENT:OnRangeAttack(enemy)
	self:PauseCoroutine(0.5)
	self:EmitFootstep()
	self:Jump()
end
```

**Purpose:** Attack by jumping (similar to headcrab).

**Why this pattern:**
- Pauses 0.5 seconds first (wind-up)
- Plays footstep sound (audio feedback)
- Jumps toward enemy
- Simple but effective attack for stick figure

**Note:** No specific jump direction - uses normal jump physics
- DrGBase AI automatically faces enemy before attacking
- Jump naturally goes toward enemy
- Could be enhanced to use Leap() for directional jump

### Patrol Behavior

```lua
function ENT:OnReachedPatrol()
	self:Wait(math.random(3, 7))
end
function ENT:OnIdle()
	self:AddPatrolPos(self:RandomPos(1500))
end
```

**Purpose:** Standard wandering behavior (same as other examples).

### Landing Sound

```lua
function ENT:OnLandOnGround()
	self:EmitFootstep()
end
```

**Purpose:** Play footstep when landing from jump.
- Called automatically by DrGBase when touching ground
- Creates satisfying audio feedback for jumps
- Uses same footstep system as walking

### Possession Controls

```lua
ENT.PossessionBinds = {
	[IN_JUMP] = {{
		coroutine = false,
		onkeypressed = function(self)
			if not self:IsOnGround() then return end
			self:EmitFootstep()
			self:Jump()
		end
	}}
}
```

**Purpose:** Let players jump when possessing.
- Only works when on ground
- Plays footstep sound
- Simple jump control
- Movement handled by `POSSESSION_MOVE_8DIR`

## Testing the NPC

### Spawning

1. Start Garry's Mod and load a map
2. Open spawn menu (Q)
3. Navigate to the "DrGBase" tab under NPCs
4. Click "Test 2D Nextbot" to spawn

### Behavior to Observe

- **Appearance:** 2D stick figure that always faces you
- **Movement:** Walks around, sprite animates
- **Colors:** Each spawn has random color tint
- **Jumping:** Jump attacks when seeing enemies
- **Climbing:** Can climb ladders and ledges automatically
- **Combat:** Fights other sprites and players
- **Possession:** Smooth 8-directional movement

### Testing Tips

- **Test billboard:** Move around sprite, it always faces you
- **Test climbing:** Spawn near ladder, watch it climb
- **Test animations:** Watch walk, idle, jump animations cycle
- **Test colors:** Spawn multiple, see color variety
- **Test footsteps:** Listen for synced footstep sounds
- **Test possession:** Very smooth controls compared to 3D models

### Specific Scenarios

1. **Spawn on multi-level map** - Watch climbing behavior
2. **Spawn multiple sprites** - They'll fight each other
3. **Place near ladder** - See ladder climbing in action
4. **Possess and explore** - Test smooth 2D movement
5. **Watch animation events** - Footsteps sync to walk frames

## Customization Ideas

### Create Your Own Sprite Sheets

To create custom sprites:

1. Create folder structure:
```
materials/
  mysprites/
    mycharacter/
      idle/
        1.png
        2.png
      walk/
        1.png
        2.png
        3.png
        4.png
      jump/
        1.png
```

2. Configure NPC:
```lua
ENT.SpriteFolder = "mysprites/mycharacter"
ENT.PrintName = "My Character"
```

**Sprite requirements:**
- PNG format with transparency
- Powers of 2 dimensions (64x64, 128x128, 256x256, etc.)
- Consistent size across all frames
- Numbered sequentially (1.png, 2.png, etc.)

### Add Attack Animation

```lua
-- Add to animations:
ENT.AttackAnimation = "attack"
ENT.AttackAnimRate = 1.0

-- In OnRangeAttack:
function ENT:OnRangeAttack(enemy)
	self:PlaySpriteAnimation("attack") -- Play attack animation
	self:PauseCoroutine(0.3) -- Wait for animation
	self:EmitFootstep()
	self:Jump() -- Then jump
end
```

### Weapon-Based Sprite NPC

```lua
-- Add shooting animation
ENT.ShootAnimation = "shoot"

function ENT:OnRangeAttack(enemy)
	self:PlaySpriteAnimation("shoot")
	self:PauseCoroutine(0.2)

	-- Fire projectile
	local bullet = {}
	bullet.Num = 1
	bullet.Src = self:GetPos() + Vector(0, 0, 30)
	bullet.Dir = (enemy:EyePos() - bullet.Src):GetNormalized()
	bullet.Spread = Vector(0.1, 0.1, 0)
	bullet.Tracer = 1
	bullet.Force = 2
	bullet.Damage = 5
	bullet.AmmoType = "Pistol"
	self:FireBullets(bullet)

	self:EmitSound("Weapon_Pistol.Single")
	self:PauseCoroutine(1) -- Cooldown
end
```

### Health Bar Above Sprite

```lua
-- CLIENT side code (outside if SERVER then):
if CLIENT then
	function ENT:Draw()
		self:DrawModel() -- Draw sprite

		-- Draw health bar above
		local pos = self:GetPos() + Vector(0, 0, 120)
		local ang = EyeAngles()
		ang:RotateAroundAxis(ang:Forward(), 90)
		ang:RotateAroundAxis(ang:Right(), 90)

		cam.Start3D2D(pos, ang, 0.1)
			local health = self:Health()
			local maxHealth = self:GetMaxHealth()
			local healthPercent = health / maxHealth

			-- Background
			surface.SetDrawColor(0, 0, 0, 200)
			surface.DrawRect(-50, -10, 100, 20)

			-- Health bar
			surface.SetDrawColor(255 * (1 - healthPercent), 255 * healthPercent, 0, 255)
			surface.DrawRect(-48, -8, 96 * healthPercent, 16)
		cam.End3D2D()
	end
end
```

### Directional Sprites (Face Direction of Movement)

```lua
-- Add multiple sprite folders for directions:
ENT.SpriteFolderRight = "mysprites/character_right"
ENT.SpriteFolderLeft = "mysprites/character_left"

function ENT:CustomThink()
	-- Switch sprite based on movement direction
	local vel = self:GetVelocity()
	if vel:Length() > 10 then
		local forward = self:GetForward()
		local right = self:GetRight()
		local dot = vel:Dot(right)

		if dot > 50 then
			self:SetSpriteFolder(self.SpriteFolderRight)
		elseif dot < -50 then
			self:SetSpriteFolder(self.SpriteFolderLeft)
		end
	end
end
```

### Size Variation

```lua
function ENT:CustomInitialize()
	self:SetSelfClassRelationship(D_HT)
	self:SetDefaultRelationship(D_HT, 2)
	self:SetPlayersRelationship(D_HT, 3)

	-- Random size
	local scale = math.random(50, 150) / 100 -- 0.5x to 1.5x
	self:SetModelScale(scale)

	-- Adjust health based on size
	self:SetHealth(self:Health() * scale)

	-- Setup animation events
	self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
		self:EmitFootstep()
	end)
	self:SpriteAnimEvent("climb", {1, 2}, function(self, frame)
		if self:IsClimbingLadder() then
			self:EmitSound("player/footsteps/ladder"..math.random(4)..".wav")
		end
	end)

	-- Random color
	self:SetColor(Color(math.random(0, 255), math.random(0, 255), math.random(0, 255)))
end
```

### Particle Effects on Sprite

```lua
function ENT:CustomInitialize()
	-- ... existing code ...

	-- Add particle effect
	local fire = ents.Create("env_sprite")
	if IsValid(fire) then
		fire:SetKeyValue("model", "sprites/glow01.spr")
		fire:SetKeyValue("scale", "0.3")
		fire:SetKeyValue("rendermode", "5")
		fire:SetKeyValue("rendercolor", "255 100 0")
		fire:SetPos(self:GetPos() + Vector(0, 0, 50))
		fire:SetParent(self)
		fire:Spawn()
		fire:Activate()
	end
end
```

### Team-Based Colors

```lua
function ENT:CustomInitialize()
	self:SetSelfClassRelationship(D_HT)

	-- Assign to team
	self.Team = math.random(1, 2)

	if self.Team == 1 then
		self:SetColor(Color(255, 0, 0)) -- Red team
		self:SetFaction(FACTION_REBELS)
	else
		self:SetColor(Color(0, 0, 255)) -- Blue team
		self:SetFaction(FACTION_COMBINE)
	end

	-- Setup animation events
	self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
		self:EmitFootstep()
	end)
end
```

## Troubleshooting

### Sprite doesn't appear or is invisible

**Problem:** Sprite sheets not found or path incorrect.

**Solutions:**
- Verify folder path: `materials/drgbase/stick_boi/`
- Check that sprite files exist and are named correctly (1.png, 2.png, etc.)
- Ensure sprites are PNG format with transparency
- Check console for missing material errors
- Try absolute path in console: `mat_texture_list 1` to see loaded materials

### Sprite appears as purple/black checkerboard

**Problem:** Material error or missing texture.

**Solutions:**
- Sprite files must be in `materials/` folder, not `models/`
- Check file format is PNG, not JPG
- Verify files aren't corrupted
- Make sure folder structure matches: `materials/[SpriteFolder]/[animation]/[frame].png`

### Animation doesn't play or is stuck

**Problem:** Frame rate or animation configuration issue.

**Solutions:**
- Check `FramesPerSecond` is set (try 6)
- Verify animation folder has multiple frames (at least 2)
- Ensure frames are numbered correctly (1.png, 2.png, not 0.png)
- Try increasing FPS: `ENT.FramesPerSecond = 12`
- Check console for Lua errors

### Sprite doesn't face camera

**Problem:** Not using sprite base class.

**Solutions:**
- Verify `ENT.Base = "drgbase_nextbot_sprite"` (not "drgbase_nextbot")
- This is automatic for sprite base
- If still not working, check for client-side errors

### Footstep sounds don't sync with animation

**Problem:** Animation event timing or frame numbers wrong.

**Solutions:**
- Check which frames have foot touching ground in sprite
- Adjust frame numbers in SpriteAnimEvent
- Example: If feet touch on frames 2 and 4:
```lua
self:SpriteAnimEvent("walk", {2, 4}, function(self, frame)
	self:EmitFootstep()
end)
```

### Climbing doesn't work

**Problem:** Climbing flags or map compatibility.

**Solutions:**
- Verify climb flags are true: `ENT.ClimbLadders = true`
- Check map has func_ladder entities (some maps don't)
- Ensure collision bounds are appropriate size
- Try spawning near known ladder
- Test on gm_construct (has ladders)

### Sprite appears too small or too large

**Problem:** Sprite image resolution or scale.

**Solutions:**
- Use SetModelScale in CustomInitialize:
```lua
self:SetModelScale(2) -- 200% size
```
- Or create larger/smaller sprite images
- Standard sprite size: 128x128 or 256x256

### Animation events fire multiple times

**Problem:** Multiple frames triggering or coroutine issues.

**Solutions:**
- Add cooldown flag:
```lua
self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
	if not self.FootstepCooldown then
		self.FootstepCooldown = true
		self:EmitFootstep()
		timer.Simple(0.1, function()
			if IsValid(self) then self.FootstepCooldown = false end
		end)
	end
end)
```

### Sprite collision is wrong

**Problem:** CollisionBounds doesn't match sprite.

**Solutions:**
- Adjust collision bounds to match sprite height:
```lua
ENT.CollisionBounds = Vector(10, 10, 64) -- For 64px tall sprite
```
- Width should stay small (sprites are thin)
- Height should match visible sprite height

### Random colors look bad

**Problem:** Some color combinations are ugly or hard to see.

**Solutions:**
- Limit color ranges:
```lua
-- Pastel colors:
self:SetColor(Color(
	math.random(150, 255),
	math.random(150, 255),
	math.random(150, 255)
))

-- Or use preset colors:
local colors = {
	Color(255, 0, 0),
	Color(0, 255, 0),
	Color(0, 0, 255),
	Color(255, 255, 0)
}
self:SetColor(table.Random(colors))
```

## See Also

- [Creating NPCs Guide](../guides/creating-npcs.md) - Comprehensive NPC creation tutorial
- [Simple Melee NPC Example](simple-melee-npc.md) - Basic 3D model NPC
- [Ranged NPC Example](ranged-npc.md) - Ranged attack patterns
- [Flying NPC Example](flying-npc.md) - Advanced movement
- [Animation API](../api/nextbot/animation.md) - Animation system reference
- [Movement API](../api/nextbot/movement.md) - Climbing and movement
- [Sprite Base Documentation](../api/nextbot/sprite.md) - Sprite-specific functions
