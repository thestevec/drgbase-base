# Table Module
**File:** `lua/drgbase/modules/table.lua`

Table utility functions for common operations and metatable manipulation.

## Functions

### table.DrG_ReadOnly(tbl)
Creates a read-only wrapper around a table.

**Parameters:**
- `tbl` (table) - The table to make read-only

**Returns:**
- (table) - Read-only proxy table

**Behavior:**
- Reading values works normally
- Writing/modifying values is silently ignored
- Original table remains unmodified and accessible

**Example:**
```lua
local data = {health = 100, name = "Player"}
local readonly = table.DrG_ReadOnly(data)

print(readonly.health)  -- 100 (reading works)
readonly.health = 50    -- Silently ignored
print(readonly.health)  -- Still 100

-- Useful for config tables
local CONFIG = table.DrG_ReadOnly({
    MAX_HEALTH = 100,
    DAMAGE = 25
})
```

### table.DrG_Default(tbl, default)
Creates a table that returns a default value for missing keys.

**Parameters:**
- `tbl` (table) - Base table
- `default` (any) - Value to return for missing keys

**Returns:**
- (table) - Table with default value metatable

**Example:**
```lua
-- Table with default value 0
local scores = table.DrG_Default({}, 0)
print(scores["Alice"])  -- 0 (doesn't exist)
scores["Bob"] = 10
print(scores["Bob"])    -- 10

-- Useful for counters
local counts = table.DrG_Default({}, 0)
for _, item in ipairs(items) do
    counts[item] = counts[item] + 1  -- No need to check if exists
end

-- Default table
local defaultEmpty = table.DrG_Default({players = {}}, {})
print(#defaultEmpty.nonexistent)  -- 0 (returns default empty table)
```

### table.DrG_Pack(...)
Packs variadic arguments into a table with size information.

**Parameters:**
- `...` - Any number of arguments

**Returns:**
- `tbl` (table) - Packed arguments
- `n` (number) - Number of arguments (including nils)

**Why:** Standard `{...}` loses nil arguments in the middle of the varargs. This preserves them.

**Example:**
```lua
-- Standard table loses middle nils
local t1 = {1, nil, 3}
print(#t1)  -- 1 (stops at first nil)

-- DrG_Pack preserves nils
local t2, n = table.DrG_Pack(1, nil, 3)
print(n)    -- 3 (correct count)

-- Useful for function argument forwarding
function Wrapper(...)
    local args, count = table.DrG_Pack(...)
    -- Can now safely pass args with nils preserved
    timer.Simple(1, function()
        OtherFunction(table.DrG_Unpack(args, count))
    end)
end
```

### table.DrG_Unpack(tbl, size, i)
Unpacks a table into multiple return values, handling nils correctly.

**Parameters:**
- `tbl` (table) - Table to unpack
- `size` (number) - Number of elements to unpack
- `i` (number, optional) - Starting index (default: 1)

**Returns:**
- (...) - Unpacked values from table

**Why:** Standard `unpack()` stops at first nil. This preserves all values up to size.

**Example:**
```lua
-- Pack and unpack with nils
local args, n = table.DrG_Pack(1, nil, 3, nil, 5)
print(table.DrG_Unpack(args, n))  -- 1, nil, 3, nil, 5

-- Partial unpack (skip first element)
local result = {table.DrG_Unpack({10, 20, 30}, 3, 2)}
print(table.concat(result, ", "))  -- 20, 30

-- Function argument forwarding
function DelayedCall(delay, func, ...)
    local args, n = table.DrG_Pack(...)
    timer.Simple(delay, function()
        func(table.DrG_Unpack(args, n))
    end)
end
```

### table.DrG_Fetch(tbl, callback)
Finds the "best" element in a table according to a comparison function.

**Parameters:**
- `tbl` (table) - Table to search
- `callback` (function) - Comparison function
  - Signature: `callback(val, currentBest, key, currentBestKey)`
  - Return `true` if `val` is better than `currentBest`

**Returns:**
- `value` (any) - The best value found
- `key` (any) - Key of the best value

**Example:**
```lua
-- Find highest value
local numbers = {a = 5, b = 12, c = 3, d = 20}
local max, key = table.DrG_Fetch(numbers, function(val, currentMax)
    return val > (currentMax or -math.huge)
end)
print(max, key)  -- 20, "d"

-- Find lowest value
local min = table.DrG_Fetch(numbers, function(val, currentMin)
    return val < (currentMin or math.huge)
end)
print(min)  -- 3

-- Find closest entity
local closest = table.DrG_Fetch(ents.GetAll(), function(ent, current)
    if not IsValid(current) then return true end
    local myPos = self:GetPos()
    return myPos:DistToSqr(ent:GetPos()) < myPos:DistToSqr(current:GetPos())
end)

-- Find player with highest score
local topPlayer = table.DrG_Fetch(player.GetAll(), function(ply, current)
    if not current then return true end
    return ply:Frags() > current:Frags()
end)
```

### table.DrG_Invert(tbl)
Inverts a table (swaps keys and values).

**Parameters:**
- `tbl` (table) - Table to invert

**Returns:**
- (table) - New table with keys/values swapped

**Example:**
```lua
-- Simple inversion
local original = {a = 1, b = 2, c = 3}
local inverted = table.DrG_Invert(original)
-- inverted = {[1] = "a", [2] = "b", [3] = "c"}

-- Lookup table creation
local weaponNames = {
    [1] = "pistol",
    [2] = "rifle",
    [3] = "shotgun"
}
local weaponIDs = table.DrG_Invert(weaponNames)
print(weaponIDs["rifle"])  -- 2

-- Enum reversal
local TEAM = {NONE = 0, RED = 1, BLUE = 2}
local TEAM_NAMES = table.DrG_Invert(TEAM)
print(TEAM_NAMES[1])  -- "RED"
```

### table.DrG_Copy(tbl, copied)
Deep copies a table, preserving shared references.

**Parameters:**
- `tbl` (table) - Table to copy
- `copied` (table, optional) - Internal cache of already-copied tables

**Returns:**
- (table) - Deep copy of the table

**Behavior:**
- Recursively copies nested tables
- Preserves shared table references (no duplication)
- Skips tables with metatables (copies reference instead)
- Safe against circular references

**Example:**
```lua
-- Deep copy
local original = {
    name = "NPC",
    stats = {health = 100, armor = 50},
    weapons = {"pistol", "rifle"}
}
local copy = table.DrG_Copy(original)
copy.stats.health = 200
print(original.stats.health)  -- Still 100

-- Shared reference preservation
local shared = {data = "shared"}
local tbl = {
    a = shared,
    b = shared
}
local copied = table.DrG_Copy(tbl)
copied.a.data = "modified"
print(copied.b.data)  -- "modified" (same reference preserved)

-- Entity references preserved (metatables)
local data = {
    owner = player.GetByID(1),  -- Entity (has metatable)
    config = {damage = 25}       -- Plain table (will be copied)
}
local copy = table.DrG_Copy(data)
print(copy.owner == data.owner)  -- true (same entity)
print(copy.config == data.config)  -- false (copied)
```

## Use Cases

### Config Protection
```lua
-- Protect config from modification
ENT.CONFIG = table.DrG_ReadOnly({
    HEALTH = 100,
    DAMAGE = 25,
    SPEED = 300
})

-- Attempts to modify are ignored
ENT.CONFIG.HEALTH = 999  -- Ignored
```

### Counting Occurrences
```lua
function CountOccurrences(items)
    local counts = table.DrG_Default({}, 0)
    for _, item in ipairs(items) do
        counts[item] = counts[item] + 1
    end
    return counts
end
```

### Network Message Forwarding
```lua
-- Preserve nil arguments across network
net.DrG_Send = function(name, ...)
    local args, n = table.DrG_Pack(...)
    net.Start(name)
    net.WriteUInt(n, 8)
    for i = 1, n do
        net.WriteType(args[i])
    end
    net.Broadcast()
end
```

### Finding Best Target
```lua
function ENT:FindBestTarget()
    local targets = self:GetPotentialTargets()
    local best = table.DrG_Fetch(targets, function(target, current)
        if not IsValid(current) then return true end
        -- Prefer closer, lower health targets
        local dist = self:GetPos():DistToSqr(target:GetPos())
        local currentDist = self:GetPos():DistToSqr(current:GetPos())
        if dist < currentDist * 0.5 then return true end
        return target:Health() < current:Health()
    end)
    return best
end
```

## Technical Details

- **ReadOnly:** Uses `__newindex` to block writes
- **Default:** Uses `__index` function to return default
- **Pack/Unpack:** Handles varargs with nils correctly
- **Fetch:** Single pass through table (O(n))
- **Invert:** Creates new table, original unchanged
- **Copy:** Recursive with cycle detection via `copied` cache
