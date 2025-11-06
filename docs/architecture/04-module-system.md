# Module System

## Overview

DrGBase uses a modular architecture where functionality is organized into focused, independent modules. Each module has a specific responsibility and exports functions/constants to the DrGBase global table or extends engine metatables.

**Three types of modules:**
1. **Core modules** (`drgbase/*.lua`) - Framework functionality
2. **Metatable extensions** (`drgbase/meta/*.lua`) - Extend engine types
3. **Utility modules** (`drgbase/modules/*.lua`) - Helper functions

Modules are loaded in order by `drgbase.lua` using the `DrGBase.IncludeFolder()` function.

## Core Modules

Each core module is documented below with its purpose, exports, and dependencies.

### colors.lua
- **Purpose:** Defines color constants for UI and console output
- **Exports:**
  - Opaque colors: `DrGBase.CLR_WHITE`, `CLR_GREEN`, `CLR_RED`, `CLR_CYAN`, `CLR_PURPLE`, `CLR_BLUE`, `CLR_ORANGE`, `CLR_DARKGRAY`, `CLR_LIGHTGRAY`
  - Transparent variants: `DrGBase.CLR_*_TR` (same colors with alpha=0)
- **Dependencies:** None
- **Size:** 29 lines
- **Realm:** Shared

### dpanels.lua
- **Purpose:** DPanel (Derma panel) utilities and helpers for UI
- **Exports:** Panel creation and management utilities
- **Dependencies:** None (client-side only)
- **Size:** 36 lines
- **Realm:** Client

### enumerations.lua
- **Purpose:** Define constants and enumerations used throughout the framework
- **Exports:**
  - Node types: `NODE_TYPE_GROUND`, `NODE_TYPE_AIR`, `NODE_TYPE_CLIMB`, `NODE_TYPE_WATER`
  - Factions: `FACTION_REBELS`, `FACTION_COMBINE`, `FACTION_ANIMALS`, `FACTION_ZOMBIES`, `FACTION_ANTLIONS`, `FACTION_GMAN`, `FACTION_BARNACLES`, `FACTION_SANIC`, `FACTION_XEN_ARMY`, `FACTION_XEN_WILDLIFE`, `FACTION_HECU`
  - Dispositions: `D_ER` (error), `D_HT` (hate), `D_FR` (fear), `D_LI` (like), `D_NU` (neutral)
  - Possession modes: `POSSESSION_MOVE_*` constants
  - AI behaviors: `AI_BEHAV_CUSTOM`, `AI_BEHAV_BASE`, `AI_BEHAV_HUMAN`
  - Patrol types: `PATROL_POS`, `PATROL_SEARCH`, `PATROL_SOUND`
- **Dependencies:** None
- **Size:** 44 lines
- **Realm:** Shared

### entity_helpers.lua
- **Purpose:** Global entity utility functions
- **Exports:** Various helper functions for entity queries and operations
- **Dependencies:** None
- **Size:** 206 lines
- **Realm:** Shared

### misc.lua
- **Purpose:** Miscellaneous framework utilities that don't fit elsewhere
- **Exports:** Various helper functions
- **Dependencies:** Multiple modules
- **Size:** 142 lines
- **Realm:** Shared

### nextbots.lua
- **Purpose:** Nextbot registration, precaching, and spawn menu integration
- **Exports:**
  - `DrGBase.AddNextbot(ENT)` - Register nextbot
  - `DrGBase.AddNextbotMixins(ENT)` - Add hook interceptors
- **Dependencies:** `entity_helpers.lua`, `spawnmenu.lua`
- **Size:** 282 lines
- **Realm:** Shared
- **ConVars:** `drgbase_precache_models`, `drgbase_precache_sounds`

### nodegraph.lua
- **Purpose:** AI node graph utilities for navigation
- **Exports:** Node graph query and manipulation functions
- **Dependencies:** None
- **Size:** 233 lines
- **Realm:** Server

### particles.lua
- **Purpose:** Particle effect system management
- **Exports:** Particle precaching and management
- **Dependencies:** Resource system
- **Size:** 77 lines
- **Realm:** Shared

### possession.lua
- **Purpose:** Player possession mechanics (control NPCs)
- **Exports:** Possession management functions
- **Dependencies:** Networking, weapon system
- **Size:** 225 lines
- **Realm:** Shared

### resources.lua
- **Purpose:** Resource management and precaching (stub/minimal)
- **Exports:** Resource management functions
- **Dependencies:** None
- **Size:** 3 lines
- **Realm:** Shared

### spawners.lua
- **Purpose:** NPC spawner entity system
- **Exports:** `DrGBase.AddSpawner(ENT)` - Register spawner entity
- **Dependencies:** Entity system
- **Size:** 78 lines
- **Realm:** Shared

### spawnmenu.lua
- **Purpose:** Integrate DrGBase entities into Garry's Mod spawn menu
- **Exports:** Spawn menu population hooks
- **Dependencies:** None
- **Size:** 123 lines
- **Realm:** Client

### weapons.lua
- **Purpose:** Weapon registration and spawn menu integration
- **Exports:**
  - `DrGBase.AddWeapon(SWEP)` - Register weapon
  - Weapon list management
- **Dependencies:** None
- **Size:** 109 lines
- **Realm:** Shared

### wrappers.lua
- **Purpose:** Entity wrapper classes for abstraction
- **Exports:** Wrapper class functions
- **Dependencies:** Entity system
- **Size:** 118 lines
- **Realm:** Shared

## Metatable Extensions

Metatable extensions add custom methods to engine types without modifying core classes.

### entity.lua

**Extended Methods:** (349 lines total)
- `ENT:DrG_IsSanic()` - Detect Sanic nextbot framework entities
- `ENT:DrG_IsDoor()` - Check if entity is a door
- `ENT:DrG_SearchBone(name)` - Find bone by name or partial match
- `ENT:DrG_TraceLine(vec, data)` - Simplified line trace from entity
- `ENT:DrG_TraceHull(vec, data)` - Simplified hull trace from entity
- `ENT:DrG_TraceLineRadial(distance, precision, data)` - Radial line traces
- `ENT:DrG_TraceHullRadial(distance, precision, data)` - Radial hull traces
- `ENT:FindInSphere(pos, radius, filter)` - Find entities in sphere
- `ENT:FindInCone(origin, direction, range, degrees, filter)` - Find entities in cone
- `ENT:FindInBox(mins, maxs, filter)` - Find entities in box
- Many more geometry and query helpers...

**Purpose:** Provide convenient entity query and trace functions

### npc.lua

**Extended Methods:** (17 lines)
- Minimal NPC-specific extensions
- Most NPC functionality is in the ENT metatable

**Purpose:** NPC-specific helper methods

### player.lua

**Extended Methods:** (276 lines)
- `Player:DrG_GetPossessedNextbot()` - Get currently possessed NPC
- `Player:DrG_IsPossessing()` - Check if possessing an NPC
- Player-specific possession helpers
- Player state management

**Purpose:** Player-related utilities and possession system support

### phys.lua

**Extended Methods:** (51 lines)
- `PhysObj:DrG_*()` methods for physics object manipulation
- Physics-based utilities

**Purpose:** Physics object helper methods

### vector.lua

**Extended Methods:** (201 lines)
- `Vector:DrG_Length2D()` - 2D vector length
- `Vector:DrG_Normalize2D()` - 2D normalization
- `Vector:DrG_Approach(target, delta)` - Smooth approach to target
- `Vector:DrG_Random()` - Random vector generation
- Geometric calculations
- Angle conversions

**Purpose:** Extended vector math operations

## Utility Modules

Utility modules provide reusable helper functions organized by category.

### coroutine.lua
- **Purpose:** Coroutine management and helpers
- **Key Functions:**
  - Coroutine creation and management
  - Non-blocking operation helpers
- **Size:** 30 lines
- **Realm:** Shared

### debugoverlay.lua
- **Purpose:** Debug visualization utilities
- **Key Functions:**
  - `debugoverlay.DrG_*()` functions for drawing debug shapes
  - Extended debug visualization helpers
- **Size:** 28 lines
- **Realm:** Shared

### math.lua
- **Purpose:** Math function extensions
- **Key Functions:**
  - `math.DrG_*()` extended math operations
  - Additional geometric calculations
- **Size:** 9 lines
- **Realm:** Shared

### navmesh.lua
- **Purpose:** Navigation mesh query helpers
- **Key Functions:**
  - Navmesh queries
  - Navigation area utilities
- **Size:** 9 lines
- **Realm:** Server

### net.lua
- **Purpose:** Simplified networking API
- **Key Functions:**
  - `net.DrG_Send(name, ...)` - Simple message sending
  - `net.DrG_Receive(name, callback)` - Message receiving
  - `net.DrG_WriteMessage(...)` - Automatic type encoding
  - `net.DrG_ReadMessage()` - Automatic type decoding
  - Callback system for request/response patterns
- **Size:** 115 lines
- **Realm:** Shared
- **Network Strings:** `DrGBaseNetMessage`, `DrGBaseNetCallbackReq`, `DrGBaseNetCallbackRes`

**Example:**
```lua
-- Send
net.DrG_Send("MyMessage", "Hello", 123, true)

-- Receive
net.DrG_Receive("MyMessage", function(ply, str, num, bool)
    print(str, num, bool)  -- "Hello", 123, true
end)
```

### render.lua
- **Purpose:** Rendering helper functions
- **Key Functions:**
  - `render.DrG_*()` rendering utilities
  - Client-side visual helpers
- **Size:** 18 lines
- **Realm:** Client

### string.lua
- **Purpose:** String manipulation extensions
- **Key Functions:**
  - `string.DrG_*()` string utilities
- **Size:** 8 lines
- **Realm:** Shared

### table.lua
- **Purpose:** Table manipulation and extensions
- **Key Functions:**
  - `table.DrG_Pack(...)` - Pack arguments into table
  - `table.DrG_Unpack(tbl, n)` - Unpack table to arguments
  - Additional table utilities
- **Size:** 55 lines
- **Realm:** Shared

### timer.lua
- **Purpose:** Timer management utilities
- **Key Functions:**
  - `timer.DrG_*()` timer helpers
  - Simplified timer creation
- **Size:** 17 lines
- **Realm:** Shared

### util.lua
- **Purpose:** General utility functions that don't fit elsewhere
- **Key Functions:**
  - `util.DrG_TraceLine(data)` - Enhanced trace line
  - `util.DrG_TraceHull(data)` - Enhanced trace hull
  - Various geometric and query utilities
- **Size:** 98 lines
- **Realm:** Shared

## Module Communication

Modules interact through several mechanisms:

### Direct Function Calls

Modules can directly call functions from other modules:

```lua
-- nextbots.lua calls functions from entity_helpers.lua
DrGBase.AddNextbot(ENT)  -- Defined in nextbots.lua

-- Uses entity helper functions internally
```

**Advantages:**
- Simple and direct
- Easy to trace code flow
- Clear dependencies

### Hook System

Modules can communicate through Garry's Mod hooks:

```lua
-- spawnmenu.lua defines a hook
hook.Add("PopulateDrGBaseSpawnmenu", "AddDrGBaseNPCs", function(pnlContent, tree, node)
    -- Populate spawn menu
end)

-- nextbots.lua calls the hook
hook.Run("PopulateDrGBaseSpawnmenu", pnlContent, tree, node)
```

**Advantages:**
- Loose coupling
- Multiple listeners
- Extensible by users

### Global State

Modules share state through the `DrGBase` global table:

```lua
-- enumerations.lua defines
FACTION_REBELS = "FACTION_HL2_REBELS"

-- Other modules use it
if npc.Faction == FACTION_REBELS then
    -- ...
end
```

**Advantages:**
- Centralized configuration
- Easy access from anywhere
- No need to pass parameters

## Adding Custom Modules

You can extend DrGBase by adding your own modules.

### Module Template

**Core module** (in `drgbase/`):
```lua
-- my_module.lua
if not DrGBase then return end

-- Module code here
local function MyPrivateHelper()
    -- Private function
end

-- Export public functions to DrGBase table
DrGBase.MyFunction = function()
    MyPrivateHelper()
    -- Public function
end

DrGBase.Print("Custom module loaded")
```

**Metatable extension** (in `drgbase/meta/`):
```lua
-- custom_entity.lua
local ENT = FindMetaTable("Entity")

function ENT:MyCustomMethod()
    -- Add method to all entities
    return self:GetPos()
end
```

**Utility module** (in `drgbase/modules/`):
```lua
-- myutil.lua
function util.DrG_MyUtility(arg)
    -- Add to util library
    return arg * 2
end
```

### Integration

**Option 1: Modify drgbase.lua**
```lua
-- In autorun/drgbase.lua, add:
DrGBase.IncludeFile("drgbase/my_module.lua")
```

**Option 2: Separate autorun file**
```lua
-- In autorun/my_drgbase_addon.lua
if not DrGBase then
    print("DrGBase not loaded!")
    return
end

DrGBase.IncludeFile("drgbase/my_module.lua")
```

**Option 3: Entity-level inclusion**
```lua
-- In your custom entity
DrGBase.IncludeFile("my_helpers.lua")
function ENT:Initialize()
    -- Use your module
end
```

### Best Practices

1. **Check DrGBase exists:** Always check `if not DrGBase then return end`
2. **Use namespacing:** Prefix functions with module name or `DrG_`
3. **Load order:** Ensure dependencies load first
4. **Document exports:** Comment what your module exports
5. **Realm awareness:** Use `if SERVER` or `if CLIENT` when needed

---

**Previous:** [Initialization System](./03-initialization.md) | **Next:** [Client-Server Architecture](./05-client-server.md)
