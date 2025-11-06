# Timer Module
**File:** `lua/drgbase/modules/timer.lua`

Timer utility functions that preserve arguments with closures.

## Functions

### timer.DrG_Simple(delay, callback, ...)
Creates a delayed function call that preserves arguments at the time of creation.

**Parameters:**
- `delay` (number) - Delay in seconds
- `callback` (function) - Function to call after delay
- `...` - Arguments to pass to callback

**Why:** Standard `timer.Simple` with closure over variables can cause issues if variables change before timer executes. This captures arguments at timer creation time.

**Example:**
```lua
-- Standard timer.Simple with closure (problematic)
for i = 1, 5 do
    timer.Simple(i, function()
        print(i)  -- Always prints 5! (closure captures reference)
    end)
end

-- DrG_Simple captures value at creation time
for i = 1, 5 do
    timer.DrG_Simple(i, function(num)
        print(num)  -- Correctly prints 1, 2, 3, 4, 5
    end, i)
end

-- Preserving multiple arguments
local ent = self
local target = enemy
timer.DrG_Simple(2, function(shooter, victim)
    if IsValid(shooter) and IsValid(victim) then
        shooter:Attack(victim)
    end
end, ent, target)

-- With nil arguments
timer.DrG_Simple(1, function(a, b, c)
    print(a, b, c)  -- 1, nil, 3
end, 1, nil, 3)
```

### timer.DrG_Loop(delay, callback, ...)
Creates a repeating timer that can dynamically adjust its interval.

**Parameters:**
- `delay` (number) - Initial delay in seconds
- `callback` (function) - Function to call repeatedly
  - Return `false` to stop the loop
  - Return number to change delay for next iteration
  - Return nothing to continue with same delay
- `...` - Arguments to pass to callback

**Behavior:**
- Executes callback after delay
- If callback returns `false`, stops looping
- If callback returns a number, uses it as next delay
- Otherwise, repeats with original delay

**Example:**
```lua
-- Simple repeating timer
timer.DrG_Loop(1, function()
    print("Tick!")
    -- Continues every 1 second
end)

-- Timer with stop condition
local count = 0
timer.DrG_Loop(0.5, function()
    count = count + 1
    print("Count:", count)
    if count >= 10 then
        return false  -- Stop after 10 iterations
    end
end)

-- Dynamic interval (accelerating timer)
local interval = 1
timer.DrG_Loop(interval, function()
    print("Beep!")
    interval = interval * 0.9  -- 10% faster each time
    if interval < 0.1 then
        return false  -- Stop when too fast
    end
    return interval  -- Use new interval
end)

-- With preserved arguments
local ent = self
timer.DrG_Loop(2, function(entity, maxIterations)
    if not IsValid(entity) then return false end

    entity:DoWork()

    maxIterations = maxIterations - 1
    if maxIterations <= 0 then
        return false  -- Stop
    end
end, ent, 5)

-- Health regeneration
local regen = 1
timer.DrG_Loop(1, function(entity, regenAmount)
    if not IsValid(entity) then return false end
    if entity:Health() >= entity:GetMaxHealth() then
        return false  -- Stop when full health
    end

    entity:SetHealth(math.min(
        entity:Health() + regenAmount,
        entity:GetMaxHealth()
    ))
end, player, regen)
```

## Use Cases

### Entity State Machine
```lua
function ENT:StartPatrol()
    timer.DrG_Loop(self.PatrolInterval, function(ent)
        if not IsValid(ent) then return false end
        if ent.State ~= "patrol" then return false end

        ent:MoveToNextWaypoint()
    end, self)
end
```

### Progressive Difficulty
```lua
-- Spawn enemies faster over time
local spawnDelay = 10
timer.DrG_Loop(spawnDelay, function()
    SpawnEnemy()

    spawnDelay = math.max(spawnDelay * 0.95, 2)  -- Min 2 seconds
    return spawnDelay  -- Use new delay
end)
```

### Countdown Timer
```lua
function StartCountdown(seconds, onTick, onComplete)
    timer.DrG_Loop(1, function(remaining, tickCallback, completeCallback)
        tickCallback(remaining)

        if remaining <= 0 then
            completeCallback()
            return false  -- Stop
        end

        return 1  -- Continue every second
    end, seconds, onTick, onComplete)
end

StartCountdown(10,
    function(time)
        print("Time remaining:", time)
    end,
    function()
        print("Countdown complete!")
    end
)
```

### Conditional Polling
```lua
-- Wait for condition to be true
function WaitForCondition(checkInterval, condition, callback)
    timer.DrG_Loop(checkInterval, function(cond, cb)
        if cond() then
            cb()
            return false  -- Stop when condition met
        end
    end, condition, callback)
end

WaitForCondition(0.1,
    function() return IsValid(targetEntity) end,
    function() print("Target found!") end
)
```

### Pulsing Effect
```lua
function ENT:StartPulse()
    local intensity = 0
    local direction = 1

    timer.DrG_Loop(0.05, function(ent)
        if not IsValid(ent) then return false end

        intensity = intensity + direction * 10
        if intensity >= 255 then
            direction = -1
        elseif intensity <= 0 then
            direction = 1
        end

        ent:SetColor(Color(255, intensity, intensity))
    end, self)
end
```

### Resource Regeneration
```lua
function ENT:StartResourceRegen()
    timer.DrG_Loop(self.RegenRate, function(ent, amount)
        if not IsValid(ent) then return false end

        local current = ent:GetResource()
        local max = ent:GetMaxResource()

        if current >= max then
            return false  -- Stop at max
        end

        ent:SetResource(math.min(current + amount, max))
    end, self, self.RegenAmount)
end
```

### Delayed Sequential Actions
```lua
-- Execute actions with delays between them
function ExecuteSequence(actions, delay)
    local index = 1
    timer.DrG_Loop(delay, function(acts, i)
        if i > #acts then return false end

        acts[i]()  -- Execute action

        return delay  -- Wait before next action
    end, actions, index)
end

ExecuteSequence({
    function() print("Step 1") end,
    function() print("Step 2") end,
    function() print("Step 3") end
}, 1)
```

## Comparison with Standard Timers

**timer.Simple with closure:**
```lua
local value = 10
timer.Simple(1, function()
    print(value)  -- Problem: prints current value of variable
end)
value = 20  -- Changes before timer executes
```

**timer.DrG_Simple:**
```lua
local value = 10
timer.DrG_Simple(1, function(v)
    print(v)  -- Prints 10 (captured at creation)
end, value)
value = 20  -- Doesn't affect timer
```

**timer.Create with Think:**
```lua
-- Requires unique name, manual removal
timer.Create("MyLoop", 1, 0, function()
    if condition then
        timer.Remove("MyLoop")  -- Manual cleanup
    end
end)
```

**timer.DrG_Loop:**
```lua
-- Automatic, no names, return false to stop
timer.DrG_Loop(1, function()
    if condition then
        return false  -- Automatic cleanup
    end
end)
```

## Implementation Details

- Uses `table.DrG_Pack()` and `table.DrG_Unpack()` for argument handling
- Preserves nil arguments correctly
- No timer names required (uses anonymous `timer.Simple`)
- DrG_Loop recreates itself on each iteration
- Dynamic interval changes apply to next iteration only

## Best Practices

1. **Always validate entities in loops:**
   ```lua
   timer.DrG_Loop(1, function(ent)
       if not IsValid(ent) then return false end
       -- Safe to use ent
   end, self)
   ```

2. **Use return values to control loops:**
   ```lua
   timer.DrG_Loop(interval, function()
       if shouldStop then return false end
       if shouldAccelerate then return interval * 0.5 end
   end)
   ```

3. **Prefer DrG_Simple for closures with changing variables:**
   ```lua
   -- Good
   for i = 1, 5 do
       timer.DrG_Simple(i, print, i)
   end

   -- Bad
   for i = 1, 5 do
       timer.Simple(i, function() print(i) end)
   end
   ```
