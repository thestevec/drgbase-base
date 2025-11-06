# Animation System

## Overview

DrGBase provides a comprehensive animation system supporting activities, sequences, animation events, and frame-based callbacks. This guide covers everything from basic animation playback to advanced techniques.

## Basic Animation Concepts

### Activities vs. Sequences

**Activities** are animation categories (ACT_WALK, ACT_RUN, ACT_IDLE)
- Same activity can map to different sequences per model
- More portable between models
- Example: `ACT_WALK` might be "walk_all" on one model, "walking" on another

**Sequences** are specific animation names ("run_all", "zombie_walk", "attack_slash")
- Model-specific
- More precise control
- Example: "zombie_walk" is a specific sequence

### Common Activities

```lua
-- Movement
ACT_IDLE           -- Standing idle
ACT_WALK           -- Walking
ACT_RUN            -- Running
ACT_JUMP           -- Jumping

-- Combat
ACT_MELEE_ATTACK1  -- Melee attack variation 1
ACT_MELEE_ATTACK2  -- Melee attack variation 2
ACT_RANGE_ATTACK1  -- Ranged attack 1
ACT_RANGE_ATTACK2  -- Ranged attack 2

-- Reactions
ACT_FLINCH_PHYSICS -- Hit by physics object
ACT_FLINCH_BACK    -- Hit from behind
ACT_FLINCH_CHEST   -- Hit in chest
ACT_FLINCH_HEAD    -- Hit in head

-- Special
ACT_CROUCH         -- Crouching
ACT_CLIMB_UP       -- Climbing up
ACT_CLIMB_DOWN     -- Climbing down
```

## Setting Default Animations

```lua
-- In ENT table:
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE
ENT.JumpAnimation = ACT_JUMP
ENT.FallAnimation = ACT_FALL
ENT.ClimbUpAnimation = ACT_CLIMB_UP
ENT.ClimbDownAnimation = ACT_CLIMB_DOWN

-- Or use sequence names:
ENT.WalkAnimation = "walk_all"
ENT.RunAnimation = "run_all"
```

## Playing Animations

### Play Activity

```lua
-- Simple playback
self:PlayActivity(ACT_MELEE_ATTACK1)

-- With callback
self:PlayActivity(ACT_MELEE_ATTACK1, function(self)
    print("Animation finished")
end)
```

### Play Sequence

```lua
-- By name
self:PlaySequence("zombie_attack_01")

-- With callback
self:PlaySequence("zombie_attack_01", function(self)
    print("Attack complete")
end)
```

### Play and Move

For animations that should play while NPC moves:

```lua
-- Play activity while moving
self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1.0, self.FaceEnemy)
-- Parameters: activity, playback_rate, callback

-- Play sequence while moving
self:PlaySequenceAndMove("run_all", 1.0, self.FaceEnemy)
```

### Playback Rate

```lua
-- Normal speed
self:PlayActivity(ACT_WALK, 1.0)

-- Double speed
self:PlayActivity(ACT_WALK, 2.0)

-- Half speed
self:PlayActivity(ACT_WALK, 0.5)

-- Change during playback
self:SetPlaybackRate(1.5)
```

## Animation Events

### OnAnimEvent Hook

Called on animation events (footsteps, attack hits, etc.):

```lua
function ENT:OnAnimEvent()
    -- Check if attacking
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({
            damage = 10,
            type = DMG_SLASH
        })
    end

    -- Footstep sounds
    if math.random(2) == 1 then
        self:EmitSound("Zombie.FootstepLeft")
    else
        self:EmitSound("Zombie.FootstepRight")
    end
end
```

### Animation Cycle

`self:GetCycle()` returns animation progress (0.0 to 1.0):

```lua
function ENT:OnAnimEvent()
    local cycle = self:GetCycle()

    if cycle > 0.25 and cycle < 0.3 then
        -- Do something at 25% of animation
        self:EmitSound("Windup")
    elseif cycle > 0.5 and cycle < 0.55 then
        -- Do something at 50%
        self:Attack({damage = 10})
    elseif cycle > 0.9 then
        -- Do something near end
        self:EmitSound("Complete")
    end
end
```

### Sequence Events

Trigger callbacks at specific animation cycles:

```lua
self:SequenceEvent("attack_slash", {0.3, 0.6}, function(self, cycle)
    print("Hit frame at cycle: " .. cycle)
    self:Attack({damage = 15})
end)

-- Or multiple callbacks
self:SequenceEvent("combo_attack", {0.2}, function(self, cycle)
    self:EmitSound("FirstHit")
    self:Attack({damage = 10})
end)

self:SequenceEvent("combo_attack", {0.7}, function(self, cycle)
    self:EmitSound("SecondHit")
    self:Attack({damage = 15})
end)
```

## Advanced Animation Techniques

### Animation Layering

```lua
function ENT:CustomThink()
    local baseSeq = self:GetSequence()

    -- Play gesture on top of base animation
    if self:IsAttacking() then
        self:AddGestureSequence(self:LookupSequence("gesture_melee_attack"))
    end
end
```

### Pose Parameters

Pose parameters control animation blending:

```lua
-- Aim pitch (up/down)
self:SetPoseParameter("aim_pitch", 45) -- Look up 45 degrees

-- Aim yaw (left/right)
self:SetPoseParameter("aim_yaw", -30) -- Look left 30 degrees

-- Move speed
self:SetPoseParameter("move_x", 0.5) -- Forward speed multiplier
self:SetPoseParameter("move_y", 0.0) -- Strafe speed multiplier

-- Example: Dynamic aim
function ENT:CustomThink()
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        local toEnemy = (enemy:GetPos() - self:GetPos()):Angle()
        local myAngles = self:GetAngles()
        local pitch = math.NormalizeAngle(toEnemy.p - myAngles.p)
        local yaw = math.NormalizeAngle(toEnemy.y - myAngles.y)

        self:SetPoseParameter("aim_pitch", pitch)
        self:SetPoseParameter("aim_yaw", yaw)
    end
end
```

### Animation Speed Based on Movement

```lua
function ENT:CustomThink()
    local speed = self:GetVelocity():Length()

    if speed > 200 then
        self:SetPlaybackRate(1.5) -- Faster animation when running
    elseif speed > 100 then
        self:SetPlaybackRate(1.0) -- Normal speed
    elseif speed > 0 then
        self:SetPlaybackRate(0.6) -- Slower animation when walking
    end
end
```

### Random Animation Selection

```lua
function ENT:OnMeleeAttack(enemy)
    local attacks = {
        ACT_MELEE_ATTACK1,
        ACT_MELEE_ATTACK2,
        "attack_slash",
        "attack_stab"
    }

    local anim = attacks[math.random(#attacks)]
    self:PlayActivityAndMove(anim, 1, self.FaceEnemy)
end

-- Or use built-in function:
function ENT:OnMeleeAttack(enemy)
    local anim = self:SelectRandomSequence(ACT_MELEE_ATTACK1)
    self:PlaySequenceAndMove(anim, 1, self.FaceEnemy)
end
```

### Context-Aware Animations

```lua
function ENT:OnMeleeAttack(enemy)
    local health_pct = self:Health() / self:GetMaxHealth()

    if health_pct < 0.3 then
        -- Desperate, frenzied attacks when low health
        self:PlayActivityAndMove(ACT_MELEE_ATTACK2, 1.5, self.FaceEnemy)
    elseif self:IsOnFire() then
        -- Different attack when on fire
        self:PlaySequenceAndMove("attack_burning", 1.2, self.FaceEnemy)
    else
        -- Normal attack
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end
end
```

### Smooth Animation Transitions

```lua
function ENT:CustomInitialize()
    self.CurrentAnimation = ACT_IDLE
    self.AnimationTransitioning = false
end

function ENT:SmoothTransition(newAnim)
    if self.AnimationTransitioning then return end
    if self.CurrentAnimation == newAnim then return end

    self.AnimationTransitioning = true

    -- Fade out current animation
    self:SetPlaybackRate(2.0) -- Speed up to finish quickly

    self:Timer(0.2, function(self)
        -- Play new animation
        self:PlayActivity(newAnim)
        self.CurrentAnimation = newAnim
        self.AnimationTransitioning = false
    end)
end
```

### Animation State Machine

```lua
function ENT:CustomInitialize()
    self.AnimState = "idle"
end

function ENT:CustomThink()
    local newState = self:DetermineAnimState()

    if newState != self.AnimState then
        self:ChangeAnimState(newState)
    end
end

function ENT:DetermineAnimState()
    if self:IsAttacking() then
        return "attacking"
    elseif self:GetVelocity():Length() > 200 then
        return "running"
    elseif self:GetVelocity():Length() > 50 then
        return "walking"
    elseif not self:IsOnGround() then
        return "airborne"
    else
        return "idle"
    end
end

function ENT:ChangeAnimState(newState)
    self.AnimState = newState

    if newState == "idle" then
        self:PlayActivity(ACT_IDLE)
    elseif newState == "walking" then
        self:PlayActivity(ACT_WALK)
    elseif newState == "running" then
        self:PlayActivity(ACT_RUN)
    elseif newState == "airborne" then
        self:PlayActivity(ACT_JUMP)
    elseif newState == "attacking" then
        -- Handled by OnMeleeAttack
    end
end
```

## Debugging Animations

### Print Available Animations

```lua
function ENT:CustomInitialize()
    local info = self:GetAnimInfo()
    print("=== Available Animations ===")
    PrintTable(info)

    -- Print sequences
    for i = 0, self:GetSequenceCount() - 1 do
        local name = self:GetSequenceName(i)
        local activity = self:GetSequenceActivity(i)
        print(i, name, activity)
    end
end
```

### Visual Debug

```lua
function ENT:CustomThink()
    if GetConVar("drgbase_debug"):GetBool() then
        -- Show current animation
        local seq = self:GetSequenceName(self:GetSequence())
        local cycle = self:GetCycle()
        debugoverlay.EntityTextAtPosition(
            self:GetPos(),
            0,
            "Anim: " .. seq,
            0.1,
            Color(255, 255, 255)
        )
        debugoverlay.EntityTextAtPosition(
            self:GetPos(),
            1,
            "Cycle: " .. math.Round(cycle, 2),
            0.1,
            Color(255, 255, 255)
        )

        -- Show skeleton
        for i = 0, self:GetBoneCount() - 1 do
            local pos = self:GetBonePosition(i)
            if pos then
                debugoverlay.Cross(pos, 2, 0.1, Color(255, 0, 0), true)
            end
        end
    end
end
```

### Check if Animation Exists

```lua
function ENT:PlayActivitySafe(activity)
    local seq = self:SelectWeightedSequence(activity)

    if seq == -1 or seq == 0 then
        print("Warning: Animation not found for activity " .. activity)
        return false
    end

    self:PlayActivity(activity)
    return true
end
```

## Sprite Animations

For sprite-based NPCs (see also [Sprite NPCs](05-sprite-npcs.md)):

```lua
ENT.Base = "drgbase_nextbot_sprite"
ENT.SpriteFolder = "my_sprites/character"
ENT.FramesPerSecond = 8

-- Define sprite animations
ENT.WalkAnimation = "walk"     -- Uses walk_1.png, walk_2.png, etc.
ENT.IdleAnimation = "idle"
ENT.AttackAnimation = "attack"

function ENT:CustomInitialize()
    -- Register sprite animation events
    self:SpriteAnimEvent("walk", {1, 3}, function(self, frame)
        self:EmitFootstep()
    end)

    self:SpriteAnimEvent("attack", {2}, function(self, frame)
        self:Attack({damage = 15})
    end)
end

-- Play sprite animation
function ENT:OnMeleeAttack(enemy)
    self:PlaySpriteAnimation("attack", 1.2, function(self)
        print("Attack animation done")
    end)
end
```

## Common Issues

### Animation not playing
```lua
-- Check if activity exists
local seq = self:SelectWeightedSequence(ACT_MELEE_ATTACK1)
if seq == -1 then
    print("Activity not found, using sequence name")
    self:PlaySequence("zombie_attack_01")
end
```

### Animation plays too fast/slow
```lua
-- Adjust playback rate
self:PlayActivity(ACT_WALK, 0.8) -- 80% speed

-- Or globally
ENT.WalkAnimRate = 0.8
ENT.RunAnimRate = 1.2
```

### Animation stutters
```lua
-- Don't call PlayActivity every frame!
-- Bad:
function ENT:CustomThink()
    self:PlayActivity(ACT_WALK) -- Restarts every frame!
end

-- Good:
function ENT:CustomThink()
    if self:GetSequence() != self:LookupSequence("walk_all") then
        self:PlayActivity(ACT_WALK) -- Only play if not already playing
    end
end
```

### Attack animation doesn't deal damage
```lua
-- Use OnAnimEvent with proper cycle check
function ENT:OnAnimEvent()
    if self:IsAttacking() then
        local cycle = self:GetCycle()
        -- Make sure this only triggers once per attack
        if cycle > 0.3 and cycle < 0.35 then
            self:Attack({damage = 10})
        end
    end
end
```

## Animation Info Structure

```lua
local info = self:GetAnimInfo()
-- Returns:
{
    sequences = {
        {name = "idle", activity = ACT_IDLE, duration = 2.5},
        {name = "walk_all", activity = ACT_WALK, duration = 1.2},
        ...
    },
    activities = {
        [ACT_IDLE] = {"idle", "idle_subtle"},
        [ACT_WALK] = {"walk_all"},
        ...
    }
}
```

## Utility Functions

```lua
-- Get animation info
local info = self:GetAnimInfo()

-- Get current sequence name
local name = self:GetSequenceName(self:GetSequence())

-- Get sequence duration
local duration = self:SequenceDuration(self:GetSequence())

-- Get sequence activity
local activity = self:GetSequenceActivity(self:GetSequence())

-- Lookup sequence by name
local seqID = self:LookupSequence("walk_all")

-- Lookup activity
local seqID = self:SelectWeightedSequence(ACT_WALK)

-- Check if animation is playing
if self:GetSequence() == self:LookupSequence("attack") then
    print("Attack animation playing")
end

-- Get bone position
local pos, ang = self:GetBonePosition(boneID)

-- Get attachment position
local attachment = self:GetAttachment(self:LookupAttachment("eyes"))
local pos = attachment.Pos
local ang = attachment.Ang
```

## Source Reference

Animations: `lua/entities/drgbase_nextbot/animations.lua` (552 lines)
Sprite Animations: `lua/entities/drgbase_nextbot_sprite/animations.lua`

## Related Documentation

- [Sprite NPCs](05-sprite-npcs.md)
- [Melee Combat](03-melee-combat.md)
- [Model System](../reference/models.md)
