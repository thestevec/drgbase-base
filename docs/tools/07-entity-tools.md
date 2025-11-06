# Entity Manipulation Tools

Entity Manipulation Tools allow you to move, scale, and modify NPCs for testing and content creation.

## Available Tools

### 1. Entity Mover
Precisely position entities in 3D space

### 2. Entity Scaler
Adjust entity size and hitbox

### 3. Godmode Tool
Make entities invulnerable

### 4. Speed Modifier
Change movement speed dynamically

## Entity Mover Tool

**Accessing:**
1. Spawn Menu > Utilities > "DrGBase Entity Mover"
2. Console: `drgbase_mover`

**Usage:**
1. Equip tool
2. **Left Click** entity - Grab
3. **Mouse movement** - Move in screen space
4. **Left Click** again - Drop
5. **Right Click** - Rotate 90 degrees
6. **Reload** - Reset position

**Console Controls:**
```
drgbase_mover_distance <number>    -- Grab distance (default: 100)
drgbase_mover_smooth <0|1>         -- Smooth movement (default: 1)
drgbase_mover_snap <number>        -- Snap to grid (0 = off, default: 0)
```

**Advanced Movement:**
- Hold **Shift** - Move faster
- Hold **Ctrl** - Move slower (precise)
- Hold **Alt** - Lock to horizontal plane
- Mouse Wheel - Adjust distance

## Entity Scaler Tool

**Accessing:**
1. Spawn Menu > Utilities > "DrGBase Entity Scaler"
2. Console: `drgbase_scale <value>`

**Usage:**
1. Equip tool
2. **Left Click** entity - Apply scale
3. **Right Click** entity - View current scale
4. **Reload** - Open scale menu

**Console:**
```
drgbase_scale 0.5     -- Half size
drgbase_scale 1.0     -- Normal size
drgbase_scale 2.0     -- Double size
drgbase_scale_reset   -- Reset to normal
```

**Features:**
- Scales model
- Scales collision hull
- Adjusts health proportionally (optional)
- Updates movement speed (optional)

**Console Variables:**
```lua
drgbase_scale_health <0|1>       -- Scale health with size (default: 1)
drgbase_scale_speed <0|1>        -- Scale speed with size (default: 1)
drgbase_scale_min <number>       -- Minimum scale (default: 0.1)
drgbase_scale_max <number>       -- Maximum scale (default: 10.0)
```

## Godmode Tool

**Accessing:**
1. Spawn Menu > Utilities > "DrGBase Godmode Tool"
2. Console: `drgbase_godmode`

**Usage:**
1. Equip tool
2. **Left Click** entity - Toggle godmode
3. **Right Click** entity - View status
4. **Reload** - Godmode all nearby

**Console:**
```
drgbase_godmode_all               -- Godmode all NPCs
drgbase_godmode_radius <radius>   -- Godmode within radius
drgbase_godmode_off_all           -- Remove godmode from all
```

**Visual Indicators:**
- Godmode NPCs glow blue
- Health bar shows infinity symbol
- Info tool shows "Godmode: Active"

## Speed Modifier Tool

**Accessing:**
1. Spawn Menu > Utilities > "DrGBase Speed Tool"
2. Console: `drgbase_speed <multiplier>`

**Usage:**
1. Equip tool
2. **Left Click** entity - Apply speed
3. **Right Click** entity - View current speed
4. **Reload** - Reset speed

**Console:**
```
drgbase_speed 0.5     -- Half speed
drgbase_speed 1.0     -- Normal speed
drgbase_speed 2.0     -- Double speed
drgbase_speed 5.0     -- Super speed
```

**Affects:**
- Walk speed
- Run speed
- Attack speed (optional)
- Animation speed (optional)

## Testing Scenarios

### Scene Setup
```
1. drgbase_ai_enabled 0 (freeze AI)
2. Use Entity Mover to position NPCs
3. Use Entity Scaler for size variety
4. drgbase_ai_enabled 1 (animate)
```

### Boss Creation
```
1. Spawn regular NPC
2. drgbase_scale 3.0 (make huge)
3. drgbase_godmode (temporary invincibility)
4. Test boss fight mechanics
5. drgbase_godmode (remove when ready)
```

### Speed Testing
```
1. Spawn multiple NPCs
2. Apply different speeds
3. Test how speed affects combat
4. Adjust until balanced
```

### Giant vs Tiny
```
1. Spawn two NPCs
2. drgbase_scale 5.0 (giant)
3. drgbase_scale 0.2 (tiny)
4. Test interaction/combat
```

## Advanced Features

### Batch Operations
```
drgbase_batch_scale <scale>       -- Scale all selected
drgbase_batch_speed <mult>        -- Speed all selected
drgbase_batch_godmode             -- Godmode all selected
```

Select entities with Physgun first, then use batch commands.

### Presets
```
drgbase_preset_save <name>        -- Save entity configuration
drgbase_preset_load <name>        -- Load entity configuration
drgbase_preset_list               -- List all presets
```

Saves position, scale, speed, and other properties.

### Entity Cloning
```
drgbase_clone                     -- Clone aimed entity
drgbase_clone_count <number>      -- Clone multiple
```

Clones entity with all modifications applied.

## Console Variables

```lua
-- Mover
drgbase_mover_distance <number>      -- Grab distance
drgbase_mover_smooth <0|1>           -- Smooth movement
drgbase_mover_snap <number>          -- Snap to grid

-- Scaler
drgbase_scale_health <0|1>           -- Scale health
drgbase_scale_speed <0|1>            -- Scale speed
drgbase_scale_collision <0|1>        -- Scale hitbox

-- Speed
drgbase_speed_affects_anims <0|1>    -- Affect animation speed
drgbase_speed_affects_attacks <0|1>  -- Affect attack speed

-- Godmode
drgbase_godmode_glow <0|1>           -- Visual glow effect
```

## Troubleshooting

**Entity Mover not working:**
- Check you're within grab distance
- Verify entity has physics
- Try increasing `drgbase_mover_distance`

**Scale not applying:**
- Verify entity is DrGBase NPC
- Check scale limits (min/max)
- Try resetting first

**Godmode not working:**
- Verify entity has OnTakeDamage
- Check entity isn't already invulnerable
- Look for console errors

## Tips

1. **Combine tools** - Use multiple tools together for complex setups
2. **Save presets** - Save configurations for reuse
3. **Test gradually** - Change one parameter at a time
4. **Document settings** - Note values that work well

## Example Workflows

### Create Mini-Boss
```
1. Spawn standard NPC
2. drgbase_scale 2.0 (larger)
3. drgbase_speed 1.5 (faster)
4. drgbase_godmode (test without dying)
5. Adjust until balanced
6. drgbase_godmode (remove)
7. drgbase_preset_save "mini_boss"
```

### Precise Scene Setup
```
1. drgbase_ai_enabled 0
2. Use Entity Mover for positioning
3. drgbase_mover_snap 16 (snap to grid)
4. Position all NPCs
5. drgbase_preset_save "scene_01"
6. drgbase_ai_enabled 1
```

### Speed Comparison Test
```
1. Spawn 3 identical NPCs
2. NPC1: drgbase_speed 0.5
3. NPC2: drgbase_speed 1.0
4. NPC3: drgbase_speed 2.0
5. Spawn enemy
6. Compare combat effectiveness
```

## Next Steps

- [AI Tools](./06-ai-tools.md) - Control NPC behavior
- [Info Tool](./02-info-tool.md) - Monitor entity properties
- [Boss NPC Example](../examples/boss-npc.md) - Create boss encounters
