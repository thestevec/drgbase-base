# Animation System

Animation control, sequences, and activities.

**File:** `animations.lua`

## Overview

The animation system provides fine-grained control over NPC animations, including sequences, activities, pose parameters, animation events, and gestures. It supports both blocking animations (wait for completion) and non-blocking gestures (layered animations). Key features:

- **Sequences** - Play specific animation sequences by name
- **Activities** - Play activities (ACT_*) that select appropriate sequences
- **Pose Parameters** - Control animation blending (aim, movement direction, etc.)
- **Animation Events** - Trigger code at specific animation frames
- **Gestures** - Layer animations over base animation
- **Movement Animations** - Play animations with synchronized movement
- **Climbing Animations** - Scale animations to match climb height

## Components

### Sequences

Sequences are the raw animation names defined in the model (e.g., "walk_all", "idle_angry").

**Playing Sequences (Blocking):**
```lua
-- Play and wait for completion
self:PlaySequenceAndWait("reload_smg1")

-- Play with custom rate
self:PlaySequenceAndWait("reload_smg1", 1.5)  -- 1.5x speed

-- Play with callback each frame
self:PlaySequenceAndWait("reload_smg1", 1, function(self, cycle)
    if cycle > 0.5 then return true end  -- Stop at 50%
end)
```

**Playing Sequences (Non-Blocking Gestures):**
```lua
-- Play as gesture layer
local duration = self:PlaySequence("gesture_signal_advance")

-- Play with custom rate and callback
self:PlaySequence("gesture_signal_advance", 1.5, function(self, cycle, layerID)
    print("Gesture at " .. (cycle * 100) .. "%")
end)
```

**Checking Sequence Status:**
```lua
if self:IsPlayingSequence("reload_smg1") then
    print("Currently reloading")
end
```

### Activities

Activities (ACT_*) are high-level animation identifiers that automatically select an appropriate sequence. For example, ACT_WALK may map to "walk_all" or "walk_angry" depending on the model.

**Playing Activities:**
```lua
-- Play and wait
self:PlayActivityAndWait(ACT_RELOAD)

-- Play as gesture
self:PlayActivity(ACT_GESTURE_RANGE_ATTACK1)

-- Check if playing
if self:IsPlayingActivity(ACT_RELOAD) then
    print("Reloading")
end
```

**Activity by Name:**
```lua
-- Get activity ID from name string
local actID = self:GetActivityIDFromName("ACT_RUN")
self:PlayActivityAndWait(actID)
```

**Random Sequence Selection:**
```lua
-- Select random sequence for activity
local seq = self:SelectRandomSequence(ACT_IDLE)
self:PlaySequenceAndWait(seq)
```

### Pose Parameters

Pose parameters blend between animations. Common uses include aim direction and movement direction.

**Basic Usage:**
```lua
-- Set pose parameter (0-1 or -1 to 1 depending on parameter)
self:SetPoseParameter("aim_yaw", 45)
self:SetPoseParameter("aim_pitch", -15)

-- Movement direction (set automatically by BodyMoveXY)
self:SetPoseParameter("move_x", 1)  -- Forward
self:SetPoseParameter("move_y", -1)  -- Left
```

**Aiming at Position:**
```lua
-- Automatically set aim pose parameters
self:DirectPoseParametersAt(enemy:GetPos(), "aim_pitch", "aim_yaw")

-- Or use shorthand (assumes pitch/yaw suffix)
self:DirectPoseParametersAt(enemy:GetPos(), "aim")
```

**Automatic Pose Parameters:**

DrGBase automatically sets these if the model has them:
- `move_x` / `move_y` - Movement direction based on velocity
- `move_yaw` - Angle between forward and velocity
- Aim parameters when calling `DirectPoseParametersAt()`

### Animation Events

Animation events trigger code at specific frames or cycles of an animation.

**Frame-Based Events:**
```lua
-- Trigger at frame 10 of sequence
local seq = self:LookupSequence("attack")
self:AddAnimEvent(seq, 10, "swing")

-- Handle event
function ENT:OnAnimEvent(event, eventID, pos, ang)
    if event == "swing" then
        print("Sword swing!")
        self:MeleeAttack()
    end
end
```

**Cycle-Based Events:**
```lua
local seq = self:LookupSequence("reload")

-- Trigger at 50% completion (cycle 0.5)
self:SequenceEvent(seq, 0.5, function(self)
    self:EmitSound("weapons/smg1/smg1_reload.wav")
end)

-- Trigger at multiple cycles
self:SequenceEvent(seq, {0.25, 0.5, 0.75}, function(self)
    print("Checkpoint")
end)

-- Clear events for sequence
self:ClearSequenceEvents(seq)
```

**Model Animation Events:**

Models can have built-in animation events. Override `HandleAnimEvent()` or `OnAnimEvent()`:
```lua
function ENT:OnAnimEvent(event, eventID, pos, ang)
    if eventID == 1 then  -- Model event ID 1
        self:EmitSound("npc/zombie/foot1.wav")
    end
end
```

### Gestures

Gestures are layered animations that play on top of the base animation, allowing simultaneous animations (e.g., walking while reloading).

**Playing Gestures:**
```lua
-- Play gesture (non-blocking)
self:PlaySequence("gesture_reload")
self:PlayActivity(ACT_GESTURE_RELOAD)

-- Gestures run on separate layer, don't block movement
```

**Raw Gesture Control:**
```lua
-- Low-level gesture control
local layerID = self:AddGestureSequence(seq)
self:SetLayerPlaybackRate(layerID, 1.5)
```

## Configuration

**Animation Properties:**
```lua
-- Default animation rates
ENT.IdleAnimRate = 1
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1
ENT.JumpAnimRate = 1
ENT.ClimbAnimRate = 1

-- Default animations
ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.JumpAnimation = ACT_JUMP
ENT.ClimbUpAnimation = "climb_up"
ENT.ClimbDownAnimation = "climb_down"
```

**Animation Update:**

`OnUpdateAnimation()` is called automatically to select the current animation based on NPC state:

```lua
function ENT:OnUpdateAnimation()
    if self:IsRunning() then return self.RunAnimation, self.RunAnimRate
    elseif self:IsMoving() then return self.WalkAnimation, self.WalkAnimRate
    else return self.IdleAnimation, self.IdleAnimRate end
end
```

**ConVars:**
- `drgbase_debug_animations` (0) - Disable auto-animation updates for debugging

## Usage Examples

**Basic Animation Playback:**
```lua
-- Wait for animation to finish
self:PlaySequenceAndWait("taunt")

-- Play gesture without blocking
self:PlaySequence("gesture_salute")

-- Activity-based
self:PlayActivityAndWait(ACT_RELOAD)
```

**Animation with Movement:**
```lua
-- Play animation and move with it
self:PlayAnimationAndMove("walk_all")

-- Play with custom options
self:PlayAnimationAndMove("walk_all", {
    rate = 1.5,          -- 1.5x speed
    multiply = Vector(2, 2, 1),  -- 2x distance
    gravity = true,      -- Apply gravity
    collisions = true,   -- Stop on collision
    stoponcollide = true -- Return on collision
})

-- With callback
self:PlayAnimationAndMove("walk_all", 1, function(self, cycle, collided)
    if collided then
        print("Hit something!")
        return true  -- Stop animation
    end
end)
```

**Absolute Movement (Scripted Movement):**
```lua
-- Move exactly as animation dictates (no gravity/collision)
self:PlayAnimationAndMoveAbsolute("jump_attack", 1)

-- Useful for:
-- - Scripted takedowns
-- - Leap attacks
-- - Cinematic movements
```

**Climbing with Height Scaling:**
```lua
-- Play climb animation scaled to exact height
local height = 100  -- Units to climb
self:PlayClimbSequence("climb_up", height)

-- Or use multiple sequences and pick closest
self:PlayClimbSequence({
    "climb_up_short",
    "climb_up_medium",
    "climb_up_tall"
}, height)

-- System automatically scales animation
```

**Reload Animation:**
```lua
function ENT:OnRangeAttack()
    if self:IsWeaponPrimaryEmpty() then
        -- Reload with animation
        self:WeaponReload("gesture_reload")

        -- Or wait for completion
        self:PlaySequenceAndWait("reload_smg1")
        self:GetWeapon():SetClip1(30)
    end
end
```

**Attack Animation with Event:**
```lua
function ENT:Initialize()
    local attackSeq = self:LookupSequence("attack")

    -- Hit enemy at frame 15
    self:AddAnimEvent(attackSeq, 15, "hit")
end

function ENT:OnMeleeAttack()
    self:PlaySequenceAndWait("attack")
end

function ENT:OnAnimEvent(event, eventID, pos, ang)
    if event == "hit" then
        -- Deal damage at exact frame
        local enemy = self:GetEnemy()
        if IsValid(enemy) and self:GetRangeTo(enemy) < 50 then
            enemy:TakeDamage(25, self, self)
        end
    end
end
```

**Pose Parameter Aiming:**
```lua
function ENT:Think()
    if self:HasEnemy() then
        -- Aim upper body at enemy
        self:DirectPoseParametersAt(self:GetEnemy():EyePos(), "aim")
    else
        -- Reset aim to neutral
        self:DirectPoseParametersAt(nil, "aim")
    end
end
```

**Custom Animation Selection:**
```lua
function ENT:OnUpdateAnimation()
    if self:Health() < self:GetMaxHealth() * 0.25 then
        -- Limping animation when injured
        return self.InjuredAnimation, 0.75
    elseif self:GetEnemy() and self:HasWeapon() then
        -- Combat stance
        return self.CombatIdleAnimation, 1
    else
        -- Default behavior
        return self.IdleAnimation, 1
    end
end
```

**Animation Change Hook:**
```lua
function ENT:OnAnimChange(oldSeq, newSeq)
    print("Animation changed: " .. oldSeq .. " -> " .. newSeq)

    -- Prevent certain animation changes
    if newSeq == "death" and self:GetGodMode() then
        return false  -- Block animation change
    end
end

function ENT:OnAnimChanged(oldSeq, newSeq)
    -- Runs in coroutine after animation actually changed
    print("Animation is now: " .. newSeq)
end
```

**Sequential Animations:**
```lua
function ENT:PerformCombo()
    self:PlaySequenceAndWait("attack1")
    self:PlaySequenceAndWait("attack2")
    self:PlaySequenceAndWait("attack3")
    print("Combo finished!")
end

-- Call in coroutine
self:RunCoroutine(self.PerformCombo)
```

**Stop Animation Early:**
```lua
self:PlaySequenceAndWait("long_taunt", 1, function(self, cycle)
    -- Stop at 50%
    if cycle >= 0.5 then
        return true
    end
end)
```

## API Reference

- [Animation Functions](../../api/nextbot/animation.md)

## Best Practices

**Blocking vs Non-Blocking:**
- Use `PlaySequenceAndWait()` for actions that should complete (reload, taunt, death)
- Use `PlaySequence()` (gestures) for reactions that shouldn't interrupt movement (flinch, wave)
- Blocking animations must be called from coroutines (behavior hooks)
- Gestures can be called anywhere

**Movement Integration:**
- Let `OnUpdateAnimation()` handle normal locomotion animations
- Use `PlayAnimationAndMove()` for scripted movement sequences
- `PlayAnimationAndMoveAbsolute()` for precise movement (attacks, jumps)
- Set `UseWalkframes = true` property to use sequence ground speed automatically

**Animation Events:**
- Use events for precise timing (footsteps, impacts, effects)
- Prefer `AddAnimEvent()` for frame-based triggers
- Use `SequenceEvent()` for percentage-based triggers (50% through reload)
- Clear events when no longer needed to prevent memory leaks

**Pose Parameters:**
- Check if model has pose parameters before using them
- Use `DirectPoseParametersAt()` for automatic aim calculation
- Pose parameters are automatically handled for movement (`move_x`, `move_y`)
- Reset pose parameters when not in use (prevents stuck aim)

**Performance:**
- Avoid calling `PlaySequenceAndWait()` outside coroutines (will error)
- Use `IsPlayingSequence()` / `IsPlayingActivity()` to avoid duplicate animations
- Clear sequence events when done with them
- Gestures automatically clean up when finished

**Model Compatibility:**
- Use activities instead of sequences for cross-model compatibility
- Check if sequence exists: `self:LookupSequence(name) != -1`
- Not all models have all activities
- Use `SelectRandomSequence()` to handle multiple sequence variants
