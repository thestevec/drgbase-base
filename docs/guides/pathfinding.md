# Pathfinding & Navigation

This guide covers the pathfinding and movement systems in DrGBase, teaching you how to make your NPCs navigate the world intelligently.

## Table of Contents

- [Introduction](#introduction)
- [Basic Pathfinding](#basic-pathfinding)
- [Path Configuration](#path-configuration)
- [Patrol System](#patrol-system)
- [Obstacles and Navigation](#obstacles-and-navigation)
- [Custom Movement Behaviors](#custom-movement-behaviors)
- [Troubleshooting](#troubleshooting)
- [Examples and Best Practices](#examples-and-best-practices)

## Introduction

DrGBase provides a comprehensive pathfinding system built on top of Garry's Mod's navigation mesh. The system handles:

- Automatic path computation and following
- Ladder and ledge climbing
- Obstacle avoidance
- Dynamic path recalculation
- Patrol behaviors
- NavArea blacklisting

The pathfinding system requires a valid navigation mesh to be loaded in the map. Use `nav_generate` in the console to create one if needed.

### How It Works

The pathfinding system uses several key components:

1. **Path Object**: Manages the computed route from point A to point B
2. **Path Generator**: Calculates the cost of traversing different terrain types
3. **FollowPath**: Executes movement along the computed path
4. **Movement Functions**: Low-level functions for moving and turning

## Basic Pathfinding

### Simple Movement to Position

The easiest way to move your NPC to a position is using `GoTo`:

```lua
function ENT:OnIdle()
    local targetPos = Vector(100, 200, 0)
    self:GoTo(targetPos)
end
```

`GoTo` is a coroutine-based function that:
- Computes a path to the target
- Follows the path automatically
- Handles obstacles and recalculation
- Returns `true` if destination reached, `false` if unreachable

### Chasing Entities

To chase a moving entity (like a player or enemy):

```lua
function ENT:OnChaseEnemy()
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        self:ChaseEntity(enemy)
    end
end
```

`ChaseEntity` continuously updates the path to follow moving targets.

### Setting Tolerance

You can specify how close the NPC needs to get to the destination:

```lua
-- Get within 50 units of the target
self:GoTo(targetPos, 50)

-- Default tolerance is 20 units
self:GoTo(targetPos)
```

### Callbacks During Movement

Monitor movement progress with callbacks:

```lua
self:GoTo(targetPos, 20, function(npc, path)
    -- This function is called every tick while moving
    if npc:GetEnemy() then
        return false  -- Stop moving if we see an enemy
    end
    -- Return true/false to stop, or nothing to continue
end)
```

### Low-Level Path Following

For more control, use `FollowPath` directly:

```lua
function ENT:CustomMovement()
    while true do
        local result = self:FollowPath(targetPos, 20)

        if result == "reached" then
            print("Destination reached!")
            break
        elseif result == "unreachable" then
            print("Cannot reach destination")
            break
        elseif result == "moving" then
            -- Still moving
        elseif result == "stuck" then
            -- NPC is stuck, HandleStuck was called
        elseif result == "obstacle" then
            -- Obstacle detected, avoiding
        elseif result == "ledge" then
            -- Climbing a ledge
        elseif result == "ladder_up" or result == "ladder_down" then
            -- Climbing a ladder
        end

        self:YieldCoroutine(true)
    end
end
```

## Path Configuration

### Path Settings

Configure the path behavior using the Path object:

```lua
function ENT:Initialize()
    -- Get the path object
    local path = self:GetPath()

    -- Set minimum look-ahead distance (default: 300)
    path:SetMinLookAheadDistance(300)

    -- Set goal tolerance (default: 20)
    path:SetGoalTolerance(20)
end
```

### NavArea Blacklisting

Prevent NPCs from using specific navigation areas:

```lua
-- Blacklist a specific area
local area = navmesh.GetNearestNavArea(dangerousPos)
self:BlacklistNavArea(area, true)

-- Remove from blacklist
self:BlacklistNavArea(area, false)

-- Check if area is blacklisted
if self:IsNavAreaBlacklisted(area) then
    print("Area is blacklisted")
end

-- Get all blacklisted areas
local blacklisted = self:BlacklistedNavAreas()
```

### Path Generator Customization

The path generator calculates the cost of different terrain types. Override these hooks to customize behavior:

```lua
-- Base cost multiplier for any path segment (default: 0)
function ENT:OnComputePath(from, to)
    return 0
end

-- Cost multiplier for climbing ladders up (default: 1)
function ENT:OnComputePathLadderUp(from, to, ladder)
    return 1  -- Higher = less preferred
end

-- Cost multiplier for climbing ladders down (default: 1)
function ENT:OnComputePathLadderDown(from, to, ladder)
    return 1
end

-- Cost multiplier for climbing ledges (default: 1)
function ENT:OnComputePathLedge(from, to, height)
    if height > 100 then
        return 5  -- Make tall ledges less desirable
    end
    return 1
end

-- Cost multiplier for small steps (default: 0)
function ENT:OnComputePathStep(from, to, height)
    return 0  -- Steps are usually fine
end

-- Cost multiplier for dropping down (default: 1)
function ENT:OnComputePathDrop(from, to, drop)
    if drop > 200 then
        return 10  -- Avoid big drops
    end
    return 1
end

-- Cost multiplier for flat terrain (default: 0)
function ENT:OnComputePathFlat(from, to)
    return 0  -- Flat is good
end

-- Cost multiplier for underwater areas (default: 1)
function ENT:OnComputePathUnderwater(from, to)
    if not self.CanSwim then
        return -1  -- Return -1 to make path impossible
    end
    return 2  -- Prefer above water
end
```

Return `-1` from any hook to make that segment impassable.

### Manual Path Operations

```lua
-- Invalidate the current path (forces recomputation)
self:InvalidatePath()

-- Update the path (advance along it)
self:UpdatePath()

-- Compute a new path
self:ComputePath(targetPos)

-- Refresh the path to the same destination
self:RefreshPath()

-- Check if last computation succeeded
if self:LastComputeSuccess() then
    print("Path was computed successfully")
end

-- Get the time of last computation
local computeTime = self:LastComputeTime()

-- Draw the path (for debugging)
self:DrawPath()
```

## Patrol System

The patrol system allows NPCs to investigate points of interest with automatic prioritization.

### Patrol Types

DrGBase includes three built-in patrol types:

1. **PATROL_POS**: Patrol to a specific position
2. **PATROL_SOUND**: Investigate a sound
3. **PATROL_SEARCH**: Search for a lost enemy

### Adding Patrols

```lua
-- Add a position patrol
self:AddPatrolPos(Vector(100, 200, 0), true)  -- true = run to position

-- Add a sound patrol
function ENT:OnHearSound(sound)
    self:AddPatrolSound(sound)
end

-- Add a search patrol (when enemy is lost)
function ENT:OnLoseEnemy(enemy)
    self:AddPatrolSearch(enemy)
end
```

### Managing Patrols

```lua
-- Get the current highest-priority patrol
local patrol = self:GetPatrol()
if patrol then
    local pos = patrol:FetchPos()
    local shouldRun = patrol:ShouldRun()
    print("Patrolling to", pos, shouldRun and "running" or "walking")
end

-- Check if NPC has any patrols
if self:HasPatrol() then
    print("Has patrol!")
end

-- Remove a specific patrol
self:RemovePatrol(patrol)

-- Clear all patrols
self:ClearPatrols()
```

### Using Patrols in Behaviors

```lua
function ENT:OnPatrol()
    local patrol = self:GetPatrol()
    if not patrol then return end

    local pos = patrol:FetchPos()
    local reached = self:GoTo(pos)

    if reached then
        patrol:OnReached(self)
        self:RemovePatrol(patrol)
    else
        patrol:OnUnreachable(self)
        self:RemovePatrol(patrol)
    end
end
```

### Patrol Priority

By default, patrols are prioritized by:
1. Validity (invalid patrols are ignored)
2. Type (higher type numbers = higher priority)
3. Time (older patrols = higher priority)

Customize patrol sorting:

```lua
function ENT:OnSortPatrols(patrol1, patrol2)
    -- Return true if patrol1 should come before patrol2
    -- Return false if patrol2 should come before patrol1
    -- Return nil to use default sorting

    -- Example: Prioritize sound patrols from weapons
    if patrol1:GetType() == PATROL_SOUND and patrol2:GetType() ~= PATROL_SOUND then
        if patrol1:GetSound().Channel == CHAN_WEAPON then
            return true
        end
    end

    return nil  -- Use default sorting
end
```

### Custom Patrol Types

Create your own patrol types:

```lua
-- In shared.lua or autorun
PATROL_CUSTOM = 4  -- Higher number = higher priority

-- In server.lua or ENT
if SERVER then
    local PatrolCustom = DrGBase.Patrol(PATROL_CUSTOM)

    function PatrolCustom:New(data)
        local patrol = {}
        patrol._data = data
        setmetatable(patrol, self)
        return patrol
    end

    function PatrolCustom:FetchPos()
        return self._data.pos
    end

    function PatrolCustom:ShouldRun()
        return self._data.urgent
    end

    function PatrolCustom:IsValid(nextbot)
        -- Check if this patrol is still valid
        return true
    end

    function PatrolCustom:OnReached(nextbot)
        print("Custom patrol reached!")
    end

    function PatrolCustom:__tostring()
        return "PatrolCustom"
    end

    -- Add method to NPC
    function ENT:AddPatrolCustom(data)
        return self:AddPatrol(PatrolCustom:New(data))
    end
end
```

## Obstacles and Navigation

### Obstacle Avoidance

DrGBase automatically avoids obstacles when the `drgbase_avoid_obstacles` convar is enabled (default: 1).

```lua
-- Manually trigger obstacle avoidance
local avoided, direction = self:AvoidObstacles(true)  -- true = forward only
if avoided then
    print("Avoiding obstacle, moving", direction)
end
```

The system detects obstacles using collision hulls and automatically adjusts movement.

### Climbing System

#### Ladders

Configure ladder climbing in your NPC:

```lua
ENT.ClimbLadders = true              -- Enable ladder climbing
ENT.ClimbLaddersUp = true            -- Can climb up ladders
ENT.ClimbLaddersDown = true          -- Can climb down ladders
ENT.ClimbLaddersUpMinHeight = 0      -- Minimum height to climb up
ENT.ClimbLaddersUpMaxHeight = 5000   -- Maximum height to climb up
ENT.ClimbLaddersDownMinHeight = 0    -- Minimum height to climb down
ENT.ClimbLaddersDownMaxHeight = 5000 -- Maximum height to climb down
ENT.LaddersUpDistance = 50           -- Distance to start climbing up
ENT.LaddersDownDistance = 50         -- Distance to start climbing down
```

Climbing hooks:

```lua
function ENT:OnStartClimbing(target, height, isDown)
    -- Called when climbing starts
    -- target is ladder entity or ledge position
    -- height is vertical distance
    -- isDown is true if climbing down

    -- Return false to prevent climbing
    -- Return true to use custom climbing (must implement CustomClimbing)
    -- Return nothing for default behavior
end

function ENT:OnClimbing(target, remaining, isDown)
    -- Called every frame while climbing
    -- remaining is distance left to climb

    -- Return true to stop climbing early
end

function ENT:OnStopClimbing(target, remaining, isDown)
    -- Called when climbing ends
    -- remaining is how much was left (0 if completed)
end

function ENT:CustomClimbing(target, height, isDown)
    -- Implement your own climbing logic
    -- Only called if OnStartClimbing returns true
end
```

Manual ladder climbing:

```lua
-- Climb up a ladder
self:ClimbLadderUp(ladderEntity)

-- Climb down a ladder
self:ClimbLadderDown(ladderEntity)

-- Check if climbing
if self:IsClimbing() then
    print("Currently climbing!")

    if self:IsClimbingLadder() then
        local isClimbing, ladder = self:IsClimbingLadder()
        print("Climbing ladder", ladder)
    elseif self:IsClimbingLedge() then
        print("Climbing ledge")
    end

    if self:IsClimbingUp() then
        print("Going up")
    elseif self:IsClimbingDown() then
        print("Going down")
    end
end
```

#### Ledges

Configure ledge climbing:

```lua
ENT.ClimbLedges = true               -- Enable ledge climbing
ENT.ClimbLedgesMinHeight = 30        -- Minimum ledge height
ENT.ClimbLedgesMaxHeight = 100       -- Maximum ledge height
ENT.LedgeDetectionDistance = 50      -- How far to detect ledges
ENT.ClimbProps = true                -- Can climb on props
ENT.ClimbOffset = Vector(0, 0, 0)    -- Offset while climbing
```

Manual ledge climbing:

```lua
-- Find a ledge in front of the NPC
local ledge = self:FindLedge()
if ledge then
    print("Found ledge at", ledge)
    self:ClimbLedge(ledge)
end

-- Find ledge on props only
local propLedge = self:FindLedge(true)
```

### Handling Stuck NPCs

```lua
function ENT:HandleStuck()
    -- Called when NPC gets stuck
    -- Default behavior: clear stuck state
    self:ClearStuck()

    -- Custom behavior:
    print("I'm stuck! Trying to get unstuck...")

    -- Option 1: Jump
    if self.CanJump then
        self:Jump()
    end

    -- Option 2: Move backward
    self:MoveBackward(100)

    -- Option 3: Try different path
    self:InvalidatePath()

    -- Always clear stuck state when done
    self:ClearStuck()
end
```

## Custom Movement Behaviors

### Direct Movement Control

```lua
-- Face towards a position or entity
self:FaceTowards(position)
self:FaceTowards(entity)

-- Instantly face a position
self:FaceInstant(position)

-- Face and wait until looking at target (coroutine)
self:FaceTo(position)

-- Face current enemy
self:FaceEnemy()

-- Approach a position or entity
self:Approach(position, multiplier)  -- multiplier defaults to 1

-- Move towards (face + approach)
self:MoveTowards(position)

-- Move away from a position
self:MoveAwayFrom(position)  -- turns away
self:MoveAwayFrom(position, true)  -- keeps facing position
```

### Directional Movement

```lua
-- Move in a direction for a distance
self:MoveForward(100)  -- Move forward 100 units
self:MoveBackward(100)
self:MoveRight(100)
self:MoveLeft(100)

-- Move with a callback every tick
self:MoveForward(100, function(npc, distanceTraveled)
    if distanceTraveled > 50 then
        print("Halfway there!")
    end

    -- Return true to stop moving early
    if npc:GetEnemy() then
        return true
    end
end)

-- Move without distance (just one frame)
self:MoveForward()
self:MoveBackward()
self:MoveRight()
self:MoveLeft()
```

### Turning

```lua
-- Turn by a specific angle
self:TurnRight(90)  -- Turn 90 degrees right
self:TurnLeft(45)   -- Turn 45 degrees left

-- Turn with callback
self:TurnRight(180, function(npc, anglesTurned)
    print("Turned", anglesTurned, "degrees so far")

    -- Return true to stop turning early
end)

-- Turn without angle (just one frame)
self:TurnRight()
self:TurnLeft()
```

### Speed Control

```lua
-- Set the NPC's movement speed
self:SetSpeed(100)  -- Sets to 100 * scale

-- Get the desired speed
local desiredSpeed = self:GetSpeed()

-- Get current actual speed
local speed = self:Speed()  -- Gets velocity length
local speedScaled = self:Speed(true)  -- Divided by scale

-- Speed comparisons (more efficient than calculating speed)
if self:IsSpeedMore(100) then print("Moving faster than 100") end
if self:IsSpeedLess(100) then print("Moving slower than 100") end
if self:IsSpeedEqual(100) then print("Moving at exactly 100") end
if self:IsSpeedMoreEqual(100) then print("Moving at or faster than 100") end
if self:IsSpeedLessEqual(100) then print("Moving at or slower than 100") end

-- Speed squared (for comparisons)
local speedSqr = self:SpeedSqr()
```

### Movement State Queries

```lua
-- Check if moving at all
if self:IsMoving() then
    print("NPC is moving")
end

-- Check movement direction
if self:IsMovingForward() then print("Moving forward") end
if self:IsMovingBackward() then print("Moving backward") end
if self:IsMovingRight() then print("Moving right") end
if self:IsMovingLeft() then print("Moving left") end
if self:IsMovingUp() then print("Moving up") end
if self:IsMovingDown() then print("Moving down") end

-- Check combined directions
if self:IsMovingForwardLeft() then print("Moving forward-left") end
if self:IsMovingForwardRight() then print("Moving forward-right") end
if self:IsMovingBackwardLeft() then print("Moving backward-left") end
if self:IsMovingBackwardRight() then print("Moving backward-right") end

-- Check turning
if self:IsTurning() then print("Turning") end
if self:IsTurningLeft() then print("Turning left") end
if self:IsTurningRight() then print("Turning right") end

-- Check if running
if self:IsRunning() then print("Running") end
```

### Custom Speed Calculation

```lua
function ENT:OnUpdateSpeed()
    -- Return a speed value to override default behavior

    if self:IsClimbing() then
        return self.ClimbSpeed
    end

    if self:HasPatrol() then
        local patrol = self:GetPatrol()
        if patrol:ShouldRun() then
            return self.RunSpeed
        end
    end

    if self:IsRunning() then
        return self.RunSpeed
    end

    return self.WalkSpeed

    -- Return -1 to use walkframe speed
    -- return -1
end
```

### Getting Movement Direction

```lua
-- Get the direction the NPC is moving relative to its facing
local moveDir = self:GetMovement()  -- Returns vector
local moveDirFlat = self:GetMovement(true)  -- Ignores Z axis

-- moveDir.x > 0 means moving forward
-- moveDir.y > 0 means moving right
-- moveDir.z > 0 means moving up
```

## Troubleshooting

### Path Not Computing

**Problem**: `GoTo` returns false immediately or NPC doesn't move.

**Solutions**:
1. Check if navigation mesh exists: `lua_run print(navmesh.IsLoaded())`
2. Generate nav mesh: `nav_generate` in console
3. Check if destination is on nav mesh:
   ```lua
   local area = navmesh.GetNearestNavArea(targetPos)
   print(IsValid(area))
   ```
4. Enable path debugging: `self:DrawPath()` in Think hook
5. Check if NPC is on ground: `print(self:GetGroundEntity())`

### NPC Gets Stuck

**Problem**: NPC stops moving and doesn't recover.

**Solutions**:
1. Implement `HandleStuck`:
   ```lua
   function ENT:HandleStuck()
       self:MoveBackward(50)
       self:InvalidatePath()
       self:ClearStuck()
   end
   ```
2. Lower path tolerance: `self:GoTo(pos, 10)`
3. Check collision hull size
4. Increase movement speed
5. Enable obstacle avoidance

### NPC Won't Climb

**Problem**: NPC avoids ladders or ledges.

**Solutions**:
1. Enable climbing:
   ```lua
   ENT.ClimbLadders = true
   ENT.ClimbLedges = true
   ```
2. Check height limits:
   ```lua
   ENT.ClimbLaddersUpMaxHeight = 5000
   ENT.ClimbLedgesMaxHeight = 200
   ```
3. Adjust path costs:
   ```lua
   function ENT:OnComputePathLadderUp(from, to, ladder)
       return 0.5  -- Make ladders more desirable
   end
   ```
4. Check if ladder is in nav mesh: Use nav_edit

### Path Takes Wrong Route

**Problem**: NPC takes a long or dangerous route.

**Solutions**:
1. Adjust path costs in hooks:
   ```lua
   function ENT:OnComputePathDrop(from, to, drop)
       if drop > 100 then
           return 10  -- Heavily penalize drops
       end
       return 1
   end
   ```
2. Blacklist dangerous areas:
   ```lua
   local area = navmesh.GetNearestNavArea(dangerPos)
   self:BlacklistNavArea(area, true)
   ```
3. Regenerate nav mesh with better connections
4. Use nav_edit to fix nav mesh issues

### Obstacle Avoidance Not Working

**Problem**: NPC runs into obstacles.

**Solutions**:
1. Enable obstacle avoidance: `drgbase_avoid_obstacles 1`
2. Check collision detection distance
3. Verify path is being followed: check `FollowPath` return values
4. Adjust collision hull size
5. Implement custom avoidance:
   ```lua
   function ENT:CustomBehavior()
       while true do
           local result = self:FollowPath(targetPos)
           if result == "obstacle" then
               -- Custom obstacle handling
               self:MoveBackward(20)
               self:TurnRight(45)
           end
           self:YieldCoroutine(true)
       end
   end
   ```

### Performance Issues

**Problem**: Pathfinding causes lag.

**Solutions**:
1. Increase compute delay: `drgbase_compute_delay 0.3`
2. Use larger tolerances: `self:GoTo(pos, 50)`
3. Cache paths when possible
4. Limit NPC count
5. Optimize path generator hooks (avoid expensive calculations)
6. Use simpler nav mesh

## Examples and Best Practices

### Example 1: Basic Enemy Chasing

```lua
function ENT:OnChaseEnemy()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    -- Chase with attack distance tolerance
    local reached = self:ChaseEntity(enemy, self.AttackRange)

    if reached then
        self:Attack(enemy)
    end
end
```

### Example 2: Patrol Between Points

```lua
function ENT:Initialize()
    -- Define patrol points
    self.PatrolPoints = {
        Vector(100, 100, 0),
        Vector(500, 100, 0),
        Vector(500, 500, 0),
        Vector(100, 500, 0),
    }
    self.CurrentPatrolIndex = 1
end

function ENT:OnPatrol()
    local point = self.PatrolPoints[self.CurrentPatrolIndex]
    local reached = self:GoTo(point, 30)

    if reached then
        -- Wait a bit before moving to next point
        self:Wait(2)

        -- Move to next point
        self.CurrentPatrolIndex = self.CurrentPatrolIndex + 1
        if self.CurrentPatrolIndex > #self.PatrolPoints then
            self.CurrentPatrolIndex = 1
        end
    end
end
```

### Example 3: Investigate Sounds

```lua
function ENT:OnHearSound(sound)
    -- Only investigate weapon sounds when idle
    if self:GetBehavior() == "idle" and sound.Channel == CHAN_WEAPON then
        self:AddPatrolSound(sound)
        self:SetBehavior("patrol")
    end
end

function ENT:OnPatrol()
    local patrol = self:GetPatrol()
    if not patrol then
        self:SetBehavior("idle")
        return
    end

    local reached = self:GoTo(patrol:FetchPos(), 50)

    if reached then
        -- Look around
        self:Wait(1)
        self:TurnRight(90)
        self:Wait(0.5)
        self:TurnLeft(180)
        self:Wait(0.5)
        self:TurnRight(90)

        -- Done investigating
        self:RemovePatrol(patrol)
    end
end
```

### Example 4: Smart Cover Movement

```lua
function ENT:OnTakeCover()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    -- Find cover position away from enemy
    local coverPos = self:FindCoverPosition(enemy:GetPos())
    if not coverPos then return end

    -- Move to cover while watching enemy
    local success = self:GoTo(coverPos, 30, function(npc, path)
        -- Keep facing enemy while moving
        npc:FaceTowards(enemy)

        -- If we get shot, find new cover
        if npc:Health() < npc._LastHealth then
            return false  -- Stop moving
        end
        npc._LastHealth = npc:Health()
    end)

    if success then
        -- Wait in cover
        self:Wait(math.random(2, 5))
    end
end

function ENT:FindCoverPosition(enemyPos)
    local myPos = self:GetPos()
    local direction = (myPos - enemyPos):GetNormalized()

    -- Try to find cover 300 units away from enemy
    for i = 1, 8 do
        local angle = (i / 8) * 360
        local testDir = direction
        testDir:Rotate(Angle(0, angle, 0))
        local testPos = myPos + testDir * 300

        -- Check if position provides cover
        local area = navmesh.GetNearestNavArea(testPos)
        if IsValid(area) then
            testPos = area:GetClosestPointOnArea(testPos)

            -- Check line of sight
            local tr = util.TraceLine({
                start = testPos + Vector(0, 0, 50),
                endpos = enemyPos + Vector(0, 0, 50),
                filter = self
            })

            if tr.Hit then
                return testPos  -- This position blocks line of sight
            end
        end
    end

    return nil
end
```

### Example 5: Complex Path Cost Customization

```lua
-- Make NPC prefer indoor routes and avoid water
function ENT:OnComputePath(from, to)
    -- Check if area is outdoors
    local center = to:GetCenter()
    local tr = util.TraceLine({
        start = center,
        endpos = center + Vector(0, 0, 10000),
        mask = MASK_SOLID_BRUSHONLY
    })

    if not tr.Hit then
        -- Outdoor area, increase cost
        return 0.5
    end

    return 0
end

function ENT:OnComputePathUnderwater(from, to)
    -- Heavily avoid water
    return 10
end

function ENT:OnComputePathLadderUp(from, to, ladder)
    local height = ladder:GetTop().z - ladder:GetBottom().z

    -- Prefer short ladders
    if height < 100 then
        return 0.5
    elseif height > 300 then
        return 3
    end

    return 1
end
```

### Example 6: Dynamic Path Adjustment

```lua
function ENT:OnThink()
    -- Periodically check if path is still optimal
    if self:IsMoving() and CurTime() - self._LastPathCheck > 2 then
        self._LastPathCheck = CurTime()

        local path = self:GetPath()
        if IsValid(path) then
            local destination = path:GetEnd()
            local currentDist = self:GetRangeTo(destination)

            -- If we're not getting closer, recompute
            if self._LastDistToGoal and currentDist >= self._LastDistToGoal then
                self:InvalidatePath()
            end

            self._LastDistToGoal = currentDist
        end
    end
end
```

## Best Practices

### 1. Always Use Coroutines for Movement

```lua
-- Good
function ENT:CustomBehavior()
    self:GoTo(pos)  -- Blocks until reached
    self:Wait(2)
    self:GoTo(nextPos)
end

-- Bad
function ENT:Think()
    if not self.moving then
        self:GoTo(pos)  -- Will block the entire game!
    end
end
```

### 2. Handle Unreachable Destinations

```lua
local reached = self:GoTo(targetPos)
if not reached then
    -- Destination unreachable, do something else
    self:SetBehavior("idle")
end
```

### 3. Use Appropriate Tolerances

```lua
-- Attacking: tight tolerance
self:ChaseEntity(enemy, self.AttackRange)

-- Patrolling: loose tolerance
self:GoTo(patrolPoint, 100)

-- Precise positioning: very tight tolerance
self:GoTo(buttonPos, 5)
```

### 4. Clean Up Patrols

```lua
-- Clear old patrols when changing behaviors
function ENT:SetBehavior(behavior)
    if behavior ~= "patrol" then
        self:ClearPatrols()
    end
    -- ... rest of behavior change code
end
```

### 5. Test Without Nav Mesh

Always ensure your NPC can still move without a nav mesh (direct line movement):

```lua
function ENT:OnIdle()
    local target = self:RandomPos()
    self:GoTo(target)  -- Works with or without nav mesh
end
```

### 6. Optimize Path Recalculation

```lua
-- Instead of computing every frame
function ENT:CustomMovement()
    while true do
        self:FollowPath(targetPos)
        self:YieldCoroutine(true)
    end
end

-- Use compute delay or manual control
local ComputeDelay = 0.3
function ENT:CustomMovement()
    local lastCompute = 0
    while true do
        if CurTime() - lastCompute > ComputeDelay then
            self:ComputePath(targetPos)
            lastCompute = CurTime()
        end
        self:FollowPath(targetPos)
        self:YieldCoroutine(true)
    end
end
```

### 7. Combine Movement with Animations

```lua
function ENT:OnUpdateSpeed()
    -- Return -1 to use animation movement speed
    if self.UseWalkframes then
        return -1
    end

    return self:IsRunning() and self.RunSpeed or self.WalkSpeed
end
```

### 8. Debug Visually

```lua
hook.Add("PostDrawTranslucentRenderables", "MyNPC_Debug", function()
    for _, npc in ipairs(ents.FindByClass("my_npc")) do
        if IsValid(npc) then
            npc:DrawPath()

            -- Draw patrol points
            local patrol = npc:GetPatrol()
            if patrol then
                local pos = patrol:FetchPos()
                render.DrawWireframeSphere(pos, 30, 12, 12, Color(255, 255, 0))
            end
        end
    end
end)
```

### 9. Save Resources

```lua
-- Don't pathfind if very close
function ENT:SmartGoTo(pos, tolerance)
    tolerance = tolerance or 20

    if self:GetRangeTo(pos) < tolerance * 1.5 then
        -- Close enough, just move directly
        while self:GetRangeTo(pos) > tolerance do
            self:MoveTowards(pos)
            self:YieldCoroutine(true)
        end
        return true
    else
        -- Far away, use pathfinding
        return self:GoTo(pos, tolerance)
    end
end
```

### 10. Handle Edge Cases

```lua
function ENT:SafeGoTo(target, tolerance)
    -- Validate target
    if isentity(target) then
        if not IsValid(target) then
            return false
        end
    elseif not isvector(target) then
        return false
    end

    -- Store starting position
    local startPos = self:GetPos()
    local startTime = CurTime()

    -- Attempt to reach target
    local reached = self:GoTo(target, tolerance, function(npc, path)
        -- Timeout after 30 seconds
        if CurTime() - startTime > 30 then
            return false
        end

        -- Give up if not making progress
        if CurTime() - startTime > 5 and npc:GetRangeTo(startPos) < 50 then
            return false
        end
    end)

    return reached
end
```

## Conclusion

The DrGBase pathfinding system is powerful and flexible. Key takeaways:

- Use `GoTo` and `ChaseEntity` for simple movement
- Customize path costs for intelligent route selection
- Implement the patrol system for investigation behaviors
- Handle climbing and obstacles for realistic navigation
- Always test with and without nav meshes
- Debug visually when troubleshooting

For more information, see:
- [Behaviors Guide](behaviors.md) - Using movement in behaviors
- [Coroutines Guide](coroutines.md) - Understanding coroutine-based movement
- [API Reference](../api/movement.md) - Complete movement API
