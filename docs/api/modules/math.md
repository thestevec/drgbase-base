# Math Module
**File:** `lua/drgbase/modules/math.lua`

Mathematical utility functions for cyclic values and animations.

## Functions

### math.DrG_Cycle(min, max, rate, offset)
Generates a smoothly cycling value over time using a sine wave.

**Parameters:**
- `min` (number) - Minimum value (default: 0)
- `max` (number) - Maximum value (default: min+1)
- `rate` (number) - Cycle rate (cycles per second, default: 1)
- `offset` (number) - Time offset in seconds (default: 0)

**Returns:**
- (number) - Current cycled value between min and max

**Behavior:**
- Uses sine wave for smooth oscillation
- Automatically swaps min/max if min > max
- Cycles continuously based on CurTime()
- Perfect for periodic animations and effects

**Example:**
```lua
-- Pulsing glow effect (0.5 to 1.0, once per second)
local glowIntensity = math.DrG_Cycle(0.5, 1.0, 1)

-- Breathing effect (slow cycle, 0.3 cycles/second)
local scale = math.DrG_Cycle(0.9, 1.1, 0.3)

-- Moving up and down
local height = math.DrG_Cycle(0, 100, 0.5)
self:SetPos(basePos + Vector(0, 0, height))

-- Phase-shifted cycles for multiple entities
for i, ent in ipairs(entities) do
    -- Each entity offset by i seconds
    local offset = i
    local height = math.DrG_Cycle(0, 50, 1, offset)
    ent:SetPos(ent.basePos + Vector(0, 0, height))
end

-- Fast pulsing warning light (2 cycles per second)
local brightness = math.DrG_Cycle(0, 255, 2)
local color = Color(255, brightness, 0)
```

## Use Cases

### Animated Visual Effects
```lua
function ENT:Draw()
    -- Pulsing size
    local scale = math.DrG_Cycle(0.8, 1.2, 0.5)
    self:SetModelScale(scale)

    -- Pulsing alpha
    local alpha = math.DrG_Cycle(100, 255, 1)
    self:SetColor(Color(255, 255, 255, alpha))
end
```

### Bobbing Motion
```lua
function ENT:Think()
    if not self.spawnPos then
        self.spawnPos = self:GetPos()
    end

    -- Gentle bobbing motion
    local bob = math.DrG_Cycle(-5, 5, 0.5)
    self:SetPos(self.spawnPos + Vector(0, 0, bob))

    self:NextThink(CurTime())
    return true
end
```

### Rotating Objects
```lua
function ENT:Think()
    -- Smooth rotation cycle
    local rotationAngle = math.DrG_Cycle(0, 360, 0.2)
    self:SetAngles(Angle(0, rotationAngle, 0))

    self:NextThink(CurTime())
    return true
end
```

### Color Cycling
```lua
-- Rainbow effect cycling through hue
local hue = math.DrG_Cycle(0, 360, 0.3)
local color = HSVToColor(hue, 1, 1)
entity:SetColor(color)
```

### Wave Patterns
```lua
-- Create a wave of entities with phase offsets
for i = 1, 10 do
    local height = math.DrG_Cycle(0, 100, 1, i * 0.1)
    entities[i]:SetPos(basePos + Vector(0, i * 50, height))
end
```

## Technical Details

**Formula:**
```lua
((sin((CurTime() - offset) * 2π * rate) + 1) / 2) * (max - min) + min
```

- Sine wave oscillates between -1 and 1
- Normalized to 0-1 range
- Scaled to min-max range
- Rate controls frequency (cycles per second)
- Offset shifts the phase in time

**Performance:**
- Very lightweight (single sine calculation)
- No memory allocation
- Safe to call every frame
