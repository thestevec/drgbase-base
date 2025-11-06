# Movement & Locomotion Functions

Movement control, pathfinding, and locomotion.

**Files:**
- `lua/entities/drgbase_nextbot/movements.lua` (702 lines)
- `lua/entities/drgbase_nextbot/locomotion.lua` (106 lines)

---

## Speed Functions

### ENT:GetSpeed()

Gets the configured speed value (independent of scale).

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `number` - The speed value

---

### ENT:SetSpeed(speed)

Sets the desired movement speed.

**Realm:** 🔴 SERVER

**Parameters:**
- `speed` (number) - Speed in units/second (automatically scaled by entity scale)

**Returns:** None

**Example:**
```lua
function ENT:CustomInitialize()
    self.WalkSpeed = 100
    self.RunSpeed = 300
end
```

---

### ENT:Speed(scale)

Gets the current velocity magnitude.

**Realm:** 🌐 SHARED

**Parameters:**
- `scale` (boolean, optional) - If true, divides speed by entity scale

**Returns:**
- `number` - Current speed in units/second

---

### ENT:SpeedSqr(scale)

Gets the squared velocity magnitude (more efficient for comparisons).

**Realm:** 🌐 SHARED

**Parameters:**
- `scale` (boolean, optional) - If true, accounts for entity scale

**Returns:**
- `number` - Squared speed value

---

### ENT:IsSpeedMore(speed, scale)

Checks if current speed is greater than a value.

**Realm:** 🌐 SHARED

**Parameters:**
- `speed` (number) - Speed to compare against
- `scale` (boolean, optional) - Account for entity scale

**Returns:**
- `boolean` - True if current speed is greater

---

### ENT:IsSpeedLess(speed, scale)

Checks if current speed is less than a value.

**Realm:** 🌐 SHARED

**Parameters:**
- `speed` (number) - Speed to compare against
- `scale` (boolean, optional) - Account for entity scale

**Returns:**
- `boolean` - True if current speed is less

---

### ENT:IsSpeedEqual(speed, scale)

Checks if current speed equals a value.

**Realm:** 🌐 SHARED

**Parameters:**
- `speed` (number) - Speed to compare against
- `scale` (boolean, optional) - Account for entity scale

**Returns:**
- `boolean` - True if speeds are equal

---

### ENT:IsSpeedMoreEqual(speed, scale)

Checks if current speed is greater than or equal to a value.

**Realm:** 🌐 SHARED

**Parameters:**
- `speed` (number) - Speed to compare against
- `scale` (boolean, optional) - Account for entity scale

**Returns:**
- `boolean` - True if current speed >= threshold

---

### ENT:IsSpeedLessEqual(speed, scale)

Checks if current speed is less than or equal to a value.

**Realm:** 🌐 SHARED

**Parameters:**
- `speed` (number) - Speed to compare against
- `scale` (boolean, optional) - Account for entity scale

**Returns:**
- `boolean` - True if current speed <= threshold

---

## Movement State Functions

### ENT:IsMoving()

Checks if the NPC is currently moving.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if velocity is not zero

---

### ENT:GetMovement(ignoreZ)

Gets the movement direction relative to the entity's angles.

**Realm:** 🌐 SHARED

**Parameters:**
- `ignoreZ` (boolean, optional) - If true, ignores vertical movement

**Returns:**
- `Vector` - Movement direction vector, or zero vector if not moving

---

### ENT:IsRunning()

Checks if the NPC is running (moving with speed boost).

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `boolean` - True if running (checks possessor IN_SPEED key or ShouldRun hook)

---

## Directional Movement Checks

### ENT:IsMovingUp()

Checks if moving upward (Z axis).

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving up

---

### ENT:IsMovingDown()

Checks if moving downward (Z axis).

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving down

---

### ENT:IsMovingForward()

Checks if moving forward relative to entity angles.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving forward

---

### ENT:IsMovingBackward()

Checks if moving backward relative to entity angles.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving backward

---

### ENT:IsMovingRight()

Checks if moving right relative to entity angles.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving right

---

### ENT:IsMovingLeft()

Checks if moving left relative to entity angles.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving left

---

### ENT:IsMovingForwardLeft()

Checks if moving forward and left simultaneously.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving forward-left

---

### ENT:IsMovingForwardRight()

Checks if moving forward and right simultaneously.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving forward-right

---

### ENT:IsMovingBackwardLeft()

Checks if moving backward and left simultaneously.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving backward-left

---

### ENT:IsMovingBackwardRight()

Checks if moving backward and right simultaneously.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if moving backward-right

---

## Turning & Rotation Functions

### ENT:IsTurning(prec)

Checks if the entity is currently turning.

**Realm:** 🌐 SHARED

**Parameters:**
- `prec` (number, optional) - Precision for angle comparison (decimal places)

**Returns:**
- `boolean` - True if yaw angle has changed since last frame

---

### ENT:IsTurningLeft(prec)

Checks if turning to the left.

**Realm:** 🌐 SHARED

**Parameters:**
- `prec` (number, optional) - Precision for angle comparison

**Returns:**
- `boolean` - True if turning left

---

### ENT:IsTurningRight(prec)

Checks if turning to the right.

**Realm:** 🌐 SHARED

**Parameters:**
- `prec` (number, optional) - Precision for angle comparison

**Returns:**
- `boolean` - True if turning right

---

### ENT:FaceTowards(pos)

Smoothly rotates to face a position over time.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector or Entity) - Position or entity to face

**Returns:** None

**Example:**
```lua
function ENT:OnIdle()
    if self:HasEnemy() then
        self:FaceTowards(self:GetEnemy())
    end
end
```

---

### ENT:FaceInstant(pos)

Instantly rotates to face a position (no smoothing).

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector or Entity) - Position or entity to face

**Returns:** None

**Example:**
```lua
function ENT:OnSpawn()
    self:FaceInstant(targetPos)
end
```

---

### ENT:FaceTo(toface)

Coroutine function that smoothly rotates until facing target.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `toface` (Vector or Entity) - Position or entity to face

**Returns:** None (yields until facing complete)

**Example:**
```lua
function ENT:OnIdle()
    self:FaceTo(self:GetEnemy())
    -- Code after this runs once facing is complete
    self:PlayAnimation("attack")
end
```

---

### ENT:FaceEnemy()

Faces the current enemy if one exists.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

---

### ENT:TurnRight(angle, callback)

Turns right by a specific angle.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `angle` (number, optional) - Degrees to turn. If omitted, turns continuously
- `callback` (function, optional) - Called each frame: `callback(self, turnedSoFar)`. Return true to stop

**Returns:** None (yields if angle specified)

**Example:**
```lua
function ENT:OnIdle()
    self:TurnRight(90) -- Turn 90 degrees right
    -- Continues after turn complete
end
```

---

### ENT:TurnLeft(angle, callback)

Turns left by a specific angle.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `angle` (number, optional) - Degrees to turn. If omitted, turns continuously
- `callback` (function, optional) - Called each frame: `callback(self, turnedSoFar)`. Return true to stop

**Returns:** None (yields if angle specified)

**Example:**
```lua
function ENT:OnIdle()
    self:TurnLeft(180, function(self, turned)
        print("Turned " .. turned .. " degrees")
    end)
end
```

---

## Basic Movement Functions

### ENT:Approach(pos, nb)

Moves toward a position without pathfinding.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector or Entity) - Target position or entity
- `nb` (number, optional) - Movement weight (default 1)

**Returns:** None

**Note:** This is a low-level function. Use `MoveTowards` or `FollowPath` for most cases.

---

### ENT:MoveTowards(pos)

Faces and moves toward a position.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector or Entity) - Target position or entity

**Returns:** None

**Example:**
```lua
function ENT:OnIdle()
    self:MoveTowards(targetPos)
end
```

---

### ENT:MoveAwayFrom(pos, face)

Moves away from a position.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector or Entity) - Position to move away from
- `face` (boolean, optional) - If true, faces the position while moving backward

**Returns:** None

**Example:**
```lua
function ENT:OnDamaged(dmg)
    if dmg:GetDamage() > 50 then
        self:MoveAwayFrom(dmg:GetAttacker(), true)
    end
end
```

---

### ENT:MoveForward(dist, callback)

Moves forward for a distance.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `dist` (number, optional) - Distance to move. If omitted, moves continuously
- `callback` (function, optional) - Called each frame: `callback(self, distanceMoved)`. Return true to stop

**Returns:** None (yields if distance specified)

**Example:**
```lua
function ENT:OnIdle()
    self:MoveForward(100) -- Move 100 units forward
    self:PlayAnimation("idle")
end
```

---

### ENT:MoveBackward(dist, callback)

Moves backward for a distance.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `dist` (number, optional) - Distance to move. If omitted, moves continuously
- `callback` (function, optional) - Called each frame: `callback(self, distanceMoved)`. Return true to stop

**Returns:** None (yields if distance specified)

---

### ENT:MoveRight(dist, callback)

Moves right (strafe) for a distance.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `dist` (number, optional) - Distance to move. If omitted, moves continuously
- `callback` (function, optional) - Called each frame: `callback(self, distanceMoved)`. Return true to stop

**Returns:** None (yields if distance specified)

**Example:**
```lua
function ENT:OnIdle()
    self:MoveRight(50) -- Strafe 50 units right
end
```

---

### ENT:MoveLeft(dist, callback)

Moves left (strafe) for a distance.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `dist` (number, optional) - Distance to move. If omitted, moves continuously
- `callback` (function, optional) - Called each frame: `callback(self, distanceMoved)`. Return true to stop

**Returns:** None (yields if distance specified)

---

## Pathfinding & Navigation

### ENT:FollowPath(pos, tolerance, generator)

Performs one step of pathfinding toward a target. Called repeatedly to follow a path.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `pos` (Vector or Entity) - Target position or entity
- `tolerance` (number, optional) - Goal tolerance in units (default 20)
- `generator` (function, optional) - Custom path generator function

**Returns:**
- `string` - Status code:
  - `"reached"` - Destination reached
  - `"unreachable"` - Cannot path to destination
  - `"moving"` - Currently moving toward goal
  - `"stuck"` - Entity is stuck
  - `"obstacle"` - Avoiding obstacle
  - `"ledge"` - Climbing ledge
  - `"ladder_up"` - Climbing ladder up
  - `"ladder_down"` - Climbing ladder down
- `any` - Additional data (e.g., ladder entity)

**Example:**
```lua
function ENT:OnIdle()
    while true do
        local status = self:FollowPath(targetPos, 50)
        if status == "reached" then
            print("Reached destination!")
            break
        elseif status == "unreachable" then
            print("Can't get there!")
            break
        end
        self:YieldCoroutine(true)
    end
end
```

---

### ENT:GoTo(pos, tolerance, callback)

Moves to a position using pathfinding. Coroutine wrapper around FollowPath.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `pos` (Vector or Entity) - Target position or entity
- `tolerance` (number, optional) - Goal tolerance in units (default 20)
- `callback` (function, optional) - Called each frame: `callback(self, path)`. Can return boolean to override result

**Returns:**
- `boolean` - True if destination reached, false if unreachable

**Example:**
```lua
function ENT:OnIdle()
    if self:GoTo(waypoint) then
        print("Arrived at waypoint!")
    else
        print("Couldn't reach waypoint")
    end
end
```

---

### ENT:ChaseEntity(ent, tolerance, callback)

Chases an entity using pathfinding. Automatically updates path as entity moves.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `ent` (Entity) - Entity to chase
- `tolerance` (number, optional) - Distance to get within (default 20)
- `callback` (function, optional) - Called each frame: `callback(self, path)`. Can return boolean to override result

**Returns:**
- `boolean` - True if caught entity, false if unreachable or entity became invalid

**Example:**
```lua
function ENT:OnChaseEnemy()
    self:ChaseEntity(self:GetEnemy(), 100, function(self, path)
        if not self:HasEnemy() then
            return false -- Stop chasing
        end
    end)
end
```

---

## Obstacle Avoidance

### ENT:AvoidObstacles(forwardOnly)

Automatically avoids nearby obstacles by adjusting movement direction.

**Realm:** 🔴 SERVER

**Parameters:**
- `forwardOnly` (boolean, optional) - Only check obstacles in front

**Returns:**
- `boolean` - True if avoiding an obstacle
- `string` - Direction being avoided (e.g., "N", "NE", "SE")

**Note:** Controlled by `drgbase_avoid_obstacles` convar. Automatically called by FollowPath.

**Example:**
```lua
function ENT:OnIdle()
    local avoiding, direction = self:AvoidObstacles(true)
    if avoiding then
        print("Avoiding obstacle to the " .. direction)
    end
end
```

---

## Climbing System

### ENT:IsClimbing()

Checks if currently climbing (ledge or ladder).

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if climbing

---

### ENT:IsClimbingUp()

Checks if climbing upward.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if climbing up

---

### ENT:IsClimbingDown()

Checks if climbing downward.

**Realm:** 🌐 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if climbing down

---

### ENT:IsClimbingLadder(ladder)

Checks if climbing a ladder.

**Realm:** 🔴 SERVER

**Parameters:**
- `ladder` (Entity, optional) - Specific ladder to check. If omitted, checks any ladder

**Returns:**
- `boolean` - True if climbing (the specified) ladder
- `Entity` - The ladder being climbed (if omitted parameter)

**Example:**
```lua
local climbing, ladder = self:IsClimbingLadder()
if climbing then
    print("Climbing ladder: " .. tostring(ladder))
end
```

---

### ENT:IsClimbingLedge()

Checks if climbing a ledge (not a ladder).

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `boolean` - True if climbing a ledge

---

### ENT:FindLedge(propOnly)

Searches for a climbable ledge in front of the entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `propOnly` (boolean, optional) - Only find ledges on props (not world)

**Returns:**
- `Vector` - Ledge top position, or nil if none found

**Configuration:**
- Requires `self.ClimbLedges = true`
- Controlled by `self.LedgeDetectionDistance`
- Height limited by `self.ClimbLedgesMinHeight` and `self.ClimbLedgesMaxHeight`
- Props require `self.ClimbProps = true`

**Example:**
```lua
function ENT:OnIdle()
    local ledge = self:FindLedge()
    if ledge then
        self:ClimbLedge(ledge)
    end
end
```

---

### ENT:ClimbLedge(ledge, callback)

Climbs a ledge to the specified position.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `ledge` (Vector) - Top of ledge position (usually from FindLedge)
- `callback` (function, optional) - Called each frame: `callback(self, ledge, remaining, down)`. Return true to stop

**Returns:** None (yields until climb complete)

**Example:**
```lua
function ENT:OnIdle()
    local ledge = self:FindLedge()
    if ledge then
        self:ClimbLedge(ledge, function(self, ledge, remaining)
            if remaining < 10 then
                print("Almost at the top!")
            end
        end)
        print("Finished climbing!")
    end
end
```

---

### ENT:ClimbLadder(ladder, down, callback)

Climbs a ladder entity.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `ladder` (Entity) - Ladder entity (func_useableladder)
- `down` (boolean) - True to climb down, false to climb up
- `callback` (function, optional) - Called each frame: `callback(self, ladder, remaining, down)`. Return true to stop

**Returns:** None (yields until climb complete)

**Configuration:**
- Requires `self.ClimbLadders = true`
- Up requires `self.ClimbLaddersUp = true`
- Down requires `self.ClimbLaddersDown = true`
- Speed controlled by `self.ClimbSpeed`

**Example:**
```lua
function ENT:OnIdle()
    local ladder = self:FindLadderNearby()
    if IsValid(ladder) then
        self:ClimbLadder(ladder, false) -- Climb up
        print("Reached top of ladder!")
    end
end
```

---

### ENT:ClimbLadderUp(ladder)

Convenience function to climb a ladder upward.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `ladder` (Entity) - Ladder entity

**Returns:** None

---

### ENT:ClimbLadderDown(ladder)

Convenience function to climb a ladder downward.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `ladder` (Entity) - Ladder entity

**Returns:** None

---

## Locomotion Properties

### ENT:GetAcceleration()

Gets current acceleration rate.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Acceleration in units/second²

---

### ENT:SetAcceleration(accel)

Sets acceleration rate.

**Realm:** 🔴 SERVER

**Parameters:**
- `accel` (number) - Acceleration in units/second²

**Returns:** None

**Example:**
```lua
function ENT:CustomInitialize()
    self.Acceleration = 500
end
```

---

### ENT:GetDeceleration()

Gets current deceleration rate.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Deceleration in units/second²

---

### ENT:SetDeceleration(decel)

Sets deceleration rate.

**Realm:** 🔴 SERVER

**Parameters:**
- `decel` (number) - Deceleration in units/second²

**Returns:** None

---

### ENT:GetJumpHeight()

Gets jump height capability.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Jump height in units

---

### ENT:SetJumpHeight(height)

Sets jump height capability.

**Realm:** 🔴 SERVER

**Parameters:**
- `height` (number) - Jump height in units

**Returns:** None

---

### ENT:GetStepHeight()

Gets maximum step height (can walk over obstacles this tall).

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Step height in units

---

### ENT:SetStepHeight(height)

Sets maximum step height.

**Realm:** 🔴 SERVER

**Parameters:**
- `height` (number) - Step height in units

**Returns:** None

**Example:**
```lua
function ENT:CustomInitialize()
    self.StepHeight = 18
end
```

---

### ENT:GetMaxYawRate()

Gets maximum turning speed.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Degrees per second

---

### ENT:SetMaxYawRate(rate)

Sets maximum turning speed.

**Realm:** 🔴 SERVER

**Parameters:**
- `rate` (number) - Degrees per second

**Returns:** None

**Example:**
```lua
function ENT:CustomInitialize()
    self.MaxYawRate = 250 -- Faster turning
end
```

---

### ENT:GetDeathDropHeight()

Gets maximum safe fall distance.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Fall height in units

---

### ENT:SetDeathDropHeight(height)

Sets maximum safe fall distance. Entity takes damage if falling further.

**Realm:** 🔴 SERVER

**Parameters:**
- `height` (number) - Fall height in units

**Returns:** None

**Example:**
```lua
function ENT:CustomInitialize()
    self.DeathDropHeight = 200
end
```

---

### ENT:GetDesiredSpeed()

Gets the target speed the locomotion system is trying to reach.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Desired speed in units/second

---

### ENT:SetDesiredSpeed(speed)

Sets the target speed for the locomotion system.

**Realm:** 🔴 SERVER

**Parameters:**
- `speed` (number) - Desired speed in units/second

**Returns:** None

---

## Stuck Detection

### ENT:IsStuck()

Checks if the locomotion system detected the entity is stuck.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `boolean` - True if stuck

---

### ENT:ClearStuck()

Clears the stuck state.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

---

### ENT:IsStuckInWorld()

Checks if entity is stuck inside world geometry.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `boolean` - True if inside world

---

## Speed Update Hook

### ENT:UpdateSpeed()

Updates the locomotion speed. Called automatically each frame.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Note:** Automatically calls `OnUpdateSpeed()` hook.

---

### ENT:OnUpdateSpeed()

Hook to determine current movement speed. Override to customize speed behavior.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Speed in units/second, or -1 to use walkframe speed

**Default Behavior:**
- Returns `ClimbSpeed` when climbing
- Returns `-1` when `UseWalkframes = true`
- Returns `RunSpeed` when running
- Returns `WalkSpeed` otherwise

**Example:**
```lua
function ENT:OnUpdateSpeed()
    if self:HasEnemy() then
        return self.RunSpeed * 1.5 -- Extra fast when enemy nearby
    end
    return self.WalkSpeed
end
```

---

## Climbing Hooks

### ENT:OnStartClimbing(target, height, down)

Called when climbing starts (ledge or ladder).

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Ledge position or ladder entity
- `height` (number) - Height to climb
- `down` (boolean) - True if climbing down

**Returns:**
- `nil` - Use default climbing behavior
- `false` - Cancel climbing
- `true` - Use custom climbing (call CustomClimbing hook)

**Example:**
```lua
function ENT:OnStartClimbing(target, height, down)
    self:EmitSound("npc/zombie/zombie_voice_idle" .. math.random(1, 14) .. ".wav")
end
```

---

### ENT:OnClimbing(target, remaining, down)

Called each frame while climbing.

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Ledge position or ladder entity
- `remaining` (number) - Distance remaining (scaled)
- `down` (boolean) - True if climbing down

**Returns:**
- `boolean` - Return true to stop climbing early

**Example:**
```lua
function ENT:OnClimbing(target, remaining, down)
    if remaining < 5 then
        -- Almost done climbing
        self:PlayAnimation("climb_top")
    end
end
```

---

### ENT:WhileClimbing(target, remaining, down)

Alternative hook called during climbing. Same as OnClimbing.

**Realm:** 🔴 SERVER

**Parameters:** Same as OnClimbing

**Returns:** Same as OnClimbing

---

### ENT:OnStopClimbing(target, remaining, down)

Called when climbing ends.

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Ledge position or ladder entity
- `remaining` (number) - Distance remaining when stopped
- `down` (boolean) - True if was climbing down

**Returns:** None

**Example:**
```lua
function ENT:OnStopClimbing(target, remaining, down)
    if remaining > 0 then
        print("Climbing interrupted!")
    else
        print("Reached the top!")
    end
end
```

---

### ENT:CustomClimbing(target, height, down)

Called to implement custom climbing behavior when OnStartClimbing returns true.

**Realm:** 🔴 SERVER (Coroutine)

**Parameters:**
- `target` (Vector or Entity) - Ledge position or ladder entity
- `height` (number) - Total height to climb
- `down` (boolean) - True if climbing down

**Returns:** None

**Example:**
```lua
function ENT:OnStartClimbing(target, height, down)
    return true -- Use custom climbing
end

function ENT:CustomClimbing(target, height, down)
    self:PlayAnimation("custom_climb")
    self:Timer(2, function()
        self:SetPos(target)
    end)
    self:YieldCoroutine(false)
end
```

---

### ENT:HandleStuck()

Called when the entity becomes stuck. Override to implement unstuck behavior.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Default:** Calls `ClearStuck()`

**Example:**
```lua
function ENT:HandleStuck()
    print("I'm stuck! Trying to jump...")
    self.loco:Jump()
    self:ClearStuck()
end
```

---

## Configuration Properties

These properties should be set in `CustomInitialize()`:

### Movement Speeds
- `WalkSpeed` (number) - Walking speed (default varies)
- `RunSpeed` (number) - Running speed (default varies)
- `ClimbSpeed` (number) - Climbing speed (default varies)

### Locomotion
- `Acceleration` (number) - Acceleration rate (default 400)
- `Deceleration` (number) - Deceleration rate (default 400)
- `JumpHeight` (number) - Jump height (default 70)
- `StepHeight` (number) - Max step height (default 18)
- `MaxYawRate` (number) - Turn speed in degrees/sec (default 250)
- `DeathDropHeight` (number) - Max safe fall distance (default 200)

### Climbing - Ledges
- `ClimbLedges` (boolean) - Enable ledge climbing (default false)
- `ClimbLedgesMinHeight` (number) - Minimum ledge height (default 40)
- `ClimbLedgesMaxHeight` (number) - Maximum ledge height (default 100)
- `LedgeDetectionDistance` (number) - How far to detect ledges (default 50)
- `ClimbOffset` (Vector) - Position offset while climbing (default Vector(0,0,0))

### Climbing - Ladders
- `ClimbLadders` (boolean) - Enable ladder climbing (default false)
- `ClimbLaddersUp` (boolean) - Enable climbing up (default false)
- `ClimbLaddersDown` (boolean) - Enable climbing down (default false)
- `ClimbLaddersUpMinHeight` (number) - Minimum ladder height for up (default 0)
- `ClimbLaddersUpMaxHeight` (number) - Maximum ladder height for up (default huge)
- `ClimbLaddersDownMinHeight` (number) - Minimum drop for down (default 0)
- `ClimbLaddersDownMaxHeight` (number) - Maximum drop for down (default huge)
- `LaddersUpDistance` (number) - Distance to detect ladder bottom (default 50)
- `LaddersDownDistance` (number) - Distance to detect ladder top (default 50)

### Climbing - Props
- `ClimbProps` (boolean) - Enable climbing on props (default false)

### Animation
- `UseWalkframes` (boolean) - Use animation walkframes for speed (default false)

---

## Console Variables

- `drgbase_compute_delay` (default 0.1) - Delay between path computations
- `drgbase_avoid_obstacles` (default 1) - Enable obstacle avoidance
- `drgbase_multiplier_speed` (default 1) - Global speed multiplier

---

## See Also

- [Path Functions](./path.md)
- [Patrol Functions](./patrol.md)
- [Animation Functions](./animation.md)
- [Movement System Guide](../../systems/movement/README.md)
