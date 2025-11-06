# Math Module
**File:** `lua/drgbase/modules/math.lua`

The math module extends Lua's built-in math library with useful functions for game development, particularly for creating smooth cyclic animations and effects.

## Overview

This module provides mathematical utilities that are commonly needed in game development, such as creating smooth oscillating values for animations, pulsing effects, and other time-based variations.

---

## Functions

### math.DrG_Cycle(min, max, rate, offset)

**Realm:** 🟣 SHARED

**Parameters:**
- `min` (number, optional, default: 0) - Minimum value of the cycle
- `max` (number, optional, default: min + 1) - Maximum value of the cycle
- `rate` (number, optional, default: 1) - Cycles per second (frequency)
- `offset` (number, optional, default: 0) - Time offset in seconds to shift the cycle phase

**Returns:**
- `number` - Current value in the cycle, oscillating smoothly between min and max

**Description:**

Creates a smoothly cycling value based on a sine wave that oscillates between a minimum and maximum value over time. This is extremely useful for creating pulsing effects, breathing animations, smooth color transitions, or any periodic variation that needs to be continuous and smooth.

The function uses `CurTime()` internally, so it automatically animates over time without requiring manual time tracking. The cycle follows a sine wave pattern, providing smooth acceleration and deceleration at the extremes rather than linear interpolation.

The rate parameter controls how many complete cycles occur per second. A rate of 1.0 means one complete cycle (min to max and back to min) per second. Higher rates create faster oscillations.

The offset parameter allows you to shift the phase of the cycle, which is useful for creating multiple synchronized but offset animations, or for starting the cycle at a specific point in its wave.

If `min` is greater than `max`, the function automatically swaps them to ensure correct behavior.

**Example:**
```lua
-- Pulsing alpha value for a fading effect
local alpha = math.DrG_Cycle(50, 255, 0.5) -- Slow pulse
surface.SetDrawColor(255, 255, 255, alpha)

-- Smooth color transition
local colorValue = math.DrG_Cycle(0, 255, 1)
local pulseColor = Color(colorValue, 100, 255 - colorValue)

-- Breathing scale effect for an entity
ENT.Think = function(self)
    local scale = math.DrG_Cycle(0.8, 1.2, 0.3)
    self:SetModelScale(scale, 0)
    self:NextThink(CurTime())
    return true
end

-- Multiple offset animations for a sequence effect
for i = 1, 5 do
    local offset = i * 0.2 -- 0.2 second offset between each
    local height = math.DrG_Cycle(0, 50, 1, offset)
    -- Use height for staggered animation
end

-- Smooth size pulsing for a glowing effect
local size = math.DrG_Cycle(32, 48, 2) -- Fast 2Hz pulse
render.DrawSprite(pos, size, size, color)

-- Oscillating movement
local oscillation = math.DrG_Cycle(-100, 100, 0.5)
local newPos = basePos + Vector(oscillation, 0, 0)

-- Create a "heartbeat" effect with specific timing
local heartbeat = math.DrG_Cycle(1, 1.5, 1.2)
entity:SetModelScale(heartbeat, 0)
```

**Notes:**
- The function uses sine wave interpolation, not linear, so the rate of change is slower at the extremes and faster in the middle
- Based on `CurTime()`, so it's affected by server/client time
- Perfect for any smooth, continuous, repeating animation or effect
- The cycle is continuous and never jumps or resets
- Using the same parameters in different locations will give synchronized values
- Use different offsets to create phase-shifted synchronized effects
- For non-smooth cycling (linear or stepped), you may need to implement your own solution

**See Also:**
- `math.sin()` - The underlying function used for the cycle
- `CurTime()` - The time function used for automatic animation
