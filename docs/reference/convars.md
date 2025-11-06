# Console Variables Reference

Complete reference of DrGBase console variables (ConVars) for configuration and debugging.

## AI Configuration

### drgbase_ai_radius
**Default**: `5000`  
**Type**: Float  
**Replicated**: Yes

Maximum distance NPCs can detect enemies.

```
drgbase_ai_radius 3000  // Reduce detection range
drgbase_ai_radius 10000 // Increase detection range
```

### drgbase_ai_sight
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Enable/disable visual enemy detection.

```
drgbase_ai_sight 0  // NPCs can't see enemies
drgbase_ai_sight 1  // Normal sight (default)
```

### drgbase_ai_hearing
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Enable/disable sound-based enemy detection.

```
drgbase_ai_hearing 0  // NPCs can't hear sounds
drgbase_ai_hearing 1  // Normal hearing (default)
```

### drgbase_ai_patrol
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Enable/disable patrol behavior when idle.

```
drgbase_ai_patrol 0  // NPCs stand still when idle
drgbase_ai_patrol 1  // NPCs patrol when idle (default)
```

### drgbase_ai_omniscient
**Default**: `0`  
**Type**: Boolean  
**Replicated**: Yes

NPCs can see through walls (developer tool).

```
drgbase_ai_omniscient 1  // NPCs see through walls
drgbase_ai_omniscient 0  // Normal sight (default)
```

## Movement Configuration

### drgbase_compute_delay
**Default**: `0.1`  
**Type**: Float  
**Replicated**: Yes

Pathfinding recalculation delay in seconds.

```
drgbase_compute_delay 0.05  // More responsive pathfinding
drgbase_compute_delay 0.2   // Less CPU usage
```

### drgbase_avoid_obstacles
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Enable/disable obstacle avoidance.

```
drgbase_avoid_obstacles 0  // NPCs walk through obstacles
drgbase_avoid_obstacles 1  // NPCs avoid obstacles (default)
```

### drgbase_multiplier_speed
**Default**: `1`  
**Type**: Float  
**Replicated**: Yes

Global speed multiplier for all DrGBase NPCs.

```
drgbase_multiplier_speed 0.5  // Half speed
drgbase_multiplier_speed 2.0  // Double speed
```

## Damage Configuration

### drgbase_multiplier_health
**Default**: `1`  
**Type**: Float  
**Replicated**: Yes

Global health multiplier for all DrGBase NPCs.

```
drgbase_multiplier_health 0.5  // Half health (easier)
drgbase_multiplier_health 2.0  // Double health (harder)
```

### drgbase_multiplier_damage_players
**Default**: `1`  
**Type**: Float  
**Replicated**: Yes

Multiplier for NPC damage to players.

```
drgbase_multiplier_damage_players 0.5  // NPCs deal half damage to players
drgbase_multiplier_damage_players 2.0  // NPCs deal double damage to players
```

### drgbase_multiplier_damage_npc
**Default**: `1`  
**Type**: Float  
**Replicated**: Yes

Multiplier for NPC damage to other NPCs.

```
drgbase_multiplier_damage_npc 0.5  // NPCs deal half damage to each other
drgbase_multiplier_damage_npc 2.0  // NPCs deal double damage to each other
```

## Ragdoll Configuration

### drgbase_remove_ragdolls
**Default**: `-1`  
**Type**: Float  
**Replicated**: Yes

Time in seconds before ragdolls are removed. `-1` = never remove.

```
drgbase_remove_ragdolls 5   // Remove after 5 seconds
drgbase_remove_ragdolls 0   // Remove immediately
drgbase_remove_ragdolls -1  // Never remove (default)
```

### drgbase_ragdoll_fadeout
**Default**: `3`  
**Type**: Float  
**Replicated**: Yes

Time in seconds for ragdoll fade-out effect.

```
drgbase_ragdoll_fadeout 1  // Quick fade
drgbase_ragdoll_fadeout 5  // Slow fade
```

### drgbase_ragdoll_collisions_disabled
**Default**: `0`  
**Type**: Boolean  
**Replicated**: Yes

Disable ragdoll collisions for better performance.

```
drgbase_ragdoll_collisions_disabled 1  // No ragdoll collisions
drgbase_ragdoll_collisions_disabled 0  // Normal collisions (default)
```

### drgbase_remove_dead
**Default**: `0`  
**Type**: Boolean  
**Replicated**: Yes

Remove NPC entity immediately on death (leave only ragdoll).

```
drgbase_remove_dead 1  // Remove dead NPCs
drgbase_remove_dead 0  // Keep dead NPCs (default)
```

## Possession System

### drgbase_possession_enable
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Enable/disable NPC possession system.

```
drgbase_possession_enable 0  // Disable possession
drgbase_possession_enable 1  // Enable possession (default)
```

### drgbase_possession_targetall
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Allow possessing all entities, not just DrGBase NPCs.

```
drgbase_possession_targetall 0  // Only DrGBase NPCs
drgbase_possession_targetall 1  // All entities (default)
```

### drgbase_possession_allow_lockon
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Allow camera lock-on during possession.

```
drgbase_possession_allow_lockon 0  // Disable lock-on
drgbase_possession_allow_lockon 1  // Enable lock-on (default)
```

### drgbase_possession_lockon_speed
**Default**: `0.05`  
**Type**: Float  
**Replicated**: Yes (Client)

Camera lock-on lerp speed.

```
drgbase_possession_lockon_speed 0.1   // Faster lock-on
drgbase_possession_lockon_speed 0.01  // Slower lock-on
```

### drgbase_possession_teleport
**Default**: `0`  
**Type**: Boolean  
**Replicated**: Yes (Client)

Teleport to possessed NPC instead of smooth transition.

```
drgbase_possession_teleport 1  // Instant teleport
drgbase_possession_teleport 0  // Smooth transition (default)
```

### drgbase_possession_exit
**Default**: `KEY_E`  
**Type**: Integer (Key code)  
**Replicated**: Yes (Client)

Key to exit possession.

```
bind e "drgbase_possession_exit"  // Default: E key
```

### drgbase_possession_view
**Default**: `KEY_V`  
**Type**: Integer (Key code)  
**Replicated**: Yes (Client)

Key to cycle camera views.

```
bind v "drgbase_possession_view"  // Default: V key
```

### drgbase_possession_climb
**Default**: `KEY_C`  
**Type**: Integer (Key code)  
**Replicated**: Yes (Client)

Key to climb while possessing.

```
bind c "drgbase_possession_climb"  // Default: C key
```

### drgbase_possession_lockon
**Default**: `KEY_L`  
**Type**: Integer (Key code)  
**Replicated**: Yes (Client)

Key to toggle camera lock-on.

```
bind l "drgbase_possession_lockon"  // Default: L key
```

## Weapon Configuration

### drgbase_give_weapons
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Allow players to give weapons to human NPCs.

```
drgbase_give_weapons 0  // Players can't give weapons
drgbase_give_weapons 1  // Players can give weapons (default)
```

## Projectile Configuration

### drgbase_projectile_tickrate
**Default**: `-1`  
**Type**: Float  
**Replicated**: Yes

Custom think rate for projectiles. `-1` = use default.

```
drgbase_projectile_tickrate 0.01  // 100 ticks per second
drgbase_projectile_tickrate 0.1   // 10 ticks per second
drgbase_projectile_tickrate -1    // Use default (automatic)
```

## Optimization

### drgbase_precache_models
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Precache NPC models on server start.

```
drgbase_precache_models 0  // Don't precache (faster load)
drgbase_precache_models 1  // Precache models (default)
```

### drgbase_precache_sounds
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Precache NPC sounds on server start.

```
drgbase_precache_sounds 0  // Don't precache (faster load)
drgbase_precache_sounds 1  // Precache sounds (default)
```

### drgbase_update_luminosity
**Default**: `1`  
**Type**: Boolean  
**Replicated**: Yes

Update player luminosity for stealth detection.

```
drgbase_update_luminosity 0  // Disable (better performance)
drgbase_update_luminosity 1  // Enable (default)
```

## Debug Tools

### drgbase_debug_relationships
**Default**: `0`  
**Type**: Boolean

Print relationship changes to console.

```
drgbase_debug_relationships 1  // Show relationship debug
```

### drgbase_debug_animations
**Default**: `0`  
**Type**: Boolean  
**Replicated**: Yes

Print animation changes to console.

```
drgbase_debug_animations 1  // Show animation debug
```

### drgbase_debug_traces
**Default**: `0`  
**Type**: Boolean

Visualize trace lines.

```
drgbase_debug_traces 1  // Show trace debug
```

### drgbase_debug_trajectories
**Default**: `0`  
**Type**: Boolean

Visualize projectile trajectories.

```
drgbase_debug_trajectories 1  // Show trajectory debug
```

## Display Options

### drgbase_display_collisions
**Default**: `0`  
**Type**: Boolean (Client)

Display NPC collision bounds.

```
drgbase_display_collisions 1  // Show collision boxes
```

### drgbase_display_sight
**Default**: `0`  
**Type**: Boolean (Client)

Display NPC sight range and FOV.

```
drgbase_display_sight 1  // Show sight visualization
```

### drgbase_navmesh_error
**Default**: `1`  
**Type**: Boolean (Client)

Show warning when navmesh is missing.

```
drgbase_navmesh_error 0  // Hide navmesh warning
```

### drgbase_adjust_ragdoll_attachments
**Default**: `0`  
**Type**: Boolean (Client)

Adjust ragdoll attachment positions.

```
drgbase_adjust_ragdoll_attachments 1  // Fix attachment positions
```

## Nodegraph Visualization

### drgbase_nodegraph_display
**Default**: `0`  
**Type**: Boolean (Client)

Display DrGBase nodegraph for debugging.

```
drgbase_nodegraph_display 1  // Show nodegraph
```

### drgbase_nodegraph_distance
**Default**: `1500`  
**Type**: Float (Client)

Maximum distance to display nodes.

```
drgbase_nodegraph_distance 3000  // Show nodes further away
```

### drgbase_nodegraph_type
**Default**: `2`  
**Type**: Integer (Client)

Nodegraph display type (0=none, 1=basic, 2=detailed).

```
drgbase_nodegraph_type 1  // Basic display
drgbase_nodegraph_type 2  // Detailed display (default)
```

### drgbase_nodegraph_transparent
**Default**: `0`  
**Type**: Boolean (Client)

Make nodegraph visualization transparent.

```
drgbase_nodegraph_transparent 1  // Transparent nodes
```

## Tool ConVars

### drgbase_tool_damage_type
Damage type for damage tool (set via tool UI).

### drgbase_tool_faction_list
Faction list for faction tool (set via tool UI).

## Garry's Mod AI ConVars

These are standard GMod ConVars that affect DrGBase NPCs:

### ai_disabled
**Default**: `0`

Disable all NPC AI globally.

```
ai_disabled 1  // Freeze all NPCs
```

### ai_ignoreplayers
**Default**: `0`

NPCs ignore players.

```
ai_ignoreplayers 1  // NPCs won't target players
```

### ai_serverragdolls
**Default**: `0`

Create ragdolls on server instead of client.

```
ai_serverragdolls 1  // Server-side ragdolls
```

### developer
**Default**: `0`

Enable developer mode for additional debug output.

```
developer 1  // Show developer messages
developer 2  // Show verbose messages
```

## Usage Examples

### Testing Configuration
```
// Easy mode
drgbase_multiplier_health 0.5
drgbase_multiplier_damage_players 0.5
drgbase_ai_radius 3000

// Hard mode
drgbase_multiplier_health 2.0
drgbase_multiplier_damage_players 2.0
drgbase_ai_omniscient 1
```

### Performance Optimization
```
// Better performance
drgbase_ragdoll_collisions_disabled 1
drgbase_remove_ragdolls 5
drgbase_compute_delay 0.15
drgbase_update_luminosity 0
```

### Debug Mode
```
// Full debug
developer 1
drgbase_debug_animations 1
drgbase_debug_relationships 1
drgbase_debug_traces 1
drgbase_display_collisions 1
drgbase_display_sight 1
drgbase_nodegraph_display 1
```

For more information, see [Developer Tools](../tools/01-overview.md).
