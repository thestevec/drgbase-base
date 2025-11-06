# Architecture Overview

## High-Level Architecture

DrGBase uses a layered, modular architecture built on top of Garry's Mod's Source engine framework. The system is designed around four key architectural principles:

### Layered Architecture (Core → Base → Custom)

DrGBase follows a three-layer architecture pattern:
- **Core Layer**: Framework utilities, helpers, and global functions
- **Base Layer**: Abstract entity classes (nextbots, weapons, projectiles)
- **Custom Layer**: User-created entities that inherit from base classes

This layering allows developers to work at the appropriate level of abstraction.

### Module-Based Design

The framework is organized into independent, focused modules:
- Core modules (`drgbase/*.lua`) - Framework-level functionality
- Metatable extensions (`drgbase/meta/*.lua`) - Extend engine types
- Utility modules (`drgbase/modules/*.lua`) - Helper functions

Each module is self-contained and has clear responsibilities.

### Event-Driven System

DrGBase heavily uses hooks and callbacks for customization:
- Override hooks like `CustomInitialize()`, `CustomThink()`
- Event hooks like `OnTakeDamage()`, `OnNewEnemy()`, `OnDeath()`
- Engine hooks integrated with custom logic

This allows developers to extend behavior without modifying core code.

### Network Architecture

Built on Source engine's client-server architecture:
- Server handles all simulation (AI, physics, damage)
- Client handles rendering and UI
- Network variables (NW2) automatically synchronize state
- Custom network messages for events and commands

## Component Relationships

The framework components are organized in a hierarchical dependency structure:

```
┌─────────────────────────────────────────────────────────┐
│                    Custom Layer                         │
│  (Your NPCs, Weapons, Projectiles)                     │
└──────────────────────┬──────────────────────────────────┘
                       │ Inherits from
┌──────────────────────▼──────────────────────────────────┐
│                     Base Layer                          │
│  drgbase_nextbot, drgbase_weapon, proj_drg_default     │
└──────────────────────┬──────────────────────────────────┘
                       │ Uses
┌──────────────────────▼──────────────────────────────────┐
│                    Core Layer                           │
│  DrGBase global, utilities, helpers, modules            │
└──────────────────────┬──────────────────────────────────┘
                       │ Built on
┌──────────────────────▼──────────────────────────────────┐
│                   Source Engine                         │
│  Garry's Mod, Lua 5.1, Game Engine                     │
└─────────────────────────────────────────────────────────┘
```

### Core Layer
The Core Layer provides framework-level functionality:
- **DrGBase global table**: Main namespace and API entry point
- **Utilities**: Helper functions for common operations
- **Module system**: File loading and inclusion
- **Metatable extensions**: Extend Entity, Player, NPC, Vector, PhysObj
- **Registration system**: Register entities, weapons, spawners

### Base Layer
The Base Layer provides abstract entity classes:
- **drgbase_nextbot**: Base NPC class with AI, movement, combat
- **drgbase_weapon**: Base weapon class with primary/secondary fire
- **proj_drg_default**: Base projectile class with physics and damage

### Extension Layer
The Extension Layer is where developers create content:
- **Custom NPCs**: Inherit from base nextbot classes
- **Custom Weapons**: Inherit from drgbase_weapon
- **Custom Projectiles**: Inherit from proj_drg_default

### Support Layer
The Support Layer provides development tools and systems:
- **Developer Tools**: STool integrations for testing
- **Spawners**: Entity spawner system
- **UI**: Spawn menu integration and panels

## System Components

DrGBase is composed of five major component systems:

### 1. Core Framework (`drgbase/`)
- **Purpose:** Provide framework-level functionality and global API
- **Key Files:**
  - `enumerations.lua` (44 lines) - Constants and enums
  - `entity_helpers.lua` (206 lines) - Entity utility functions
  - `nextbots.lua` (282 lines) - Nextbot registration system
  - `nodegraph.lua` (233 lines) - AI node graph utilities
  - `possession.lua` (225 lines) - Player possession mechanics
  - `misc.lua` (142 lines) - Miscellaneous utilities
  - `weapons.lua` (109 lines) - Weapon registration
  - `spawners.lua` (78 lines) - Spawner system
- **Responsibilities:**
  - Global DrGBase table and API
  - Entity registration and precaching
  - Spawn menu integration
  - Possession system management
  - AI node graph utilities
  - Framework utilities and helpers

### 2. Entity System (`entities/`)
- **Purpose:** Base entity classes for NPCs, projectiles, and spawners
- **Key Files:**
  - `drgbase_nextbot/` (6,238 lines total) - Base NPC class
  - `drgbase_nextbot_human/` - Human-specific variant
  - `drgbase_nextbot_sprite/` - Sprite-based variant
  - `proj_drg_default/` - Base projectile class
  - `spwn_drg_default.lua` - Base spawner entity
- **Responsibilities:**
  - AI logic and behavior
  - Movement and pathfinding
  - Combat and weapons
  - Animations and effects
  - Relationships and factions
  - Detection and awareness
  - Health and status

### 3. Weapon System (`weapons/`)
- **Purpose:** Base weapon classes and possession tools
- **Key Files:**
  - `drgbase_weapon/` - Base SWEP class
  - `drgbase_possession.lua` - Possession weapon
  - `drgbase_possessor.lua` - Possessor tool
  - `gmod_tool/stools/drgbase_tool_*.lua` - Developer tools
- **Responsibilities:**
  - Primary and secondary fire modes
  - Weapon holding and animations
  - Possession mechanics
  - Developer testing tools

### 4. Metatable Extensions (`drgbase/meta/`)
- **Purpose:** Extend engine metatables with custom functionality
- **Key Files:**
  - `entity.lua` (349 lines) - Entity metatable extensions
  - `player.lua` (276 lines) - Player metatable extensions
  - `vector.lua` (201 lines) - Vector metatable extensions
  - `phys.lua` (51 lines) - PhysObj metatable extensions
  - `npc.lua` (17 lines) - NPC metatable extensions
- **Responsibilities:**
  - Add helper methods to engine types
  - Provide convenient utility functions
  - Extend entity, player, NPC, vector, and physics object capabilities

### 5. Utility Modules (`drgbase/modules/`)
- **Purpose:** Reusable helper functions and extensions
- **Key Files:**
  - `net.lua` (115 lines) - Networking utilities
  - `util.lua` (98 lines) - General utilities
  - `table.lua` (55 lines) - Table extensions
  - `coroutine.lua` (30 lines) - Coroutine helpers
  - `debugoverlay.lua` (28 lines) - Debug visualization
  - `render.lua` (18 lines) - Rendering helpers
  - `timer.lua` (17 lines) - Timer utilities
- **Responsibilities:**
  - Simplified networking API
  - Math and geometry utilities
  - Table and string manipulation
  - Debug visualization
  - Coroutine management

## Data Flow

Data flows through DrGBase in three primary patterns: initialization, runtime, and event handling.

### Initialization Flow

```
Autorun
  ↓
Core Modules Load
  ↓
Metatable Extensions
  ↓
Utility Modules
  ↓
Base Entities Register
  ↓
Custom Entities Inherit & Register
  ↓
System Ready
```

### Runtime Flow

```
Entity Spawns
  ↓
Initialize() Called
  ↓
Think() Loop Begins
  ↓
Hook System Processes Events
  ↓
AI/Movement/Combat Systems Update
  ↓
Network State Synced to Clients
```

### Event Flow

```
Game Event (e.g., Take Damage)
  ↓
Engine Calls Hook
  ↓
DrGBase Processes
  ↓
Custom Hook Called
  ↓
Result Returned to Engine
```

## Execution Lifecycle

Entities in DrGBase follow a well-defined lifecycle from spawn to removal:

### Nextbot Lifecycle

1. **Spawn**
   - `ENT:Initialize()`
   - `ENT:CustomInitialize()`
   - Resources loaded
   - Network vars initialized

2. **Active**
   - `ENT:Think()` - Every tick
   - `ENT:CustomThink()` - Per-entity logic
   - AI updates
   - Movement updates
   - Combat system updates

3. **Events**
   - `ENT:OnTakeDamage()`
   - `ENT:OnNewEnemy()`
   - `ENT:OnMeleeAttack()`
   - Custom hooks

4. **Death**
   - `ENT:OnDeath()`
   - Ragdoll creation
   - Cleanup
   - `ENT:OnRemove()`

### Weapon Lifecycle

1. **Deploy**
   - `SWEP:Initialize()`
   - Resources loaded
   - `SWEP:Deploy()`

2. **Active**
   - `SWEP:Think()` - Every tick
   - `SWEP:PrimaryAttack()` - On primary fire
   - `SWEP:SecondaryAttack()` - On secondary fire
   - `SWEP:Reload()` - On reload

3. **Holster**
   - `SWEP:Holster()`
   - Cleanup

### Projectile Lifecycle

1. **Spawn**
   - `ENT:Initialize()`
   - Physics initialized
   - Velocity applied

2. **Flight**
   - `ENT:Think()` - Every tick
   - `ENT:PhysicsCollide()` - On collision
   - Trail/effects rendering

3. **Impact**
   - Damage dealt
   - Impact effects
   - `ENT:Remove()` or bounce

## Threading Model

DrGBase runs on Source engine's single-threaded model with coroutines for complex behaviors:

### Think Loop
- Runs on server every tick
- Updates AI, movement, combat

### Coroutines
- Used for complex behaviors
- Non-blocking operations
- Sequential action execution

### Network Updates
- Automatic state synchronization
- Manual network messages for events

## Memory Management

DrGBase follows Lua's garbage collection model with specific patterns for game entities:

### Entity References
Entities reference each other primarily through:
- Entity handles (validated with `IsValid()` before use)
- Weak references in tables (cleared when entity is removed)
- Enemy/target tracking (automatically cleared on entity removal)

Best practice: Always validate entities with `IsValid()` before accessing them.

### Cleanup
Cleanup occurs automatically and manually:
- **Automatic**: Lua garbage collector handles unused tables/data
- **Manual**: `ENT:OnRemove()` hook for cleanup on entity removal
- **Timer cleanup**: Timers use unique names to prevent leaks
- **Network cleanup**: Network messages cleaned up on disconnect

### Resource Management
Resources are precached during initialization:
- **Models**: `util.PrecacheModel()` called for all NPC models
- **Sounds**: `util.PrecacheSound()` for sound tables
- **Materials**: Automatically precached via `resource.AddFile()`
- **ConVars**: `drgbase_precache_models` and `drgbase_precache_sounds` control precaching

## Performance Considerations

DrGBase is designed with performance in mind for multiplayer environments:

### Update Frequency
- **Every tick (Think)**:
  - AI decision making
  - Movement updates
  - Target validation
  - Weapon think logic

- **Less frequent** (throttled or conditional):
  - Pathfinding (only when needed)
  - Detection scans (field-of-view checks)
  - Relationship queries (cached results)
  - Sound detection (event-based)

### Network Optimization
Network traffic is minimized through:
- **NW2 variables**: Only changed values are sent
- **Selective updates**: Clients only receive relevant state
- **Event-based messaging**: Network messages only for significant events
- **Client prediction**: Animations predicted client-side when possible

### Computational Optimization
Expensive operations are optimized:
- **Pathfinding**: Cached paths, recomputed only when invalidated
- **Line of sight**: Spatial partitioning for entity queries
- **Relationship lookups**: Results cached per entity pair
- **Entity queries**: Use radius-based searches instead of all entities
- **Coroutines**: Break up long operations across multiple frames

## Extension Points

DrGBase provides multiple extension points for customization without modifying core code:

### Hook System
Developers extend behavior by overriding hooks:
- **Initialize hooks**: `CustomInitialize()`, `CustomPostInitialize()`
- **Think hooks**: `CustomThink()`, `CustomThinkAI()`
- **Damage hooks**: `OnTakeDamage()`, `OnDeath()`, `OnDamageEntity()`
- **Combat hooks**: `OnMeleeAttack()`, `OnRangeAttack()`, `OnReload()`
- **AI hooks**: `OnNewEnemy()`, `OnLostEnemy()`, `OnUpdateRelationship()`
- **Movement hooks**: `OnNavAreaChanged()`, `OnLeaveGround()`, `OnLandOnGround()`
- **Animation hooks**: `OnAnimEvent()`, `OnSequenceFinished()`

See [Nextbot Hooks API](../api/nextbot/hooks.md) for complete list.

### Custom Modules
Add your own modules to extend functionality:
```lua
-- my_module.lua in drgbase/modules/
if not DrGBase then return end

function DrGBase.MyCustomFunction()
    -- Your code
end
```

### Metatable Extensions
Extend existing engine metatables:
```lua
-- Extend Entity metatable
local ENT = FindMetaTable("Entity")
function ENT:MyHelper()
    -- Custom helper method
end
```

## Dependencies

DrGBase has minimal external dependencies:

### Required
- **Garry's Mod**: The game itself (Source Engine)
- **Lua 5.1+**: Scripting language (included with Garry's Mod)

### Optional
- **Navigation meshes**: Required for advanced pathfinding (generated in-game)
- **Source SDK models**: For using HL2/HL2:EP1/EP2 models in NPCs

### No External Libraries
DrGBase is self-contained and doesn't require external Lua libraries or dependencies beyond what Garry's Mod provides.

---

**Next:** [File Structure](./02-file-structure.md)
