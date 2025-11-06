# Render Module
**File:** `lua/drgbase/modules/render.lua`

The render module provides enhanced rendering utilities for creating visual effects and 3D sprites.

## Overview

This module extends Garry's Mod's rendering capabilities with convenient functions for drawing billboarded sprites in 3D space. These sprites automatically face the camera and can be dynamically colored and lit.

**Important:** This module only loads on the client realm.

---

## Functions

### render.DrG_DrawSprite(sprite, pos, size, options)

**Realm:** 🔵 CLIENT

**Parameters:**
- `sprite` (string) - Path to the sprite material
- `pos` (Vector) - World position to draw the sprite
- `size` (number, optional, default: 100) - Size of the sprite in units
- `options` (table, optional) - Configuration table with the following fields:
  - `origin` (Vector, optional) - Point that the sprite should face (defaults to `EyePos()`)
  - `color` (Color, optional, default: white) - Color tint for the sprite
  - `lighting` (boolean, optional, default: false) - Whether to apply world lighting to the sprite
  - `rotation` (number, optional, default: 0) - Rotation angle in degrees

**Returns:**
- Nothing

**Description:**

Draws a billboarded sprite in 3D space that automatically faces the camera (or specified origin point). The sprite is rendered as a flat quad that always rotates to face the viewer, making it perfect for effects like glows, markers, icons, or particle-like visuals.

The function uses `DrGBase.Material()` to load the sprite material, which should be a valid material path. If the material doesn't exist or is an error material, the function returns without drawing anything.

The sprite is drawn using `render.DrawQuadEasy()` with an automatically calculated normal vector that makes it face the origin point. The Z component of the normal is set to 0, which keeps the sprite upright and prevents it from tilting up or down with the camera angle.

Size is automatically clamped to a minimum of 0, and if not provided defaults to 100 units.

The lighting option applies world lighting at the sprite's position, multiplying the color by the ambient light intensity to make the sprite blend naturally with the environment lighting. This creates more realistic integration with the game world.

**Example:**
```lua
-- Simple sprite at a position
local pos = Entity(1):GetPos() + Vector(0, 0, 50)
render.DrG_DrawSprite("sprites/glow04", pos, 64)

-- Colored sprite with custom size
render.DrG_DrawSprite("sprites/light_glow02", pos, 128, {
    color = Color(255, 0, 0, 200)
})

-- Rotating sprite
local rotation = CurTime() * 90 -- 90 degrees per second
render.DrG_DrawSprite("sprites/glow05", pos, 96, {
    color = Color(100, 200, 255),
    rotation = rotation
})

-- Lit sprite that responds to environment lighting
render.DrG_DrawSprite("sprites/glow04", pos, 80, {
    color = Color(255, 255, 0),
    lighting = true -- Will appear darker in shadows
})

-- Draw sprite facing a specific point instead of camera
local targetPos = Entity(2):GetPos()
render.DrG_DrawSprite("sprites/redglow1", pos, 100, {
    origin = targetPos, -- Faces this point instead of camera
    color = Color(255, 100, 100)
})

-- Combine multiple options
hook.Add("PostDrawTranslucentRenderables", "DrawMarker", function()
    local markerPos = Vector(0, 0, 100)
    local pulseSize = 50 + math.sin(CurTime() * 3) * 20
    local alpha = 150 + math.sin(CurTime() * 5) * 50

    render.DrG_DrawSprite("sprites/glow04", markerPos, pulseSize, {
        color = Color(0, 255, 255, alpha),
        rotation = CurTime() * 45,
        lighting = true
    })
end)

-- Draw objective markers
for _, objective in ipairs(GetActiveObjectives()) do
    render.DrG_DrawSprite("sprites/light_glow02", objective.pos, 72, {
        color = objective.completed and Color(0, 255, 0) or Color(255, 255, 0),
        lighting = false
    })
end
```

**Notes:**
- Only works on CLIENT realm
- Best used in rendering hooks like `PostDrawTranslucentRenderables` or `PostDrawOpaqueRenderables`
- The sprite automatically faces the camera (or specified origin)
- Size is clamped to minimum 0 to prevent negative sizes
- Material errors are silently handled - no sprite is drawn if material is invalid
- The lighting option uses `render.GetLightColor()` at the sprite position
- Rotation is applied after the billboard calculation
- Z component of normal is zeroed to keep sprites upright
- Use translucent sprites (with alpha) for best visual results

**See Also:**
- `render.DrawQuadEasy()` - Underlying GMod render function
- `DrGBase.Material()` - Material loading function used internally
- `render.GetLightColor()` - Used for lighting calculations
