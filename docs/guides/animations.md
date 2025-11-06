# Animation System Guide

Learn how to configure and control animations for your DrGBase nextbots.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Basic Animation Configuration](#basic-animation-configuration)
3. [Playing Animations](#playing-animations)
4. [Pose Parameters](#pose-parameters)
5. [Animation Events and Callbacks](#animation-events-and-callbacks)
6. [Common Patterns](#common-patterns)
7. [Troubleshooting](#troubleshooting)

---

## Introduction

The DrGBase animation system provides powerful tools for controlling your nextbot's animations. It handles:

- **Automatic animation management** - Idle, walk, run animations based on movement state
- **Manual animation control** - Play specific animations when needed
- **Animation blending** - Layer gestures on top of base animations
- **Motion-matched movement** - Entity moves according to animation data
- **Pose parameters** - Dynamic aiming, looking, and procedural adjustments
- **Animation events** - Trigger code at specific points in animations

### Key Concepts

**Sequences vs Activities:**
- **Sequence** - A specific animation by name (e.g., "walk_all")
- **Activity** - An abstract animation type that maps to one or more sequences (e.g., `ACT_WALK`)
- Activities allow the system to randomly select from multiple sequences, adding variety

**Animation Layers:**
- **Base layer** - The main animation (idle, walk, run, etc.)
- **Gesture layers** - Additional animations that blend on top (reload, signal, etc.)

**Playback Modes:**
- **Non-blocking** - Animation plays as a gesture, doesn't interrupt base animation
- **Blocking** - Animation replaces base animation and pauses the coroutine
- **Movement** - Animation plays and moves the entity according to motion data

---

## Basic Animation Configuration

Configure your nextbot's automatic animations by setting these properties in your entity file.

### Default Animation Properties

```lua
ENT.Base = "drgbase_nextbot"

-- Basic movement animations
ENT.IdleAnimation = ACT_IDLE
ENT.IdleAnimRate = 1

ENT.WalkAnimation = ACT_WALK
ENT.WalkAnimRate = 1

ENT.RunAnimation = ACT_RUN
ENT.RunAnimRate = 1

ENT.JumpAnimation = ACT_JUMP
ENT.JumpAnimRate = 1

-- Climbing animations (if climbing is enabled)
ENT.ClimbUpAnimation = ACT_CLIMB_UP
ENT.ClimbDownAnimation = ACT_CLIMB_DOWN
ENT.ClimbAnimRate = 1
```

### Using Sequence Names

You can use sequence names instead of activities:

```lua
-- Use specific sequences
ENT.IdleAnimation = "idle_subtle"
ENT.WalkAnimation = "walk_all"
ENT.RunAnimation = "run_all"
```

**Tip:** Use activities when possible - they allow the engine to pick from multiple similar animations randomly, adding variety to your nextbot's behavior.

### Checking Available Animations

To see what animations your model has:

1. Spawn your nextbot
2. Open console and type: `ent_info` while looking at it
3. Or use the Model Viewer: `Tools > Model Viewer` in GMod

### Example: Zombie Nextbot

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Zombie"
ENT.Model = "models/zombie/classic.mdl"

-- Zombies are slow
ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_WALK  -- Use walk animation for running too
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1.5  -- Speed up walk animation when running
```

### Example: Fast Headcrab

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Fast Headcrab"
ENT.Model = "models/headcrabclassic.mdl"

ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN

-- Headcrabs are very active
ENT.IdleAnimRate = 1.2
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1.5
```

---

## Playing Animations

### Automatic Animation System

By default, DrGBase automatically chooses animations based on your nextbot's state:

```lua
-- This happens automatically!
-- No code needed - just set the properties shown above
```

The system checks (in order):
1. Is the nextbot down or dead? → No animation
2. Is it climbing up? → `ClimbUpAnimation`
3. Is it climbing down? → `ClimbDownAnimation`
4. Is it in the air? → `JumpAnimation`
5. Is it running? → `RunAnimation`
6. Is it moving? → `WalkAnimation`
7. Otherwise → `IdleAnimation`

### Overriding Automatic Animations

You can customize the automatic animation logic:

```lua
function ENT:OnUpdateAnimation()
    -- Custom animation state machine
    if self:IsDown() or self:IsDead() then
        return -- No animation when down/dead
    end

    -- Check custom states first
    if self:GetIsCharging() then
        return ACT_RUN_PROTECTED, 2.0  -- Fast charge animation
    end

    if self:GetIsCrouching() then
        if self:IsMoving() then
            return ACT_WALK_CROUCH
        else
            return ACT_CROUCH_IDLE
        end
    end

    -- Fall back to default behavior
    if self:IsRunning() then return self.RunAnimation, self.RunAnimRate
    elseif self:IsMoving() then return self.WalkAnimation, self.WalkAnimRate
    else return self.IdleAnimation, self.IdleAnimRate end
end
```

### Playing One-Off Animations

#### Non-Blocking Gestures

Use `PlayAnimation()` to play gestures that don't interrupt movement:

```lua
function ENT:OnRangeAttack(target)
    -- Play attack gesture while continuing to move
    self:PlayAnimation(ACT_GESTURE_RANGE_ATTACK1, 1.0)

    -- Fire weapon
    self:Timer(0.3, function()
        self:ShootAtTarget(target)
    end)
end
```

#### Blocking Animations

Use `PlayAnimationAndWait()` to play an animation and pause execution:

```lua
function ENT:CustomBehavior()
    -- Must be in a coroutine (like AIBehaviour or ReactInCoroutine)

    -- Move to target
    self:FollowPath(target)

    -- Stop and play animation, wait until complete
    self:PlayAnimationAndWait(ACT_MELEE_ATTACK1, 1.0)

    -- Now deal damage
    self:DealDamageInRadius(100, self.MeleeAttackRange)

    -- Wait a bit
    self:PauseCoroutine(2.0)
end
```

#### Animations with Movement

Use `PlayAnimationAndMove()` to play animations that move the entity:

```lua
function ENT:DoDodgeRoll()
    -- Play roll animation and move according to its motion data
    self:PlayAnimationAndMove("roll_forward", {
        rate = 1.2,          -- Play 20% faster
        gravity = false,     -- Ignore gravity during roll
        collisions = true    -- Stop if we hit something
    })
end
```

### Advanced Animation Control

#### Animation Callbacks

Execute code during animation playback:

```lua
-- Gesture with callback
self:PlayAnimation(ACT_RELOAD, 1.0, function(self, cycle, layerID)
    -- cycle = 0.0 to 1.0 (animation progress)

    if cycle >= 0.5 and not self.reloadSoundPlayed then
        self:EmitSound("weapons/reload.wav")
        self.reloadSoundPlayed = true
    end

    if cycle >= 1.0 then
        self.reloadSoundPlayed = false
        return true  -- Stop callback
    end
end)

-- Blocking animation with callback
self:PlayAnimationAndWait("special_attack", 1.0, function(self, cycle)
    if cycle >= 0.3 then
        self:DealDamage()
        return true  -- Stop animation early
    end
end)
```

#### Stopping Animations Early

```lua
function ENT:DoInterruptibleAttack()
    self:PlayAnimationAndWait("long_attack", 1.0, function(self, cycle)
        -- Check if we should cancel
        if not IsValid(self:GetEnemy()) then
            return true  -- Cancel animation
        end

        if cycle >= 0.6 then
            -- Attack happens at 60% through
            self:MeleeAttack()
        end
    end)
end
```

#### Complex Movement Animations

```lua
function ENT:DoChargeAttack(target)
    local startPos = self:GetPos()

    self:PlayAnimationAndMove(ACT_CHARGE, {
        rate = 1.5,
        gravity = false,
        stoponcollide = true,  -- Stop when hitting something
        multiply = Vector(1.5, 1.5, 1)  -- Move 50% further
    }, function(self, cycle, collided)
        -- Check collision
        if collided then
            -- Hit something!
            self:EmitSound("physics/metal/metal_solid_impact_hard.wav")
            self:DealDamageInRadius(50, 100)
            return true  -- Stop animation
        end

        -- Create trail effect
        if math.random() > 0.7 then
            local effectdata = EffectData()
            effectdata:SetOrigin(self:GetPos())
            util.Effect("ManhackSparks", effectdata)
        end
    end)
end
```

### Absolute Movement

For animations that should ignore physics:

```lua
function ENT:DoTeleportDash()
    -- Dash forward, ignoring gravity and collisions
    self:PlayAnimationAndMoveAbsolute("dash_forward", 2.0)

    -- Create effect at start and end
    self:CreateEffect("teleport_start")
    self:Timer(0.5, function()
        self:CreateEffect("teleport_end")
    end)
end
```

### Climbing Animations

DrGBase handles climbing automatically, but you can manually play climb animations:

```lua
function ENT:ClimbLedge(height)
    -- Automatically scales animation to match height
    self:PlayClimbAnimation(self.ClimbUpAnimation, height, 1.0)
end

-- Or with multiple animations (picks best match for height)
function ENT:ClimbLedge(height)
    self:PlayClimbAnimation({
        "climb_small",   -- ~32 units
        "climb_medium",  -- ~64 units
        "climb_large"    -- ~128 units
    }, height, 1.0)
end
```

---

## Pose Parameters

Pose parameters allow dynamic adjustments to animations without switching animations.

### Common Pose Parameters

Most player models include these pose parameters:

- `aim_pitch` / `aim_yaw` - Aiming direction
- `head_pitch` / `head_yaw` - Head looking direction
- `move_x` / `move_y` - Movement direction (automatically set by DrGBase)
- `move_yaw` - Movement angle relative to facing

### Setting Pose Parameters

```lua
-- Manual control
self:SetPoseParameter("aim_pitch", 45)
self:SetPoseParameter("aim_yaw", -20)

-- Get current value
local currentPitch = self:GetPoseParameter("aim_pitch")
```

### Aiming at Targets

Use `DirectPoseParametersAt()` to automatically aim pose parameters:

```lua
function ENT:Think()
    local enemy = self:GetEnemy()

    if IsValid(enemy) then
        -- Aim head at enemy
        self:DirectPoseParametersAt(enemy, "aim_pitch", "aim_yaw")

        -- Or use shortened syntax (adds _pitch and _yaw automatically)
        self:DirectPoseParametersAt(enemy, "aim")
    else
        -- Reset to neutral
        self:DirectPoseParametersAt(nil, "aim")
    end
end
```

### Advanced Pose Parameter Usage

```lua
function ENT:CustomThink()
    local enemy = self:GetEnemy()

    if IsValid(enemy) then
        -- Get eye position for more accurate aiming
        local eyeBone = self:LookupBone("ValveBiped.Bip01_Head1")
        local eyePos = self:GetBonePosition(eyeBone)

        -- Aim from eye position
        self:DirectPoseParametersAt(
            enemy:WorldSpaceCenter(),  -- Target position
            "aim_pitch",                -- Pitch parameter
            "aim_yaw",                  -- Yaw parameter
            eyePos                      -- Origin point
        )

        -- Also turn body toward target
        self:DirectPoseParametersAt(enemy, "body_yaw")
    end
end
```

### Example: Looking Around

```lua
function ENT:OnIdle()
    -- Look at random positions while idle
    self:ReactInCoroutine(function()
        while not self:HasEnemy() do
            -- Pick random position to look at
            local lookPos = self:GetPos() + VectorRand() * 500
            lookPos.z = lookPos.z + math.random(-100, 100)

            -- Smoothly interpolate to look position
            local duration = 2.0
            local startTime = CurTime()

            while CurTime() < startTime + duration do
                local progress = (CurTime() - startTime) / duration
                self:DirectPoseParametersAt(lookPos, "aim")
                self:YieldCoroutine()
            end

            -- Hold for a bit
            self:PauseCoroutine(math.random(1, 3))
        end
    end)
end
```

---

## Animation Events and Callbacks

Animation events allow you to trigger code at specific points in an animation.

### Built-in Animation Events

Many models have built-in events in their animations:

```lua
function ENT:OnAnimEvent(options, event, pos, angles)
    if event == 1 or event == 2 or event == 3 or event == 4 then
        -- Footstep events (many models use events 1-4)
        self:EmitSound("npc/zombie/foot" .. math.random(1, 3) .. ".wav")

        -- Create dust effect
        local tr = util.TraceLine({
            start = pos,
            endpos = pos - Vector(0, 0, 20),
            filter = self
        })

        if tr.Hit then
            local effectdata = EffectData()
            effectdata:SetOrigin(tr.HitPos)
            effectdata:SetScale(1)
            util.Effect("ManhackSparks", effectdata)
        end

        return true
    end

    if event == 5 then
        -- Custom attack event
        self:MeleeAttack()
        return true
    end
end
```

### Adding Custom Events

You can add events to sequences that don't have them:

```lua
function ENT:Initialize()
    -- Add attack event at frame 15 of melee animation
    self:AddAnimEvent("melee_attack", 15, 100)  -- Event ID 100

    -- Add multiple footstep events
    self:AddAnimEvent("walk", {10, 25}, 1)  -- Event 1 at frames 10 and 25
end

function ENT:OnAnimEvent(options, event, pos, angles)
    if event == 100 then
        -- Our custom attack event
        self:DealDamageInRadius(50, self.MeleeAttackRange)
        return true
    end

    if event == 1 then
        self:EmitSound("footstep.wav")
        return true
    end
end
```

### Sequence Events (Cycle-Based)

For more control, use `SequenceEvent()` to trigger at specific animation cycles:

```lua
function ENT:Initialize()
    -- Fire callback at 50% through attack animation
    self:SequenceEvent("attack", 0.5, function(self)
        self:FireProjectile()
    end)

    -- Fire at multiple points
    self:SequenceEvent("combo_attack", {0.3, 0.6, 0.9}, function(self)
        self:MeleeAttack()
    end)

    -- Pass additional arguments
    self:SequenceEvent("reload", 0.7, function(self, msg)
        self:EmitSound("weapons/reload.wav")
        print(msg)
    end, "Reload complete!")
end
```

### Clearing Events

```lua
-- Clear specific sequence
self:ClearSequenceEvents("attack")

-- Clear all sequence events
self:ClearSequenceEvents()
```

### Example: Complete Attack System

```lua
function ENT:Initialize()
    -- Set up animation events
    self:SequenceEvent("attack_slash", 0.5, function(self)
        self:DoSlashDamage()
    end)

    self:SequenceEvent("attack_stab", 0.6, function(self)
        self:DoStabDamage()
    end)

    self:SequenceEvent("roar", {0.3, 0.6}, function(self)
        self:EmitSound("npc/zombie/zombie_voice_idle" .. math.random(1, 14) .. ".wav")
    end)
end

function ENT:OnMeleeAttack(target)
    -- Pick random attack
    local attacks = {"attack_slash", "attack_stab"}
    local attack = attacks[math.random(#attacks)]

    -- Play attack animation
    self:PlayAnimationAndWait(attack, 1.0)

    return true  -- We handled the attack
end

function ENT:DoSlashDamage()
    -- Wide arc attack
    local targets = self:FindInCone(self:GetPos(), self:GetForward(), 100, 60)
    for _, ent in ipairs(targets) do
        if self:GetRelationship(ent) == D_HT then
            self:DealDamageTo(ent, 25)
        end
    end

    -- Effect
    self:EmitSound("weapons/knife/knife_slash.wav")
end

function ENT:DoStabDamage()
    -- Forward stab
    local tr = util.TraceLine({
        start = self:GetPos() + Vector(0, 0, 40),
        endpos = self:GetPos() + self:GetForward() * 60 + Vector(0, 0, 40),
        filter = self
    })

    if IsValid(tr.Entity) and self:GetRelationship(tr.Entity) == D_HT then
        self:DealDamageTo(tr.Entity, 40)
        self:EmitSound("weapons/knife/knife_hitwall1.wav")
    end
end
```

---

## Common Patterns

### Pattern: State-Based Animations

```lua
-- Use NW vars to track animation states
function ENT:Initialize()
    self:SetNWBool("IsCharging", false)
    self:SetNWBool("IsDefending", false)
end

function ENT:OnUpdateAnimation()
    if self:IsDown() or self:IsDead() then return end

    -- Priority order matters!
    if self:GetNWBool("IsCharging") then
        return "charge", 2.0
    end

    if self:GetNWBool("IsDefending") then
        return "defend_idle"
    end

    -- Default movement animations
    if self:IsRunning() then return self.RunAnimation, self.RunAnimRate
    elseif self:IsMoving() then return self.WalkAnimation, self.WalkAnimRate
    else return self.IdleAnimation, self.IdleAnimRate end
end

function ENT:StartCharging()
    self:SetNWBool("IsCharging", true)
    self:UpdateAnimation()
end

function ENT:StopCharging()
    self:SetNWBool("IsCharging", false)
    self:UpdateAnimation()
end
```

### Pattern: Animation Sequences

Chain multiple animations together:

```lua
function ENT:DoCoolMoveSequence()
    -- Must be in coroutine

    -- Play animations in sequence
    self:PlayAnimationAndWait("windup", 1.0)
    self:PlayAnimationAndMove("spin_attack", 1.5)
    self:PlayAnimationAndWait("recovery", 0.8)

    -- Continue with normal behavior
end
```

### Pattern: Interrupt Protection

Prevent interrupting important animations:

```lua
function ENT:OnAnimChange(oldSeq, newSeq)
    -- Don't interrupt these animations
    local protected = {
        "special_attack",
        "death",
        "spawn"
    }

    if table.HasValue(protected, oldSeq) then
        return false  -- Block animation change
    end
end
```

### Pattern: Speed-Adjusted Animations

Automatically adjust animation speed to movement:

```lua
-- This happens automatically for walk/run animations!
-- The playback rate matches velocity

-- But you can customize it:
function ENT:OnUpdateAnimation()
    if self:IsMoving() then
        local speed = self:GetVelocity():Length()
        local rate = speed / 100  -- Adjust as needed
        return self.WalkAnimation, rate
    end

    return self.IdleAnimation, 1.0
end
```

### Pattern: Synchronized Actions

Synchronize effects with animations:

```lua
function ENT:DoGroundPound()
    local duration = self:PlayAnimationAndWait("ground_pound", 1.0, function(self, cycle)
        -- Impact at 70% through animation
        if cycle >= 0.7 and not self.hasImpacted then
            self.hasImpacted = true
            self:CreateShockwave()
            self:DealDamageInRadius(100, 300)
            util.ScreenShake(self:GetPos(), 10, 5, 1, 500)
        end
    end)

    self.hasImpacted = false
end
```

### Pattern: Layered Gestures

Combine base animation with gestures:

```lua
function ENT:OnRangeAttack(target)
    -- Keep moving/idle animation as base
    -- Add shooting gesture on top

    if self:HasWeapon() then
        local weapon = self:GetWeapon()

        -- Play reload gesture if needed
        if weapon:Clip1() <= 0 then
            self:PlayAnimation(ACT_RELOAD, 1.0)
            self:Timer(1.5, function()
                weapon:SetClip1(weapon:GetMaxClip1())
            end)
        else
            -- Play shoot gesture
            self:PlayAnimation(ACT_GESTURE_RANGE_ATTACK1, 1.5, function(self, cycle)
                if cycle >= 0.3 and cycle < 0.4 then
                    self:FireWeapon()
                end
            end)
        end
    end

    return true  -- We handled it
end
```

### Pattern: Context-Aware Animations

Different animations based on situation:

```lua
function ENT:OnUpdateAnimation()
    if self:IsDown() or self:IsDead() then return end

    local enemy = self:GetEnemy()

    -- No enemy: relaxed animations
    if not IsValid(enemy) then
        if self:IsMoving() then
            return "walk_relaxed", 0.9
        else
            return "idle_relaxed", 1.0
        end
    end

    -- Has enemy: alert animations
    if self:IsMoving() then
        if self:IsRunning() then
            return "run_alert", 1.2
        else
            return "walk_alert", 1.1
        end
    else
        return "idle_alert", 1.0
    end
end
```

---

## Troubleshooting

### Animation Not Playing

**Problem:** Animation doesn't appear to play

**Solutions:**
1. Check if the animation exists:
```lua
local seq = self:LookupSequence("my_animation")
if seq == -1 then
    print("Animation not found!")
end
```

2. Check if another animation is blocking it:
```lua
if self:IsPlayingAnimation() then
    print("Already playing animation:", self:GetSequenceName(self:GetSequence()))
end
```

3. Use the debug convar:
```
drgbase_debug_animations 1
```

### Animation Plays Too Fast/Slow

**Problem:** Animation doesn't match movement speed

**Solutions:**
1. Adjust the animation rate property:
```lua
ENT.WalkAnimRate = 1.2  -- Play 20% faster
```

2. For manual animations, set the rate:
```lua
self:PlayAnimation(ACT_WALK, 1.5)  -- 1.5x speed
```

3. Let DrGBase handle it automatically (it matches playback to velocity)

### Animation Stutters or Jitters

**Problem:** Animation looks choppy

**Solutions:**
1. Check your `Think()` frequency (don't call UpdateAnimation too often)
2. Make sure frame rate is stable
3. Don't rapidly change animations:
```lua
-- Bad: Changes every think
function ENT:OnUpdateAnimation()
    return math.random() > 0.5 and ACT_IDLE or ACT_WALK  -- Don't do this!
end

-- Good: Only changes when state changes
function ENT:OnUpdateAnimation()
    if self:IsMoving() then
        return ACT_WALK
    else
        return ACT_IDLE
    end
end
```

### Pose Parameters Don't Work

**Problem:** Pose parameters have no effect

**Solutions:**
1. Check if your model has the pose parameter:
```lua
local min, max = self:GetPoseParameterRange(self:LookupPoseParameter("aim_pitch"))
print("Range:", min, max)  -- If 0,0 then parameter doesn't exist
```

2. Make sure you're calling it every frame:
```lua
function ENT:Think()
    if IsValid(self:GetEnemy()) then
        self:DirectPoseParametersAt(self:GetEnemy(), "aim")
    end
end
```

3. Some models have unusual parameter names - check in Model Viewer

### PlayAnimationAndWait Doesn't Wait

**Problem:** `PlayAnimationAndWait()` returns immediately

**Solution:** It must be called from a coroutine:
```lua
-- Wrong:
function ENT:CustomThink()
    self:PlayAnimationAndWait("attack", 1.0)  -- Won't work!
end

-- Right:
function ENT:AIBehaviour()
    self:PlayAnimationAndWait("attack", 1.0)  -- Works!
end

-- Or use ReactInCoroutine:
function ENT:CustomThink()
    self:ReactInCoroutine(function()
        self:PlayAnimationAndWait("attack", 1.0)  -- Works!
    end)
end
```

### Movement Animation Doesn't Move

**Problem:** Using `PlayAnimationAndMove()` but entity doesn't move

**Solutions:**
1. Check if animation has movement data:
```lua
local seq = self:LookupSequence("my_anim")
local success, movement = self:GetSequenceMovement(seq, 0, 1)
print("Has movement:", success, movement)
```

2. Many animations don't have movement data - you may need to use absolute movement:
```lua
self:PlayAnimationAndMoveAbsolute("attack", 1.0)
```

3. Or manually move during callback:
```lua
self:PlayAnimationAndWait("attack", 1.0, function(self, cycle)
    self:SetPos(self:GetPos() + self:GetForward() * 5)
end)
```

### Animation Events Not Firing

**Problem:** `OnAnimEvent` not being called

**Solutions:**
1. Check if animation has events (use Model Viewer)
2. Make sure you're returning `true` if you handle the event:
```lua
function ENT:OnAnimEvent(options, event, pos, angles)
    if event == 1 then
        print("Footstep!")
        return true  -- Important!
    end
end
```

3. Add events manually if missing:
```lua
function ENT:Initialize()
    self:AddAnimEvent("walk", {10, 20}, 1)  -- Add footstep events
end
```

### Animation Doesn't Loop

**Problem:** Animation plays once then stops

**Solution:** Check if animation is set to loop in the model. Most movement animations loop automatically. For manual animations:
```lua
-- Use gesture (non-blocking) for looping
self:PlayAnimation("idle_loop", 1.0)

-- Or play repeatedly in coroutine
self:ReactInCoroutine(function()
    while true do
        self:PlayAnimationAndWait("idle_no_loop", 1.0)
    end
end)
```

### Entity Slides During Animation

**Problem:** Entity's position updates but animation plays in place

**Solutions:**
1. Use `PlayAnimationAndMove()` instead of `PlayAnimationAndWait()`
2. Or disable automatic movement:
```lua
self.UseWalkframes = false  -- Disable automatic walkframe-based movement
```

---

## Additional Resources

- [Animation API Reference](/docs/api/nextbot/animation.md) - Complete function documentation
- [Base Entity Configuration](/docs/api/nextbot/base-config.md) - All animation properties
- [Behavior System Guide](/docs/guides/behaviors.md) - Using animations in AI behaviors
- [Model Specifications](/docs/reference/models.md) - Model requirements and animation standards

---

## Quick Reference

### Properties
```lua
ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.JumpAnimation = ACT_JUMP
ENT.ClimbUpAnimation = ACT_CLIMB_UP
ENT.ClimbDownAnimation = ACT_CLIMB_DOWN

ENT.IdleAnimRate = 1
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1
ENT.JumpAnimRate = 1
ENT.ClimbAnimRate = 1
```

### Playing Animations
```lua
-- Gesture (non-blocking)
self:PlayAnimation(anim, rate, callback)

-- Blocking (waits in coroutine)
self:PlayAnimationAndWait(anim, rate, callback)

-- With movement
self:PlayAnimationAndMove(anim, options, callback)
self:PlayAnimationAndMoveAbsolute(anim, options, callback)

-- Climbing
self:PlayClimbAnimation(anim, height, rate, callback)
```

### Pose Parameters
```lua
-- Set manually
self:SetPoseParameter("aim_pitch", 45)

-- Aim at target
self:DirectPoseParametersAt(target, "aim_pitch", "aim_yaw")
self:DirectPoseParametersAt(target, "aim")  -- Shortened
```

### Animation Events
```lua
-- Hook
function ENT:OnAnimEvent(options, event, pos, angles)
    if event == 1 then
        -- Handle event
        return true
    end
end

-- Add events
self:AddAnimEvent(seq, frames, event)
self:SequenceEvent(seq, cycles, callback, ...)
self:ClearSequenceEvents(seq)
```

### Hooks
```lua
function ENT:OnUpdateAnimation()
    return anim, rate
end

function ENT:OnAnimChange(oldSeq, newSeq)
    return false  -- Prevent change
end

function ENT:OnAnimEvent(options, event, pos, angles)
    return true  -- Handled
end
```
