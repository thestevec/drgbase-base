# Coroutine Module
**File:** `lua/drgbase/modules/coroutine.lua`

Automatic coroutine management system that handles resuming and cleanup of coroutines.

## Overview

DrGBase provides a coroutine management system that automatically resumes suspended coroutines every Think cycle and cleans up dead ones. This eliminates the need to manually manage coroutine lifecycle.

## Functions

### coroutine.DrG_Create(todo, call)
Creates and registers a managed coroutine.

**Parameters:**
- `todo` (function) - The function to run as a coroutine
- `call` (function, optional) - Callback function called when coroutine completes
  - Receives two parameters: `ok` (boolean success status), `args` (return value from coroutine)

**Returns:**
- `cor` (thread) - The created coroutine
- `id` (number) - Unique ID for the coroutine

**Behavior:**
- Coroutine automatically resumes every Think cycle
- When coroutine completes (status becomes "dead"), the callback is called
- Dead coroutines are automatically removed from management

**Example:**
```lua
-- Simple coroutine
local cor, id = coroutine.DrG_Create(function()
    print("Starting task")
    coroutine.yield()
    print("Continuing after yield")
    coroutine.yield()
    print("Task complete")
    return "success"
end)

-- Coroutine with completion callback
coroutine.DrG_Create(function()
    for i = 1, 5 do
        print("Step", i)
        coroutine.yield()
    end
    return "done"
end, function(ok, result)
    print("Coroutine finished:", ok, result)
end)

-- Long-running background task
coroutine.DrG_Create(function()
    while true do
        -- Do work
        self:ProcessTasks()

        -- Yield to prevent freezing
        coroutine.yield()

        -- Continue next frame
    end
end)
```

### coroutine.DrG_Remove(id)
Manually removes a coroutine from management.

**Parameters:**
- `id` (number) - Coroutine ID returned from `coroutine.DrG_Create()`

**Note:** Dead coroutines are automatically removed, so manual removal is typically only needed to cancel running coroutines.

**Example:**
```lua
local cor, id = coroutine.DrG_Create(function()
    while true do
        -- Some work
        coroutine.yield()
    end
end)

-- Cancel the coroutine later
timer.Simple(5, function()
    coroutine.DrG_Remove(id)
end)
```

## Use Cases

### Spreading Work Across Frames
```lua
-- Process large dataset without freezing the server
coroutine.DrG_Create(function()
    for i, data in ipairs(largeDataset) do
        ProcessData(data)

        -- Yield every 100 items to prevent lag
        if i % 100 == 0 then
            coroutine.yield()
        end
    end
end, function(ok, result)
    print("Processing complete!")
end)
```

### Sequential Async Operations
```lua
coroutine.DrG_Create(function()
    print("Phase 1: Initialization")
    coroutine.yield()

    print("Phase 2: Loading")
    coroutine.yield()

    print("Phase 3: Complete")
end)
```

### Background AI Processing
```lua
function ENT:Initialize()
    self.aiCoroutine = coroutine.DrG_Create(function()
        while IsValid(self) do
            self:ThinkAI()
            coroutine.yield()
        end
    end)
end
```

## Implementation Details

- Coroutines are stored in a table indexed by unique IDs
- A Think hook automatically resumes all suspended coroutines
- When a coroutine's status is "dead", its completion callback is called once
- Dead coroutines are automatically cleaned up
- No limit on number of concurrent coroutines
