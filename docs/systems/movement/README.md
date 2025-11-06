# Movement & Pathfinding System

Locomotion, pathfinding, and navigation.

**Files:** `movements.lua`, `locomotion.lua`, `path.lua`, `patrol.lua`

## Overview

The movement system handles NPC locomotion, pathfinding with navigation meshes, climbing (ladders and ledges), and patrol behaviors. It integrates seamlessly with Garry's Mod's navmesh system and provides high-level movement functions.

Key features:
- **Locomotion** - Speed, acceleration, turning, and ground movement
- **Pathfinding** - Automatic navigation using navmeshes
- **Jumping & Climbing** - Ledge and ladder climbing with customizable heights
- **Patrol System** - Position, sound, and search patrol types
- **Obstacle Avoidance** - Automatic collision avoidance

## Components

### Locomotion

Basic movement mechanics controlled by the CLuaLocomotion interface.

**Speed Control:**
```lua
-- Get/set desired speed
self:SetSpeed(300)  -- Units per second
local speed = self:GetSpeed()

-- Speed automatically scales with model scale
-- Actual speed = SetSpeed * GetScale()
```

**Movement Parameters:**
```lua
self:SetAcceleration(1000)      -- Speed up rate
self:SetDeceleration(1000)      -- Slow down rate
self:SetJumpHeight(72)          -- Jump height in units
self:SetStepHeight(18)          -- Max step-up height
self:SetMaxYawRate(250)         -- Turning speed (degrees/sec)
self:SetDeathDropHeight(200)    -- Max safe fall distance
```

**Movement Queries:**
```lua
if self:IsMoving() then end         -- Has velocity
if self:IsRunning() then end        -- Moving fast (via ShouldRun hook)
if self:IsTurning() then end        -- Changing yaw
if self:IsOnGround() then end       -- On ground surface
if self:IsClimbing() then end       -- Climbing ladder/ledge
if self:IsStuck() then end          -- Stuck in geometry
```

**Directional Movement:**
```lua
self:MoveForward(distance)   -- Move forward
self:MoveBackward(distance)  -- Move backward
self:MoveRight(distance)     -- Strafe right
self:MoveLeft(distance)      -- Strafe left
self:TurnRight(angle)        -- Turn right (degrees)
self:TurnLeft(angle)         -- Turn left (degrees)
```

**Facing:**
```lua
self:FaceTowards(pos)     -- Gradually face position
self:FaceInstant(pos)     -- Instantly face position
self:FaceTo(pos)          -- Turn until facing (blocking)
self:FaceEnemy()          -- Face current enemy
```

### Pathfinding

Automatic navigation using Garry's Mod navmeshes.

**High-Level Pathfinding:**
```lua
-- Go to position (blocking, returns true if reached)
if self:GoTo(targetPos) then
    print("Reached destination!")
end

-- Chase entity (blocking)
self:ChaseEntity(enemy, tolerance)

-- Low-level path following (single step, non-blocking)
local result = self:FollowPath(targetPos)
-- Returns: "reached", "unreachable", "moving", "stuck", "obstacle", "ledge", "ladder_up", "ladder_down"
```

**Path Configuration:**
```lua
local path = self:GetPath()
path:SetGoalTolerance(20)           -- Distance considered "reached"
path:SetMinLookAheadDistance(300)   -- Look-ahead for smoothing

-- Manual path computation
self:ComputePath(targetPos, generatorFunc)
self:InvalidatePath()  -- Clear current path
```

**Path Generator (Custom Costs):**

Override `GetPathGenerator()` or use hooks to customize pathfinding:
```lua
function ENT:OnComputePath(fromArea, toArea)
    return 0  -- Extra cost (0 = no penalty)
end

function ENT:OnComputePathLadderUp(from, to, ladder)
    return 1  -- Cost multiplier (1 = normal, -1 = forbidden)
end

function ENT:OnComputePathLedge(from, to, height)
    return 1
end
```

**Navmesh Blacklisting:**
```lua
-- Block specific nav areas
self:BlacklistNavArea(area, true)
self:IsNavAreaBlacklisted(area)
```

### Jumping & Climbing

Vertical movement includes ledge climbing and ladder climbing.

**Ledges:**
```lua
-- Auto-detection
local ledge = self:FindLedge()  -- Returns vector or nil
if ledge then
    self:ClimbLedge(ledge)  -- Blocking coroutine
end
```

**Ledge Properties:**
```lua
ENT.ClimbLedges = true                -- Enable ledge climbing
ENT.ClimbLedgesMinHeight = 40         -- Minimum climbable height
ENT.ClimbLedgesMaxHeight = 120        -- Maximum climbable height
ENT.LedgeDetectionDistance = 50       -- Forward detection range
ENT.ClimbProps = false                -- Can climb prop_physics
```

**Ladders:**
```lua
-- Manual ladder climbing
self:ClimbLadderUp(ladder)    -- Blocking
self:ClimbLadderDown(ladder)  -- Blocking

-- Queries
if self:IsClimbingLadder(ladder) then end
if self:IsClimbingLedge() then end
```

**Ladder Properties:**
```lua
ENT.ClimbLadders = true
ENT.ClimbLaddersUp = true
ENT.ClimbLaddersDown = true
ENT.ClimbLaddersUpMinHeight = 0
ENT.ClimbLaddersUpMaxHeight = 999999
ENT.ClimbLaddersDownMinHeight = 0
ENT.ClimbLaddersDownMaxHeight = 999999
ENT.LaddersUpDistance = 50     -- Detection range
ENT.LaddersDownDistance = 50
```

**Climbing Hooks:**
```lua
function ENT:OnStartClimbing(target, height, down)
    -- Return false to cancel
    -- Return true to use custom climbing (call CustomClimbing)
    -- Return nothing for default behavior
end

function ENT:WhileClimbing(target, remaining, down)
    -- Called each frame
    -- Return true to stop early
end

function ENT:OnStopClimbing(target, height, down)
    -- Called when climb finishes
end

function ENT:CustomClimbing(target, height, down)
    -- Implement custom climbing behavior
end
```

### Patrol System

NPCs can patrol to positions, sounds, or search areas.

**Patrol Types:**
- `PATROL_POS` - Go to specific position
- `PATROL_SOUND` - Investigate sound source
- `PATROL_SEARCH` - Search for lost enemy

**Adding Patrols:**
```lua
-- Patrol to position
self:AddPatrolPos(Vector(100, 200, 0), run)

-- Investigate sound
self:AddPatrolSound(soundData)

-- Search for enemy
self:AddPatrolSearch(enemy)
```

**Patrol Queries:**
```lua
local patrol = self:GetPatrol()  -- Get current patrol
if self:HasPatrol() then end     -- Has any patrol

-- Manual control
self:RemovePatrol(patrol)
self:ClearPatrols()  -- Remove all
```

**Patrol Hooks:**
```lua
function ENT:OnSortPatrols(patrol1, patrol2)
    -- Return true if patrol1 has higher priority
end

function ENT:OnReachedPatrol()
    self:Wait(math.random(3, 7))  -- Default: wait before continuing
end

function ENT:OnPatrolUnreachable()
    -- Patrol cannot be reached
end

function ENT:WhilePatrolling()
    -- Called each frame while moving to patrol
end
```

## Configuration

**Locomotion Properties:**
```lua
ENT.WalkSpeed = 100
ENT.RunSpeed = 300
ENT.Acceleration = 1000
ENT.Deceleration = 1000
ENT.JumpHeight = 72
ENT.StepHeight = 18
ENT.MaxYawRate = 250
ENT.DeathDropHeight = 200
ENT.UseWalkframes = false  -- Use animation ground speed
```

**Climbing Properties:**
```lua
ENT.ClimbSpeed = 60
ENT.ClimbOffset = Vector(0, 0, 0)  -- Offset during climb
ENT.ClimbLedges = true
ENT.ClimbLedgesMinHeight = 40
ENT.ClimbLedgesMaxHeight = 120
ENT.ClimbProps = false
ENT.ClimbLadders = true
ENT.ClimbLaddersUp = true
ENT.ClimbLaddersDown = true
```

**ConVars:**
```lua
drgbase_compute_delay (0.1)         -- Path recompute interval
drgbase_avoid_obstacles (1)         -- Enable obstacle avoidance
drgbase_multiplier_speed (1)        -- Global speed multiplier
```

## Usage Examples

**Basic Movement:**
```lua
function ENT:OnChaseEnemy()
    local enemy = self:GetEnemy()
    if self:GoTo(enemy:GetPos()) then
        print("Reached enemy!")
    end
end
```

**Patrol Behavior:**
```lua
function ENT:OnIdle()
    -- Wander randomly
    local randomPos = self:RandomPos(1500)
    self:AddPatrolPos(randomPos)
end

function ENT:OnSound(ent, sound)
    -- Investigate sounds
    if sound.Channel == CHAN_WEAPON then
        self:AddPatrolSound(sound)
    end
end
```

**Custom Path Costs:**
```lua
function ENT:OnComputePathLadderUp(from, to, ladder)
    -- Avoid ladders unless necessary
    return 5  -- 5x cost penalty
end

function ENT:OnComputePathLedge(from, to, height)
    -- Prefer ledges
    return 0.5  -- 50% cost
end
```

**Movement with Callback:**
```lua
self:GoTo(targetPos, 20, function(self, path)
    -- Called each frame while moving
    if self:HasEnemy() then
        return false  -- Abort movement
    end
end)
```

**Obstacle Avoidance:**
```lua
local avoided, direction = self:AvoidObstacles(forwardOnly)
if avoided then
    print("Avoiding obstacle: " .. direction)
end
```

## API Reference

- [Movement Functions](../../api/nextbot/movement.md)
- [Path Functions](../../api/nextbot/path.md)
- [Patrol Functions](../../api/nextbot/patrol.md)

## Best Practices

**Pathfinding:**
- Always use `GoTo()` or `ChaseEntity()` for high-level navigation
- Use `FollowPath()` only for custom movement loops
- Set reasonable `GoalTolerance` (default: 20 units)
- Navmesh must be present for pathfinding to work

**Climbing:**
- Test ledge heights with your model scale
- Use `ClimbProps = false` unless NPCs should climb physics props
- Adjust `ClimbOffset` if NPC clips into walls during climb
- Implement `OnStartClimbing()` to prevent climbing in certain conditions

**Performance:**
- Paths recompute based on `drgbase_compute_delay`
- Use `InvalidatePath()` sparingly (forces immediate recompute)
- Blacklist nav areas to prevent expensive failed pathfinds
- Lower `drgbase_ai_radius` to reduce enemy search radius

**Patrol System:**
- Implement `OnSortPatrols()` to prioritize patrol types
- Use `PATROL_SOUND` for investigation behavior
- Clear old patrols to prevent stale waypoints
- Patrols are sorted by type priority then time added
