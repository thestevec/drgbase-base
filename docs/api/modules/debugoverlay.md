# Debug Overlay Module
**File:** `lua/drgbase/modules/debugoverlay.lua`

The debug overlay module provides enhanced visualization functions for debugging projectile trajectories and physics calculations.

## Overview

This module extends Garry's Mod's debug overlay system with specialized functions for visualizing complex motion paths, particularly useful for debugging ballistic projectiles, grenades, and other physics-based movement.

---

## Functions

### debugoverlay.DrG_Trajectory(start, velocity, lifetime, color, ignoreZ, options)

**Realm:** 🟣 SHARED

**Parameters:**
- `start` (Vector) - Starting position of the trajectory
- `velocity` (Vector) - Initial velocity vector
- `lifetime` (number) - How long the debug overlay should remain visible in seconds
- `color` (Color or function) - Color of the trajectory line. Can be a Color object or a function that takes time `t` and returns a Color
- `ignoreZ` (boolean) - Whether to ignore Z-axis for rendering (makes line visible through walls)
- `options` (table, optional) - Configuration table with the following fields:
  - `ballistic` (boolean) - Whether this is a ballistic trajectory (affected by gravity)
  - `from` (number, default: 0) - Starting time for trajectory visualization
  - `to` (number, default: 10) - Ending time for trajectory visualization
  - `increments` (number, default: 0.01) - Time step between each line segment (smaller = smoother)
  - `colors` (function) - Custom color function for different trajectory segments
  - `height` (boolean, default: true) - If true, draws a vertical line from the highest point to the ground for ballistic trajectories

**Returns:**
- Nothing

**Description:**

Visualizes a projectile trajectory path using debug overlay lines. This function calculates and draws the predicted path of a projectile given its starting position and velocity. It's particularly useful for debugging grenade throws, ballistic weapons, or any physics-based projectile motion.

The function uses `Vector:DrG_TrajectoryInfo()` internally to compute the trajectory predictions. For ballistic trajectories, it can optionally show the apex (highest point) and draw a vertical line to the ground for better spatial understanding.

The trajectory is drawn as a series of connected line segments, where each segment represents a small time increment. The color can be dynamic, changing based on the time parameter, which is useful for showing trajectory evolution or different trajectory phases.

**Example:**
```lua
-- Simple ballistic trajectory visualization
local startPos = Entity(1):GetPos() + Vector(0, 0, 64)
local velocity = Entity(1):GetAimVector() * 1000 + Vector(0, 0, 200)

debugoverlay.DrG_Trajectory(startPos, velocity, 5, Color(255, 0, 0), false, {
    ballistic = true,
    from = 0,
    to = 3,
    increments = 0.05,
    height = true
})

-- Dynamic color based on time (fade from red to blue)
debugoverlay.DrG_Trajectory(startPos, velocity, 5, function(t)
    local ratio = t / 3
    return Color(255 * (1 - ratio), 0, 255 * ratio)
end, false, {
    ballistic = true,
    from = 0,
    to = 3,
    increments = 0.02
})

-- Non-ballistic trajectory (straight line)
debugoverlay.DrG_Trajectory(startPos, velocity, 5, Color(0, 255, 0), false, {
    ballistic = false,
    from = 0,
    to = 2,
    increments = 0.1
})

-- Grenade throw preview
hook.Add("Think", "GrenadePreview", function()
    local ply = LocalPlayer()
    if ply:GetActiveWeapon():GetClass() == "weapon_grenade" then
        local startPos = ply:GetShootPos()
        local aimVec = ply:GetAimVector()
        local throwVelocity = aimVec * 800 + Vector(0, 0, 100)

        debugoverlay.DrG_Trajectory(startPos, throwVelocity, 0.1, Color(255, 200, 0), false, {
            ballistic = true,
            from = 0,
            to = 5,
            increments = 0.05,
            height = true
        })
    end
end)
```

**Notes:**
- Requires `Vector:DrG_TrajectoryInfo()` to be available (part of DrGBase vector extensions)
- The `increments` value affects both performance and visual quality - smaller values are smoother but more expensive
- When using a color function, it receives the time parameter `t` ranging from `options.from` to `options.to`
- The height line for ballistic trajectories traces down to find the ground using a simple vertical trace
- Use `ignoreZ = true` to see the trajectory through walls and obstacles
- Lifetime should be set appropriately for your use case (0.1 for continuous updates in Think, longer for one-time visualization)

**See Also:**
- `Vector:DrG_TrajectoryInfo()` - Used internally for trajectory calculations
- `debugoverlay.Line()` - GMod function used for rendering
