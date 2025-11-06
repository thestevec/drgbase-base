# NavMesh Module
**File:** `lua/drgbase/modules/navmesh.lua`

The navmesh module provides enhanced navigation mesh utilities for AI pathfinding and navigation.

## Overview

This module extends Garry's Mod's navigation mesh system by providing optimized lookup functions. It builds an internal cache of nav areas indexed by their center positions for fast retrieval. This is particularly useful when you have a position and need to find the corresponding navigation area without iterating through all areas.

**Important:** This module only loads on the server realm.

---

## Functions

### navmesh.DrG_GetAreaFromCenter(pos)

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector) - The center position of the nav area to find

**Returns:**
- `CNavArea` or `nil` - The navigation area with the matching center position, or nil if not found

**Description:**

Retrieves a navigation area by its center position using an optimized lookup table. This function provides O(1) constant-time lookup of nav areas when you know the exact center position, as opposed to iterating through all nav areas.

The function maintains an internal cache that maps stringified center positions to their corresponding nav areas. This cache is built automatically when the module loads by iterating through all nav areas once via `navmesh.GetAllNavAreas()`.

This is particularly useful when you need to:
- Store nav area references by their center positions (which are serializable)
- Quickly retrieve nav areas after reconstructing their center position
- Work with nav area data that has been transmitted over the network or saved to disk

The position must match exactly with a nav area's center point. If you need to find the nearest nav area to an arbitrary position, use `navmesh.GetNearestNavArea()` instead.

**Example:**
```lua
-- Store a nav area reference by its center position
local myNavArea = navmesh.GetNearestNavArea(entity:GetPos())
if myNavArea then
    local centerPos = myNavArea:GetCenter()
    -- Store centerPos in a database or entity data

    -- Later, retrieve the nav area using the stored position
    local retrievedArea = navmesh.DrG_GetAreaFromCenter(centerPos)
    if retrievedArea then
        print("Found the same nav area!")
    end
end

-- Working with saved nav area data
local savedNavCenters = {
    Vector(100, 200, 0),
    Vector(500, -300, 64),
    Vector(-200, 400, 128)
}

for _, centerPos in ipairs(savedNavCenters) do
    local area = navmesh.DrG_GetAreaFromCenter(centerPos)
    if area then
        -- Spawn entities or set up waypoints at these areas
        local spawnPos = area:GetCenter() + Vector(0, 0, 10)
        local npc = ents.Create("npc_zombie")
        npc:SetPos(spawnPos)
        npc:Spawn()
    end
end

-- Validating nav areas after loading
function ValidateNavPoint(centerPos)
    local area = navmesh.DrG_GetAreaFromCenter(centerPos)
    if not area then
        print("Warning: Nav area no longer exists at", centerPos)
        return false
    end
    if area:IsBlocked() then
        print("Warning: Nav area is blocked")
        return false
    end
    return true
end

-- Working with networked nav data
-- SERVER:
util.AddNetworkString("SendNavArea")
net.Start("SendNavArea")
net.WriteVector(myNavArea:GetCenter())
net.Send(player)

-- CLIENT receives center, sends back to server for processing
net.Receive("SendNavArea", function()
    local centerPos = net.ReadVector()
    -- Send back to server
    net.Start("ProcessNavArea")
    net.WriteVector(centerPos)
    net.SendToServer()
end)

-- SERVER processes:
net.Receive("ProcessNavArea", function(len, ply)
    local centerPos = net.ReadVector()
    local area = navmesh.DrG_GetAreaFromCenter(centerPos)
    if area then
        -- Use the area for pathfinding
    end
end)
```

**Notes:**
- Only works on SERVER realm - returns nil on client
- Requires an exact match of the center position (Vector comparison by string representation)
- The internal cache is built once during module initialization
- If the navmesh is regenerated during runtime, this cache will be outdated
- Use `tostring(pos)` for debugging - this is how positions are compared internally
- For finding the nearest nav area to a position, use `navmesh.GetNearestNavArea()` instead
- For general nav area iteration, use `navmesh.GetAllNavAreas()` instead

**See Also:**
- `navmesh.GetNearestNavArea()` - Find the closest nav area to any position
- `navmesh.GetAllNavAreas()` - Get all nav areas in the map
- `CNavArea:GetCenter()` - Get the center position of a nav area
