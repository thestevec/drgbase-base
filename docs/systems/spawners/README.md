# Spawner System

NPC spawning system.

**File:** `spawners.lua`

## Overview

The spawner system provides a registry for spawner entities and integration with the spawn menu. Spawners create NPCs at specified locations with customizable quantities and spawn areas.

**Key Features:**
- **Spawner Registration** - Register spawner entities for spawn menu
- **Spawn Menu Integration** - Automatic category organization
- **Programmatic Creation** - Create spawners via code

## Registration

```lua
ENT.PrintName = "Zombie Spawner"
ENT.Category = "DrGBase"
ENT.Spawnable = true

DrGBase.AddSpawner(ENT)
```

## Creating Spawners

```lua
-- Create spawner programmatically
local spawner = DrGBase.CreateSpawner(
    Vector(0, 0, 0),    -- Position
    "npc_zombie",       -- NPC class or table
    500,                -- Spawn radius
    5                   -- Max quantity
)
```
