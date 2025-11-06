# String Module
**File:** `lua/drgbase/modules/string.lua`

String utility functions for number formatting.

## Functions

### string.DrG_Number(nb, size)
Pads a number with leading zeros to reach a minimum length.

**Parameters:**
- `nb` (any) - Number to format (converted to string)
- `size` (number) - Minimum total length

**Returns:**
- (string) - Zero-padded number string

**Behavior:**
- Converts input to string
- Prepends zeros until string reaches specified size
- If string is already longer than size, returns unchanged

**Example:**
```lua
-- Basic padding
print(string.DrG_Number(5, 3))        -- "005"
print(string.DrG_Number(42, 3))       -- "042"
print(string.DrG_Number(100, 3))      -- "100"
print(string.DrG_Number(1234, 3))     -- "1234" (already longer)

-- Timer displays
local minutes = 5
local seconds = 7
local display = minutes .. ":" .. string.DrG_Number(seconds, 2)
print(display)  -- "5:07"

-- File naming
for i = 1, 100 do
    local filename = "file_" .. string.DrG_Number(i, 4) .. ".txt"
    print(filename)  -- "file_0001.txt", "file_0002.txt", etc.
end

-- Entity IDs
local id = 42
local formattedID = "NPC_" .. string.DrG_Number(id, 5)
print(formattedID)  -- "NPC_00042"
```

## Use Cases

### Digital Clock Display
```lua
function FormatTime(hours, minutes, seconds)
    return string.DrG_Number(hours, 2) .. ":" ..
           string.DrG_Number(minutes, 2) .. ":" ..
           string.DrG_Number(seconds, 2)
end

print(FormatTime(9, 5, 3))   -- "09:05:03"
print(FormatTime(14, 30, 45))  -- "14:30:45"
```

### Countdown Timer
```lua
function ENT:DrawTimer()
    local timeLeft = math.max(0, self.EndTime - CurTime())
    local minutes = math.floor(timeLeft / 60)
    local seconds = math.floor(timeLeft % 60)

    local display = string.DrG_Number(minutes, 2) .. ":" ..
                    string.DrG_Number(seconds, 2)

    draw.SimpleText(display, "HudHintTextLarge", x, y, color)
end
```

### Sequential File Generation
```lua
-- Generate numbered files with consistent naming
for i = 1, 1000 do
    local path = "data/saves/save_" .. string.DrG_Number(i, 4) .. ".txt"
    file.Write(path, data)
end
-- Creates: save_0001.txt, save_0002.txt, ..., save_1000.txt
```

### Entity Naming System
```lua
function ENT:Initialize()
    -- Generate unique padded ID
    local id = self:EntIndex()
    self.DisplayName = "UNIT-" .. string.DrG_Number(id, 6)
end
-- Creates: UNIT-000001, UNIT-000042, etc.
```

### Score Display
```lua
-- Scoreboard with padded scores
function DrawScoreboard()
    for i, ply in ipairs(player.GetAll()) do
        local score = ply:Frags()
        local scoreText = string.DrG_Number(score, 5)
        draw.Text({
            text = scoreText,
            pos = {x, y + i * 20},
            font = "ScoreFont"
        })
    end
end
```

### Version Numbers
```lua
-- Consistent version formatting
function GetVersionString(major, minor, patch)
    return string.DrG_Number(major, 2) .. "." ..
           string.DrG_Number(minor, 2) .. "." ..
           string.DrG_Number(patch, 3)
end

print(GetVersionString(1, 5, 27))  -- "01.05.027"
```

### Leaderboard Rankings
```lua
-- Display rankings with padded numbers
function DisplayRankings(players)
    for rank, ply in ipairs(players) do
        local rankText = "#" .. string.DrG_Number(rank, 3)
        print(rankText, ply:Nick())
    end
end
-- Outputs: #001 PlayerName, #002 PlayerName, etc.
```

### Progress Indicators
```lua
-- Show completion percentage
function DrawProgress(current, total)
    local percentage = math.floor((current / total) * 100)
    local text = string.DrG_Number(percentage, 3) .. "%"
    -- "005%", "042%", "100%"
end
```

## Technical Details

**Algorithm:**
```lua
function string.DrG_Number(nb, size)
    nb = tostring(nb)
    while #nb < size do
        nb = "0"..nb
    end
    return nb
end
```

**Performance:**
- String concatenation in loop (not optimal for large padding)
- Fine for typical use cases (padding to 2-10 digits)
- For large padding amounts, consider `string.format()` instead

**Alternatives:**
```lua
-- Using string.format (more efficient for large padding)
string.format("%05d", 42)  -- "00042"

-- But DrG_Number works with any input type
string.DrG_Number("42", 5)  -- "00042"
string.DrG_Number(42.5, 5)  -- "042.5"
```

## Notes

- Works with any input that can be converted to string
- Useful for display purposes, sorting, file naming
- Does not truncate if input is longer than specified size
- For negative numbers, the minus sign counts as a character
