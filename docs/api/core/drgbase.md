# DrGBase Global API

Global `DrGBase` table functions and properties.

**File:** `lua/autorun/drgbase.lua` and various module files

## Registration Functions

### DrGBase.AddNextbot(ENT)

Registers a nextbot entity with the DrGBase framework. Handles model and sound precaching, killicon registration, and spawnmenu integration.

**Realm:** 🟣 SHARED

**Parameters:**
- `ENT` (table) - Entity table with nextbot definition
  - Must have `PrintName` and `Category` properties
  - Optional: `Models` (table), `OnSpawnSounds`, `OnIdleSounds`, `OnDamageSounds`, `OnDeathSounds`
  - Optional: `Killicon` (table) with `icon` and `color` properties
  - Optional: `Spawnable` (boolean) - Defaults to true

**Returns:**
- `boolean` - true if registered successfully, false if PrintName or Category missing

**Example:**
```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "My Nextbot"
ENT.Category = "My NPCs"
ENT.Models = {"models/player.mdl"}
-- ... rest of entity definition ...
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Details:**
- On server: Precaches models and sounds (if convars enabled), adds mixins for hook handling
- On client: Adds language strings and killicons
- Automatically adds to spawnmenu under specified category
- Prints confirmation message when loaded

---

### DrGBase.AddWeapon(SWEP)

Registers a weapon with the DrGBase framework and adds it to the spawnmenu.

**Realm:** 🟣 SHARED

**Parameters:**
- `SWEP` (table) - Weapon table definition
  - Must have `PrintName` and `Category` properties
  - Optional: `Killicon` (table) with `icon` and `color` properties

**Returns:** None

**Example:**
```lua
SWEP.Base = "drgbase_weapon"
SWEP.PrintName = "My Weapon"
SWEP.Category = "DrGBase"
SWEP.Spawnable = true
-- ... weapon definition ...
DrGBase.AddWeapon(SWEP)
```

**Details:**
- On server: Adds weapon material to downloads
- On client: Registers language strings and killicons
- Adds to DrGBaseWeapons list for spawnmenu integration

---

### DrGBase.AddSpawner(ENT)

Registers a spawner entity with the framework.

**Realm:** 🟣 SHARED

**Parameters:**
- `ENT` (table) - Spawner entity table
  - Must have `PrintName` and `Category` properties
  - Optional: `Spawnable` (boolean) - Defaults to true

**Returns:**
- `boolean` - true if registered successfully, false if PrintName or Category missing

**Example:**
```lua
ENT.Base = "drgbase_spawner"
ENT.PrintName = "Custom Spawner"
ENT.Category = "DrGBase"
-- ... spawner definition ...
DrGBase.AddSpawner(ENT)
```

**Details:**
- Registers spawner in both NPC list and DrGBaseSpawners list
- Automatically appears in DrGBase spawnmenu tab

---

### DrGBase.AddParticles(pcf, particles)

Loads and precaches particle systems from a PCF file.

**Realm:** 🟣 SHARED

**Parameters:**
- `pcf` (string) - PCF filename (without path, automatically prefixed with "particles/")
- `particles` (string or table) - Particle system name(s) to precache

**Returns:** None

**Example:**
```lua
-- Single particle
DrGBase.AddParticles("myeffects.pcf", "my_particle_effect")

-- Multiple particles
DrGBase.AddParticles("myeffects.pcf", {
    "particle_one",
    "particle_two",
    "particle_three"
})
```

---

## File Inclusion Functions

### DrGBase.IncludeFile(fileName)

Intelligently includes a Lua file with automatic realm detection based on filename prefix.

**Realm:** 🟣 SHARED

**Parameters:**
- `fileName` (string) - File path relative to lua/

**Returns:**
- `any` - Return value from the included file (server-side for sv_ files, client-side for cl_ files, both for shared)

**Behavior:**
- Files prefixed with `sv_` only run on server
- Files prefixed with `cl_` are sent to client via AddCSLuaFile and run only on client
- Files with no prefix are shared: AddCSLuaFile'd and run on both realms

**Example:**
```lua
DrGBase.IncludeFile("drgbase/mymodule.lua")      -- Shared (runs on both)
DrGBase.IncludeFile("drgbase/sv_server.lua")     -- Server only
DrGBase.IncludeFile("drgbase/cl_client.lua")     -- Client only (sent to client)
```

**Notes:**
- Automatically prints inclusion message to console
- More convenient than manually checking SERVER/CLIENT and calling AddCSLuaFile

---

### DrGBase.IncludeFiles(fileNames)

Includes multiple files and returns their results in a table.

**Realm:** 🟣 SHARED

**Parameters:**
- `fileNames` (table) - Array of file paths relative to lua/

**Returns:**
- `table` - Table mapping filenames to their return values

**Example:**
```lua
local modules = DrGBase.IncludeFiles({
    "mymodule/core.lua",
    "mymodule/helpers.lua",
    "mymodule/config.lua"
})
-- Access return values: modules["mymodule/core.lua"]
```

---

### DrGBase.IncludeFolder(folder)

Includes all Lua files in a folder (non-recursive).

**Realm:** 🟣 SHARED

**Parameters:**
- `folder` (string) - Folder path relative to lua/

**Returns:**
- `table` - Table mapping filenames to their return values

**Example:**
```lua
DrGBase.IncludeFolder("drgbase/meta")  -- Includes all .lua files in drgbase/meta/
```

**Details:**
- Only includes files directly in the specified folder (not subfolders)
- Uses file.Find to discover Lua files
- Each file is processed through DrGBase.IncludeFile

---

### DrGBase.RecursiveInclude(folder)

Recursively includes all Lua files in a folder and all its subfolders.

**Realm:** 🟣 SHARED

**Parameters:**
- `folder` (string) - Folder path relative to lua/

**Returns:**
- `table` - Table mapping all filenames to their return values

**Example:**
```lua
DrGBase.RecursiveInclude("drgbase")  -- Includes all .lua files in drgbase/ and subdirectories
```

**Details:**
- Recursively traverses all subdirectories
- Merges results from all folders into a single table
- Useful for loading entire module hierarchies

---

## Output Functions

### DrGBase.Print(msg, options)

Prints a formatted message to console or chat with DrGBase branding.

**Realm:** 🟣 SHARED

**Parameters:**
- `msg` (string) - Message to print
- `options` (table, optional) - Formatting options:
  - `chat` (boolean) - If true, prints to chat instead of console
  - `title` (Color) - Color for "[DrGBase]" prefix (default: green on client, cyan on server)
  - `color` (Color) - Color for the message text (default: white)
  - `player` (Player) - If chat is true on server, send to specific player (otherwise broadcasts)

**Returns:** None

**Example:**
```lua
-- Simple console print
DrGBase.Print("Hello World")

-- Colored console message
DrGBase.Print("Warning!", {color = DrGBase.CLR_ORANGE})

-- Chat message (on server, sends to all players)
DrGBase.Print("Server announcement", {chat = true})

-- Chat message to specific player
DrGBase.Print("Private message", {chat = true, player = ply})

-- Custom title color
DrGBase.Print("Custom", {title = DrGBase.CLR_PURPLE})
```

**Details:**
- On server with chat option: Uses networking to display in client chat
- Automatically formats with "[DrGBase]" prefix
- Default colors: green title for info, cyan for server messages, orange for client

---

### DrGBase.Info(msg, options)

Prints an info message with green "[DrGBase]" prefix.

**Realm:** 🟣 SHARED

**Parameters:**
- `msg` (string) - Message to print
- `options` (table, optional) - Same options as DrGBase.Print (title color is forced to green)

**Returns:** None

**Example:**
```lua
DrGBase.Info("Module loaded successfully")
DrGBase.Info("Entity spawned", {chat = true})
```

---

### DrGBase.Error(msg, options)

Prints an error message in red.

**Realm:** 🟣 SHARED

**Parameters:**
- `msg` (string) - Error message to print
- `options` (table, optional) - Same options as DrGBase.Print (color is forced to red)

**Returns:** None

**Example:**
```lua
DrGBase.Error("Failed to load configuration file")
DrGBase.Error("Invalid entity", {chat = true})
```

---

### DrGBase.ErrorInfo(msg, options)

Prints an error message in red with green "[DrGBase]" prefix.

**Realm:** 🟣 SHARED

**Parameters:**
- `msg` (string) - Error message to print
- `options` (table, optional) - Same options as DrGBase.Print (title green, color red)

**Returns:** None

**Example:**
```lua
DrGBase.ErrorInfo("Invalid parameter: expected string, got nil")
```

---

## Entity Creation Functions (SERVER)

### DrGBase.CreateSpawner(pos, tospawn, radius, quantity, class)

Creates and configures a spawner entity programmatically.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector) - Position to place the spawner
- `tospawn` (string or table) - Entity class name(s) to spawn
  - If string: Single entity class
  - If table: Key-value pairs of {class = count}
- `radius` (number) - Spawn radius
- `quantity` (number) - Maximum number of entities to spawn
- `class` (string, optional) - Spawner class to use (default: "spwn_drg_default")

**Returns:**
- `Entity` - The created spawner entity, or NULL if creation failed

**Example:**
```lua
-- Simple spawner
local spawner = DrGBase.CreateSpawner(
    Vector(0, 0, 0),
    "npc_drg_zombie",
    500,
    10
)

-- Multi-entity spawner
local spawner = DrGBase.CreateSpawner(
    Vector(100, 200, 50),
    {
        ["npc_drg_zombie"] = 5,
        ["npc_drg_headcrab"] = 3
    },
    1000,
    8
)
```

---

### DrGBase.CreateProjectile(model, binds)

Creates a customizable projectile entity with callback hooks.

**Realm:** 🔴 SERVER

**Parameters:**
- `model` (string or table) - Model path(s) for the projectile
  - If table: Randomly selects one model from the array
- `binds` (table, optional) - Callback functions:
  - `Init` (function) - Called on projectile initialization
  - `Think` (function) - Called every tick
  - `Contact` (function) - Called when projectile hits something
  - `Use` (function) - Called when projectile is used
  - `DealtDamage` (function) - Called when projectile deals damage
  - `TakeDamage` (function) - Called when projectile takes damage
  - `Remove` (function) - Called when projectile is removed

**Returns:**
- `Entity` - The created projectile entity, or NULL if creation failed

**Example:**
```lua
local proj = DrGBase.CreateProjectile("models/weapons/w_missile.mdl", {
    Init = function(self)
        self:SetCollisionGroup(COLLISION_GROUP_PROJECTILE)
        self:PhysicsInit(SOLID_VPHYSICS)
    end,
    Contact = function(self, data)
        local effectdata = EffectData()
        effectdata:SetOrigin(data.HitPos)
        util.Effect("Explosion", effectdata)
        self:Remove()
    end,
    Think = function(self)
        -- Custom projectile behavior
    end
})

-- Set projectile properties
proj:SetPos(startPos)
proj:SetAngles(angle)
local phys = proj:GetPhysicsObject()
if IsValid(phys) then
    phys:SetVelocity(direction * 2000)
end
```

---

### DrGBase.ParticleEffect(effect, data)

Creates a particle effect entity with advanced configuration options.

**Realm:** 🔴 SERVER

**Parameters:**
- `effect` (string) - Particle effect name
- `data` (table) - Configuration:
  - `pos` (Vector) - Position
  - `ang` (Angle) - Angles
  - `active` (boolean) - Start active (default: true)
  - `parent` (Entity) - Parent entity
  - `attachment` (string) - Attachment point name
  - `keepoffset` (boolean) - Maintain offset when parenting
  - `cpoints` (table) - Control point data (array of tables with same structure)

**Returns:**
- `Entity` - The created particle system entity, or NULL if creation failed

**Example:**
```lua
-- Simple particle at position
local particle = DrGBase.ParticleEffect("explosion_huge", {
    pos = Vector(0, 0, 0),
    ang = Angle(0, 0, 0)
})

-- Particle attached to entity
local particle = DrGBase.ParticleEffect("fire_jet", {
    parent = ent,
    attachment = "muzzle"
})

-- Particle with control points
local particle = DrGBase.ParticleEffect("beam_effect", {
    pos = startPos,
    cpoints = {
        {pos = endPos}  -- Control point 1
    }
})
```

---

### DrGBase.SimpleParticleEffect(effect, arg1, arg2)

Simplified particle creation with automatic parameter detection.

**Realm:** 🔴 SERVER

**Parameters:**
- `effect` (string) - Particle effect name
- `arg1` (Entity or Vector) - Parent entity OR position
- `arg2` (string or Angle) - Attachment name (if arg1 is Entity) OR angles (if arg1 is Vector)

**Returns:**
- `Entity` - The created particle system entity

**Example:**
```lua
-- Particle at entity attachment
DrGBase.SimpleParticleEffect("smoke_effect", weapon, "muzzle")

-- Particle at world position
DrGBase.SimpleParticleEffect("explosion", Vector(100, 200, 50), Angle(0, 0, 0))
```

---

## Utility Functions

### DrGBase.GetNextbots()

Returns a table of all currently spawned DrGBase nextbots.

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:**
- `table` - Table of spawned DrGBase nextbot entities

**Example:**
```lua
local bots = DrGBase.GetNextbots()
for i, bot in ipairs(bots) do
    print(bot:GetClass(), bot:Health())
end
```

---

### DrGBase.IsMeleeWeapon(weapon)

Checks if a weapon is considered a melee weapon based on hold type and properties.

**Realm:** 🟣 SHARED

**Parameters:**
- `weapon` (Weapon) - Weapon entity to check

**Returns:**
- `boolean` - true if weapon is melee

**Example:**
```lua
local wep = ent:GetActiveWeapon()
if DrGBase.IsMeleeWeapon(wep) then
    print("Entity is using a melee weapon")
end
```

**Details:**
- Checks hold type for: "melee", "melee2", "fist", "knife"
- Also checks weapon.DrGBase_Melee property
- Checks if hold type contains "melee"

---

### DrGBase.IsTarget(ent)

Determines if an entity is a valid target for AI targeting.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to check

**Returns:**
- `boolean` - true if entity can be targeted

**Example:**
```lua
for _, ent in ipairs(ents.FindInSphere(pos, 1000)) do
    if DrGBase.IsTarget(ent) then
        -- This entity can be targeted by NPCs
    end
end
```

**Details:**
- Excludes template NPCs (SF_NPC_TEMPLATE flag)
- Blacklists: bullseye, grenades, tripmines, satchels, grubs, cockroaches
- Whitelists: replicator entities
- Returns true for: entities with DrGBase_Target property, NextBots, Players, NPCs

---

### DrGBase.CanAttack(ent)

Determines if an entity can be attacked (includes non-living entities like physics props).

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to check

**Returns:**
- `boolean` - true if entity can be attacked

**Example:**
```lua
if DrGBase.CanAttack(target) then
    self:Attack(target)
end
```

**Details:**
- Excludes players who are possessing NPCs
- Returns true for all valid targets (via DrGBase.IsTarget)
- Also returns true for entities with valid physics objects

---

### DrGBase.Blind()

Creates a new blind data object for blinding effects.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `BlindData` - Blind data object with methods:
  - `:GetDuration()` - Returns duration in seconds
  - `:SetDuration(duration)` - Sets duration
  - `:ScaleDuration(scale)` - Multiplies duration by scale
  - `:GetAttacker()` - Returns attacker entity
  - `:SetAttacker(attacker)` - Sets attacker
  - `:GetInflictor()` - Returns inflictor entity
  - `:SetInflictor(inflictor)` - Sets inflictor

**Example:**
```lua
local blind = DrGBase.Blind()
blind:SetDuration(5)
blind:SetAttacker(attacker)
blind:SetInflictor(weapon)

-- Apply to target (assuming target has :Blind method)
target:Blind(blind)
```

---

### DrGBase.Material(name, ...)

Cached material loading for client-side rendering.

**Realm:** 🔵 CLIENT

**Parameters:**
- `name` (string) - Material path
- `...` - Additional Material() parameters

**Returns:**
- `IMaterial` - Cached material object

**Example:**
```lua
local mat = DrGBase.Material("models/debug/debugwhite")
surface.SetMaterial(mat)
surface.DrawTexturedRect(x, y, w, h)
```

**Details:**
- Caches materials to avoid repeated Material() calls
- Improves performance for frequently used materials
- Returns cached version on subsequent calls

---

## Spawnmenu Functions (CLIENT)

### DrGBase.GetIcon(name)

Gets a custom icon for a spawnmenu category.

**Realm:** 🔵 CLIENT

**Parameters:**
- `name` (string) - Category name

**Returns:**
- `string` or `nil` - Icon path, or nil if not set

**Example:**
```lua
local icon = DrGBase.GetIcon("My Category")
```

---

### DrGBase.SetIcon(name, icon)

Sets a custom icon for a spawnmenu category.

**Realm:** 🔵 CLIENT

**Parameters:**
- `name` (string) - Category name
- `icon` (string) - Icon path (e.g., "icon16/star.png")

**Returns:** None

**Example:**
```lua
DrGBase.SetIcon("My NPCs", "icon16/user.png")
```

---

### DrGBase.DListView(columns, options)

Creates a DListView panel with optional convar synchronization.

**Realm:** 🔵 CLIENT

**Parameters:**
- `columns` (table) - Array of column names
- `options` (table, optional):
  - `convar` (string) - ConVar name for data persistence (stored as JSON)

**Returns:**
- `Panel` - DListView panel

**Example:**
```lua
local list = DrGBase.DListView({"Name", "Value", "Status"}, {
    convar = "my_addon_listdata"
})

-- Add lines (automatically saved to convar)
list:AddLine("Item 1", "100", "Active")
list:AddLine("Item 2", "200", "Inactive")

-- Remove line by ID
list:RemoveLine(1)

-- Clear all
list:Clear()
```

**Details:**
- When convar is specified, list data is automatically saved/loaded as JSON
- Data persists across game sessions
- Updates automatically when convar changes

---

## Nodegraph Functions

### DrGBase.GetNodegraph()

Returns the internal AI nodegraph table.

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:**
- `table` - Array of Node objects

**Example:**
```lua
local nodes = DrGBase.GetNodegraph()
print("Total nodes:", #nodes)
```

**Notes:**
- Nodegraph is loaded from .ain files on server
- Each node has position, type, and links to other nodes

---

### DrGBase.ClosestNode(pos)

Finds the closest nodegraph node to a position.

**Realm:** 🟣 SHARED

**Parameters:**
- `pos` (Vector) - Position to search from

**Returns:**
- `Node` or `nil` - Closest node object

**Example:**
```lua
local node = DrGBase.ClosestNode(self:GetPos())
if node then
    print("Closest node:", node:GetID(), node:GetPos())
end
```

---

### DrGBase.NodegraphAstar(pos, goal, callback)

Performs A* pathfinding along the nodegraph.

**Realm:** 🟣 SHARED

**Parameters:**
- `pos` (Vector) - Start position
- `goal` (Vector) - Goal position
- `callback` (function, optional) - Custom cost function(node1, node2) -> cost

**Returns:**
- `table` - Array of Vector positions forming the path
- `boolean` - Success status

**Example:**
```lua
local path, success = DrGBase.NodegraphAstar(
    self:GetPos(),
    target:GetPos()
)

if success then
    for i, pos in ipairs(path) do
        -- Follow path positions
    end
end
```

**Notes:**
- Automatically finds closest nodes to start and goal positions
- Last position in path is adjusted to exact goal position
- Uses DrGBase.Astar internally with nodegraph neighbors

---

### DrGBase.RefreshNodegraph()

Reloads the nodegraph from the current map's .ain file.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Example:**
```lua
-- After map change or nodegraph update
DrGBase.RefreshNodegraph()
```

---

### DrGBase.BroadcastNodegraph()

Broadcasts the nodegraph to all connected clients.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Notes:**
- Currently disabled in code (commented out)
- Used for client-side nodegraph visualization

---

### DrGBase.SendNodegraph(ply)

Sends the nodegraph to a specific player.

**Realm:** 🔴 SERVER

**Parameters:**
- `ply` (Player) - Player to send to

**Returns:** None

**Notes:**
- Currently disabled in code (commented out)
- Called automatically on PlayerInitialSpawn

---

## Global Properties

### DrGBase.Icon

Path to the DrGBase icon used in spawnmenus and UI.

**Type:** `string`

**Value:** `"drgbase/icon16.png"`

**Example:**
```lua
print("DrGBase icon:", DrGBase.Icon)
```

---

### DrGBase.DefaultFootsteps

Table mapping material types to footstep sound arrays for nextbot footstep sounds.

**Type:** `table`

**Structure:**
```lua
DrGBase.DefaultFootsteps[MAT_TYPE] = {
    "sound1.wav",
    "sound2.wav",
    -- ...
}
```

**Example:**
```lua
-- Get footsteps for concrete
local sounds = DrGBase.DefaultFootsteps[MAT_CONCRETE]
local footstep = sounds[math.random(#sounds)]
self:EmitSound(footstep)
```

**Supported Materials:**
- MAT_CONCRETE, MAT_DIRT, MAT_FLESH, MAT_GRATE, MAT_METAL
- MAT_SAND, MAT_TILE, MAT_GRASS, MAT_VENT, MAT_WOOD
- MAT_GLASS, MAT_SLOSH (water), MAT_PLASTIC, MAT_SNOW
- And more (see source for complete list)

---

## See Also

- [Colors](./colors.md) - DrGBase color constants
- [Entity Helpers](./entity-helpers.md) - Entity utility functions
- [Enumerations](./enumerations.md) - DrGBase enumerations
