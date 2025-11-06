# Animation System Functions

Animation control, sequences, and activities.

**File:** `lua/entities/drgbase_nextbot/animations.lua` (552 lines)

---

## Playing Animations

### ENT:PlayAnimation(anim, rate, callback)

Plays an animation as a gesture (layer) on top of the current animation.

**Realm:** 🔴 SERVER

**Parameters:**
- `anim` (string or number) - Sequence name or activity ID
- `rate` (number, optional) - Playback speed multiplier (default: 1.0)
- `callback` (function, optional) - Function called each frame: `callback(self, cycle, layerID)`, return `true` to stop

**Returns:**
- `number` - Animation duration in seconds

**Example:**
```lua
-- Play a reload gesture
self:PlayAnimation(ACT_RELOAD, 1.5)

-- Play with callback
self:PlayAnimation("gesture_shoot", 1.0, function(self, cycle, layerID)
    if cycle > 0.5 then
        -- Fire weapon halfway through animation
        self:FireProjectile()
        return true -- Stop callback
    end
end)
```

**Notes:**
- Does not interrupt current animation
- Plays on a separate animation layer
- If `anim` is a string, plays the sequence by name
- If `anim` is a number, plays the activity and selects a random weighted sequence

---

### ENT:PlayAnimationAndWait(anim, rate, callback)

Plays an animation and pauses the coroutine until it completes.

**Realm:** 🔴 SERVER

**Parameters:**
- `anim` (string or number) - Sequence name or activity ID
- `rate` (number, optional) - Playback speed multiplier (default: 1.0)
- `callback` (function, optional) - Function called each frame: `callback(self, cycle)`, return `true` to stop

**Returns:**
- `number` - Actual time waited in seconds

**Example:**
```lua
function ENT:CustomBehavior()
    -- Play attack animation and wait for completion
    self:PlayAnimationAndWait(ACT_MELEE_ATTACK1, 1.0)

    -- Now deal damage after animation completes
    self:DealDamage()

    -- Play cooldown animation
    self:PlayAnimationAndWait("reload", 0.8, function(self, cycle)
        if cycle > 0.5 then
            -- Halfway through, spawn effect
            self:CreateReloadEffect()
        end
    end)
end
```

**Notes:**
- MUST be called from within a coroutine
- Blocks execution until animation finishes
- Replaces current animation
- Can be interrupted by `OnAnimChange` returning `false`

---

### ENT:PlayAnimationAndMove(anim, options, callback)

Plays an animation and moves the entity according to the animation's motion data.

**Realm:** 🔴 SERVER

**Parameters:**
- `anim` (string or number) - Sequence name or activity ID
- `options` (number or table) - Playback rate, or table of options:
  - `rate` (number) - Playback speed multiplier
  - `gravity` (boolean) - Apply gravity during animation (default: true)
  - `collisions` (boolean) - Stop on collisions (default: true)
  - `stoponcollide` (boolean) - End animation on collision
  - `gravityoncollide` (boolean) - Enable gravity after collision
  - `multiply` (Vector) - Scale movement per axis
- `callback` (function, optional) - Function called each frame: `callback(self, cycle, collided)`

**Returns:**
- `number` - Animation duration

**Example:**
```lua
-- Jump forward with animation
self:PlayAnimationAndMove("jump_forward", 1.0)

-- Charge attack with collision detection
self:PlayAnimationAndMove(ACT_CHARGE, {
    rate = 1.5,
    gravity = false,
    stoponcollide = true
}, function(self, cycle, collided)
    if collided then
        self:DealDamage(50)
        return true -- Stop animation
    end
end)
```

---

### ENT:PlayAnimationAndMoveAbsolute(anim, options, callback)

Like `PlayAnimationAndMove`, but ignores gravity and collisions by default.

**Realm:** 🔴 SERVER

**Parameters:**
- Same as `PlayAnimationAndMove`

**Example:**
```lua
-- Teleport dash animation
self:PlayAnimationAndMoveAbsolute("dash", 2.0)
```

---

### ENT:PlayClimbAnimation(anim, height, rate, callback)

Plays a climb animation scaled to a specific height.

**Realm:** 🔴 SERVER

**Parameters:**
- `anim` (string, number, or table) - Animation or table of animations (picks closest match)
- `height` (number) - Target height to climb
- `rate` (number, optional) - Playback speed
- `callback` (function, optional) - Called each frame

**Example:**
```lua
-- Climb a ledge
local ledgeHeight = 64
self:PlayClimbAnimation("climb_up", ledgeHeight, 1.0)

-- Auto-select best climb animation for height
self:PlayClimbAnimation({
    "climb_up_32",
    "climb_up_64",
    "climb_up_128"
}, ledgeHeight)
```

---

### ENT:SetAnimation(anim)

Sets the current animation.

**Realm:** 🟣 SHARED

**Parameters:**
- `anim` (string or number) - Animation to set

**Returns:** None

---

### ENT:GetAnimation()

Gets current animation.

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:**
- `number` - Current animation/sequence

---

## Sequences

### ENT:LookupSequence(name)

Looks up sequence ID by name.

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Sequence name

**Returns:**
- `number` - Sequence ID, or -1 if not found

---

### ENT:GetSequence()

Gets current sequence ID.

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:**
- `number` - Current sequence

---

### ENT:SetSequence(seq)

Sets sequence by ID.

**Realm:** 🟣 SHARED

**Parameters:**
- `seq` (number) - Sequence ID

**Returns:** None

---

## Activities

### ENT:SelectWeightedSequence(activity)

Selects a random weighted sequence for an activity.

**Realm:** 🟣 SHARED

**Parameters:**
- `activity` (number) - Activity ID

**Returns:**
- `number` - Selected sequence ID

---

### ENT:GetActivity()

Gets current activity.

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:**
- `number` - Current activity

---

## Animation Events

### ENT:OnAnimEvent(options, event, pos, angles)

Hook called when an animation event occurs.

**Realm:** 🔴 SERVER

**Parameters:**
- `options` (string) - Event data/options from the animation
- `event` (number) - Event ID
- `pos` (Vector) - Entity position
- `angles` (Angle) - Entity angles

**Returns:**
- `boolean` - True if handled (optional)

**Example:**
```lua
function ENT:OnAnimEvent(options, event, pos, angles)
    if event == 1 then
        -- Footstep event
        self:EmitSound("npc/footstep.wav")
        return true
    elseif event == 2 then
        -- Attack event
        self:MeleeAttack()
        return true
    end
end
```

**Common Animation Events:**
- `AE_CL_PLAYSOUND` - Play a sound (options = sound path)
- `1-4` - Footstep events (can be used for custom sounds/effects)
- Custom events defined in model animations

**See Also:**
- `ENT:AddAnimEvent()` - Add custom events to sequences
- `ENT:HandleAnimEvent()` - Internal event handler

---

## Pose Parameters

### ENT:SetPoseParameter(name, value)

Sets a pose parameter.

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Parameter name
- `value` (number) - Parameter value

**Returns:** None

**Example:**
```lua
self:SetPoseParameter("aim_pitch", 45)
```

---

### ENT:GetPoseParameter(name)

Gets pose parameter value.

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Parameter name

**Returns:**
- `number` - Parameter value

---

### ENT:DirectPoseParametersAt(pos, pitch, yaw, center)

Automatically sets pose parameters to aim at a position.

**Realm:** 🟣 SHARED

**Parameters:**
- `pos` (Vector or Entity) - Target position or entity
- `pitch` (string or table) - Pitch parameter name, or table {pitch, yaw}
- `yaw` (string, optional) - Yaw parameter name
- `center` (Vector, optional) - Origin point (default: WorldSpaceCenter)

**Example:**
```lua
-- Aim head at target
self:DirectPoseParametersAt(target, "aim_pitch", "aim_yaw")

-- Simplified syntax
self:DirectPoseParametersAt(target, "aim") -- Uses "aim_pitch" and "aim_yaw"

-- With custom center
self:DirectPoseParametersAt(target:GetPos(), "head_pitch", "head_yaw", self:GetBonePosition(self:LookupBone("head")))
```

---

## Animation Event Helpers

### ENT:SequenceEvent(seq, cycles, callback, ...)

Registers a callback to fire at specific cycle points during a sequence.

**Realm:** 🟣 SHARED

**Parameters:**
- `seq` (string, number, or table) - Sequence name, ID, or table of sequences
- `cycles` (number or table) - Cycle point(s) to trigger (0.0 to 1.0)
- `callback` (function) - Function to call: `callback(self, ...args)`
- `...` - Additional arguments to pass to callback

**Example:**
```lua
-- Fire at halfway point
self:SequenceEvent("attack", 0.5, function(self)
    self:FireWeapon()
end)

-- Fire at multiple points
self:SequenceEvent("combo_attack", {0.3, 0.6, 0.9}, function(self, hitNum)
    self:MeleeAttack()
end)
```

---

### ENT:AddAnimEvent(seq, frames, event)

Adds animation events to a sequence at specific frame numbers.

**Realm:** 🟣 SHARED

**Parameters:**
- `seq` (string, number, or table) - Sequence(s) to add events to
- `frames` (number or table) - Frame number(s) to trigger event
- `event` (number) - Event ID to trigger

**Example:**
```lua
-- Add footstep events
self:AddAnimEvent("walk", {10, 20}, 1) -- Event 1 at frames 10 and 20
```

---

### ENT:ClearSequenceEvents(seq)

Clears registered sequence events.

**Realm:** 🟣 SHARED

**Parameters:**
- `seq` (string, number, table, or nil) - Sequence(s) to clear, or nil for all

**Example:**
```lua
-- Clear specific sequence events
self:ClearSequenceEvents("attack")

-- Clear all events
self:ClearSequenceEvents()
```

---

## Animation Status

### ENT:IsPlayingAnimation()

Checks if an animation is currently playing (blocking the main animation).

**Realm:** 🔴 SERVER

**Returns:**
- `boolean` - True if playing a blocking animation

---

### ENT:IsPlayingSequence(seq)

Checks if a specific sequence is playing (either main or as gesture).

**Realm:** 🔴 SERVER

**Parameters:**
- `seq` (string or number) - Sequence name or ID

**Returns:**
- `boolean` - True if sequence is playing

---

### ENT:IsPlayingActivity(act)

Checks if a specific activity is playing.

**Realm:** 🔴 SERVER

**Parameters:**
- `act` (number) - Activity ID

**Returns:**
- `boolean` - True if activity is playing

**Example:**
```lua
if self:IsPlayingActivity(ACT_RELOAD) then
    -- Can't attack while reloading
    return
end
```

---

## Gestures & Layers

Gestures are animations played on separate layers that blend with the main animation.

### ENT:AddGesture(activity)

Adds a gesture animation.

**Realm:** 🟣 SHARED

**Parameters:**
- `activity` (number) - Gesture activity

**Returns:**
- `number` - Gesture ID

---

### ENT:RemoveGesture(gesture)

Removes a gesture.

**Realm:** 🟣 SHARED

**Parameters:**
- `gesture` (number) - Gesture ID

**Returns:** None

---

## Animation Speed

### ENT:SetPlaybackRate(rate)

Sets animation playback speed.

**Realm:** 🟣 SHARED

**Parameters:**
- `rate` (number) - Playback multiplier (1.0 = normal)

**Returns:** None

---

### ENT:GetPlaybackRate()

Gets playback rate.

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:**
- `number` - Current playback rate

---

## Animation Update System

### ENT:UpdateAnimation()

Updates the entity's animation based on movement state. Called automatically each think.

**Realm:** 🔴 SERVER

**Notes:**
- Calls `OnUpdateAnimation()` to determine which animation to play
- Automatically manages animation transitions
- Respects `IsPlayingAnimation()` - won't change animation if one is actively playing
- Handles playback rate adjustments for movement animations

---

### ENT:OnUpdateAnimation()

Hook to determine which animation should play based on entity state.

**Realm:** 🔴 SERVER

**Returns:**
- `anim` (string or number) - Animation to play (sequence name or activity ID)
- `rate` (number, optional) - Playback rate

**Example:**
```lua
function ENT:OnUpdateAnimation()
    if self:IsDown() or self:IsDead() then
        return -- No animation
    end

    -- Custom animation logic
    if self:IsAttacking() then
        return ACT_MELEE_ATTACK1, 1.5
    elseif self:IsClimbingUp() then
        return self.ClimbUpAnimation, self.ClimbAnimRate
    elseif self:IsRunning() then
        return self.RunAnimation, self.RunAnimRate
    elseif self:IsMoving() then
        return self.WalkAnimation, self.WalkAnimRate
    else
        return self.IdleAnimation, self.IdleAnimRate
    end
end
```

---

### ENT:OnAnimChange(oldSeq, newSeq)

Hook called before an animation changes.

**Realm:** 🔴 SERVER

**Parameters:**
- `oldSeq` (string) - Old sequence name
- `newSeq` (string) - New sequence name

**Returns:**
- `boolean` - Return `false` to prevent the animation change

**Example:**
```lua
function ENT:OnAnimChange(oldSeq, newSeq)
    -- Prevent interrupting attack animations
    if oldSeq == "attack_heavy" then
        return false
    end
end
```

---

### ENT:BodyUpdate()

Called to update body animations. Override for custom body animation logic.

**Realm:** 🔴 SERVER

**Default Implementation:**
```lua
function ENT:BodyUpdate()
    self:BodyMoveXY()
end
```

---

## Helper Functions

### ENT:SelectRandomSequence(activity)

Selects a random weighted sequence for an activity.

**Realm:** 🟣 SHARED

**Parameters:**
- `activity` (number) - Activity ID

**Returns:**
- `number` - Selected sequence ID

---

### ENT:GetActivityIDFromName(name)

Gets an activity ID from its name string.

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Activity name (e.g., "ACT_WALK")

**Returns:**
- `number` - Activity ID, or `ACT_INVALID` if not found

---

### ENT:GetAnimInfoSequence(seq)

Gets animation info for a sequence.

**Realm:** 🟣 SHARED

**Parameters:**
- `seq` (string or number) - Sequence name or ID

**Returns:**
- `table` - Animation info table (includes `numframes`, etc.)

---

## Animation Blending

Animation blending occurs automatically through the gesture/layer system. The engine blends:

- Main animation (base layer)
- Gesture animations (additional layers via `PlayAnimation()`)
- Pose parameters (for aiming, looking, etc.)

**Automatic Blending:**
- Movement animations automatically adjust playback rate to match velocity
- Turning animations blend with walking animations
- Pose parameters for `move_x`, `move_y`, and `move_yaw` are set automatically

**Example:**
```lua
-- Play idle animation on base layer
self:ResetSequence(self:LookupSequence("idle"))

-- Add aiming gesture on top
self:PlayAnimation(ACT_GESTURE_RANGE_ATTACK1)

-- Adjust aim with pose parameters
self:DirectPoseParametersAt(target, "aim")
```

---

## Sprite Animations

For sprite-based nextbots (`drgbase_nextbot_sprite`), the animation system works differently:

- Uses frame-based sprite sheets instead of 3D model sequences
- Configure sprite animations in the entity's `SpriteData` table
- Animations are defined as frame ranges in the sprite sheet

**Note:** Sprite nextbots use `IsDrGNextbotSprite = true` and skip 3D animation functions like `BodyMoveXY()`.

**See Also:**
- Sprite Nextbot Guide (coming soon)
- `lua/entities/drgbase_nextbot_sprite/` for implementation details

---

## Related Hooks

- `ENT:OnAnimationStart(anim)` - Called when animation starts
- `ENT:OnAnimationComplete(anim)` - Called when animation completes
- `ENT:OnAnimEvent(event, options)` - Called for animation events

---

## See Also

- [Base Configuration](./base-config.md) - Animation properties
- [Animation System Guide](../../systems/animation/README.md)
- [Animation Events Reference](../../reference/anim-events.md)
