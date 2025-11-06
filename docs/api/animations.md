# Animation System API Reference

The animation system provides comprehensive control over entity animations, sequences, activities, and movement-synced animations.

**Source Files:**
- `lua/entities/drgbase_nextbot/animations.lua`

---

## Configuration

### ConVars

#### `drgbase_debug_animations`
```lua
CreateConVar("drgbase_debug_animations", "0", {FCVAR_ARCHIVE, FCVAR_NOTIFY, FCVAR_REPLICATED})
```
Enable animation debugging mode. When enabled, prevents automatic animation updates for NPCs with disabled AI.

**Realm:** SHARED
**Type:** Boolean (0/1)
**Default:** 0

---

## Animation Information

### ENT:GetAnimInfoSequence
```lua
table ENT:GetAnimInfoSequence(number|string seq)
```
Retrieves detailed animation information for a given sequence.

**Realm:** SHARED
**Parameters:**
- `seq` (number|string) - Sequence index or name
**Returns:** (table) Animation information table, or empty table if invalid
**Example:**
```lua
local animInfo = self:GetAnimInfoSequence("walk_all")
if animInfo.numframes then
    print("Animation has " .. animInfo.numframes .. " frames")
end
```

---

### ENT:GetActivityIDFromName
```lua
number ENT:GetActivityIDFromName(string name)
```
Converts an activity name to its numeric ID. Results are cached for performance.

**Realm:** SHARED
**Parameters:**
- `name` (string) - Activity name (e.g., "ACT_WALK")
**Returns:** (number) Activity ID, or ACT_INVALID if not found
**Example:**
```lua
local walkID = self:GetActivityIDFromName("ACT_WALK")
if walkID ~= ACT_INVALID then
    self:StartActivity(walkID)
end
```

---

## Sequence Selection

### ENT:SelectRandomSequence
```lua
number ENT:SelectRandomSequence(number activity)
```
Selects a random sequence for the given activity using a random seed.

**Realm:** SHARED
**Parameters:**
- `activity` (number) - Activity ID
**Returns:** (number) Selected sequence index
**Example:**
```lua
local walkSeq = self:SelectRandomSequence(ACT_WALK)
self:ResetSequence(walkSeq)
```

---

## Animation Events

### ENT:SequenceEvent
```lua
ENT:SequenceEvent(number|string|table seq, number|table cycles, function callback, ...)
```
Registers a callback to execute at specific cycle points during a sequence.

**Realm:** SHARED
**Parameters:**
- `seq` (number|string|table) - Sequence(s) to attach event to
- `cycles` (number|table) - Cycle point(s) to trigger at (0-1)
- `callback` (function) - Function to call: `function(ENT self, ...)`
- `...` - Additional arguments passed to callback
**Example:**
```lua
-- Fire a sound at 50% through the animation
self:SequenceEvent("attack", 0.5, function(self)
    self:EmitSound("npc.attack")
end)

-- Multiple events
self:SequenceEvent("walk", {0.25, 0.75}, function(self)
    self:EmitSound("npc.footstep")
end)
```

---

### ENT:ClearSequenceEvents
```lua
ENT:ClearSequenceEvents([number|string|table seq])
```
Clears registered sequence events.

**Realm:** SHARED
**Parameters:**
- `seq` (number|string|table|nil) - Sequence(s) to clear, or nil to clear all
**Example:**
```lua
-- Clear specific sequence
self:ClearSequenceEvents("attack")

-- Clear all events
self:ClearSequenceEvents()
```

---

### ENT:AddAnimEvent
```lua
ENT:AddAnimEvent(number|string|table seq, number|table frames, any event)
```
Adds animation events at specific frame numbers. Automatically converts frames to cycles.

**Realm:** SHARED
**Parameters:**
- `seq` (number|string|table) - Sequence(s) to attach event to
- `frames` (number|table) - Frame number(s) to trigger at
- `event` - Event data to pass to `OnAnimEvent`
**Example:**
```lua
-- Fire event at frame 10
self:AddAnimEvent("attack", 10, "ATTACK_HIT")

-- Multiple frames
self:AddAnimEvent("walk", {5, 15}, "FOOTSTEP")
```

---

### ENT:OnAnimEvent
```lua
ENT:OnAnimEvent(any options, number event, Vector pos, Angle angle)
```
**Hook** called when an animation event fires.

**Realm:** SHARED
**Parameters:**
- `options` - Event data from AddAnimEvent or model event
- `event` (number) - Event type ID
- `pos` (Vector) - Entity position
- `angle` (Angle) - Entity angles
**Example:**
```lua
function ENT:OnAnimEvent(options, event, pos, angle)
    if options == "ATTACK_HIT" then
        self:MeleeAttack()
    elseif options == "FOOTSTEP" then
        self:EmitSound("npc.footstep")
    end
end
```

---

## Pose Parameters

### ENT:DirectPoseParametersAt
```lua
ENT:DirectPoseParametersAt(Vector|Entity pos, string pitch, string yaw, [Vector center])
ENT:DirectPoseParametersAt(Vector|Entity pos, string baseName, [Vector center])
```
Automatically sets pitch and yaw pose parameters to aim at a position.

**Realm:** SHARED
**Parameters:**
- `pos` (Vector|Entity) - Target position or entity
- `pitch` (string) - Pitch pose parameter name, or base name (auto-appends "_pitch"/"_yaw")
- `yaw` (string) - Yaw pose parameter name (optional with base name)
- `center` (Vector) - Center point for angle calculation (default: WorldSpaceCenter)
**Example:**
```lua
-- Using base name
self:DirectPoseParametersAt(enemy, "aim") -- Uses "aim_pitch" and "aim_yaw"

-- Using explicit names
self:DirectPoseParametersAt(enemy, "aim_pitch", "aim_yaw")

-- With custom center
self:DirectPoseParametersAt(enemy, "aim", self:GetAttachment(1).Pos)
```

---

## Animation Playback (SERVER)

### ENT:IsPlayingAnimation
```lua
boolean ENT:IsPlayingAnimation()
```
Checks if currently playing a blocking animation (PlaySequenceAndWait).

**Realm:** SERVER
**Returns:** (boolean) True if playing blocking animation
**Example:**
```lua
if not self:IsPlayingAnimation() then
    self:PlayActivityAndWait(ACT_MELEE_ATTACK1)
end
```

---

### ENT:IsPlayingSequence
```lua
boolean ENT:IsPlayingSequence(number|string seq)
```
Checks if a specific sequence is currently playing (blocking or gesture).

**Realm:** SERVER
**Parameters:**
- `seq` (number|string) - Sequence index or name
**Returns:** (boolean) True if sequence is playing
**Example:**
```lua
if self:IsPlayingSequence("attack") then
    -- Attack is active
end
```

---

### ENT:IsPlayingActivity
```lua
boolean ENT:IsPlayingActivity(number activity)
```
Checks if a specific activity is currently playing as a blocking animation.

**Realm:** SERVER
**Parameters:**
- `activity` (number) - Activity ID
**Returns:** (boolean) True if activity is playing
**Example:**
```lua
if self:IsPlayingActivity(ACT_MELEE_ATTACK1) then
    -- Melee attack is active
end
```

---

## Blocking Animations (SERVER)

Blocking animations pause coroutine execution until complete.

### ENT:PlaySequenceAndWait
```lua
number ENT:PlaySequenceAndWait(number|string seq, [number rate], [function callback])
```
Plays a sequence and blocks until complete.

**Realm:** SERVER
**Parameters:**
- `seq` (number|string) - Sequence index or name
- `rate` (number) - Playback rate (default: 1)
- `callback` (function) - Called each frame: `function(ENT self, number cycle)`. Return true to stop early.
**Returns:** (number) Duration played in seconds
**Example:**
```lua
-- Basic usage
self:PlaySequenceAndWait("attack")

-- With custom rate
self:PlaySequenceAndWait("attack", 2) -- 2x speed

-- With callback
self:PlaySequenceAndWait("attack", 1, function(self, cycle)
    if cycle > 0.5 and not self.attackHit then
        self.attackHit = true
        self:MeleeAttack()
    end
    if not IsValid(self:GetEnemy()) then
        return true -- Stop early
    end
end)
```

---

### ENT:PlayActivityAndWait
```lua
number ENT:PlayActivityAndWait(number activity, [number rate], [function callback])
```
Plays an activity and blocks until complete. Automatically selects a random sequence.

**Realm:** SERVER
**Parameters:**
- `activity` (number) - Activity ID
- `rate` (number) - Playback rate (default: 1)
- `callback` (function) - Called each frame
**Returns:** (number) Duration played in seconds
**Example:**
```lua
self:PlayActivityAndWait(ACT_MELEE_ATTACK1)
```

---

### ENT:PlayAnimationAndWait
```lua
number ENT:PlayAnimationAndWait(number|string anim, [number rate], [function callback])
```
Plays either a sequence (string) or activity (number) and blocks until complete.

**Realm:** SERVER
**Parameters:**
- `anim` (number|string) - Activity ID or sequence name
- `rate` (number) - Playback rate (default: 1)
- `callback` (function) - Called each frame
**Returns:** (number) Duration played in seconds
**Example:**
```lua
-- Play by sequence name
self:PlayAnimationAndWait("attack")

-- Play by activity ID
self:PlayAnimationAndWait(ACT_MELEE_ATTACK1)
```

---

## Movement Animations (SERVER)

Movement animations automatically sync position/rotation with animation movement data.

### ENT:PlaySequenceAndMove
```lua
number ENT:PlaySequenceAndMove(number|string seq, [table|number options], [function callback])
```
Plays a sequence with synchronized movement from animation data.

**Realm:** SERVER
**Parameters:**
- `seq` (number|string) - Sequence index or name
- `options` (table|number) - Options table or playback rate:
  - `rate` (number) - Playback rate (default: 1)
  - `gravity` (boolean) - Apply gravity (default: true)
  - `collisions` (boolean) - Stop on collision (default: true)
  - `stoponcollide` (boolean) - End animation on collision (default: false)
  - `gravityoncollide` (boolean) - Enable gravity after collision (default: false)
  - `multiply` (Vector) - Scale movement per axis (default: Vector(1,1,1))
- `callback` (function) - Called each frame: `function(ENT self, number cycle, boolean collided)`
**Returns:** (number) Duration played in seconds
**Example:**
```lua
-- Basic movement
self:PlaySequenceAndMove("walk_to_door")

-- With options
self:PlaySequenceAndMove("jump_over", {
    rate = 1.5,
    gravity = false,
    collisions = true,
    stoponcollide = true
})

-- Scale movement
self:PlaySequenceAndMove("climb", {
    multiply = Vector(1, 1, 2) -- Double Z movement
})
```

---

### ENT:PlayActivityAndMove
```lua
number ENT:PlayActivityAndMove(number activity, [table|number options], [function callback])
```
Plays an activity with synchronized movement.

**Realm:** SERVER
**Parameters:**
- Same as PlaySequenceAndMove
**Returns:** (number) Duration played in seconds

---

### ENT:PlayAnimationAndMove
```lua
number ENT:PlayAnimationAndMove(number|string anim, [table|number options], [function callback])
```
Plays either a sequence (string) or activity (number) with synchronized movement.

**Realm:** SERVER
**Parameters:**
- Same as PlaySequenceAndMove
**Returns:** (number) Duration played in seconds

---

### ENT:PlaySequenceAndMoveAbsolute
```lua
number ENT:PlaySequenceAndMoveAbsolute(number|string seq, [table|number options], [function callback])
```
Plays a sequence with absolute movement (no gravity, no collisions).

**Realm:** SERVER
**Parameters:**
- `seq` (number|string) - Sequence index or name
- `options` (table|number) - Options table or playback rate
- `callback` (function) - Called each frame
**Returns:** (number) Duration played in seconds
**Example:**
```lua
-- Perfect animation movement
self:PlaySequenceAndMoveAbsolute("climb_ladder")
```

---

### ENT:PlayActivityAndMoveAbsolute
```lua
number ENT:PlayActivityAndMoveAbsolute(number activity, [table|number options], [function callback])
```
Plays an activity with absolute movement.

**Realm:** SERVER

---

### ENT:PlayAnimationAndMoveAbsolute
```lua
number ENT:PlayAnimationAndMoveAbsolute(number|string anim, [table|number options], [function callback])
```
Plays either a sequence (string) or activity (number) with absolute movement.

**Realm:** SERVER

---

## Gesture Animations (SERVER)

Gestures play as animation layers without blocking.

### ENT:PlaySequence
```lua
number ENT:PlaySequence(number|string seq, [number rate], [function callback])
```
Plays a sequence as a non-blocking gesture layer.

**Realm:** SERVER
**Parameters:**
- `seq` (number|string) - Sequence index or name
- `rate` (number) - Playback rate (default: 1)
- `callback` (function) - Called each frame: `function(ENT self, number cycle, number layerID)`. Return true to stop early.
**Returns:** (number) Duration in seconds, or 0 if failed
**Example:**
```lua
-- Play gesture
self:PlaySequence("gesture_signal")

-- With callback
self:PlaySequence("gesture_signal", 1, function(self, cycle, layerID)
    if cycle > 0.5 then
        self:EmitSound("radio.beep")
        return true -- Stop early
    end
end)
```

---

### ENT:PlayActivity
```lua
number ENT:PlayActivity(number activity, [number rate], [function callback])
```
Plays an activity as a non-blocking gesture layer.

**Realm:** SERVER
**Parameters:**
- Same as PlaySequence
**Returns:** (number) Duration in seconds

---

### ENT:PlayAnimation
```lua
number ENT:PlayAnimation(number|string anim, [number rate], [function callback])
```
Plays either a sequence (string) or activity (number) as a gesture.

**Realm:** SERVER
**Parameters:**
- Same as PlaySequence
**Returns:** (number) Duration in seconds

---

## Climbing Animations (SERVER)

Specialized movement animations for climbing.

### ENT:PlayClimbSequence
```lua
number ENT:PlayClimbSequence(number|string|table seq, number height, [number rate], [function callback])
```
Plays a climb sequence scaled to a specific height.

**Realm:** SERVER
**Parameters:**
- `seq` (number|string|table) - Sequence(s) to use. If table, selects closest match to height.
- `height` (number) - Target height in units
- `rate` (number) - Playback rate (default: 1)
- `callback` (function) - Called each frame
**Returns:** (number) Duration played in seconds
**Example:**
```lua
-- Climb 64 units
self:PlayClimbSequence("climb_up", 64)

-- Auto-select from multiple sequences
self:PlayClimbSequence({
    "climb_up_36",
    "climb_up_64",
    "climb_up_128"
}, 72) -- Selects closest match
```

---

### ENT:PlayClimbActivity
```lua
number ENT:PlayClimbActivity(number activity, number height, [number rate], [function callback])
```
Plays a climb activity scaled to a specific height.

**Realm:** SERVER

---

### ENT:PlayClimbAnimation
```lua
number ENT:PlayClimbAnimation(number|string anim, number height, [number rate], [function callback])
```
Plays either a climb sequence (string) or activity (number).

**Realm:** SERVER

---

## Animation System (SERVER)

### ENT:UpdateAnimation
```lua
ENT:UpdateAnimation()
```
Updates the current animation based on movement state. Called automatically by BodyUpdate.

**Realm:** SERVER
**Note:** Only functions when not playing a blocking animation.
**Example:**
```lua
-- Manually trigger animation update
self:UpdateAnimation()
```

---

### ENT:OnUpdateAnimation
```lua
number|string, [number] ENT:OnUpdateAnimation()
```
**Hook** to determine which animation should play based on current state.

**Realm:** SERVER
**Returns:**
- (number|string) - Activity ID or sequence name
- (number) - Optional playback rate
**Example:**
```lua
function ENT:OnUpdateAnimation()
    if self:IsDown() then return ACT_DIEBACKWARD end
    if not self:IsOnGround() then return ACT_GLIDE end
    if self:IsRunning() then return ACT_RUN, 1.2 end
    if self:IsMoving() then return ACT_WALK end
    return ACT_IDLE
end
```

**Default Behavior:**
```lua
function ENT:OnUpdateAnimation()
    if self:IsDown() or self:IsDead() then return end
    if self:IsClimbingUp() then return self.ClimbUpAnimation, self.ClimbAnimRate
    elseif self:IsClimbingDown() then return self.ClimbDownAnimation, self.ClimbAnimRate
    elseif not self:IsOnGround() then return self.JumpAnimation, self.JumpAnimRate
    elseif self:IsRunning() then return self.RunAnimation, self.RunAnimRate
    elseif self:IsMoving() then return self.WalkAnimation, self.WalkAnimRate
    else return self.IdleAnimation, self.IdleAnimRate end
end
```

---

### ENT:OnAnimChange
```lua
boolean ENT:OnAnimChange(string oldSeq, string newSeq)
```
**Hook** called before animation changes. Return false to prevent the change.

**Realm:** SERVER
**Parameters:**
- `oldSeq` (string) - Old sequence name
- `newSeq` (string) - New sequence name
**Returns:** (boolean) Return false to prevent animation change
**Example:**
```lua
function ENT:OnAnimChange(oldSeq, newSeq)
    if self:IsAttacking() and newSeq ~= "attack" then
        return false -- Don't interrupt attack
    end
end
```

---

### ENT:OnAnimChanged
```lua
ENT:OnAnimChanged(string oldSeq, string newSeq, number delay)
```
**Hook** called after animation changes (runs in coroutine).

**Realm:** SERVER
**Parameters:**
- `oldSeq` (string) - Old sequence name
- `newSeq` (string) - New sequence name
- `delay` (number) - Delay before callback fired
**Example:**
```lua
function ENT:OnAnimChanged(oldSeq, newSeq, delay)
    if newSeq == "attack" then
        self:EmitSound("npc.attack_start")
    end
end
```

---

### ENT:BodyUpdate
```lua
ENT:BodyUpdate()
```
**Hook** called each think to update body/animation. Default calls BodyMoveXY.

**Realm:** SERVER
**Example:**
```lua
function ENT:BodyUpdate()
    -- Custom body update logic
    self:BodyMoveXY()
end
```

---

### ENT:HandleAnimEvent
```lua
ENT:HandleAnimEvent(number event, _, _, _, any options)
```
**Hook** called for model-defined animation events. Forwards to OnAnimEvent.

**Realm:** SERVER
**Parameters:**
- `event` (number) - Event type ID
- `options` - Event data from model

---

## Related Configuration Properties

These properties affect animation behavior (defined in `shared.lua`):

```lua
ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.JumpAnimation = ACT_JUMP
ENT.ClimbUpAnimation = "climb_up"
ENT.ClimbDownAnimation = "climb_down"

ENT.IdleAnimRate = 1
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1
ENT.JumpAnimRate = 1
ENT.ClimbAnimRate = 1
```

---

## Advanced: Meta Extensions

The animation system extends the NextBot metatable:

### NextBot:BodyMoveXY
Overridden to automatically handle:
- Pose parameter updates (`move_x`, `move_y`, `move_yaw`)
- Playback rate syncing with movement speed
- Turn animation speed matching

### NextBot:GetActivity
Returns current sequence activity for DrGBase nextbots.

### NextBot:StartActivity
Starts an activity (selects random sequence and resets).

---

## See Also

- [Status System](status.md) - Health and state management
- [Movement System](../guides/movement.md) - Movement and navigation
- [AI System](../guides/ai.md) - AI behaviors that use animations
