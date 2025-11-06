# Sprite-Based NPC Example

This example demonstrates creating a 2D sprite-based NPC using the DrGBase sprite nextbot system. Perfect for creating 2D characters, retro-style enemies, or billboard-style NPCs.

## Overview

This example creates a "Stickman" NPC that:
- Uses 2D sprite animations instead of 3D models
- Has walk, idle, jump, and climb animations
- Can climb ladders and ledges
- Has proper sprite animation events
- Uses customizable sprite folders

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot_sprite"  -- Use sprite base

-- Misc --
ENT.PrintName = "Stickman"
ENT.Category = "Custom NPCs"
ENT.CollisionBounds = Vector(10, 10, 80)

-- Stats --
ENT.SpawnHealth = 60

-- AI --
ENT.RangeAttackRange = 200
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 200
ENT.AvoidEnemyRange = 0

-- Relationships --
ENT.Factions = {}

-- Movements --
ENT.RunSpeed = 200
ENT.WalkSpeed = 100
ENT.JumpHeight = 200

-- Sprite Animations --
ENT.SpriteFolder = "drgbase/stick_boi"  -- Folder in materials/
ENT.FramesPerSecond = 6  -- Animation speed
ENT.WalkAnimation = "walk"  -- Name of animation folder
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
ENT.EyeOffset = Vector(0, 0, 40)

if SERVER then

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- Set random color for variation
        self:SetColor(Color(
            math.random(0, 255),
            math.random(0, 255),
            math.random(0, 255)
        ))

        -- Set up animation events
        self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
            self:EmitFootstep()
        end)

        self:SpriteAnimEvent("climb", {1, 2}, function(self, frame)
            if self:IsClimbingLadder() then
                self:EmitSound("player/footsteps/ladder"..math.random(4)..".wav")
            end
        end)
    end

    -- AI --

    function ENT:OnRangeAttack(enemy)
        self:PauseCoroutine(0.5)
        self:EmitFootstep()
        self:Jump()
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1000))
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(2, 5))
    end

    -- Events --

    function ENT:OnLandOnGround()
        self:EmitFootstep()
    end

end

-- Register --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Sprite Folder Structure

Organize your sprites in `materials/`:
```
materials/
└── mysprites/
    └── character/
        ├── idle/
        │   ├── 1.png
        │   ├── 2.png
        │   └── 3.png
        ├── walk/
        │   ├── 1.png
        │   ├── 2.png
        │   ├── 3.png
        │   └── 4.png
        ├── jump/
        │   └── 1.png
        └── attack/
            ├── 1.png
            └── 2.png
```

Then set: `ENT.SpriteFolder = "mysprites/character"`

## Step-by-Step Explanation

### 1. Use Sprite Base

```lua
ENT.Base = "drgbase_nextbot_sprite"  -- Important!
```

Use the sprite-specific base instead of the regular nextbot base.

### 2. Configure Sprite Animations

```lua
ENT.SpriteFolder = "drgbase/stick_boi"
ENT.FramesPerSecond = 6
ENT.WalkAnimation = "walk"
ENT.IdleAnimation = "idle"
```

- `SpriteFolder` - Path to sprite folder in materials/
- `FramesPerSecond` - How fast animations play
- Animation properties reference folder names

### 3. Animation Events

```lua
self:SpriteAnimEvent("walk", {1, 2}, function(self, frame)
    self:EmitFootstep()
end)
```

Triggers code on specific animation frames. Perfect for footsteps, attack hit timing, etc.

### 4. Collision Bounds

```lua
ENT.CollisionBounds = Vector(10, 10, 80)
```

Sprites need thin collision boxes (small X and Y, taller Z).

## Customization Examples

### Create Attack Animation

```lua
ENT.AttackAnimation = "attack"

function ENT:OnRangeAttack(enemy)
    self:PlaySpriteAnimation("attack", false)  -- Play once
    self:PauseCoroutine(0.3)

    -- Deal damage at specific frame
    self:Attack({damage = 20, type = DMG_CLUB})

    self:PauseCoroutine(0.5)
end
```

### Multiple Color Variants

```lua
function ENT:CustomInitialize()
    local colors = {
        Color(255, 0, 0),     -- Red
        Color(0, 255, 0),     -- Green
        Color(0, 0, 255),     -- Blue
        Color(255, 255, 0),   -- Yellow
    }

    self:SetColor(colors[math.random(#colors)])
end
```

### Sprite Size Variation

```lua
ENT.SpriteScale = {0.5, 1.5}  -- Random size between 0.5x and 1.5x

-- Or fixed size:
ENT.SpriteScale = 2.0  -- 2x size
```

### Custom Death Animation

```lua
ENT.DeathAnimation = "death"

function ENT:OnDeath(dmg, hitgroup)
    self:PlaySpriteAnimation("death", false)
    self:PauseCoroutine(0.5)  -- Wait for animation
end
```

## Common Issues and Solutions

### Issue: Sprites Don't Show

**Solutions:**
1. Verify sprite folder path is correct
2. Check sprites are in `materials/` folder (not `models/`)
3. Ensure PNG files are numbered: `1.png`, `2.png`, etc.
4. Files must be lowercase
5. Restart GMod to reload materials

### Issue: Animations Don't Play

**Solutions:**
1. Verify animation folder exists: `materials/yourfolder/walk/`
2. Check folder names match animation properties
3. Ensure at least one frame exists in each animation folder
4. Verify `FramesPerSecond` is greater than 0

### Issue: Sprite Faces Wrong Direction

Sprites auto-face the camera. For directional sprites:
```lua
ENT.SpriteDirectional = true  -- Use directional sprites
ENT.SpriteAngles = Angle(0, 90, 0)  -- Adjust facing
```

### Issue: Collision Issues

```lua
-- Adjust collision bounds
ENT.CollisionBounds = Vector(15, 15, 70)  -- Width, depth, height

-- Make sure Z is tall enough for the sprite height
```

## Performance Tips

1. **Optimize frame count** - Don't use too many frames per animation
```lua
-- Good: 4-8 frames per animation
-- Bad: 30+ frames (wastes memory)
```

2. **Limit animation events**
```lua
-- Only trigger events on necessary frames
self:SpriteAnimEvent("walk", {1, 3}, callback)  -- Every other step
```

3. **Use appropriate resolution**
```lua
-- 64x64 to 256x256 is usually sufficient
-- Don't use 1024x1024+ sprites (performance hit)
```

## Next Steps

- Try [Flying NPC Example](./flying-npc.md) for aerial movement
- Learn [Advanced AI](./advanced-ai.md) for smarter behaviors
- Explore [Custom Projectiles](./custom-projectile.md) for sprite-based projectiles
