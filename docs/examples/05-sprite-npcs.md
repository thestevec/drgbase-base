# 2D Sprite-Based NPCs

## Overview

DrGBase supports 2D sprite-based NPCs perfect for creating retro-style enemies, paper characters, or billboard creatures. Sprites use frame-based animations instead of 3D model animations.

## Basic Sprite NPC

### Complete Example

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot_sprite" -- Use sprite base class

-- Basic Info
ENT.PrintName = "Stick Figure"
ENT.Category = "My NPCs"
ENT.CollisionBounds = Vector(10, 10, 100)

-- AI
ENT.RangeAttackRange = 200
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 200

-- Sprite Animations
ENT.SpriteFolder = "drgbase/stick_boi"  -- Path in materials/
ENT.FramesPerSecond = 6                 -- Animation speed

-- Animation Definitions
ENT.WalkAnimation = "walk"     -- materials/drgbase/stick_boi/walk_1.png, walk_2.png, etc.
ENT.WalkAnimRate = 0.5
ENT.RunAnimation = "walk"
ENT.IdleAnimation = "idle"     -- materials/drgbase/stick_boi/idle_1.png, etc.
ENT.JumpAnimation = "jump"

-- Climbing (sprites can climb!)
ENT.ClimbLedges = true
ENT.ClimbProps = true
ENT.ClimbLadders = true
ENT.ClimbLaddersUp = true
ENT.ClimbLaddersDown = true
ENT.ClimbUpAnimation = "climb"
ENT.ClimbDownAnimation = "climb"
ENT.ClimbAnimRate = 0.5

if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- Register animation events (like footsteps)
        self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
            self:EmitFootstep()
        end)

        self:SpriteAnimEvent("climb", {1, 2}, function(self, frame)
            if self:IsClimbingLadder() then
                self:EmitSound("player/footsteps/ladder"..math.random(4)..".wav")
            end
        end)

        -- Random color for variety
        self:SetColor(Color(math.random(0, 255), math.random(0, 255), math.random(0, 255)))
    end

    function ENT:OnRangeAttack(enemy)
        self:PauseCoroutine(0.5)
        self:EmitFootstep()
        self:Jump()
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(3, 7))
    end

    function ENT:OnLandOnGround()
        self:EmitFootstep()
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Sprite Setup

### 1. Creating Sprite Files

Sprites are stored in `materials/your_folder/`:

```
materials/
  └── my_sprites/
      └── character/
          ├── idle_1.png
          ├── idle_2.png
          ├── idle_3.png
          ├── walk_1.png
          ├── walk_2.png
          ├── walk_3.png
          ├── walk_4.png
          ├── jump_1.png
          ├── attack_1.png
          ├── attack_2.png
          └── attack_3.png
```

**Naming convention:** `[animation_name]_[frame_number].png`

### 2. Configuring Sprites

```lua
ENT.SpriteFolder = "my_sprites/character"  -- Path relative to materials/
ENT.FramesPerSecond = 8                    -- Global animation speed
```

### 3. Defining Animations

```lua
ENT.IdleAnimation = "idle"       -- Uses idle_1.png, idle_2.png, idle_3.png
ENT.WalkAnimation = "walk"       -- Uses walk_1.png through walk_4.png
ENT.RunAnimation = "run"         -- Uses run_1.png, run_2.png, etc.
ENT.JumpAnimation = "jump"       -- Uses jump_1.png
ENT.AttackAnimation = "attack"   -- Custom animation
```

### 4. Animation Speed Control

```lua
ENT.FramesPerSecond = 8    -- Base speed for all animations
ENT.WalkAnimRate = 0.5     -- Walk plays at 50% speed (slower)
ENT.RunAnimRate = 1.5      -- Run plays at 150% speed (faster)
ENT.ClimbAnimRate = 0.75   -- Climb plays at 75% speed
```

## Animation Events

Register callbacks for specific frames:

```lua
function ENT:CustomInitialize()
    -- Footsteps on frames 1 and 3 of walk animation
    self:SpriteAnimEvent("walk", {1, 3}, function(self, frame)
        self:EmitFootstep()
        -- Dust particle
        local effectdata = EffectData()
        effectdata:SetOrigin(self:GetPos())
        util.Effect("ManhackSparks", effectdata)
    end)

    -- Attack damage on frame 2
    self:SpriteAnimEvent("attack", {2}, function(self, frame)
        self:Attack({
            damage = 15,
            type = DMG_SLASH
        })
        self:EmitSound("Weapon.Swing")
    end)

    -- Landing sound on frame 1 of jump
    self:SpriteAnimEvent("jump", {1}, function(self, frame)
        if self:IsOnGround() then
            self:EmitSound("Player.FallDamage")
        end
    end)
end
```

**SpriteAnimEvent(animation, frames, callback)**
- `animation`: Animation name (string)
- `frames`: Frame numbers (table of numbers)
- `callback`: Function called when frame is displayed

## Custom Sprite Animations

### Playing Custom Animations

```lua
function ENT:OnMeleeAttack(enemy)
    self:PlaySpriteAnimation("attack", 1.0, function(self)
        print("Attack animation finished!")
    end)
end

function ENT:OnRangeAttack(enemy)
    self:PlaySpriteAnimation("shoot", 1.5) -- Play at 150% speed
    self:Wait(0.5)
end
```

### Looping vs. One-Shot

```lua
-- One-shot animation (plays once then returns to idle)
self:PlaySpriteAnimation("attack", 1.0, function(self)
    -- Called when animation completes
    self:EmitSound("AttackEnd")
end)

-- Looping animation (plays continuously)
self:SetSpriteAnimation("walk") -- Loops until changed
```

## Advanced Sprite Techniques

### Multiple Sprite Sets

```lua
function ENT:CustomInitialize()
    local variants = {
        "sprites/character_red",
        "sprites/character_blue",
        "sprites/character_green"
    }

    self.SpriteFolder = variants[math.random(#variants)]
    self:InitializeSprites() -- Reload sprite definitions
end
```

### Directional Sprites

Create sprites for different directions:

```
materials/
  └── character/
      ├── idle_north_1.png
      ├── idle_north_2.png
      ├── idle_south_1.png
      ├── idle_south_2.png
      ├── idle_east_1.png
      ├── idle_east_2.png
      ├── idle_west_1.png
      └── idle_west_2.png
```

```lua
function ENT:CustomThink()
    -- Update sprite based on movement direction
    local dir = self:GetVelocity():GetNormalized()
    local forward = self:GetForward()
    local right = self:GetAngles():Right()

    local dotForward = dir:Dot(forward)
    local dotRight = dir:Dot(right)

    local suffix = ""
    if math.abs(dotForward) > math.abs(dotRight) then
        suffix = dotForward > 0 and "_north" or "_south"
    else
        suffix = dotRight > 0 and "_east" or "_west"
    end

    if self:IsMoving() then
        self:SetSpriteAnimation("walk" .. suffix)
    else
        self:SetSpriteAnimation("idle" .. suffix)
    end
end
```

### Sprite Scaling

```lua
function ENT:CustomInitialize()
    self:SetModelScale(2.0) -- 2x size
    -- Or vary size
    self:SetModelScale(math.Rand(0.5, 1.5))
end
```

### Sprite Effects

```lua
function ENT:OnTakeDamage(dmg)
    -- Flash red when hit
    self:SetColor(Color(255, 100, 100))

    self:Timer(0.1, function(self)
        self:SetColor(Color(255, 255, 255))
    end)
end

function ENT:OnDeath(dmg)
    -- Fade out
    local alpha = 255
    self:LoopTimer(0.05, function(self)
        alpha = alpha - 15
        if alpha <= 0 then
            self:Remove()
            return true -- Stop timer
        end
        local col = self:GetColor()
        col.a = alpha
        self:SetColor(col)
    end)
end
```

### Paper Mario Style

```lua
ENT.CollisionBounds = Vector(5, 5, 50) -- Thin collision

function ENT:CustomInitialize()
    -- Always face camera
    self:SetSolid(SOLID_BBOX)
    self:SetMoveType(MOVETYPE_STEP)

    -- Make truly 2D (face player)
    self:LoopTimer(0.1, function(self)
        local ply = self:GetEnemy()
        if IsValid(ply) and ply:IsPlayer() then
            local ang = (ply:GetPos() - self:GetPos()):Angle()
            ang.p = 0
            ang.r = 0
            self:SetAngles(ang)
        end
    end)
end
```

## Climbing System

Sprite NPCs can climb walls and ladders:

```lua
ENT.ClimbLedges = true       -- Climb up ledges
ENT.ClimbProps = true        -- Climb up props
ENT.ClimbLadders = true      -- Use func_ladder entities
ENT.ClimbLaddersUp = true
ENT.ClimbLaddersDown = true
ENT.ClimbSpeed = 60          -- Climbing movement speed
ENT.ClimbUpAnimation = "climb"
ENT.ClimbDownAnimation = "climb"
ENT.ClimbAnimRate = 0.75
```

**Climbing detection:**
```lua
function ENT:CustomThink()
    if self:IsClimbing() then
        print("Currently climbing!")
    end

    if self:IsClimbingLadder() then
        print("On a ladder!")
    end
end
```

## Example: Platformer Enemy

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot_sprite"

ENT.PrintName = "Platformer Enemy"
ENT.Category = "DrGBase"
ENT.CollisionBounds = Vector(8, 8, 16)

-- Stats
ENT.SpawnHealth = 30
ENT.WalkSpeed = 80
ENT.RunSpeed = 150
ENT.JumpHeight = 200

-- AI
ENT.MeleeAttackRange = 40
ENT.RangeAttackRange = 0

-- Sprites
ENT.SpriteFolder = "sprites/goomba"
ENT.FramesPerSecond = 8
ENT.WalkAnimation = "walk"
ENT.IdleAnimation = "idle"
ENT.JumpAnimation = "jump"
ENT.AttackAnimation = "attack"

if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- Footstep sounds
        self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
            self:EmitSound("NPC.FootstepWood")
        end)

        -- Attack damage on frame 2
        self:SpriteAnimEvent("attack", {2}, function(self, frame)
            self:Attack({
                damage = 10,
                type = DMG_CRUSH
            })
        end)
    end

    function ENT:OnMeleeAttack(enemy)
        self:PlaySpriteAnimation("attack", 1.2, function(self)
            -- Attack complete
        end)
    end

    function ENT:OnIdle()
        -- Patrol behavior
        if math.random(3) == 1 then
            self:Jump() -- Random jumps
        end

        self:AddPatrolPos(self:RandomPos(500))
    end

    function ENT:OnTakeDamage(dmg)
        -- Stomp damage from above = instant kill
        if dmg:GetDamagePosition().z > self:GetPos().z + 10 then
            self:SetHealth(0)
            self:EmitSound("Goomba.Stomp")

            -- Squash effect
            self:SetModelScale(1, 0.1) -- Instant squash
            return
        end

        -- Flash white
        self:SetColor(Color(255, 255, 255))
        self:Timer(0.1, function(self)
            self:SetColor(self.OriginalColor or Color(255, 255, 255))
        end)
    end

    function ENT:OnDeath(dmg)
        -- Flip upside down and fall
        local ang = self:GetAngles()
        ang:RotateAroundAxis(ang:Forward(), 180)
        self:SetAngles(ang)

        self:SetVelocity(Vector(0, 0, 200))
        self:Wait(2)
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Sprite Image Guidelines

### Resolution
- **Recommended:** 64x64 to 256x256 pixels
- Higher resolution = more detail but larger file size
- Keep consistent across all frames

### Format
- **PNG** with transparency (alpha channel)
- **VTF** (Valve Texture Format) for optimization

### Style Tips
1. **Consistent size:** All frames should be same dimensions
2. **Center pivot:** Character should be centered in image
3. **Transparent background:** Use alpha channel
4. **Frame count:** 2-4 frames for idle, 4-8 for walk, 1-3 for attack
5. **Smooth transitions:** First and last frame should flow together for loops

### Converting to VTF

Use VTFEdit to convert PNG to VTF:
1. Open PNG in VTFEdit
2. Tools → Convert Folder
3. Enable "Alpha Channel"
4. Export to `materials/` folder

## Troubleshooting

### Sprites don't show
- Check material path in `SpriteFolder`
- Ensure files named correctly: `animation_1.png`, `animation_2.png`
- Verify PNG files have transparency
- Check console for missing texture errors

### Animations don't play
- Verify animation name matches file prefix
- Check `FramesPerSecond` is > 0
- Use `print()` in `SpriteAnimEvent` to debug

### Sprite faces wrong direction
- Sprites auto-billboard to camera by default
- Override with custom angle logic if needed

### Climbing doesn't work
- Ensure `ClimbLedges`/`ClimbLadders` enabled
- Check navmesh exists
- Verify collision bounds aren't too large

## Source Reference

Source: `lua/entities/npc_drg_testsprite.lua`
Base class: `lua/entities/drgbase_nextbot_sprite/`

## Related Documentation

- [Animation System](09-animations.md)
- [Sprite Base Class](../reference/sprite-base.md)
- [Material System](../guides/materials.md)
