# Utility Modules API

Extended functionality for Garry's Mod built-in libraries and DrGBase-specific utilities.

**Location:** `lua/drgbase/modules/`

## Overview

DrGBase extends Garry's Mod's standard libraries with additional utility functions that simplify common tasks like networking, table manipulation, tracing, and timing operations.

## Modules

### Core Utilities
- **[util.md](./util.md)** - Trace functions with debug visualization, damage handling, and color interpolation
- **[table.md](./table.md)** - Table manipulation: read-only wrappers, deep copy, fetch/find operations
- **[string.md](./string.md)** - String formatting, zero-padded numbers

### Timing & Async
- **[timer.md](./timer.md)** - Argument-preserving timers, repeating loops with dynamic intervals
- **[coroutine.md](./coroutine.md)** - Automatic coroutine management with lifecycle handling

### Math & Positioning
- **[math.md](./math.md)** - Cyclic values for animations and periodic effects
- **[navmesh.md](./navmesh.md)** - Fast navigation area lookups (server-only)

### Rendering & Visualization
- **[render.md](./render.md)** - 2D sprites in 3D space with billboard effect (client-only)
- **[debugoverlay.md](./debugoverlay.md)** - Trajectory visualization for projectiles

### Networking
- **[net.md](./net.md)** - Simplified networking with automatic serialization and callbacks

## Common Use Cases

### Networking Made Simple
```lua
-- Server
net.DrG_Receive("PlayerAction", function(ply, actionType, data)
    HandlePlayerAction(ply, actionType, data)
end)

-- Client
net.DrG_Send("PlayerAction", "jump", {height = 100})
```

### Automatic Coroutine Management
```lua
coroutine.DrG_Create(function()
    for i = 1, 100 do
        ProcessItem(i)
        coroutine.yield()  -- Automatically resumed next Think
    end
end)
```

### Argument-Safe Timers
```lua
for i = 1, 5 do
    timer.DrG_Simple(i, function(num)
        print(num)  -- Correctly prints 1, 2, 3, 4, 5
    end, i)
end
```

### Cyclic Animations
```lua
-- Smooth pulsing effect
local scale = math.DrG_Cycle(0.8, 1.2, 0.5)
ent:SetModelScale(scale)
```

### Advanced Table Operations
```lua
-- Deep copy with shared reference preservation
local copy = table.DrG_Copy(originalTable)

-- Find best target
local closest = table.DrG_Fetch(enemies, function(enemy, current)
    return GetDistance(enemy) < GetDistance(current)
end)
```

## Design Philosophy

All DrGBase utility modules follow these principles:

1. **Preserve Arguments**: Functions like `timer.DrG_Simple` capture arguments at call time
2. **Automatic Management**: Systems like coroutines self-manage lifecycle
3. **Nil-Safe**: Table pack/unpack preserve nil values
4. **Debug-Friendly**: Built-in visualization (traces, trajectories)
5. **Consistent Naming**: All functions prefixed with `DrG_`

## Performance Considerations

- **util.DrG_TraceLine**: Debug visualization has performance cost when enabled
- **table.DrG_Copy**: Recursive, use sparingly on large tables
- **render.DrG_DrawSprite**: Lighting calculation is expensive
- **net.DrG_***: Slight overhead for automatic serialization
- **coroutine.DrG_Create**: Runs every Think, limit concurrent count
