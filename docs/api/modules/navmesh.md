# NavMesh Module
**File:** `lua/drgbase/modules/navmesh.lua`

**Realm:** Server only

Navigation mesh utility functions for efficient area lookups.

## Overview

Provides optimized lookup of navigation areas by their center position. The module pre-caches all nav areas at startup for O(1) lookups.

## Functions

### navmesh.DrG_GetAreaFromCenter(pos)
Gets a navigation area by its center position.

**Parameters:**
- `pos` (Vector) - The center position of the nav area

**Returns:**
- (CNavArea or nil) - The nav area with this center, or nil if not found

**Behavior:**
- Uses a pre-built lookup table for instant access
- Compares positions as strings for exact matching
- Returns nil if no area has exactly this center position

**Example:**
```lua
-- Get area from a known center point
local center = Vector(100, 200, 0)
local area = navmesh.DrG_GetAreaFromCenter(center)

if area then
    print("Found area:", area:GetID())
end

-- Check if a position is a nav area center
local pos = entity:GetPos()
local area = navmesh.DrG_GetAreaFromCenter(pos)
if area then
    print("Entity is at nav area center")
end
```

## Use Cases

### Reverse Lookup from Position
```lua
-- When you have a center position and need the area
function ENT:OnReachedDestination()
    local myPos = self:GetPos()
    local area = navmesh.DrG_GetAreaFromCenter(myPos)

    if area then
        print("Reached area ID:", area:GetID())
        -- Do something with the area
    end
end
```

### Validating Stored Positions
```lua
-- Check if a saved position is still a valid nav area center
function RestoreNavArea(savedPos)
    local area = navmesh.DrG_GetAreaFromCenter(savedPos)

    if area then
        return area
    else
        print("Saved nav area no longer exists")
        return nil
    end
end
```

### Area-Based Triggers
```lua
-- Trigger events when NPC reaches specific nav areas
function ENT:Think()
    local currentArea = self:GetCurrentNavArea()
    if currentArea then
        local center = currentArea:GetCenter()
        local area = navmesh.DrG_GetAreaFromCenter(center)

        -- Trigger area-specific logic
        if self.checkpoints[center] then
            self:OnCheckpointReached(area)
        end
    end
end
```

## Implementation Details

**Initialization:**
- On load, iterates through all nav areas via `navmesh.GetAllNavAreas()`
- Builds a lookup table: `{[tostring(center)] = area}`
- Center positions are converted to strings for table keys

**Performance:**
- O(1) lookup time (hash table)
- No runtime overhead
- Memory: ~20 bytes per nav area
- Built once at startup

**Limitations:**
- Only works with exact center positions
- Position must match exactly (no tolerance)
- Server-side only (CLIENT checks are at file start)

## Related Functions

- `navmesh.GetAllNavAreas()` - Gets all nav areas
- `CNavArea:GetCenter()` - Gets center of a nav area
- `navmesh.GetNearestNavArea()` - Finds nearest nav area to a position
