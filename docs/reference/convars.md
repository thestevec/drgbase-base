# ConVar Reference

All DrGBase console variables (ConVars).

## Overview

Console variables allow runtime configuration of DrGBase behavior without code changes. All DrGBase convars are prefixed with `drgbase_` and can be changed via the console or server.cfg.

**Setting ConVars:**
```
// In console
drgbase_ai_patrol 0

// In server.cfg
drgbase_multiplier_health 2
```

---

## Complete ConVar List

### AI & Behavior

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_ai_patrol` | `1` | Enable/disable NPC patrol behavior | nextbot/shared.lua |
| `drgbase_ai_sight` | `1` | Enable/disable NPC sight detection | nextbot/detection.lua |
| `drgbase_ai_hearing` | `1` | Enable/disable NPC hearing detection | nextbot/detection.lua |
| `drgbase_ai_radius` | `5000` | Maximum radius for enemy detection (units) | nextbot/ai.lua |
| `drgbase_ai_omniscient` | `0` | Make all NPCs omniscient (see through walls, always aware) | nextbot/awareness.lua |

### Movement & Pathfinding

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_compute_delay` | `0.1` | Delay between path computations (seconds) | nextbot/movements.lua |
| `drgbase_avoid_obstacles` | `1` | Enable/disable obstacle avoidance | nextbot/movements.lua |
| `drgbase_multiplier_speed` | `1` | Global speed multiplier for all NPCs | nextbot/movements.lua |

### Combat & Damage

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_multiplier_health` | `1` | Global health multiplier for all NPCs | nextbot/shared.lua |
| `drgbase_multiplier_damage_players` | `1` | Damage multiplier when NPCs attack players | nextbot/hooks.lua |
| `drgbase_multiplier_damage_npc` | `1` | Damage multiplier when NPCs attack other NPCs | nextbot/hooks.lua |

### Ragdolls & Death

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_remove_dead` | `0` | Remove NPC entities immediately on death (no ragdoll) | nextbot/hooks.lua |
| `drgbase_remove_ragdolls` | `-1` | Time to remove ragdolls (-1 = never, 0 = instant) | nextbot/misc.lua |
| `drgbase_ragdoll_fadeout` | `3` | Time for ragdoll fade-out effect (seconds) | nextbot/misc.lua |
| `drgbase_ragdoll_collisions_disabled` | `0` | Disable ragdoll collisions with world | nextbot/misc.lua |

### Resources & Precaching

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_precache_models` | `1` | Automatically precache NPC models | nextbots.lua |
| `drgbase_precache_sounds` | `1` | Automatically precache NPC sounds | nextbots.lua |

### Possession System

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_possession_enable` | `1` | Enable/disable possession system | possession.lua |
| `drgbase_possession_allow_lockon` | `1` | Allow lock-on targeting in possession mode | possession.lua |
| `drgbase_possession_targetall` | `1` | Allow targeting all entities (not just possessable) | nextbot/misc.lua |

### Weapons

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_give_weapons` | `1` | Allow players to give weapons to NPCs via USE key | weapons.lua |

### Projectiles

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_projectile_tickrate` | `-1` | Custom tickrate for projectiles (-1 = engine default) | proj_drg_default/shared.lua |

### Player Systems

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_update_luminosity` | `1` | Update player luminosity for NPC vision | meta/player.lua |

### Debug & Visualization

| Name | Default | Description | File |
|------|---------|-------------|------|
| `drgbase_debug_animations` | `0` | Show animation debug information | nextbot/animations.lua |
| `drgbase_debug_relationships` | `0` | Show relationship debug information | nextbot/relationships.lua |
| `drgbase_debug_traces` | `0` | Visualize trace lines | modules/util.lua |
| `drgbase_debug_trajectories` | `0` | Visualize projectile trajectories | meta/phys.lua |
| `drgbase_nodegraph_display` | `0` | Display navigation nodegraph | nodegraph.lua |
| `drgbase_nodegraph_distance` | `1500` | Maximum distance to display nodegraph nodes | nodegraph.lua |
| `drgbase_nodegraph_type` | `2` | Type of nodegraph display (0-2) | nodegraph.lua |
| `drgbase_nodegraph_transparent` | `0` | Make nodegraph display transparent | nodegraph.lua |

---

## ConVar Flags

Most DrGBase convars use these flags:
- **FCVAR_ARCHIVE** - Saved to config file
- **FCVAR_NOTIFY** - Notifies players when changed
- **FCVAR_REPLICATED** - Synchronized between server and clients

Debug convars typically don't have these flags for performance.

---

## Usage Examples

### Adjusting Difficulty

```lua
// Make NPCs tougher
drgbase_multiplier_health 2
drgbase_multiplier_damage_npc 1.5

// Make NPCs easier
drgbase_multiplier_health 0.5
drgbase_multiplier_damage_players 0.5
```

### Performance Optimization

```lua
// Reduce computation load
drgbase_compute_delay 0.2
drgbase_avoid_obstacles 0

// Quick ragdoll cleanup
drgbase_remove_ragdolls 5
drgbase_ragdoll_collisions_disabled 1
```

### Debugging

```lua
// Enable debug visuals
drgbase_debug_animations 1
drgbase_debug_relationships 1
drgbase_debug_traces 1
drgbase_nodegraph_display 1
```

---

## See Also

- [Configuration Guide](../guides/configuration.md)
- [Performance Tuning](../guides/performance.md)
- [Debugging](../guides/debugging.md)
