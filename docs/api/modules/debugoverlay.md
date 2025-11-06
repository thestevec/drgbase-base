# Debug Overlay Module
**File:** `lua/drgbase/modules/debugoverlay.lua`

Debug visualization functions for projectile trajectories.

## Functions

### debugoverlay.DrG_Trajectory(start, velocity, lifetime, color, ignoreZ, options)
Visualizes a projectile trajectory path with debug overlays.

**Parameters:**
- `start` (Vector) - Starting position of the trajectory
- `velocity` (Vector) - Initial velocity vector
- `lifetime` (number) - How long (in seconds) the overlay should be visible
- `color` (Color or function) - Line color
  - If Color: All lines use this color
  - If function: Called with `time` parameter, should return Color for that point
- `ignoreZ` (boolean) - Whether to ignore Z-axis depth testing
- `options` (table) - Trajectory visualization options:
  - `ballistic` (boolean) - Whether trajectory is affected by gravity (default: false)
  - `from` (number) - Start time of trajectory to visualize (default: 0)
  - `to` (number) - End time of trajectory to visualize (default: 10)
  - `increments` (number) - Time step between line segments (default: 0.01)
  - `colors` (function) - Color function for custom coloring
  - `height` (boolean) - Show height indicator line for ballistic trajectories (default: true)

**Behavior:**
- Draws a series of connected line segments following the trajectory
- Uses Vector:DrG_TrajectoryInfo() to predict trajectory (see [Vector Meta Extensions](../meta/vector.md))
- For ballistic trajectories with `height` enabled, draws a vertical line from the highest point to the ground

**Example:**
```lua
-- Simple straight-line trajectory
debugoverlay.DrG_Trajectory(
    self:GetPos(),
    self:GetForward() * 1000,
    5,  -- 5 second lifetime
    Color(255, 0, 0),
    false,
    {}
)

-- Ballistic trajectory with gravity
debugoverlay.DrG_Trajectory(
    self:GetPos(),
    self:GetAngles():Forward() * 800 + Vector(0, 0, 400),
    10,
    Color(0, 255, 0),
    false,
    {
        ballistic = true,  -- Affected by gravity
        from = 0,
        to = 5,
        height = true  -- Show height indicator
    }
)

-- Color gradient based on time
debugoverlay.DrG_Trajectory(
    startPos,
    velocity,
    5,
    function(t)
        -- Fade from red to blue over time
        local ratio = t / 5
        return Color(255 * (1-ratio), 0, 255 * ratio)
    end,
    false,
    {
        ballistic = true,
        to = 5
    }
)

-- Show where a projectile will be at specific time range
debugoverlay.DrG_Trajectory(
    self:GetPos(),
    self:GetAimVector() * 1500,
    3,
    Color(255, 255, 0),
    false,
    {
        ballistic = true,
        from = 1,  -- Start visualization at 1 second
        to = 3,    -- End at 3 seconds
        increments = 0.05  -- Larger segments for performance
    }
)
```

## Use Cases

### Debugging Projectile Paths
```lua
-- In projectile initialization
function ENT:Initialize()
    if GetConVar("developer"):GetBool() then
        debugoverlay.DrG_Trajectory(
            self:GetPos(),
            self:GetVelocity(),
            10,
            Color(255, 0, 0),
            false,
            {
                ballistic = self.Ballistic,
                to = 5
            }
        )
    end
end
```

### Visualizing Attack Ranges
```lua
-- Show where an NPC's projectile will go
function ENT:VisualizeAttack(target)
    local velocity = (target:GetPos() - self:GetPos()):GetNormalized() * 1000

    debugoverlay.DrG_Trajectory(
        self:GetShootPos(),
        velocity,
        5,
        Color(255, 100, 0),
        false,
        {
            ballistic = true,
            height = true
        }
    )
end
```

### Arc Trajectory Preview
```lua
-- Show grenade arc with height indicator
concommand.Add("preview_throw", function(ply)
    local startPos = ply:GetShootPos()
    local velocity = ply:GetAimVector() * 800 + Vector(0, 0, 300)

    debugoverlay.DrG_Trajectory(
        startPos,
        velocity,
        10,
        Color(0, 255, 255),
        false,
        {
            ballistic = true,
            to = 3,
            height = true
        }
    )
end)
```

## Related Functions

- [Vector:DrG_TrajectoryInfo()](../meta/vector.md) - Generates trajectory prediction data
- Standard debugoverlay functions - Base Garry's Mod debug overlay system
