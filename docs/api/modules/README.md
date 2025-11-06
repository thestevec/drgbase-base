# Utility Modules API

Helper functions and utilities.

**Location:** `lua/drgbase/modules/`

## Modules

- **[coroutine.md](./coroutine.md)** - Coroutine helpers for managing async operations
- **[debugoverlay.md](./debugoverlay.md)** - Debug visualization tools (trajectories, traces)
- **[math.md](./math.md)** - Math extensions for animations and physics
- **[navmesh.md](./navmesh.md)** - Navigation mesh helpers for pathfinding
- **[net.md](./net.md)** - Enhanced networking system with message helpers and RPC callbacks
- **[render.md](./render.md)** - Rendering helpers for sprites and effects
- **[string.md](./string.md)** - String extensions for formatting
- **[table.md](./table.md)** - Table extensions (deep copy, readonly, fetch, pack/unpack)
- **[timer.md](./timer.md)** - Timer management with loops and delays
- **[util.md](./util.md)** - Utility functions (tracing, damage, colors)

## Module Overview

These modules extend Garry's Mod's built-in libraries with DrGBase-specific functionality. All module functions are prefixed with `DrG_` to avoid conflicts (e.g., `net.DrG_Send`, `table.DrG_Copy`).

### Key Modules

- **net.md** - Most important for multiplayer features, provides simplified networking
- **table.md** - Essential for data manipulation and nil-safe operations
- **util.md** - Common utilities for tracing, damage handling, and effects
- **timer.md** - Simplified timer management for delayed actions
