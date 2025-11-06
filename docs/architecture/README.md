# Architecture

Understanding DrGBase's architecture will help you build better addons and troubleshoot issues effectively.

## Contents

1. **[Overview](./01-overview.md)**
   - High-level architecture
   - Component relationships
   - Data flow
   - Execution lifecycle

2. **[File Structure](./02-file-structure.md)**
   - Directory organization
   - File naming conventions
   - Module locations
   - Base classes

3. **[Initialization System](./03-initialization.md)**
   - Autorun entry point
   - Load order
   - Module initialization
   - Dependency management

4. **[Module System](./04-module-system.md)**
   - Core modules
   - Utility modules
   - Metatable extensions
   - Module communication

5. **[Client-Server Architecture](./05-client-server.md)**
   - Realm separation
   - Network communication
   - State synchronization
   - Shared code

6. **[Design Patterns](./06-design-patterns.md)**
   - Inheritance pattern
   - Hook system
   - Registry pattern
   - Factory pattern
   - Observer pattern

## Architecture Diagrams

### Component Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     DrGBase Framework                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Autorun   │  │    Core     │  │  Modules    │         │
│  │  Entry Point│──▶│  Functions  │──▶│  & Utils    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                               │
│  ┌─────────────────────────────────────────────────┐        │
│  │           Base Classes (Entities)                │        │
│  │  ┌──────────────┐  ┌──────────────┐            │        │
│  │  │  drgbase_    │  │  drgbase_    │            │        │
│  │  │  nextbot     │  │  weapon      │            │        │
│  │  └──────────────┘  └──────────────┘            │        │
│  └─────────────────────────────────────────────────┘        │
│                         ▲                                     │
│                         │                                     │
│  ┌──────────────────────┴────────────────────────┐          │
│  │        Custom Entities (User Code)             │          │
│  │  ┌──────────────┐  ┌──────────────┐          │          │
│  │  │  npc_drg_*   │  │ weapon_drg_* │          │          │
│  │  └──────────────┘  └──────────────┘          │          │
│  └─────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow Diagram

```
  Player Input
       │
       ▼
  ┌─────────┐         ┌──────────┐         ┌─────────┐
  │ Client  │────────▶│ Network  │────────▶│ Server  │
  │ (UI)    │◀────────│  Layer   │◀────────│  (AI)   │
  └─────────┘         └──────────┘         └─────────┘
       │                                          │
       ▼                                          ▼
  Rendering                               Game Logic
  Prediction                              Physics/AI
  Sounds                                  Damage/State
```

### Class Hierarchy

```
base_nextbot (Garry's Mod)
    │
    ├─▶ drgbase_nextbot (DrGBase Base)
    │       │
    │       ├─▶ drgbase_nextbot_human (Humanoid Variant)
    │       │       └─▶ npc_drg_* (User NPCs)
    │       │
    │       └─▶ drgbase_nextbot_sprite (2D Sprite)
    │               └─▶ npc_drg_testsprite
    │
weapon_base (Garry's Mod)
    │
    └─▶ drgbase_weapon (DrGBase Base)
            └─▶ weapon_drg_* (User Weapons)

base_entity (Garry's Mod)
    │
    ├─▶ proj_drg_default (DrGBase Projectile)
    │       └─▶ proj_drg_* (User Projectiles)
    │
    └─▶ spwn_drg_default (DrGBase Spawner)
            └─▶ spwn_drg_* (User Spawners)
```

## Quick Reference

### Load Order

1. `autorun/drgbase.lua` - Entry point
2. Core modules (`drgbase/*.lua`)
3. Metatable extensions (`drgbase/meta/*.lua`)
4. Utility modules (`drgbase/modules/*.lua`)
5. Base entities (`entities/drgbase_*/`)
6. Custom entities inherit from bases

### Key Files

- **Entry Point:** `lua/autorun/drgbase.lua`
- **Base Nextbot:** `lua/entities/drgbase_nextbot/shared.lua`
- **Base Weapon:** `lua/weapons/drgbase_weapon/shared.lua`
- **Networking:** `lua/drgbase/modules/net.lua`
- **AI System:** `lua/entities/drgbase_nextbot/ai.lua`

---

**For Implementation Details:** See [Core Systems](../systems/README.md)
**For API Details:** See [API Reference](../api/README.md)
