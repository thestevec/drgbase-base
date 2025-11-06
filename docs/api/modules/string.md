# String Module
**File:** `lua/drgbase/modules/string.lua`

The string module extends Lua's built-in string library with useful formatting functions.

## Overview

This module provides string utilities commonly needed in game development, particularly for formatting numbers with leading zeros for display purposes.

---

## Functions

### string.DrG_Number(nb, size)

**Realm:** 🟣 SHARED

**Parameters:**
- `nb` (any) - The number (or value) to format. Will be converted to string with `tostring()`
- `size` (number) - The minimum length of the resulting string

**Returns:**
- `string` - The formatted string with leading zeros

**Description:**

Formats a number with leading zeros to ensure it reaches a minimum string length. This is commonly used for displaying counters, timers, IDs, or any numeric value where you want consistent width formatting.

The function first converts the input to a string using `tostring()`, then prepends "0" characters until the string reaches the desired size. If the string is already equal to or longer than the specified size, it returns unchanged.

This is particularly useful for:
- Displaying time in MM:SS format
- Formatting countdown timers
- Creating fixed-width numeric displays
- Generating zero-padded IDs or codes
- Creating consistent UI layouts with aligned numbers

**Example:**
```lua
-- Format single digits for time display
local minutes = 5
local seconds = 7
print(string.DrG_Number(minutes, 2) .. ":" .. string.DrG_Number(seconds, 2))
-- Output: "05:07"

-- Format timer countdown
local timeLeft = 3
print("Time: " .. string.DrG_Number(timeLeft, 3))
-- Output: "Time: 003"

-- Format entity IDs for display
for i = 1, 25 do
    local id = string.DrG_Number(i, 4)
    print("Entity_" .. id) -- Entity_0001, Entity_0002, etc.
end

-- Create a formatted scoreboard
local function FormatScore(score)
    return string.DrG_Number(score, 6)
end
print("Score: " .. FormatScore(152))
-- Output: "Score: 000152"

-- Format milliseconds for precise timing
local ms = 45
local timeDisplay = string.DrG_Number(ms, 3) .. "ms"
-- Output: "045ms"

-- Works with larger numbers too (doesn't truncate)
print(string.DrG_Number(12345, 3))
-- Output: "12345" (already longer than 3)

-- HUD timer display
hook.Add("HUDPaint", "ShowTimer", function()
    local timeLeft = math.floor(GetGlobalFloat("RoundTimeLeft"))
    local minutes = math.floor(timeLeft / 60)
    local seconds = timeLeft % 60

    local display = string.DrG_Number(minutes, 2) .. ":" .. string.DrG_Number(seconds, 2)
    draw.SimpleText(display, "DermaLarge", ScrW() / 2, 50, Color(255, 255, 255), TEXT_ALIGN_CENTER)
end)

-- Format kill counts
local kills = player.GetCount()
draw.SimpleText("Kills: " .. string.DrG_Number(kills, 3), "Default", x, y, color)

-- Generate sequential file names
for i = 1, 100 do
    local filename = "screenshot_" .. string.DrG_Number(i, 4) .. ".jpg"
    -- screenshot_0001.jpg, screenshot_0002.jpg, etc.
end
```

**Notes:**
- Input is converted to string with `tostring()`, so any type can be passed
- Does not truncate if the input is already longer than the specified size
- Adds leading zeros, not trailing zeros
- Size parameter must be a number
- Useful for creating fixed-width displays in UI
- Common sizes: 2 for double-digit displays, 3 for triple-digit, etc.
- Perfect for displaying time in MM:SS or HH:MM:SS format

**See Also:**
- `string.format()` - More flexible string formatting (can use "%02d" for similar results)
- `tostring()` - Used internally for conversion
