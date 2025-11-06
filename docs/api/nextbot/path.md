# Path & Pathfinding Functions

Pathfinding and navigation mesh functions.

**File:** `lua/entities/drgbase_nextbot/path.lua` (159 lines)

---

## Overview

The path system provides advanced pathfinding using Garry's Mod's navigation mesh. It includes path computation, cost calculation, area blacklisting, and custom path generators.

**Key Features:**
- Navigation mesh pathfinding with A* algorithm
- Custom cost generators for terrain preferences
- Area blacklisting to avoid specific regions
- Automatic handling of ladders, ledges, drops, and steps
- Path caching and recomputation

---

## Path Management

### ENT:GetPath()

Gets or creates the entity's pathfinding object.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `PathFollower` - The path object

**Note:** Automatically creates a new path with MinLookAheadDistance of 300 if one doesn't exist.

**Example:**
```lua
function ENT:OnIdle()
    local path = self:GetPath()
    if IsValid(path) then
        print("Path length: " .. path:GetLength())
    end
end
```

---

### ENT:InvalidatePath()

Invalidates and clears the current path.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Example:**
```lua
function ENT:OnDamaged(dmg)
    self:InvalidatePath() -- Recalculate path after taking damage
end
```

---

### ENT:ComputePath(pos, generator)

Computes a new path to the target position.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector) - Target position
- `generator` (function, optional) - Custom cost generator function (uses `GetPathGenerator()` if omitted)

**Returns:**
- `boolean` - True if path was successfully computed

**Example:**
```lua
function ENT:OnIdle()
    if self:ComputePath(targetPos) then
        print("Path computed successfully!")
    else
        print("No path found")
    end
end
```

---

### ENT:RefreshPath(generator)

Recomputes the path to the same destination.

**Realm:** 🔴 SERVER

**Parameters:**
- `generator` (function, optional) - Custom cost generator function

**Returns:**
- `boolean` - True if path was successfully computed

**Example:**
```lua
function ENT:OnIdle()
    if self:LastComputeSuccess() then
        self:RefreshPath() -- Update path to same target
    end
end
```

---

### ENT:UpdatePath()

Updates the path follower's current segment.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Note:** Called automatically by `FollowPath()`. Advances to next segment when current goal is reached.

---

### ENT:DrawPath()

Draws debug visualization of the current path.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Example:**
```lua
function ENT:OnThink()
    if GetConVar("developer"):GetBool() then
        self:DrawPath()
    end
end
```

---

### ENT:LastComputeSuccess()

Checks if the last path computation succeeded.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `boolean` - True if last compute succeeded

**Example:**
```lua
function ENT:OnIdle()
    self:ComputePath(targetPos)
    if not self:LastComputeSuccess() then
        print("Path computation failed!")
    end
end
```

---

### ENT:LastComputeTime()

Gets the time of the last path computation.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - CurTime() when path was last computed, or -1 if no path exists

**Example:**
```lua
function ENT:OnIdle()
    local time = self:LastComputeTime()
    if CurTime() - time > 5 then
        self:RefreshPath() -- Refresh old paths
    end
end
```

---

## Navigation Area Blacklisting

### ENT:BlacklistNavArea(area, blacklist)

Adds or removes a navigation area from the blacklist.

**Realm:** 🔴 SERVER

**Parameters:**
- `area` (CNavArea) - Navigation area to blacklist
- `blacklist` (boolean) - True to blacklist, false to unblacklist

**Returns:** None

**Example:**
```lua
function ENT:OnDamaged(dmg, attacker)
    local area = navmesh.GetNearestNavArea(self:GetPos())
    if IsValid(area) then
        self:BlacklistNavArea(area, true) -- Avoid this area
    end
end
```

---

### ENT:IsNavAreaBlacklisted(area)

Checks if a navigation area is blacklisted.

**Realm:** 🔴 SERVER

**Parameters:**
- `area` (CNavArea) - Navigation area to check

**Returns:**
- `boolean` - True if blacklisted

---

### ENT:BlacklistedNavAreas()

Gets all blacklisted navigation areas.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `table` - Array of blacklisted CNavArea objects

**Example:**
```lua
function ENT:CustomInitialize()
    -- Blacklist dangerous areas on spawn
    local dangerZone = navmesh.GetNearestNavArea(Vector(0, 0, 0))
    self:BlacklistNavArea(dangerZone, true)
end

function ENT:OnThink()
    local blacklisted = self:BlacklistedNavAreas()
    print("Avoiding " .. #blacklisted .. " areas")
end
```

---

## Path Cost Generator

### ENT:GetPathGenerator()

Returns the default path cost generator function used for pathfinding.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `function` - Cost generator function with signature: `function(area, fromArea, ladder, elevator, length)`

**Cost Generator Return Values:**
- `number >= 0` - Cost to traverse this segment (0 = free, higher = more expensive)
- `-1` - Area is not traversable (blocked)

**Cost Calculation:**
The generator calculates cost based on:
- Distance between areas
- Ladder climb costs (with height limits)
- Ledge climb costs (with height limits)
- Step heights
- Drop heights
- Underwater areas
- Custom hooks for each terrain type

**Example:**
```lua
function ENT:CustomInitialize()
    -- Override to prefer outdoor paths
    local oldGenerator = self:GetPathGenerator()

    function myGenerator(area, fromArea, ladder, elevator, length)
        local baseCost = oldGenerator(area, fromArea, ladder, elevator, length)
        if baseCost < 0 then return -1 end

        -- Prefer outdoor areas
        if area:IsUnderwater() then
            return baseCost * 3 -- Triple cost for water
        end

        return baseCost
    end

    -- Use custom generator
    self:ComputePath(targetPos, myGenerator)
end
```

---

## Path Cost Hooks

These hooks are called by the default path generator to calculate costs. Return values affect pathfinding behavior.

### ENT:OnComputePath(from, to)

Called for every area-to-area transition during pathfinding.

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area

**Returns:**
- `number >= 0` - Additional cost multiplier (0 = no extra cost, 1 = double cost)
- `-1` - Block this path segment

**Default:** Returns 0 (no extra cost)

**Example:**
```lua
function ENT:OnComputePath(from, to)
    -- Prefer areas near walls
    local toCenter = to:GetCenter()
    local trace = util.TraceLine({
        start = toCenter,
        endpos = toCenter + Vector(100, 0, 0),
        mask = MASK_SOLID_BRUSHONLY
    })

    if trace.Hit then
        return 0 -- Lower cost near walls
    else
        return 0.5 -- Higher cost in open areas
    end
end
```

---

### ENT:OnComputePathFlat(from, to)

Called for flat (same-height) area transitions.

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area

**Returns:**
- `number >= 0` - Cost multiplier
- `-1` - Block this path

**Default:** Returns 0

---

### ENT:OnComputePathStep(from, to, height)

Called for small height increases (within step height).

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area
- `height` (number) - Step height

**Returns:**
- `number >= 0` - Cost multiplier
- `-1` - Block this path

**Default:** Returns 0

**Example:**
```lua
function ENT:OnComputePathStep(from, to, height)
    -- Make steps more expensive for injured NPCs
    if self:Health() < self:GetMaxHealth() * 0.5 then
        return 2 -- Double cost when injured
    end
    return 0
end
```

---

### ENT:OnComputePathLedge(from, to, height)

Called for ledge climbs (height > step height, requires climbing).

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area
- `height` (number) - Ledge height

**Returns:**
- `number >= 0` - Cost multiplier (default 1)
- `-1` - Block this path

**Default:** Returns 1 (double cost)

**Note:** Only called if `ClimbLedges = true` and height is within min/max limits.

**Example:**
```lua
function ENT:OnComputePathLedge(from, to, height)
    if height > 60 then
        return 3 -- High ledges are expensive
    end
    return 1
end
```

---

### ENT:OnComputePathDrop(from, to, drop)

Called for drops (falling to lower areas).

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area
- `drop` (number) - Drop height

**Returns:**
- `number >= 0` - Cost multiplier (default 1)
- `-1` - Block this path

**Default:** Returns 1 (double cost)

**Note:** Only called for drops within DeathDropHeight limit.

---

### ENT:OnComputePathJump(from, to, height)

Called for jumps (not currently used by default generator, but available for custom implementations).

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area
- `height` (number) - Jump height

**Returns:**
- `number >= 0` - Cost multiplier (default 1)
- `-1` - Block this path

**Default:** Returns 1

---

### ENT:OnComputePathLadderUp(from, to, ladder)

Called for climbing ladders upward.

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area (ladder bottom)
- `to` (CNavArea) - Destination navigation area (ladder top)
- `ladder` (CNavLadder) - Ladder object

**Returns:**
- `number >= 0` - Cost multiplier (default 1)
- `-1` - Block this ladder

**Default:** Returns 1 (double cost)

**Note:** Only called if `ClimbLadders = true` and `ClimbLaddersUp = true`, and ladder height is within limits.

**Example:**
```lua
function ENT:OnComputePathLadderUp(from, to, ladder)
    local height = ladder:GetTop().z - ladder:GetBottom().z
    if height > 200 then
        return 5 -- Very expensive for tall ladders
    end
    return 1
end
```

---

### ENT:OnComputePathLadderDown(from, to, ladder)

Called for climbing ladders downward.

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area (ladder top)
- `to` (CNavArea) - Destination navigation area (ladder bottom)
- `ladder` (CNavLadder) - Ladder object

**Returns:**
- `number >= 0` - Cost multiplier (default 1)
- `-1` - Block this ladder

**Default:** Returns 1 (double cost)

**Note:** Only called if `ClimbLadders = true` and `ClimbLaddersDown = true`, and drop is within limits.

---

### ENT:OnComputePathUnderwater(from, to)

Called for underwater area transitions.

**Realm:** 🔴 SERVER

**Parameters:**
- `from` (CNavArea) - Source navigation area
- `to` (CNavArea) - Destination navigation area (underwater)

**Returns:**
- `number >= 0` - Cost multiplier (default 1)
- `-1` - Block underwater paths

**Default:** Returns 1 (double cost)

**Example:**
```lua
function ENT:OnComputePathUnderwater(from, to)
    return -1 -- Never go underwater
end
```

---

## Configuration Properties

These properties control pathfinding behavior. Set them in `CustomInitialize()`:

### Path Computation
- `drgbase_compute_delay` (convar, default 0.1) - Minimum delay between path recalculations

### Climbing Limits
- `ClimbLadders` (boolean) - Enable ladder use (default false)
- `ClimbLaddersUp` (boolean) - Enable climbing up ladders (default false)
- `ClimbLaddersDown` (boolean) - Enable climbing down ladders (default false)
- `ClimbLaddersUpMinHeight` (number) - Minimum ladder height for up (default 0)
- `ClimbLaddersUpMaxHeight` (number) - Maximum ladder height for up (default huge)
- `ClimbLaddersDownMinHeight` (number) - Minimum drop for down (default 0)
- `ClimbLaddersDownMaxHeight` (number) - Maximum drop for down (default huge)

### Ledge Climbing
- `ClimbLedges` (boolean) - Enable ledge climbing (default false)
- `ClimbLedgesMinHeight` (number) - Minimum climbable ledge height (default 40)
- `ClimbLedgesMaxHeight` (number) - Maximum climbable ledge height (default 100)

---

## Usage Examples

### Basic Pathfinding

```lua
function ENT:OnIdle()
    local path = self:GetPath()

    if self:ComputePath(targetPos) then
        while IsValid(path) do
            path:Update(self)
            self:YieldCoroutine(true)
        end
        print("Reached destination!")
    else
        print("No path available")
    end
end
```

### Custom Cost Generator

```lua
function ENT:CustomInitialize()
    self.preferredHeight = 100
end

function ENT:OnComputePath(from, to)
    -- Prefer areas at a certain height
    local heightDiff = math.abs(to:GetCenter().z - self.preferredHeight)
    if heightDiff > 200 then
        return 2 -- Double cost for areas far from preferred height
    end
    return 0
end
```

### Area Blacklisting

```lua
function ENT:OnTakeDamage(dmg)
    -- Avoid the area where we took damage
    local area = navmesh.GetNearestNavArea(self:GetPos())
    if IsValid(area) then
        self:BlacklistNavArea(area, true)
        self:Timer(10, function()
            self:BlacklistNavArea(area, false) -- Unblacklist after 10s
        end)
    end
end
```

### Terrain Preference

```lua
function ENT:OnComputePathLadderUp(from, to, ladder)
    return 10 -- Heavily avoid ladders
end

function ENT:OnComputePathUnderwater(from, to)
    return -1 -- Never go underwater
end

function ENT:OnComputePathDrop(from, to, drop)
    if drop > 100 then
        return 5 -- Avoid large drops
    end
    return 0
end
```

---

## Technical Details

### Path Segment Types

The path system recognizes these segment types (from PathFollower):
- Type 0: Normal ground movement
- Type 2: End of path (within goal tolerance)
- Type 4: Ladder up
- Type 5: Ladder down

### Path Recomputation

Paths are automatically recomputed when:
- Target has moved significantly from last computation
- Remaining path length is less than half the distance target moved
- Path becomes invalid

### Cost Calculation Formula

```
finalCost = baseCost + (distance * costMultiplier)

Where:
- baseCost = cost accumulated so far
- distance = area-to-area distance (or ladder length)
- costMultiplier = sum of all hook return values
```

---

## See Also

- [Movement Functions](./movement.md) - Path following and locomotion
- [Patrol Functions](./patrol.md) - Patrol point system
- [Navigation Mesh Guide](../../guides/navmesh.md) - Setting up navigation meshes
