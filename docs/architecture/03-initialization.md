# Initialization System

## Entry Point

The DrGBase framework initializes through `lua/autorun/drgbase.lua`.

### Autorun System

Garry's Mod automatically executes all Lua files in the `lua/autorun/` directory when the game starts. DrGBase uses this mechanism as its entry point:

- **Execution timing**: Runs after the game engine initializes but before entities load
- **Realm execution**: Runs on both server and client (realm-specific code handled internally)
- **Load order**: Autorun files execute in alphabetical order (drgbase.lua loads early)

This ensures DrGBase is fully initialized before any entities, weapons, or other addons that depend on it.

## Initialization Sequence

DrGBase initialization follows a specific sequence to ensure proper dependency loading:

### Phase 1: Global Setup

```lua
-- Create global table
DrGBase = DrGBase or {}

-- Setup print functions
DrGBase.Print()
DrGBase.Info()
DrGBase.Error()
DrGBase.ErrorInfo()
```

### Phase 2: Include System Setup

```lua
-- Define file inclusion functions
DrGBase.IncludeFile(fileName)
DrGBase.IncludeFolder(folder)
DrGBase.RecursiveInclude(folder)
```

### Phase 3: Core Modules

```lua
-- Load core functionality
drgbase/colors.lua
drgbase/enumerations.lua
drgbase/entity_helpers.lua
drgbase/misc.lua
-- etc.
```

### Phase 4: Metatable Extensions

```lua
-- Extend engine metatables
drgbase/meta/entity.lua
drgbase/meta/npc.lua
drgbase/meta/player.lua
drgbase/meta/phys.lua
drgbase/meta/vector.lua
```

### Phase 5: Utility Modules

```lua
-- Load utility modules
drgbase/modules/coroutine.lua
drgbase/modules/math.lua
drgbase/modules/net.lua
-- etc.
```

### Phase 6: Systems Initialization

```lua
-- Load system modules
drgbase/nextbots.lua       -- Nextbot registry
drgbase/weapons.lua         -- Weapon registry
drgbase/spawners.lua        -- Spawner system
drgbase/possession.lua      -- Possession system
drgbase/spawnmenu.lua       -- Spawn menu integration
```

### Phase 7: Base Entities

<!-- Garry's Mod automatically loads entities/ after autorun -->

### Phase 8: Network Setup

```lua
if SERVER then
    -- Register network strings for client-server communication
    util.AddNetworkString("DrGBaseChatPrint")
    -- Additional network strings registered by modules
end
```

### Phase 9: Base Entities

Garry's Mod automatically loads `lua/entities/` after autorun:
- Base classes loaded first (drgbase_nextbot, drgbase_weapon, etc.)
- Example NPCs loaded next
- Custom addon entities loaded last

## Load Order Dependencies

Understanding dependencies is crucial for troubleshooting initialization issues.

### Critical Dependencies

```
enumerations.lua → (defines constants used by all modules)
    ↓
entity_helpers.lua → (utilities used by nextbot system)
    ↓
meta extensions → (extend engine types before systems use them)
    ↓
modules → (utilities used by core systems)
    ↓
core systems → (nextbots.lua, weapons.lua, etc.)
    ↓
base entities → (drgbase_nextbot, drgbase_weapon)
    ↓
custom entities → (your NPCs and weapons)
```

### Module Dependencies

**Core Module Dependencies:**
- `nextbots.lua` depends on: `entity_helpers.lua`, `spawnmenu.lua`
- `spawners.lua` depends on: `nextbots.lua`, `entity_helpers.lua`
- `possession.lua` depends on: `weapons.lua`, networking
- `wrappers.lua` depends on: `entity_helpers.lua`

**No Circular Dependencies:**
DrGBase is designed to avoid circular dependencies. Each module can be loaded independently in the defined order.

## File Inclusion System

DrGBase provides custom file inclusion functions that handle realm separation automatically.

### DrGBase.IncludeFile(fileName)

**How it works:**
1. Extracts filename from path
2. Checks for realm prefix (`sv_`, `cl_`)
3. Handles file based on prefix:
   - **sv_**: Server-only (only included on SERVER realm)
   - **cl_**: Client-only (sent to clients, included on CLIENT realm)
   - **No prefix**: Shared (sent to clients and included on both)
4. Prints loading message to console
5. Returns the result of `include(fileName)`

**Realm Detection Logic:**
```lua
if string.StartWith(last, "sv_") then
    if SERVER then return include(fileName) end  -- Server only
elseif string.StartWith(last, "cl_") then
    if CLIENT then return include(fileName) end  -- Client only
else
    AddCSLuaFile(fileName)  -- Send to clients
    return include(fileName)  -- Run on both
end
```

**Key Points:**
- `AddCSLuaFile()`: Tells server to send file to clients
- `include()`: Actually loads and executes the file
- Server-only files are never sent to clients (security)

```lua
-- Server-only file
DrGBase.IncludeFile("sv_myfile.lua")  -- Only runs on server

-- Client-only file
DrGBase.IncludeFile("cl_myfile.lua")  -- Sent to client, runs there

-- Shared file
DrGBase.IncludeFile("sh_myfile.lua")  -- Runs on both, sent to client
```

### DrGBase.IncludeFolder(folder)

**How it works:**
1. Uses `file.Find(folder.."/*.lua", "LUA")` to get all Lua files
2. Loops through each file and calls `DrGBase.IncludeFile()`
3. Returns a table mapping filenames to their return values
4. Prints "Include folder" message to console

**Order of inclusion:**
Files are included in the order returned by `file.Find()`, which is typically alphabetical. This is usually sufficient since files in the same folder shouldn't depend on each other's load order.

**Usage:**
```lua
DrGBase.IncludeFolder("drgbase")          -- Load all core modules
DrGBase.IncludeFolder("drgbase/meta")     -- Load metatable extensions
DrGBase.IncludeFolder("drgbase/modules")  -- Load utility modules
```

### DrGBase.RecursiveInclude(folder)

**How it works:**
1. Calls `DrGBase.IncludeFolder(folder)` to load all files in current folder
2. Uses `file.Find(folder.."/*", "LUA")` to find subfolders
3. Recursively calls itself for each subfolder
4. Merges all return values into a single table

**Use cases:**
- Loading entire module hierarchies
- Loading entity folders with subfiles
- Useful when you have nested module organization

**Not used by default in DrGBase** because it explicitly loads specific folders (better control over load order).

## Realm Handling

DrGBase handles server and client initialization differently based on the realm.

### Server Initialization

**Server-specific actions:**
- Registers network strings: `util.AddNetworkString("DrGBaseChatPrint")`
- Loads server-only files (sv_* prefix)
- Sends client files to clients: `AddCSLuaFile()`
- Adds resource files: `resource.AddFile("materials/entities/...")`
- Precaches models and sounds
- Registers nextbot/weapon mixins (hooks interception)

### Client Initialization

**Client-specific actions:**
- Receives and loads files sent from server
- Registers language strings: `language.Add(class, PrintName)`
- Sets up killicons: `killicon.Add(class, icon, color)`
- Loads spawn menu integration
- Sets up UI panels and client-side rendering
- Displays "Hi! :)" message in console

### Shared Initialization

**Runs on both realms:**
- Creates DrGBase global table
- Loads core modules (enumerations, entity_helpers, etc.)
- Loads metatable extensions
- Loads utility modules
- Defines entity base classes
- Sets up configuration properties

## Network String Registration

Network strings must be registered before use for client-server communication:

```lua
if SERVER then
    util.AddNetworkString("DrGBaseChatPrint")
    -- Additional network strings registered by individual modules
end
```

**How it works:**
- Network strings are identifiers for network messages
- Must be registered on **server only** before sending
- Client automatically receives registered string list
- Attempting to use unregistered strings causes errors

**DrGBase Network Strings:**
- `DrGBaseChatPrint`: Used for server-to-client chat messages
- Additional strings registered by modules as needed

## Entity Registration

Entities register themselves to appear in spawn menus and integrate with DrGBase systems.

### Nextbots

Nextbots register using `DrGBase.AddNextbot(ENT)` called from entity file:

```lua
AddCSLuaFile()  -- Send to clients
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "My NPC"
ENT.Category = "DrGBase"
-- ... configuration ...
DrGBase.AddNextbot(ENT)  -- Register with framework
```

**What AddNextbot does:**
1. Validates PrintName and Category exist
2. Precaches models (if enabled): `util.PrecacheModel(model)`
3. Precaches sounds (if enabled): `util.PrecacheSound(sound)`
4. **Client:** Registers language string and killicon
5. **Server:** Adds resource files, sets up hook mixins
6. Adds to spawn menu list
7. Prints confirmation message

### Weapons

Weapons register using `DrGBase.AddWeapon(SWEP)`:

```lua
SWEP.Base = "drgbase_weapon"
SWEP.PrintName = "My Weapon"
SWEP.Category = "DrGBase"
DrGBase.AddWeapon(SWEP)
```

**What AddWeapon does:**
1. Validates PrintName and Category
2. Registers language string and killicon
3. Adds resource files for weapon icon
4. Adds to DrGBase weapons list
5. Integrates with spawn menu

### Spawners

Spawners register using `DrGBase.AddSpawner(ENT)`:

```lua
ENT.Base = "base_entity"
ENT.PrintName = "NPC Spawner"
ENT.Category = "DrGBase"
DrGBase.AddSpawner(ENT)
```

Similar registration process to nextbots.

## Initialization Hooks

Hooks are called at specific points during initialization to allow customization.

### Framework Hooks

**Client-Side:**
```lua
hook.Add("Initialize", "DrGBaseHello", function()
    DrGBase.Info("Hi! :)")
end)
```

This hook runs when the client finishes loading. DrGBase uses it to print a friendly message.

**Entity Registration Hooks:**
- `PopulateDrGBaseSpawnmenu`: Called to populate spawn menu with DrGBase entities

### Entity Hooks

When an entity spawns, these hooks are called in order:

1. **ENT:Initialize()** (Framework method)
   - Sets up core entity systems
   - Initializes AI, movement, weapons, etc.
   - Loads resources
   - Sets up network variables
   - **Calls ENT:CustomInitialize() at the end**

2. **ENT:CustomInitialize()** (Your override)
   - This is where you add custom spawn logic
   - Override this hook in your entity
   - Called automatically by Initialize()

Example:
```lua
function ENT:CustomInitialize()
    self:SetColor(Color(255, 0, 0))  -- Red NPC
    self:GiveWeapon("weapon_drg_ar2")
end
```

## Precaching

Precaching loads resources into memory at initialization to prevent lag during gameplay.

### Model Precaching

DrGBase automatically precaches all models defined in `ENT.Models`:

```lua
if PrecacheModels:GetBool() then
    for i, model in ipairs(ENT.Models or {}) do
        util.PrecacheModel(model)
    end
end
```

**What precaching does:**
- Loads model data into memory
- Prevents stuttering when model first appears
- Improves performance in multiplayer

### Sound Precaching

DrGBase automatically precaches sounds from sound tables:

```lua
if PrecacheSounds:GetBool() then
    for i, sounds in ipairs({
        ENT.OnSpawnSounds,
        ENT.OnIdleSounds,
        ENT.OnDamageSounds,
        ENT.OnDeathSounds
    }) do
        for h, soundName in ipairs(sounds) do
            util.PrecacheSound(soundName)
        end
    end
end
```

### ConVar Control

Two ConVars control precaching behavior:

```
drgbase_precache_models 1  // Default: 1 (enabled)
drgbase_precache_sounds 1  // Default: 1 (enabled)
```

**When to disable:**
- Testing with many NPCs (faster load times)
- Memory-constrained environments
- Development iteration

**Production recommendation:** Keep both enabled (1)

## Debugging Initialization

When troubleshooting initialization problems, the console is your primary tool.

### Console Output

DrGBase prints detailed loading information:

```
[DrGBase] Include folder 'drgbase'.
[DrGBase] Include file 'drgbase/colors.lua'.
[DrGBase] Include file 'drgbase/enumerations.lua'.
[DrGBase] Include file 'drgbase/entity_helpers.lua'.
...
[DrGBase] Include folder 'drgbase/meta'.
[DrGBase] Include file 'drgbase/meta/entity.lua'.
...
[DrGBase] Nextbot 'npc_drg_zombie': loaded.
[DrGBase] Weapon 'weapon_drg_ar2': loaded.
```

**What to look for:**
- Green `[DrGBase]` prefix indicates successful loading
- Red error messages indicate problems
- Load order: modules before entities
- All expected entities should show "loaded" messages

### Common Issues

**1. "attempt to index global 'DrGBase' (a nil value)"**
- **Cause:** Custom code runs before DrGBase initializes
- **Fix:** Ensure DrGBase loads first (alphabetically, "drgbase" comes early)

**2. "ENT.Base is nil" or inheritance issues**
- **Cause:** Base class not loaded yet
- **Fix:** Check that base entities load before derived entities

**3. Missing models/sounds in-game**
- **Cause:** Precaching disabled or paths incorrect
- **Fix:** Verify ConVars enabled, check model paths

**4. Network string errors**
- **Cause:** Network string used before registration
- **Fix:** Ensure strings registered in autorun, not entity init

**5. Client not receiving files**
- **Cause:** Missing `AddCSLuaFile()` call
- **Fix:** Use `DrGBase.IncludeFile()` which handles this automatically

## Lazy Loading

DrGBase does **not** use lazy loading - all modules are loaded at initialization.

**Why no lazy loading:**
- Source engine loads all entities at startup anyway
- Performance impact is negligible (~12k lines)
- Predictable load order
- Simpler debugging
- All systems available immediately

**Exception:** Spawn menu content is populated on-demand when the player opens the spawn menu (handled by Garry's Mod).

## Hot Reloading

During development, you can reload DrGBase without restarting the server.

### lua_openscript

Use console commands to reload files:

```lua
-- Server-side reload
lua_openscript autorun/drgbase.lua

-- Client-side reload (run on client or from server)
lua_openscript_cl autorun/drgbase.lua

-- Reload specific module
lua_openscript drgbase/nextbots.lua
```

### Easier Reload: Entity Refresh

For entity changes only:
```
-- In console
ent_remove npc_drg_zombie  // Remove all instances
// Then respawn from spawn menu
```

Or use the reload addon from Workshop.

### Caveats

**What doesn't hot-reload well:**
- **Metatable extensions**: Once added, hard to remove
- **Hook additions**: Multiple hooks may stack
- **Global state**: Variables persist between reloads
- **Network strings**: Can't re-register (restart required)
- **Entity base classes**: Changes require full restart

**Best practice for development:**
- Reload specific entity files instead of full framework
- Use `lua_openscript entities/npc_drg_zombie.lua`
- Restart server for core framework changes
- Use dev environment with fast restart times

---

**Previous:** [File Structure](./02-file-structure.md) | **Next:** [Module System](./04-module-system.md)
