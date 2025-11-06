# Util Module
**File:** `lua/drgbase/modules/util.lua`

The util module extends Garry's Mod's utility functions with enhanced tracing, damage management, and color manipulation.

## Overview

This module provides:
- **Debug Tracing**: Trace functions with optional debug visualization (controlled by ConVar)
- **Radial Tracing**: Perform multiple traces in a circular pattern
- **Damage Serialization**: Save and load damage information for network transmission or storage
- **Color Interpolation**: Smoothly blend between colors

---

## Tracing Functions

### util.DrG_TraceLine(data)

**Realm:** 🟣 SHARED

**Parameters:**
- `data` (table) - Standard trace data table (same as `util.TraceLine`)

**Returns:**
- `table` - Trace result (same format as `util.TraceLine`)

**Description:**

Performs a line trace with optional debug visualization. This is a drop-in replacement for `util.TraceLine()` that adds the ability to visualize traces for debugging purposes.

When the ConVar `drgbase_debug_traces` is greater than 0, the trace is visualized with colored lines:
- Green line: Unobstructed path (no hit)
- Red line: Path up to hit point
- White line: Remaining path after hit point

The visualization duration equals the ConVar value in seconds. Set to 0 to disable visualization.

**Example:**
```lua
-- Basic trace
local tr = util.DrG_TraceLine({
    start = ply:GetShootPos(),
    endpos = ply:GetShootPos() + ply:GetAimVector() * 10000,
    filter = ply
})

if tr.Hit then
    print("Hit:", tr.Entity)
end

-- Enable debug visualization
ConVarExists("drgbase_debug_traces") and GetConVar("drgbase_debug_traces"):SetFloat(5)

-- Now traces will show colored lines for 5 seconds
local tr = util.DrG_TraceLine({
    start = Vector(0, 0, 100),
    endpos = Vector(1000, 0, 100)
})

-- Use in weapon code for debugging
function SWEP:PrimaryAttack()
    local tr = util.DrG_TraceLine({
        start = self.Owner:GetShootPos(),
        endpos = self.Owner:GetShootPos() + self.Owner:GetAimVector() * 2000,
        filter = self.Owner,
        mask = MASK_SHOT
    })

    if tr.Hit then
        tr.Entity:TakeDamage(20, self.Owner, self)
    end
end
```

**Notes:**
- Identical to `util.TraceLine()` when debug visualization is disabled
- Debug lines use `debugoverlay.Line()`
- ConVar must be > 0 to show visualization
- Visualization duration = ConVar value in seconds
- Perfect for debugging line-of-sight, weapon traces, or AI vision

**See Also:**
- [util.DrG_TraceHull](#utilDrG_TraceHulldata) - For hull traces with debug
- [util.DrG_TraceLineRadial](#utilDrG_TraceLineRadialdist-precision-data) - For radial line traces
- `util.TraceLine()` - Standard GMod function

---

### util.DrG_TraceHull(data)

**Realm:** 🟣 SHARED

**Parameters:**
- `data` (table) - Standard hull trace data table (same as `util.TraceHull`)

**Returns:**
- `table` - Trace result (same format as `util.TraceHull`)

**Description:**

Performs a hull trace (swept box) with optional debug visualization. This is a drop-in replacement for `util.TraceHull()` that adds debugging capabilities.

When the ConVar `drgbase_debug_traces` is greater than 0, the trace is visualized with:
- White line showing the path
- Box overlay at the hit position showing the hull size
- Green box for successful (no collision) traces
- Red box for collision traces (with transparent alpha)

**Example:**
```lua
-- Check if a player-sized volume can fit
local tr = util.DrG_TraceHull({
    start = Vector(0, 0, 100),
    endpos = Vector(0, 0, 0),
    mins = Vector(-16, -16, 0),
    maxs = Vector(16, 16, 72),
    filter = player
})

if not tr.Hit then
    print("Space is clear for player")
end

-- Check entity movement path
local tr = util.DrG_TraceHull({
    start = entity:GetPos(),
    endpos = entity:GetPos() + moveDirection * 100,
    mins = entity:OBBMins(),
    maxs = entity:OBBMaxs(),
    filter = entity
})

-- Physics-based movement checking
function ENT:CanMoveTo(targetPos)
    local tr = util.DrG_TraceHull({
        start = self:GetPos(),
        endpos = targetPos,
        mins = self:OBBMins(),
        maxs = self:OBBMaxs(),
        filter = self,
        mask = MASK_NPCSOLID
    })

    return not tr.Hit or tr.Fraction > 0.99
end
```

**Notes:**
- Identical to `util.TraceHull()` when debug visualization is disabled
- Shows hull volume at impact point for easier debugging
- ConVar must be > 0 to show visualization
- Useful for checking if spaces can fit entities
- Essential for movement validation and pathfinding

**See Also:**
- [util.DrG_TraceLine](#utilDrG_TraceLinedata) - For line traces with debug
- [util.DrG_TraceHullRadial](#utilDrG_TraceHullRadialdist-precision-data) - For radial hull traces
- `util.TraceHull()` - Standard GMod function

---

### util.DrG_TraceLineRadial(dist, precision, data)

**Realm:** 🟣 SHARED

**Parameters:**
- `dist` (number) - Distance of each trace ray
- `precision` (number) - Number of traces to perform (evenly distributed in 360 degrees)
- `data` (table) - Base trace data table. The `endpos` field will be overwritten for each trace

**Returns:**
- `table` - Array of trace results, sorted by distance (closest hit first)

**Description:**

Performs multiple line traces in a radial pattern around a central point, useful for detecting objects in all directions. The traces are evenly distributed in a 360-degree horizontal circle.

Each trace rotates around the Y-axis (horizontal plane) based on the precision. Higher precision means more traces and better coverage but higher performance cost.

The results are sorted by distance from the start point, with the closest hit first. This makes it easy to find the nearest obstacle or entity in any direction.

**Example:**
```lua
-- Detect nearby enemies in all directions
local traces = util.DrG_TraceLineRadial(500, 12, {
    start = self:GetPos() + Vector(0, 0, 50),
    filter = self,
    mask = MASK_SHOT
})

-- Find closest enemy
for _, tr in ipairs(traces) do
    if IsValid(tr.Entity) and tr.Entity:IsNPC() then
        self:SetEnemy(tr.Entity)
        break
    end
end

-- Create a proximity sensor
local traces = util.DrG_TraceLineRadial(200, 8, {
    start = sensor:GetPos(),
    filter = sensor
})

local closestDist = math.huge
for _, tr in ipairs(traces) do
    if tr.Hit then
        closestDist = math.min(closestDist, tr.Fraction * 200)
    end
end

print("Closest obstacle:", closestDist, "units away")

-- AI vision in all directions
function ENT:Think()
    local traces = util.DrG_TraceLineRadial(1000, 16, {
        start = self:EyePos(),
        filter = self,
        mask = MASK_VISIBLE
    })

    for _, tr in ipairs(traces) do
        if IsValid(tr.Entity) and tr.Entity:IsPlayer() then
            self:AlertToPlayer(tr.Entity)
        end
    end
end

-- Create a "radar" effect
local traces = util.DrG_TraceLineRadial(2000, 32, {
    start = LocalPlayer():GetPos(),
    filter = LocalPlayer()
})

-- Process each direction for rendering
for i, tr in ipairs(traces) do
    local angle = i * (360 / 32)
    -- Draw radar blip based on tr.Fraction
end
```

**Notes:**
- Traces are evenly distributed in a full 360-degree circle
- Results are sorted by distance (closest first)
- All traces use the same `data` table (except `endpos`)
- Higher precision = more accurate but more expensive
- Traces are performed on the horizontal plane (rotates around Y-axis only)
- Typical precision values: 8-12 for rough checks, 16-32 for detailed scans
- Supports debug visualization via `drgbase_debug_traces` ConVar

**See Also:**
- [util.DrG_TraceLine](#utilDrG_TraceLinedata) - Single line trace
- [util.DrG_TraceHullRadial](#utilDrG_TraceHullRadialdist-precision-data) - Radial hull traces

---

### util.DrG_TraceHullRadial(dist, precision, data)

**Realm:** 🟣 SHARED

**Parameters:**
- `dist` (number) - Distance of each trace ray
- `precision` (number) - Number of traces to perform (evenly distributed in 360 degrees)
- `data` (table) - Base hull trace data table. The `endpos` field will be overwritten for each trace. Must include `mins` and `maxs`

**Returns:**
- `table` - Array of trace results, sorted by distance (closest hit first)

**Description:**

Performs multiple hull traces in a radial pattern around a central point. This is similar to `util.DrG_TraceLineRadial()` but uses hull traces instead of line traces, making it better for checking if an entity or volume can fit in various directions.

The traces sweep a box volume (defined by `mins` and `maxs`) in all directions around the start point. This is perfect for:
- Finding valid movement directions for entities
- Checking if an entity can fit in nearby spaces
- Creating area-aware AI that avoids tight spaces
- Physics-based navigation

Results are sorted by distance with closest hits first.

**Example:**
```lua
-- Find valid movement directions
local traces = util.DrG_TraceHullRadial(100, 8, {
    start = self:GetPos(),
    mins = self:OBBMins(),
    maxs = self:OBBMaxs(),
    filter = self,
    mask = MASK_NPCSOLID
})

-- Find first clear direction
for _, tr in ipairs(traces) do
    if not tr.Hit then
        local direction = (tr.HitPos - tr.StartPos):GetNormalized()
        self:SetVelocity(direction * 200)
        break
    end
end

-- Check which directions have space
local traces = util.DrG_TraceHullRadial(50, 12, {
    start = entity:GetPos(),
    mins = Vector(-16, -16, 0),
    maxs = Vector(16, 16, 72),
    filter = entity
})

local openDirections = {}
for i, tr in ipairs(traces) do
    if not tr.Hit then
        local angle = i * (360 / 12)
        table.insert(openDirections, angle)
    end
end

-- Avoid obstacles by checking surrounding space
function ENT:AvoidObstacles()
    local traces = util.DrG_TraceHullRadial(200, 16, {
        start = self:GetPos() + Vector(0, 0, 32),
        mins = Vector(-20, -20, -20),
        maxs = Vector(20, 20, 20),
        filter = self
    })

    -- Find the direction with most free space
    local bestDir = nil
    local bestDist = 0

    for i, tr in ipairs(traces) do
        local dist = tr.Fraction * 200
        if dist > bestDist then
            bestDist = dist
            local angle = i * (360 / 16)
            bestDir = Vector(math.cos(math.rad(angle)), math.sin(math.rad(angle)), 0)
        end
    end

    if bestDir then
        self:SetMoveDirection(bestDir)
    end
end

-- Check if NPC can fit through doorways
local traces = util.DrG_TraceHullRadial(100, 8, {
    start = npc:GetPos(),
    mins = npc:OBBMins() * 0.9,
    maxs = npc:OBBMaxs() * 0.9,
    filter = npc,
    mask = MASK_NPCSOLID
})

local canFit = false
for _, tr in ipairs(traces) do
    if tr.Fraction > 0.8 then -- Can move 80% of the distance
        canFit = true
        break
    end
end
```

**Notes:**
- Uses hull traces (swept boxes) instead of line traces
- Results are sorted by distance (closest first)
- Requires `mins` and `maxs` in the data table
- More expensive than `util.DrG_TraceLineRadial()` but more accurate for entity movement
- Perfect for navigation and pathfinding
- Supports debug visualization via `drgbase_debug_traces` ConVar
- Traces rotate around Y-axis (horizontal plane)

**See Also:**
- [util.DrG_TraceHull](#utilDrG_TraceHulldata) - Single hull trace
- [util.DrG_TraceLineRadial](#utilDrG_TraceLineRadialdist-precision-data) - Radial line traces

---

## Damage Functions

### util.DrG_SaveDmg(dmg)

**Realm:** 🟣 SHARED

**Parameters:**
- `dmg` (CTakeDamageInfo) - Damage info object to save

**Returns:**
- `table` - Table containing all damage information

**Description:**

Converts a CTakeDamageInfo object into a Lua table that can be stored, transmitted over network, or saved to disk. This is essential for damage replication, delayed damage application, or network transmission of damage information.

The saved table contains all standard damage fields:
- `ammoType` - Type of ammunition
- `attacker` - Entity that caused the damage
- `baseDamage` - Base damage amount
- `damage` - Actual damage amount
- `damageBonus` - Bonus damage
- `damageCustom` - Custom damage identifier
- `damageForce` - Force vector applied
- `damagePosition` - World position of damage
- `damageType` - Damage type flags (DMG_BULLET, etc.)
- `inflictor` - Entity that inflicted damage (weapon)
- `maxDamage` - Maximum damage
- `reportedPosition` - Reported position

**Example:**
```lua
-- Save damage for delayed application
hook.Add("EntityTakeDamage", "SaveDamage", function(target, dmg)
    if target.DelayDamage then
        local savedDmg = util.DrG_SaveDmg(dmg)
        timer.Simple(2, function()
            if IsValid(target) then
                local newDmg = util.DrG_LoadDmg(savedDmg)
                target:TakeDamageInfo(newDmg)
            end
        end)
        return true -- Cancel original damage
    end
end)

-- Network damage information
-- SERVER
util.AddNetworkString("ReplicateDamage")
hook.Add("EntityTakeDamage", "NetworkDamage", function(target, dmg)
    if ShouldReplicateDamage(target) then
        local savedDmg = util.DrG_SaveDmg(dmg)
        net.Start("ReplicateDamage")
        net.WriteEntity(target)
        net.WriteTable(savedDmg)
        net.Broadcast()
    end
end)

-- Store damage history
local damageHistory = {}
hook.Add("EntityTakeDamage", "DamageHistory", function(target, dmg)
    if not damageHistory[target] then
        damageHistory[target] = {}
    end
    table.insert(damageHistory[target], {
        time = CurTime(),
        damage = util.DrG_SaveDmg(dmg)
    })
end)

-- Replay damage after respawn
function RespawnWithDamageReplay(ply, savedDamages)
    ply:Spawn()
    timer.Simple(0.1, function()
        for _, saved in ipairs(savedDamages) do
            local dmg = util.DrG_LoadDmg(saved)
            ply:TakeDamageInfo(dmg)
        end
    end)
end
```

**Notes:**
- Creates a serializable copy of damage information
- Entity references (attacker, inflictor) are stored directly (not serialized)
- Invalid entities may cause issues when loading, always validate with `IsValid()`
- Use with `util.DrG_LoadDmg()` to restore the damage info
- Perfect for networking, storage, or delayed damage application

**See Also:**
- [util.DrG_LoadDmg](#utilDrG_LoadDmgdata) - Restore damage from saved table

---

### util.DrG_LoadDmg(data)

**Realm:** 🟣 SHARED

**Parameters:**
- `data` (table) - Table containing damage information (from `util.DrG_SaveDmg`)

**Returns:**
- `CTakeDamageInfo` - Reconstructed damage info object

**Description:**

Converts a saved damage table back into a CTakeDamageInfo object that can be used with `Entity:TakeDamageInfo()`. This reconstructs damage information from a table created by `util.DrG_SaveDmg()`.

The function validates entity references (attacker and inflictor) before setting them, skipping invalid entities. If the input is not a table, returns an empty DamageInfo object.

**Example:**
```lua
-- Restore saved damage
local savedDmg = util.DrG_SaveDmg(originalDmg)
timer.Simple(5, function()
    local dmg = util.DrG_LoadDmg(savedDmg)
    entity:TakeDamageInfo(dmg)
end)

-- Network damage from server to client
-- SERVER
net.Start("ShowDamageEffect")
net.WriteTable(util.DrG_SaveDmg(dmg))
net.Send(ply)

-- CLIENT
net.Receive("ShowDamageEffect", function()
    local data = net.ReadTable()
    local dmg = util.DrG_LoadDmg(data)
    -- Use damage info for effects
    local force = dmg:GetDamageForce()
    local pos = dmg:GetDamagePosition()
    CreateBloodEffect(pos, force)
end)

-- Apply stored damage
function ApplyStoredDamage(target, damageTable)
    local dmg = util.DrG_LoadDmg(damageTable)
    if IsValid(target) then
        target:TakeDamageInfo(dmg)
    end
end

-- Damage over time from saved template
local templateDamage = util.DrG_SaveDmg(baseDmg)
timer.DrG_Loop(1, function(target, template)
    if not IsValid(target) or target:Health() <= 0 then
        return false
    end

    local dmg = util.DrG_LoadDmg(template)
    dmg:SetDamage(10) -- Modify damage amount
    target:TakeDamageInfo(dmg)
end, entity, templateDamage)
```

**Notes:**
- Returns empty DamageInfo if data is not a table
- Validates entities before setting (skips invalid attackers/inflictors)
- Entities may become invalid between save and load - always check
- All damage properties are restored from the table
- Safe to use with networked or stored damage data

**See Also:**
- [util.DrG_SaveDmg](#utilDrG_SaveDmgdmg) - Save damage to table
- `Entity:TakeDamageInfo()` - Apply damage to entity

---

## Color Functions

### util.DrG_MergeColors(ratio, max, low)

**Realm:** 🟣 SHARED

**Parameters:**
- `ratio` (number) - Interpolation ratio between 0 and 1 (clamped automatically)
- `max` (Color) - Color at ratio = 1
- `low` (Color) - Color at ratio = 0

**Returns:**
- `Color` - Interpolated color

**Description:**

Linearly interpolates between two colors based on a ratio. When ratio is 0, returns the low color. When ratio is 1, returns the max color. Values between 0 and 1 blend smoothly between the two colors.

This is perfect for:
- Health bars that change color based on health percentage
- Temperature or danger indicators
- Smooth color transitions
- Visual feedback for progress or intensity

The ratio is automatically clamped between 0 and 1, so you don't need to worry about values outside this range.

**Example:**
```lua
-- Health bar color
local healthRatio = ply:Health() / ply:GetMaxHealth()
local color = util.DrG_MergeColors(healthRatio,
    Color(0, 255, 0),   -- Green at full health
    Color(255, 0, 0)    -- Red at low health
)
draw.RoundedBox(0, x, y, w, h, color)

-- Temperature indicator
local temp = GetTemperature() -- 0 to 100
local tempRatio = temp / 100
local tempColor = util.DrG_MergeColors(tempRatio,
    Color(255, 0, 0),    -- Hot (red)
    Color(0, 100, 255)   -- Cold (blue)
)

-- Charge indicator
local chargePercent = weapon:GetCharge() / weapon:GetMaxCharge()
local chargeColor = util.DrG_MergeColors(chargePercent,
    Color(255, 255, 0),  -- Fully charged (yellow)
    Color(100, 100, 100) -- Empty (gray)
)

-- Smooth threat level coloring
local threatLevel = CalculateThreatLevel() -- Returns 0.0 to 1.0
local threatColor = util.DrG_MergeColors(threatLevel,
    Color(255, 0, 0, 255),    -- Maximum threat
    Color(255, 255, 0, 100)   -- Low threat
)

-- Blend colors over time
local timeRatio = (CurTime() % 2) / 2 -- 0 to 1 over 2 seconds
local color = util.DrG_MergeColors(timeRatio,
    Color(255, 0, 255),  -- Purple
    Color(0, 255, 255)   -- Cyan
)

-- Distance-based color
local dist = localPlayer:GetPos():Distance(target:GetPos())
local distRatio = math.Clamp(dist / 1000, 0, 1)
local distColor = util.DrG_MergeColors(1 - distRatio, -- Inverted
    Color(0, 255, 0),    -- Close (green)
    Color(255, 0, 0)     -- Far (red)
)

-- Speed indicator
local speed = entity:GetVelocity():Length()
local maxSpeed = 500
local speedRatio = speed / maxSpeed
local speedColor = util.DrG_MergeColors(speedRatio,
    Color(255, 100, 0),  -- Fast (orange)
    Color(200, 200, 200) -- Slow (gray)
)
```

**Notes:**
- Ratio is automatically clamped between 0 and 1
- Interpolates all RGBA channels (including alpha)
- Linear interpolation (not gamma-corrected)
- No need to clamp input ratio manually
- Perfect for dynamic UI coloring
- Can create smooth gradients by calling repeatedly with changing ratios

**See Also:**
- `Color()` - GMod color construction
- `ColorAlpha()` - For just changing alpha
