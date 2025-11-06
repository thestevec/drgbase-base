# File Structure

## Complete Directory Tree

Below is the complete annotated directory structure of DrGBase:

```
drgbase/
├── lua/
│   ├── autorun/
│   │   └── drgbase.lua                      # Main entry point
│   │
│   ├── drgbase/                             # Core framework
│   │   ├── colors.lua                       # Color definitions
│   │   ├── dpanels.lua                      # UI panel utilities
│   │   ├── enumerations.lua                 # Constants & enums
│   │   ├── entity_helpers.lua               # Entity utility functions
│   │   ├── misc.lua                         # Miscellaneous utilities
│   │   ├── nextbots.lua                     # Nextbot registration
│   │   ├── nodegraph.lua                    # Node graph utilities
│   │   ├── particles.lua                    # Particle effects
│   │   ├── possession.lua                   # Possession mechanics
│   │   ├── resources.lua                    # Resource management
│   │   ├── spawners.lua                     # Spawner system
│   │   ├── spawnmenu.lua                    # Spawn menu integration
│   │   ├── weapons.lua                      # Weapon registry
│   │   ├── wrappers.lua                     # Entity wrappers
│   │   │
│   │   ├── meta/                            # Metatable extensions
│   │   │   ├── entity.lua                   # Entity extensions
│   │   │   ├── npc.lua                      # NPC extensions
│   │   │   ├── phys.lua                     # Physics extensions
│   │   │   ├── player.lua                   # Player extensions
│   │   │   └── vector.lua                   # Vector extensions
│   │   │
│   │   └── modules/                         # Utility modules
│   │       ├── coroutine.lua                # Coroutine helpers
│   │       ├── debugoverlay.lua             # Debug visualization
│   │       ├── math.lua                     # Math extensions
│   │       ├── navmesh.lua                  # Navigation helpers
│   │       ├── net.lua                      # Networking system
│   │       ├── render.lua                   # Rendering helpers
│   │       ├── string.lua                   # String extensions
│   │       ├── table.lua                    # Table extensions
│   │       ├── timer.lua                    # Timer system
│   │       └── util.lua                     # Utility functions
│   │
│   ├── entities/
│   │   ├── drgbase_entity.lua               # Base entity
│   │   │
│   │   ├── drgbase_nextbot/                 # Base nextbot (6,238 lines)
│   │   │   ├── shared.lua                   # Base config (621 lines)
│   │   │   ├── ai.lua                       # AI system (187 lines)
│   │   │   ├── animations.lua               # Animations (552 lines)
│   │   │   ├── awareness.lua                # Awareness (262 lines)
│   │   │   ├── behaviours.lua               # Behaviors (152 lines)
│   │   │   ├── blank.lua                    # Template (30 lines)
│   │   │   ├── detection.lua                # Detection (244 lines)
│   │   │   ├── hooks.lua                    # Hooks (300 lines)
│   │   │   ├── inventory.lua                # Inventory (30 lines)
│   │   │   ├── locomotion.lua               # Locomotion (106 lines)
│   │   │   ├── misc.lua                     # Misc (675 lines)
│   │   │   ├── movements.lua                # Movement (702 lines)
│   │   │   ├── path.lua                     # Pathfinding (158 lines)
│   │   │   ├── patrol.lua                   # Patrol (226 lines)
│   │   │   ├── possession.lua               # Possession (420 lines)
│   │   │   ├── relationships.lua            # Relationships (831 lines)
│   │   │   ├── status.lua                   # Health (205 lines)
│   │   │   └── weapons.lua                  # Weapons (537 lines)
│   │   │
│   │   ├── drgbase_nextbot_human/           # Human variant
│   │   ├── drgbase_nextbot_sprite/          # Sprite variant
│   │   │
│   │   ├── proj_drg_default/                # Base projectile
│   │   │
│   │   ├── npc_drg_*.lua                    # Example NPCs
│   │   ├── proj_drg_*.lua                   # Example projectiles
│   │   └── spwn_drg_*.lua                   # Spawner entities
│   │
│   ├── weapons/
│   │   ├── drgbase_weapon/                  # Base weapon
│   │   │   ├── shared.lua
│   │   │   ├── primary.lua
│   │   │   ├── secondary.lua
│   │   │   ├── misc.lua
│   │   │   └── meta.lua
│   │   │
│   │   ├── drgbase_possession.lua           # Possession weapon
│   │   ├── drgbase_possessor.lua            # Possessor weapon
│   │   ├── weapon_drg_*.lua                 # Example weapons
│   │   │
│   │   └── gmod_tool/stools/                # Developer tools
│   │       └── drgbase_tool_*.lua
│   │
│   └── effects/
│       └── drg_blood_explosion.lua          # Blood effect
│
├── materials/
│   ├── entities/                            # Entity icons
│   ├── drgbase/                             # Framework materials
│   └── weapons/                             # Weapon icons
│
└── particles/
    └── drgbase.pcf                          # Particle definitions
```

## File Naming Conventions

DrGBase uses a consistent naming convention system to organize code and control realm execution:

### Prefix System

#### Entity Files
- `drgbase_` - Framework base classes
- `npc_drg_` - Example/included NPCs
- `proj_drg_` - Example projectiles
- `spwn_drg_` - Spawner entities

#### Tool Files
- `drgbase_tool_` - Developer tools

#### Weapon Files
- `drgbase_` - Framework weapons
- `weapon_drg_` - Example weapons

### Realm Prefixes

Files can use realm prefixes in their names:
- `sv_*.lua` - Server-only files
- `cl_*.lua` - Client-only files
- No prefix - Shared files

The framework's `IncludeFile()` function handles these automatically.

## Module Organization

Modules are organized by functionality and purpose, with clear separation of concerns.

### Core Modules (`drgbase/`)

Each module handles a specific aspect of the framework:

| File | Purpose | Size | Realm |
|------|---------|------|-------|
| `colors.lua` | Color definitions for UI and console | 29 lines | Shared |
| `dpanels.lua` | DPanel utilities and UI helpers | 36 lines | Client |
| `enumerations.lua` | Constants, enums, factions | 44 lines | Shared |
| `entity_helpers.lua` | Entity utility functions | 206 lines | Shared |
| `misc.lua` | Miscellaneous framework utilities | 142 lines | Shared |
| `nextbots.lua` | Nextbot registration and precaching | 282 lines | Shared |
| `nodegraph.lua` | AI node graph utilities | 233 lines | Server |
| `particles.lua` | Particle effect management | 77 lines | Shared |
| `possession.lua` | Player possession mechanics | 225 lines | Shared |
| `resources.lua` | Resource management stub | 3 lines | Shared |
| `spawners.lua` | NPC spawner system | 78 lines | Shared |
| `spawnmenu.lua` | Spawn menu integration | 123 lines | Client |
| `weapons.lua` | Weapon registration system | 109 lines | Shared |
| `wrappers.lua` | Entity wrapper classes | 118 lines | Shared |

**Total:** 1,705 lines

### Metatable Extensions (`drgbase/meta/`)

Extend engine metatables with custom methods:

| File | Purpose | Size | Extends |
|------|---------|------|---------|
| `entity.lua` | Entity metatable extensions | 349 lines | Entity |
| `player.lua` | Player metatable extensions | 276 lines | Player |
| `vector.lua` | Vector metatable extensions | 201 lines | Vector |
| `phys.lua` | PhysObj metatable extensions | 51 lines | PhysObj |
| `npc.lua` | NPC metatable extensions | 17 lines | NPC |

**Total:** 894 lines

### Utility Modules (`drgbase/modules/`)

Reusable helper functions organized by category:

| File | Purpose | Size | Realm |
|------|---------|------|-------|
| `net.lua` | Simplified networking API | 115 lines | Shared |
| `util.lua` | General utility functions | 98 lines | Shared |
| `table.lua` | Table manipulation extensions | 55 lines | Shared |
| `coroutine.lua` | Coroutine management helpers | 30 lines | Shared |
| `debugoverlay.lua` | Debug visualization utilities | 28 lines | Shared |
| `render.lua` | Rendering helper functions | 18 lines | Client |
| `timer.lua` | Timer utilities | 17 lines | Shared |
| `math.lua` | Math extensions | 9 lines | Shared |
| `navmesh.lua` | Navigation mesh helpers | 9 lines | Server |
| `string.lua` | String extensions | 8 lines | Shared |

**Total:** 387 lines

## Base Classes

DrGBase provides three primary base classes for content creation:

### Nextbot Hierarchy

```
drgbase_nextbot (base)
├── drgbase_nextbot_human
├── drgbase_nextbot_sprite
└── Custom NPCs (your addons)
```

### Weapon Hierarchy

```
drgbase_weapon (base)
└── Custom Weapons (your addons)
```

### Projectile Hierarchy

```
proj_drg_default (base)
└── Custom Projectiles (your addons)
```

## File Sizes & Complexity

Understanding file sizes helps identify where most of the framework's functionality resides:

### Largest Files

1. `relationships.lua` - 831 lines
2. `movements.lua` - 702 lines
3. `misc.lua` - 675 lines
4. `shared.lua` - 621 lines
5. `animations.lua` - 552 lines
6. `weapons.lua` - 537 lines

### System Totals

- **Total Codebase:** ~12,808 lines across 87 Lua files
- **Base Nextbot:** 6,238 lines (48.6% of codebase)
- **Core Framework:** 1,705 lines (13.3% of codebase)
- **Metatable Extensions:** 894 lines (7.0% of codebase)
- **Utility Modules:** 387 lines (3.0% of codebase)
- **Weapons & Projectiles:** ~2,000 lines (15.6% of codebase)
- **Examples & Tools:** ~1,584 lines (12.4% of codebase)

## Adding Your Own Files

When creating custom content with DrGBase, follow these structure guidelines:

### Custom Addon Structure

**Recommended folder structure for a custom addon:**

```
your_addon/
├── lua/
│   ├── entities/
│   │   ├── npc_your_zombie.lua              # Simple NPC (single file)
│   │   └── npc_your_boss/                   # Complex NPC (folder)
│   │       ├── shared.lua                   # Configuration
│   │       ├── ai.lua                       # Custom AI logic
│   │       └── weapons.lua                  # Custom combat
│   ├── weapons/
│   │   └── weapon_your_rifle.lua            # Custom weapon
│   └── autorun/
│       └── your_addon_init.lua              # Optional initialization
├── materials/
│   ├── entities/
│   │   ├── npc_your_zombie.png              # Spawnmenu icon (64x64)
│   │   └── npc_your_boss.png
│   └── weapons/
│       └── weapon_your_rifle.png
└── sound/
    └── your_addon/
        ├── zombie_attack.wav
        └── rifle_shoot.wav
```

### Integration Points

Your code interfaces with DrGBase at these key points:

1. **Entity Base Class**
   ```lua
   ENT.Base = "drgbase_nextbot"  -- Inherit from DrGBase nextbot
   ```

2. **Registration** (optional, automatic for entities/)
   ```lua
   DrGBase.AddNextbot(ENT)  -- Register in spawn menu
   ```

3. **Hooks and Overrides**
   ```lua
   function ENT:CustomInitialize()  -- Extension point
   function ENT:OnTakeDamage(dmg)   -- Event hook
   ```

4. **Helper Functions**
   ```lua
   self:FindInCone(...)         -- Use metatable extensions
   DrGBase.FindInSphere(...)    -- Use global helpers
   ```

### File Naming Best Practices

- **Entity files**: `npc_yourmod_zombie.lua` (prefix with `npc_yourmod_`)
- **Weapon files**: `weapon_yourmod_rifle.lua`
- **Unique names**: Avoid conflicts with other addons
- **Lowercase**: Use lowercase with underscores

---

**Previous:** [Overview](./01-overview.md) | **Next:** [Initialization System](./03-initialization.md)
