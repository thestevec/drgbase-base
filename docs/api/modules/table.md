# Table Module
**File:** `lua/drgbase/modules/table.lua`

The table module extends Lua's built-in table library with advanced manipulation and utility functions.

## Overview

This module provides powerful table utilities including read-only proxies, default value tables, varargs packing/unpacking, custom fetching, key-value inversion, and deep copying. These functions are essential for advanced Lua programming patterns.

---

## Functions

### table.DrG_ReadOnly(tbl)

**Realm:** 🟣 SHARED

**Parameters:**
- `tbl` (table) - The table to make read-only

**Returns:**
- `table` - A read-only proxy of the original table

**Description:**

Creates a read-only proxy of a table using metatables. The proxy allows reading values from the original table but silently ignores any attempts to modify, add, or remove values.

This is useful for:
- Protecting configuration tables from accidental modification
- Creating immutable data structures
- Exposing internal data safely to other code
- Preventing unintended side effects

The original table remains unchanged and can still be modified directly. Only the returned proxy is read-only.

**Example:**
```lua
-- Protect a configuration table
local config = {
    maxHealth = 100,
    speed = 200,
    damage = 25
}
local readOnlyConfig = table.DrG_ReadOnly(config)

-- Reading works fine
print(readOnlyConfig.maxHealth) -- 100

-- Writing is silently ignored
readOnlyConfig.maxHealth = 500
print(readOnlyConfig.maxHealth) -- Still 100

-- Original table can still be modified
config.maxHealth = 500
print(readOnlyConfig.maxHealth) -- Now 500 (proxy reads from original)

-- Expose entity data safely
function ENT:GetSafeData()
    return table.DrG_ReadOnly(self.data)
end

-- Protect shared constants
GAME_CONSTANTS = table.DrG_ReadOnly({
    GRAVITY = 600,
    MAX_PLAYERS = 32,
    ROUND_TIME = 300
})
```

**Notes:**
- Modifications are silently ignored (no error is thrown)
- The original table can still be modified directly
- Changes to the original table are visible through the proxy
- Nested tables are NOT automatically read-only
- Reading values has no performance penalty
- Use this to prevent accidental modifications, not for security

**See Also:**
- [table.DrG_Copy](#tableDrG_Copytbl-copied) - For creating independent copies

---

### table.DrG_Default(tbl, default)

**Realm:** 🟣 SHARED

**Parameters:**
- `tbl` (table) - The table to add default values to
- `default` (any) - The default value to return for missing keys

**Returns:**
- `table` - The table with default value behavior

**Description:**

Modifies a table so that accessing any non-existent key returns a default value instead of `nil`. This is implemented using the `__index` metamethod.

This is extremely useful for:
- Counters and accumulators (default 0)
- Boolean flags (default false)
- String builders (default "")
- Avoiding nil checks in loops

The default value is returned for ANY missing key, making code cleaner and more concise when working with sparse data.

**Example:**
```lua
-- Counter with automatic zero initialization
local kills = table.DrG_Default({}, 0)
kills["player1"] = kills["player1"] + 1 -- No need to check if nil
kills["player2"] = kills["player2"] + 5
print(kills["player3"]) -- 0 (not nil)

-- Boolean flags with default false
local permissions = table.DrG_Default({}, false)
permissions["admin"] = true
if permissions["moderator"] then -- false, not nil
    print("Is moderator")
end

-- Accumulating values without nil checks
local scores = table.DrG_Default({}, 0)
for _, player in ipairs(player.GetAll()) do
    scores[player:Team()] = scores[player:Team()] + player:Frags()
end

-- String building
local messages = table.DrG_Default({}, "")
messages["log"] = messages["log"] .. "Event occurred\n"

-- Tracking entity states
local entityStates = table.DrG_Default({}, "idle")
entityStates[entity] = "active"
if entityStates[otherEntity] == "idle" then -- Default is "idle"
    print("Entity is idle")
end

-- Nested defaults for complex structures
local playerData = {}
setmetatable(playerData, {
    __index = function()
        return table.DrG_Default({}, 0)
    end
})
playerData["Player1"]["kills"] = 5 -- Auto-creates inner table with defaults
```

**Notes:**
- Modifies the original table's metatable
- Returns the same table (not a copy)
- Default value is shared across all missing keys
- Be careful with mutable defaults (tables, etc.) as they're shared by reference
- Does not create keys, just returns default for missing ones
- Existing keys are unaffected

**See Also:**
- Lua metatables documentation

---

### table.DrG_Pack(...)

**Realm:** 🟣 SHARED

**Parameters:**
- `...` (any) - Variable number of arguments to pack

**Returns:**
- `table` - Table containing all arguments
- `number` - The exact count of arguments (handles nil values correctly)

**Description:**

Packs varargs (`...`) into a table along with an accurate count. Unlike the standard Lua `{...}` syntax, this function correctly handles `nil` values in the middle of the argument list by also returning the exact argument count.

This is critical for:
- Preserving nil arguments when passing varargs
- Network message serialization
- Function call forwarding
- Coroutine argument passing

The second return value is crucial because `#table` would give incorrect results if the arguments contain `nil` values.

**Example:**
```lua
-- Basic packing
local args, count = table.DrG_Pack("hello", 42, true)
print(count) -- 3
print(args[1], args[2], args[3]) -- hello  42  true

-- Correctly handles nil values
local args, count = table.DrG_Pack("a", nil, "b", nil, "c")
print(count) -- 5 (not 3!)
print(args[3]) -- "b"

-- Forward function arguments
function WrapperFunction(...)
    local args, count = table.DrG_Pack(...)
    -- Store or transmit args
    timer.Simple(1, function()
        RealFunction(table.DrG_Unpack(args, count))
    end)
end

-- Network transmission
function net.SendArgs(...)
    local args, count = table.DrG_Pack(...)
    net.WriteUInt(count, 8)
    for i = 1, count do
        net.WriteType(args[i])
    end
end

-- Store callback arguments
local storedCalls = {}
function DelayedCall(callback, ...)
    local args, count = table.DrG_Pack(...)
    table.insert(storedCalls, {
        callback = callback,
        args = args,
        count = count
    })
end
```

**Notes:**
- Returns both the table AND the count - always use both
- Essential for correctly handling nil values in varargs
- Used internally by DrGBase networking functions
- Pair with `table.DrG_Unpack()` to restore varargs
- The count is from `select("#", ...)` which handles nil correctly

**See Also:**
- [table.DrG_Unpack](#tableDrG_Unpacktbl-size-i) - Unpacks back to varargs

---

### table.DrG_Unpack(tbl, size, i)

**Realm:** 🟣 SHARED

**Parameters:**
- `tbl` (table) - The table to unpack
- `size` (number) - The number of elements to unpack (use count from `table.DrG_Pack`)
- `i` (number, optional, default: 1) - Starting index

**Returns:**
- `...` - Multiple return values from the table

**Description:**

Unpacks a table into multiple return values (varargs), correctly handling `nil` values. Unlike Lua's `unpack()` or `table.unpack()`, this function uses an explicit size parameter to ensure `nil` values in the middle of the array are properly included.

This function is recursive and works by unpacking one element at a time until it reaches the size limit. It's designed to work perfectly with `table.DrG_Pack()`.

**Example:**
```lua
-- Basic unpacking
local args, count = table.DrG_Pack("a", "b", "c")
print(table.DrG_Unpack(args, count)) -- a  b  c

-- Correctly handles nil values
local args, count = table.DrG_Pack("x", nil, "y", nil, "z")
local a, b, c, d, e = table.DrG_Unpack(args, count)
print(a, b, c, d, e) -- x  nil  y  nil  z

-- Forward arguments through timer
function DelayedFunction(delay, callback, ...)
    local args, count = table.DrG_Pack(...)
    timer.Simple(delay, function()
        callback(table.DrG_Unpack(args, count))
    end)
end

-- Restore network arguments
local args, count = net.DrG_ReadMessage()
MyFunction(table.DrG_Unpack(args, count))

-- Partial unpacking from specific index
local args = {"a", "b", "c", "d"}
print(table.DrG_Unpack(args, 4, 2)) -- b  c  d (starts from index 2)
```

**Notes:**
- Requires the size parameter to handle nil values correctly
- Use the count returned by `table.DrG_Pack()`
- Implemented recursively (may have stack limits for very large arrays)
- Default starting index is 1
- Essential for DrGBase's networking and timer functions

**See Also:**
- [table.DrG_Pack](#tableDrG_Pack) - Packs varargs with count

---

### table.DrG_Fetch(tbl, callback)

**Realm:** 🟣 SHARED

**Parameters:**
- `tbl` (table) - The table to search through
- `callback` (function) - Comparison function `(val, fetched, key, fetchedKey)` that returns `true` if `val` should replace `fetched`

**Returns:**
- `any` - The fetched value
- `any` - The key of the fetched value

**Description:**

Iterates through a table and returns the value (and key) that best matches the criteria defined by the callback function. This is a generic fetch/find function that can be used to find minimums, maximums, or any custom comparison.

The callback function receives:
- `val` - Current value being evaluated
- `fetched` - Currently fetched (best) value
- `key` - Key of current value
- `fetchedKey` - Key of currently fetched value

Return `true` from the callback if the current value should replace the fetched value.

**Example:**
```lua
-- Find maximum value
local numbers = {a = 5, b = 12, c = 3, d = 9}
local max, maxKey = table.DrG_Fetch(numbers, function(val, fetched)
    return fetched == nil or val > fetched
end)
print(max, maxKey) -- 12  b

-- Find minimum value
local min, minKey = table.DrG_Fetch(numbers, function(val, fetched)
    return fetched == nil or val < fetched
end)
print(min, minKey) -- 3  c

-- Find closest player to a position
local pos = Vector(0, 0, 0)
local closest = table.DrG_Fetch(player.GetAll(), function(ply, fetched)
    if not fetched then return true end
    return ply:GetPos():Distance(pos) < fetched:GetPos():Distance(pos)
end)

-- Find highest health entity
local healthiest = table.DrG_Fetch(ents.GetAll(), function(ent, fetched)
    if not IsValid(ent) or not ent:Health() then return false end
    return not fetched or ent:Health() > fetched:Health()
end)

-- Find alphabetically first name
local names = {john = true, alice = true, bob = true}
local first = table.DrG_Fetch(names, function(val, fetched, key, fetchedKey)
    return not fetchedKey or key < fetchedKey
end)
print(first) -- alice

-- Custom scoring system
local players = player.GetAll()
local function GetScore(ply)
    return ply:Frags() * 10 + ply:Deaths() * -5
end
local winner = table.DrG_Fetch(players, function(ply, fetched)
    if not fetched then return true end
    return GetScore(ply) > GetScore(fetched)
end)
```

**Notes:**
- Returns both the value and the key
- The callback's first invocation has `fetched` as `nil`
- Always check for `nil` in your callback
- Works with any table structure (arrays or dictionaries)
- Can be used for min/max finding, closest entity searches, etc.
- More flexible than table.sort for finding a single best match

**See Also:**
- `table.sort()` - For sorting entire tables

---

### table.DrG_Invert(tbl)

**Realm:** 🟣 SHARED

**Parameters:**
- `tbl` (table) - The table to invert

**Returns:**
- `table` - New table with keys and values swapped

**Description:**

Creates a new table where the keys and values are swapped. Values become keys and keys become values. This is useful for creating reverse lookups or converting value lists into lookup tables.

If multiple keys have the same value, only one will appear in the result (the last one processed by `pairs()`).

**Example:**
```lua
-- Create reverse lookup
local teamNames = {
    [1] = "Red",
    [2] = "Blue",
    [3] = "Green"
}
local teamIDs = table.DrG_Invert(teamNames)
print(teamIDs["Blue"]) -- 2

-- Convert array to lookup table
local allowedWeapons = {"weapon_pistol", "weapon_smg", "weapon_shotgun"}
local weaponLookup = table.DrG_Invert(allowedWeapons)
if weaponLookup[weapon:GetClass()] then
    print("Weapon is allowed")
end

-- Error code mapping
local errorCodes = {
    SUCCESS = 0,
    NOT_FOUND = 404,
    SERVER_ERROR = 500
}
local codeNames = table.DrG_Invert(errorCodes)
print("Error " .. code .. ": " .. codeNames[code])

-- Enum to string conversion
local STATE = {
    IDLE = 1,
    ACTIVE = 2,
    DEAD = 3
}
local STATE_NAMES = table.DrG_Invert(STATE)
print("State: " .. STATE_NAMES[entity.state])
```

**Notes:**
- Creates a NEW table (original is unchanged)
- If multiple keys have the same value, behavior is undefined (depends on `pairs()` order)
- Useful for creating reverse lookups
- Both keys and values should be usable as table keys (numbers, strings, etc.)
- Does not work with nested tables

**See Also:**
- [table.DrG_Copy](#tableDrG_Copytbl-copied) - For deep copying

---

### table.DrG_Copy(tbl, copied)

**Realm:** 🟣 SHARED

**Parameters:**
- `tbl` (table) - The table to copy
- `copied` (table, optional) - Internal cache for handling circular references (leave empty for normal use)

**Returns:**
- `table` - Deep copy of the table

**Description:**

Creates a deep copy of a table, recursively copying all nested tables. This creates a completely independent copy where modifications to the copy don't affect the original.

The function handles:
- Nested tables at any depth
- Circular references (tables that reference themselves)
- Tables with metatables that are not tables themselves

Tables are copied recursively, but other types (numbers, strings, functions, entities, etc.) are copied by reference. The function maintains a cache to handle circular references properly.

**Example:**
```lua
-- Basic deep copy
local original = {
    name = "Player",
    stats = {
        health = 100,
        armor = 50
    }
}
local copy = table.DrG_Copy(original)
copy.stats.health = 200
print(original.stats.health) -- Still 100 (independent copy)

-- Copy with nested structures
local data = {
    level1 = {
        level2 = {
            level3 = {
                value = 42
            }
        }
    }
}
local dataCopy = table.DrG_Copy(data)

-- Handles circular references
local circular = {}
circular.self = circular
circular.data = {value = 10}
local circularCopy = table.DrG_Copy(circular)
-- No infinite recursion!

-- Copy entity configuration
local npcConfig = {
    health = 100,
    weapons = {"weapon_pistol", "weapon_smg"},
    behavior = {
        aggressive = true,
        searchRadius = 500
    }
}
local copiedConfig = table.DrG_Copy(npcConfig)
copiedConfig.health = 200 -- Original unchanged

-- Safely modify shared tables
function ENT:Initialize()
    self.data = table.DrG_Copy(self.BaseData)
    -- Now self.data is independent
end
```

**Notes:**
- Creates a deep copy (all nested tables are copied)
- Handles circular references automatically
- Non-table values are copied by reference (functions, entities, etc.)
- The `copied` parameter is for internal use - don't pass it manually
- More expensive than shallow copy, but safe for complex structures
- Does not copy metatables (except to check if they're tables)
- Essential for creating independent copies of configuration tables

**See Also:**
- `table.Copy()` - GMod's shallow copy function
- [table.DrG_ReadOnly](#tableDrG_ReadOnlytbl) - For read-only access without copying
