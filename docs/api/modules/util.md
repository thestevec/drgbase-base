# Util Module
**File:** `lua/drgbase/modules/util.lua`

Utility functions for tracing, damage handling, and color operations.

## Trace Functions

### util.DrG_TraceLine(data)
Performs a line trace with optional debug visualization.

**Parameters:**
- `data` (table) - Standard TraceLine data table
  - `start` (Vector) - Starting position
  - `endpos` (Vector) - Ending position
  - `filter` - Entity or table of entities to filter
  - `mask` (number) - Collision mask (optional)

**Returns:**
- (table) - Trace result table

**Debug Visualization:**
Uses the `drgbase_debug_traces` console variable. When set to a value > 0, traces are visualized for that many seconds:
- Red line: Hit something
- Green line: No hit
- White line: Remaining distance to endpos

**Example:**
```lua
local tr = util.DrG_TraceLine({
    start = self:GetPos(),
    endpos = self:GetPos() + self:GetForward() * 1000,
    filter = self
})
if tr.Hit then
    print("Hit entity:", tr.Entity)
end
```

### util.DrG_TraceHull(data)
Performs a hull trace with optional debug visualization.

**Parameters:**
- `data` (table) - Standard TraceHull data table
  - `start` (Vector) - Starting position
  - `endpos` (Vector) - Ending position
  - `mins` (Vector) - Hull minimum bounds
  - `maxs` (Vector) - Hull maximum bounds
  - `filter` - Entity or table of entities to filter
  - `mask` (number) - Collision mask (optional)

**Returns:**
- (table) - Trace result table

**Debug Visualization:**
Uses the `drgbase_debug_traces` console variable to visualize hull traces with a box overlay.

### util.DrG_TraceLineRadial(dist, precision, data)
Performs multiple line traces in a radial pattern around the starting point.

**Parameters:**
- `dist` (number) - Distance to trace
- `precision` (number) - Number of traces to perform (evenly distributed around 360°)
- `data` (table) - Base trace data (endpos will be overwritten)

**Returns:**
- (table) - Array of trace results, sorted by distance (closest first)

**Example:**
```lua
-- Check for obstacles in all directions
local traces = util.DrG_TraceLineRadial(500, 8, {
    start = self:GetPos(),
    filter = self
})
-- First trace in array is the closest hit
if traces[1].Hit then
    print("Closest obstacle:", traces[1].Entity)
end
```

### util.DrG_TraceHullRadial(dist, precision, data)
Performs multiple hull traces in a radial pattern around the starting point.

**Parameters:**
- `dist` (number) - Distance to trace
- `precision` (number) - Number of traces to perform (evenly distributed around 360°)
- `data` (table) - Base trace data including mins/maxs (endpos will be overwritten)

**Returns:**
- (table) - Array of trace results, sorted by distance (closest first)

## Damage Functions

### util.DrG_SaveDmg(dmg)
Converts a DamageInfo object into a serializable table.

**Parameters:**
- `dmg` (CTakeDamageInfo) - DamageInfo object to save

**Returns:**
- (table) - Table containing all damage information:
  - `ammoType` - Ammo type index
  - `attacker` - Attacking entity
  - `baseDamage` - Base damage value
  - `damage` - Final damage value
  - `damageBonus` - Bonus damage
  - `damageCustom` - Custom damage type
  - `damageForce` - Damage force vector
  - `damagePosition` - Position where damage occurred
  - `damageType` - Damage type flags
  - `inflictor` - Weapon/inflictor entity
  - `maxDamage` - Maximum damage
  - `reportedPosition` - Reported damage position

**Use Case:**
Useful for storing damage information to apply later or send over the network.

### util.DrG_LoadDmg(data)
Converts a saved damage table back into a DamageInfo object.

**Parameters:**
- `data` (table) - Damage data table from `util.DrG_SaveDmg()`

**Returns:**
- (CTakeDamageInfo) - Reconstructed DamageInfo object

**Example:**
```lua
-- Save damage for later
local savedDamage = util.DrG_SaveDmg(dmginfo)

-- Apply it later
timer.Simple(1, function()
    local dmg = util.DrG_LoadDmg(savedDamage)
    target:TakeDamageInfo(dmg)
end)
```

## Color Functions

### util.DrG_MergeColors(ratio, max, low)
Interpolates between two colors based on a ratio.

**Parameters:**
- `ratio` (number) - Interpolation ratio (0-1, clamped)
  - `0` = Returns low color
  - `1` = Returns max color
  - `0.5` = Returns color halfway between
- `max` (Color) - Color when ratio is 1
- `low` (Color) - Color when ratio is 0

**Returns:**
- (Color) - Interpolated color

**Example:**
```lua
-- Health-based color
local healthRatio = self:Health() / self:GetMaxHealth()
local color = util.DrG_MergeColors(
    healthRatio,
    Color(0, 255, 0),  -- Full health = green
    Color(255, 0, 0)   -- No health = red
)
```

## Console Variables

### drgbase_debug_traces
**Type:** Number (default: 0)

Controls trace debug visualization:
- `0` - Disabled
- `> 0` - Show traces for specified number of seconds

**Example:**
```
drgbase_debug_traces 5  -- Show traces for 5 seconds
```
