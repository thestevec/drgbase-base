# Coroutine Module
**File:** `lua/drgbase/modules/coroutine.lua`

The coroutine module extends Lua's built-in coroutine functionality with automatic management and lifecycle handling through the Think hook.

## Overview

DrGBase provides a coroutine management system that automatically resumes suspended coroutines every Think cycle and handles completion callbacks. This is useful for creating asynchronous operations, state machines, or time-based sequences without blocking execution.

---

## Functions

### coroutine.DrG_Create(todo, call)

**Realm:** 🟣 SHARED

**Parameters:**
- `todo` (function) - The coroutine function to execute
- `call` (function, optional) - Callback function executed when the coroutine finishes. Receives two parameters: `ok` (boolean success status) and `args` (return value from the coroutine)

**Returns:**
- `coroutine` - The created coroutine object
- `number` - Unique ID for this coroutine

**Description:**

Creates and registers a managed coroutine that will be automatically resumed every Think cycle. The coroutine is tracked internally and will execute until it completes or yields. When the coroutine finishes (status becomes "dead"), the optional callback function is invoked with the coroutine's final return values.

The coroutine will be automatically removed from the management system when it finishes execution.

**Example:**
```lua
-- Simple async operation with callback
local cor, id = coroutine.DrG_Create(function()
    print("Starting async task...")
    coroutine.yield() -- Pause for one Think cycle
    print("Continuing after one frame...")
    coroutine.yield() -- Pause again
    print("Task complete!")
    return true, "Success"
end, function(ok, result)
    print("Coroutine finished:", ok, result)
end)

-- State machine example
local state = "idle"
coroutine.DrG_Create(function()
    while true do
        if state == "idle" then
            print("Waiting...")
            coroutine.yield()
        elseif state == "active" then
            print("Processing...")
            state = "idle"
            coroutine.yield()
        end
    end
end)
```

**Notes:**
- Coroutines are resumed automatically every Think hook
- Use `coroutine.yield()` to pause execution until the next Think cycle
- The completion callback is optional but useful for handling results
- Coroutines are automatically cleaned up when they finish

**See Also:**
- [coroutine.DrG_Remove](#coroutineDrG_Removeid)

---

### coroutine.DrG_Remove(id)

**Realm:** 🟣 SHARED

**Parameters:**
- `id` (number) - The unique ID returned by `coroutine.DrG_Create`

**Returns:**
- Nothing

**Description:**

Manually removes a managed coroutine from the automatic execution system. This stops the coroutine from being resumed and cleans up its internal tracking data.

Normally, coroutines are automatically removed when they finish execution. Use this function when you need to cancel a coroutine before it completes naturally.

**Example:**
```lua
-- Create a long-running coroutine
local cor, id = coroutine.DrG_Create(function()
    for i = 1, 100 do
        print("Iteration", i)
        coroutine.yield()
    end
end)

-- Cancel it after some condition
timer.Simple(2, function()
    print("Canceling coroutine...")
    coroutine.DrG_Remove(id)
end)
```

**Notes:**
- This does not stop the coroutine if it's currently running, it only prevents future resumes
- Removing a coroutine does not trigger its completion callback
- Safe to call multiple times with the same ID

**See Also:**
- [coroutine.DrG_Create](#coroutineDrG_Createtodo-call)
