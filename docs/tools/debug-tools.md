# Debug & Testing Tools

## Overview

DrGBase provides several tools specifically for debugging and testing NPC behavior. These tools modify NPC state to help isolate and test specific features.

---

## Godmode Tool

### Purpose
Enable/disable invincibility for NPCs.

### Usage
- **Left Click on NPC**: Toggle godmode on/off

### Visual Feedback
- 🟢 **Green Halo**: Godmode enabled (invincible)
- 🔴 **Red Halo**: Godmode disabled (normal)
- All DrGBase NPCs show halos when tool is equipped

### Use Cases
1. **Combat Testing**: Test attacks without killing NPC
2. **Animation Testing**: Watch full attack sequences
3. **AI Testing**: Observe behavior under fire
4. **Balance Testing**: See how long NPC survives

### Example Workflow
```
1. Spawn test NPC
2. Equip Godmode Tool
3. Click NPC (Green halo = enabled)
4. Test combat mechanics
5. Disable when done testing
```

### Console Command
```lua
-- Enable godmode
Entity(1):SetGodMode(true)

-- Check status
print(Entity(1):GetGodMode())
```

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_godmode.lua`

---

## Disable AI Tool

### Purpose
Freeze AI processing to inspect or position NPCs.

### Usage
- **Left Click on NPC**: Toggle AI on/off

### Visual Feedback
- 🟢 **Green Halo**: AI enabled (active)
- 🔴 **Red Halo**: AI disabled (frozen)
- Info Tool shows "AI Disabled" warning when viewing

### What It Does
When AI is disabled:
- ❌ No enemy detection
- ❌ No movement/pathfinding
- ❌ No attacking
- ❌ No idle behavior
- ✅ Still takes damage
- ✅ Still plays animations (manually)
- ✅ Still responds to physics

### Use Cases
1. **Positioning**: Place NPCs without them wandering
2. **Screenshot/Video**: Static NPC for filming
3. **Animation Testing**: Test animations without AI interference
4. **Performance Testing**: Measure AI performance cost
5. **Debugging**: Isolate non-AI issues

### Example Workflows

**Positioning Multiple NPCs:**
```
1. Spawn NPCs
2. Disable AI on all
3. Use Physgun to position precisely
4. Re-enable AI when ready
5. NPCs start behavior from new positions
```

**Testing Animation Without AI:**
```
1. Spawn NPC
2. Disable AI
3. Use console to play animations:
   lua_run Entity(1):PlayActivity(ACT_MELEE_ATTACK1)
4. Watch animation without AI interruption
```

### Console Commands
```lua
-- Disable AI
Entity(1):DisableAI(true)

-- Enable AI
Entity(1):DisableAI(false)

-- Check status
print(Entity(1):IsAIDisabled())
```

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_disableai.lua`

---

## No Target Tool

### Purpose
Make player invisible to NPC AI (can't be detected).

### Usage
- **Left Click on Player**: Toggle no-target on/off

### Visual Feedback
- 🟢 **Green Halo**: No-target enabled (invisible to NPCs)
- 🔴 **Red Halo**: No-target disabled (normal detection)
- Shows halos on all players when tool equipped

### What It Does
When no-target is enabled:
- ❌ NPCs can't detect player
- ❌ NPCs can't target player as enemy
- ❌ NPCs ignore player damage
- ✅ Player can still damage NPCs
- ✅ Player can observe without interference

### Use Cases
1. **Observation**: Watch NPC behavior without interaction
2. **Testing**: Test NPC vs. NPC combat
3. **Debugging**: Isolate detection issues
4. **Filming**: Record NPC behavior naturally
5. **Safe Testing**: Test hostile NPCs safely

### Example Workflows

**Observe NPC Behavior:**
```
1. Spawn NPCs
2. Enable no-target on yourself
3. Walk among NPCs undetected
4. Observe natural behavior
```

**Test NPC vs. NPC Combat:**
```
1. Spawn two enemy factions
2. Enable no-target on yourself
3. Watch them fight without targeting you
4. Gather combat data
```

### Console Commands
```lua
-- Enable no-target
Player(1):DrG_SetNoTarget(true)

-- Check status
print(Player(1):DrG_GetNoTarget())
```

### Important Notes
- No-target persists until disabled
- Doesn't affect other players
- NPCs can still see you visually (model is visible)
- Just makes AI ignore you as target

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_notarget.lua`

---

## Omniscient Tool

### Purpose
Make NPC detect all enemies instantly (all-seeing).

### Usage
- **Left Click on NPC**: Toggle omniscient on/off

### Visual Feedback
- 🟢 **Green Halo**: Omniscient enabled (all-seeing)
- 🔴 **Red Halo**: Omniscient disabled (normal detection)

### What It Does
When omniscient is enabled:
- ✅ Detects all enemies instantly
- ✅ Ignores FOV restrictions
- ✅ Ignores sight range limits
- ✅ Ignores line-of-sight blockers
- ✅ Always knows enemy position

### Use Cases
1. **Combat Testing**: Force immediate combat
2. **AI Testing**: Test combat behavior without detection delay
3. **Debugging**: Bypass detection issues
4. **Testing Aggressive Behavior**: Ensure NPCs always engage

### Example Workflows

**Force Immediate Combat:**
```
1. Spawn NPC
2. Enable omniscient
3. Spawn far away
4. NPC immediately engages
5. Test combat from start
```

**Test Detection vs. Combat:**
```
1. Test combat with omniscient enabled (perfect detection)
2. Disable omniscient
3. Test again with normal detection
4. Compare behavior differences
```

### Console Commands
```lua
-- Enable omniscient
Entity(1):SetOmniscient(true)

-- Check status
print(Entity(1):IsOmniscient())
```

### Normal Detection vs. Omniscient

| Feature | Normal Detection | Omniscient |
|---------|------------------|------------|
| FOV Check | Required | Ignored |
| Range Check | Required | Ignored |
| Line of Sight | Required | Ignored |
| Detection Delay | Yes | Instant |
| Awareness Caching | Yes | Always aware |

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_omniscient.lua`

---

## Damage Tool

### Purpose
Apply custom damage to NPCs for testing.

### Usage
- **Configure damage amount in tool panel**
- **Left Click on NPC**: Deal damage

### Configuration
Tool panel options:
- Damage amount (slider/number)
- Damage type (dropdown)
- Hit location (hitgroup)

### Damage Types Available
```
DMG_GENERIC
DMG_CRUSH
DMG_BULLET
DMG_SLASH
DMG_BURN
DMG_VEHICLE
DMG_FALL
DMG_BLAST
DMG_CLUB
DMG_SHOCK
DMG_SONIC
DMG_ENERGYBEAM
DMG_POISON
DMG_RADIATION
```

### Use Cases
1. **Health Testing**: Test health values
2. **Damage Reaction**: Test flinch/damage responses
3. **Death Testing**: Test death behavior
4. **Damage Type Testing**: Test different damage types
5. **Balance Testing**: Fine-tune damage values

### Example Workflows

**Test Death Behavior:**
```
1. Spawn NPC
2. Set damage to NPC's max health
3. Click NPC
4. Watch death animation/effects
```

**Test Damage Types:**
```
1. Spawn NPC
2. Set damage to 10, type = BULLET
3. Click NPC, observe reaction
4. Change to FIRE, click again
5. Compare behaviors
```

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_damage.lua`

---

## Scale Tool

### Purpose
Dynamically adjust NPC model size.

### Usage
- **Configure scale in tool panel (0.1 to 10.0)**
- **Left Click on NPC**: Apply scale

### Visual Feedback
- NPC model size changes immediately
- Collision bounds adjust automatically
- All animations scale proportionally

### Use Cases
1. **Boss Variants**: Create giant versions
2. **Minion Variants**: Create tiny versions
3. **Visual Variety**: Vary NPC sizes
4. **Testing**: Test size-based mechanics

### Example Values
```
0.5  = Half size (tiny)
1.0  = Normal size
2.0  = Double size (large)
5.0  = 5x size (huge)
10.0 = Maximum size (gigantic)
```

### Console Command
```lua
Entity(1):SetModelScale(2.0) -- Double size
```

### Important Notes
- Health doesn't scale automatically
- Speed doesn't scale automatically
- Damage doesn't scale automatically
- Only visual size changes

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_scale.lua`

---

## Mover Tool

### Purpose
Reposition NPCs without physics disruption.

### Usage
- **Left Click on NPC**: Select NPC
- **Left Click on Surface**: Move NPC to clicked position
- **Right Click**: Rotate NPC
- **Reload**: Deselect

### Use Cases
1. **Precise Positioning**: Place NPCs exactly
2. **No Physics Disruption**: Move without ragdolling
3. **Quick Repositioning**: Fast placement
4. **Map Setup**: Position NPCs for scenarios

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_mover.lua`

---

## Relationship Tool (Simple)

### Purpose
Quickly set relationships between entities.

### Usage
- **Configure disposition in panel**
- **Left Click on Entity 1, then Entity 2**: Set relationship

### Dispositions
- **Like (D_LI)**: Friendly
- **Hate (D_HT)**: Hostile
- **Fear (D_FR)**: Afraid
- **Neutral (D_NU)**: Ignore

### Use Cases
1. **Quick Testing**: Test relationship changes
2. **Dynamic Relationships**: Change allegiances mid-game
3. **Debugging**: Override faction relationships

### Source
File: `lua/weapons/gmod_tool/stools/drgbase_tool_relationship_simple.lua`

---

## Tool Combinations

### Combination 1: Static Combat Test
```
1. Disable AI Tool → Freeze attacker
2. Godmode Tool → Make defender invincible
3. Damage Tool → Test specific damage amounts
4. Info Tool → Monitor health/stats
```

### Combination 2: Behavior Observation
```
1. No Target Tool → Become invisible
2. Omniscient Tool → Force NPC engagement
3. Info Tool → Monitor AI state
4. Watch natural combat behavior
```

### Combination 3: Precise Setup
```
1. Disable AI Tool → Freeze NPCs
2. Mover Tool → Position exactly
3. Faction Tool → Set teams
4. Disable AI Tool → Re-enable, start scenario
```

## Best Practices

1. **Always have Info Tool ready**: Quick diagnostics
2. **Use Godmode for non-lethal tests**: Prevents premature death
3. **Disable AI when positioning**: Prevents wandering
4. **No Target for observation**: Watch without interference
5. **Omniscient for forced combat**: Skip detection phase

## See Also

- [Info Tool](info-tool.md) - Detailed diagnostics
- [Faction Tool](faction-tool.md) - Faction management
- [Console Commands](console-commands.md) - All commands
