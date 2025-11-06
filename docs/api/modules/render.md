# Render Module
**File:** `lua/drgbase/modules/render.lua`

**Realm:** Client only

Rendering utility functions for drawing 2D sprites in 3D space.

## Functions

### render.DrG_DrawSprite(sprite, pos, size, options)
Draws a 2D sprite (texture) in 3D space that always faces towards the viewer.

**Parameters:**
- `sprite` (string) - Material path (e.g. `"sprites/light_glow02"`)
- `pos` (Vector) - 3D world position where sprite should be drawn
- `size` (number) - Size of the sprite in world units (clamped to positive, default: 100)
- `options` (table) - Drawing options:
  - `origin` (Vector) - Position to face towards (default: `EyePos()`)
  - `color` (Color) - Sprite color (default: white)
  - `lighting` (boolean) - Apply world lighting to sprite (default: false)
  - `rotation` (number) - Rotation angle in degrees (default: 0)

**Behavior:**
- Sprite always faces the origin position (billboard effect)
- Z-axis rotation is fixed (sprite stays upright)
- Returns silently if material doesn't exist
- With `lighting` enabled, sprite brightness matches world light at its position

**Example:**
```lua
-- Simple white glow
render.DrG_DrawSprite("sprites/light_glow02", Vector(0, 0, 100), 50, {})

-- Colored glow with rotation
render.DrG_DrawSprite("sprites/glow04", self:GetPos(), 80, {
    color = Color(255, 0, 0, 200),
    rotation = 45
})

-- Sprite affected by world lighting
render.DrG_DrawSprite("effects/spark", pos, 40, {
    color = Color(255, 255, 0),
    lighting = true
})

-- Sprite facing specific entity
render.DrG_DrawSprite("sprites/powerup_effects", pos, 60, {
    origin = player:GetPos(),
    color = Color(0, 255, 255),
    rotation = CurTime() * 90  -- Rotating
})
```

## Use Cases

### Glowing Effects
```lua
function ENT:Draw()
    self:DrawModel()

    -- Add glow effect above entity
    local glowPos = self:GetPos() + Vector(0, 0, 50)
    render.DrG_DrawSprite("sprites/light_glow02", glowPos, 80, {
        color = Color(0, 255, 255, 150)
    })
end
```

### Animated Sprites
```lua
function ENT:Draw()
    -- Pulsing size
    local size = 50 + math.sin(CurTime() * 3) * 20

    -- Rotating sprite
    local rotation = CurTime() * 100

    render.DrG_DrawSprite("sprites/glow04", self:GetPos(), size, {
        color = Color(255, 200, 0),
        rotation = rotation
    })
end
```

### Power-Up Visual
```lua
function ENT:Draw()
    self:DrawModel()

    -- Orbiting sprites
    for i = 1, 4 do
        local angle = (CurTime() + i * 90) * 50
        local offset = Vector(
            math.cos(math.rad(angle)) * 30,
            math.sin(math.rad(angle)) * 30,
            math.sin(CurTime() * 2) * 10
        )

        render.DrG_DrawSprite(
            "sprites/powerup_effects",
            self:GetPos() + offset,
            20,
            {
                color = Color(255, 255, 0, 200),
                lighting = true
            }
        )
    end
end
```

### Health Indicator
```lua
function ENT:Draw()
    self:DrawModel()

    if self:Health() < self:GetMaxHealth() * 0.3 then
        -- Low health warning sprite
        local pos = self:GetPos() + Vector(0, 0, self:OBBMaxs().z + 10)

        render.DrG_DrawSprite("sprites/redglow1", pos, 30, {
            color = Color(255, 0, 0, 100 + math.sin(CurTime() * 10) * 100),
            rotation = CurTime() * 45
        })
    end
end
```

### Lighting-Affected Sprites
```lua
-- Sprite that dims in dark areas
hook.Add("PostDrawTranslucentRenderables", "DrawLights", function()
    for _, light in ipairs(GetLights()) do
        render.DrG_DrawSprite("sprites/light_glow02", light.pos, light.size, {
            color = light.color,
            lighting = true  -- Dims in dark areas
        })
    end
end)
```

### Direction Indicators
```lua
-- Arrow pointing to objective
function DrawObjectiveMarker(pos)
    local plyPos = LocalPlayer():GetPos()
    local direction = (pos - plyPos):GetNormalized()
    local markerPos = plyPos + direction * 100 + Vector(0, 0, 50)

    render.DrG_DrawSprite("sprites/arrow", markerPos, 40, {
        color = Color(255, 255, 0),
        rotation = math.deg(math.atan2(direction.y, direction.x))
    })
end
```

## Implementation Details

**Billboard Effect:**
- Calculates facing direction from sprite position to origin (default: player eye)
- Z component is zeroed to keep sprite upright
- Sprite rotates around its vertical axis to face viewer

**Lighting Calculation:**
- Queries `render.GetLightColor(pos)` for world lighting at position
- Converts RGB to brightness percentage
- Multiplies sprite color by brightness

**Material Handling:**
- Uses `DrGBase.Material()` to cache materials
- Silently fails if material is error/missing
- Material caching prevents repeated file lookups

**Performance:**
- Single quad draw per call
- Minimal CPU overhead
- Material caching reduces I/O
- Lighting query is relatively expensive (use sparingly)

## Related Functions

- `render.DrawQuadEasy()` - Underlying draw function
- `render.SetMaterial()` - Sets active material
- `render.GetLightColor()` - Gets lighting at position
- `DrGBase.Material()` - Material caching system
