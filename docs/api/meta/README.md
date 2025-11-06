# Metatable Extensions API

Extensions to Garry's Mod metatables.

**Location:** `lua/drgbase/meta/`

## Files

- **[entity.md](./entity.md)** - Entity metatable extensions
- **[npc.md](./npc.md)** - NPC metatable extensions
- **[player.md](./player.md)** - Player metatable extensions
- **[physobj.md](./physobj.md)** - PhysObj metatable extensions
- **[vector.md](./vector.md)** - Vector metatable extensions

## Overview

DrGBase extends several Garry's Mod metatables with additional helper functions. These extensions add DrGBase-specific functionality like faction management, relationship handling, and utility methods.

### What Gets Extended

- **Entity:** DrGBase-specific entity functions (factions, relationships, effects)
- **NPC:** NPC-specific helpers (enhanced AI, detection, status checks)
- **Player:** Player-specific utilities (inventory, stats, possession)
- **PhysObj:** Physics object utilities (material checks, advanced manipulation)
- **Vector:** Vector math extensions (angles, distance checks, transformations)

All extensions are prefixed with `DrG_` or follow DrGBase naming conventions to avoid conflicts with other addons.

---

See [Architecture](../../architecture/README.md)
