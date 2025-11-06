# Timer Module
**File:** `lua/drgbase/modules/timer.lua`

The timer module extends Garry's Mod's built-in timer library with enhanced argument passing and looping capabilities.

## Overview

This module provides timer functions that solve a common problem: passing arguments to delayed callbacks. Unlike `timer.Simple()` which can't directly pass arguments to the callback, these functions use `table.DrG_Pack` and `table.DrG_Unpack` to preserve arguments, including `nil` values.

---

## Functions

### timer.DrG_Simple(delay, callback, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `delay` (number) - Time in seconds before the callback is executed
- `callback` (function) - Function to execute after the delay
- `...` (any) - Arguments to pass to the callback function

**Returns:**
- Nothing

**Description:**

Creates a simple delayed timer that calls the specified function after a delay, passing along any provided arguments. This is an enhanced version of `timer.Simple()` that properly forwards arguments to the callback.

The function uses `table.DrG_Pack()` and `table.DrG_Unpack()` internally to preserve all arguments, including `nil` values in the middle of the argument list. This makes it much more convenient than `timer.Simple()` when you need to pass data to the delayed callback.

The arguments are captured at the time the timer is created, so later changes to variables won't affect the callback.

**Example:**
```lua
-- Pass arguments to a delayed function
timer.DrG_Simple(2, function(name, health, armor)
    print(name, "has", health, "HP and", armor, "armor")
end, "Player", 100, 50)

-- Works with nil values
timer.DrG_Simple(1, function(a, b, c)
    print(a, b, c) -- "first"  nil  "third"
end, "first", nil, "third")

-- Delayed entity operations
local ent = ents.Create("prop_physics")
ent:SetPos(Vector(0, 0, 100))
ent:Spawn()

timer.DrG_Simple(0.5, function(entity, targetPos)
    if IsValid(entity) then
        entity:SetPos(targetPos)
    end
end, ent, Vector(100, 200, 0))

-- Capture player data
for _, ply in ipairs(player.GetAll()) do
    local name = ply:Nick()
    local steamid = ply:SteamID()

    timer.DrG_Simple(5, function(playerName, playerID)
        print("Delayed message for", playerName, playerID)
    end, name, steamid)
end

-- Schedule entity cleanup
function ScheduleRemoval(entity, delay)
    timer.DrG_Simple(delay, function(ent)
        if IsValid(ent) then
            ent:Remove()
        end
    end, entity)
end

-- Delayed notification with data
timer.DrG_Simple(3, function(message, color, duration)
    NotifyPlayer(message, color, duration)
end, "Game starting!", Color(255, 255, 0), 5)
```

**Notes:**
- Arguments are captured when the timer is created, not when it executes
- Correctly handles `nil` values in arguments
- Does not create a named timer (use `timer.Create()` if you need that)
- The timer cannot be cancelled (it's anonymous)
- More convenient than using closures or `timer.Simple()` for passing data
- No performance penalty compared to `timer.Simple()` with manual closures

**See Also:**
- [timer.DrG_Loop](#timerDrG_Loopdelay-callback-) - For repeating timers with arguments
- `timer.Simple()` - GMod's basic simple timer
- [table.DrG_Pack](#tableDrG_Pack) - Used internally for argument preservation

---

### timer.DrG_Loop(delay, callback, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `delay` (number) - Time in seconds between each callback execution
- `callback` (function) - Function to execute repeatedly. Can return `false` to stop looping, or a number to change the delay
- `...` (any) - Arguments to pass to the callback function each time

**Returns:**
- Nothing

**Description:**

Creates a looping timer that repeatedly calls a function with preserved arguments. Unlike `timer.Create()` with infinite repetitions, this provides more control over the loop behavior through the callback's return value.

The callback function is called repeatedly with the provided arguments. The callback can control the timer's behavior by returning:
- Nothing (or `nil`): Continue looping with the same delay
- `false`: Stop the loop completely
- A number: Continue looping but change the delay to this new value

This is extremely useful for:
- Countdown timers that stop at zero
- Polling operations that stop when a condition is met
- Dynamic timing where delays change based on game state
- Periodic checks that terminate themselves

The arguments are captured when the timer starts, so the same values are passed to every iteration.

**Example:**
```lua
-- Simple repeating timer
timer.DrG_Loop(1, function(message)
    print(message) -- Prints every second
end, "Tick")

-- Countdown timer that stops
local count = 10
timer.DrG_Loop(1, function()
    print("Countdown:", count)
    count = count - 1
    if count <= 0 then
        print("Done!")
        return false -- Stop the loop
    end
end)

-- Dynamic delay - speeds up over time
local currentDelay = 2
timer.DrG_Loop(currentDelay, function()
    print("Pulse")
    currentDelay = math.max(0.1, currentDelay - 0.2)
    return currentDelay -- Change the delay
end)

-- Entity health check that stops when dead
timer.DrG_Loop(0.5, function(entity)
    if not IsValid(entity) then return false end
    if entity:Health() <= 0 then return false end

    print(entity, "health:", entity:Health())
end, myEntity)

-- Polling until condition is met
timer.DrG_Loop(0.1, function(targetPos)
    local dist = LocalPlayer():GetPos():Distance(targetPos)
    if dist < 100 then
        print("Reached destination!")
        return false -- Stop polling
    end
    print("Distance:", math.floor(dist))
end, Vector(1000, 500, 0))

-- Resource regeneration that changes rate
local resources = 0
timer.DrG_Loop(1, function(maxResources, regenRate)
    resources = resources + regenRate
    if resources >= maxResources then
        resources = maxResources
        print("Resources full!")
        return false
    end
    print("Resources:", resources)

    -- Slow down as we approach max
    local remaining = maxResources - resources
    if remaining < 20 then
        return 2 -- Slower regeneration
    end
end, 100, 5)

-- Heartbeat with passed data
timer.DrG_Loop(5, function(serverName, checkUrl)
    http.Fetch(checkUrl, function(body)
        print(serverName, "is alive")
    end)
end, "MyServer", "http://example.com/check")

-- Wave spawner that stops after final wave
local wave = 1
timer.DrG_Loop(30, function(maxWaves, spawnFunc)
    print("Starting wave", wave)
    spawnFunc(wave)

    wave = wave + 1
    if wave > maxWaves then
        print("All waves complete!")
        return false
    end
end, 10, function(waveNum)
    -- Spawn enemies for this wave
end)
```

**Notes:**
- Return `false` from callback to stop the loop
- Return a number to change the delay for the next iteration
- Return nothing/`nil` to continue with the same delay
- Arguments are captured once and reused for all iterations
- The timer is anonymous and cannot be retrieved or cancelled externally
- Each iteration creates a new `timer.Simple()` internally
- Handles `nil` values in arguments correctly
- Good for self-managing timers that don't need external control

**See Also:**
- [timer.DrG_Simple](#timerDrG_Simpledelay-callback-) - For single delayed execution
- `timer.Create()` - For named timers with more control
- [table.DrG_Unpack](#tableDrG_Unpacktbl-size-i) - Used internally for argument restoration
