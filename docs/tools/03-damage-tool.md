# Inflict Damage Tool

## Overview

The Inflict Damage tool allows you to apply customizable damage to any entity in the game. You can specify the amount of damage and select multiple damage types (crush, slash, burn, etc.) to test how entities respond to different kinds of damage. This is essential for testing damage resistance, damage effects, and nextbot health systems.

**Tool Name**: `Inflict Damage`

**Primary Use**: Testing entity damage responses and health systems

## How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Inflict Damage**
5. In the context menu (C key), configure:
   - Damage amount using the slider
   - Damage type(s) by clicking on the list
6. Aim at an entity and left-click to damage it
7. Press Reload (R) to damage yourself

## Actions

### Left Click
**Inflict Damage to Entity**
- Deals the configured amount of damage to the entity you're aiming at
- Applies all selected damage types
- Works on any entity that can take damage (NPCs, nextbots, props, players, etc.)
- Damage force is applied in the direction of the hit normal
- The damage source is set as yourself and your active weapon

### Right Click
**No Action**
- Right-clicking has no function with this tool

### Reload (R Key)
**Damage Yourself**
- Applies the configured damage to yourself
- Useful for testing damage effects from the receiving end
- Damage is applied as if coming from behind you
- Be careful with high damage values!

## Options/Settings

### Damage Amount Slider
**Range**: 0 to 10,000
**Default**: 0
**ConVar**: `drgbase_tool_damage_value`

Controls how much damage to inflict. Set this to match your testing needs:
- 10-50: Light damage for testing reactions
- 100-500: Moderate damage for general testing
- 1000+: Heavy damage for stress testing or instant kills

### Damage Types List
**ConVar**: `drgbase_tool_damage_type`

Select one or more damage types to apply. Click on a type to toggle it on/off:

| Damage Type | Description | Common Uses |
|-------------|-------------|-------------|
| **Crush** | Physical crushing damage | Falling objects, being crushed |
| **Slash** | Cutting/slashing damage | Melee weapons, claws |
| **Blast** | Explosive damage | Grenades, explosions |
| **Burn** | Fire damage | Fire, lava, hot surfaces |
| **Slow burn** | Damage over time from fire | Persistent burning |
| **Shock** | Electrical damage | Lightning, electrical hazards |
| **Plasma** | Energy weapon damage | Sci-fi energy weapons |
| **Dissolve** | Dissolving damage | Acid that dissolves entities |
| **Sonic** | Sound-based damage | Sonic weapons |
| **Poison** | Toxic damage | Poison gas, toxic substances |
| **Acid** | Corrosive damage | Acid, corrosive materials |
| **Radiation** | Radioactive damage | Nuclear hazards |
| **Neurotoxin** | Nerve gas damage | Chemical weapons |

**How to Select Types**:
- Click on any type in the list to toggle it
- When enabled, "Enabled" column shows "True"
- When disabled, "Enabled" column shows "False"
- You can combine multiple damage types for mixed damage
- All selected types are applied simultaneously

## Examples

### Example 1: Test Basic Health System
1. Set damage to 25
2. Select "Slash" damage type only
3. Spawn a nextbot
4. Left-click the nextbot multiple times
5. Watch health decrease with the Info tool

### Example 2: Test Fire Resistance
1. Set damage to 100
2. Select both "Burn" and "Slow burn" damage types
3. Left-click a nextbot that should resist fire
4. Verify damage is reduced or negated

### Example 3: Test Explosive Knockback
1. Set damage to 500
2. Select "Blast" damage type
3. Left-click a nextbot
4. Observe the physical force and ragdoll effects

### Example 4: Test Damage Over Time
1. Set damage to 10
2. Select "Poison" and "Radiation"
3. Left-click a nextbot
4. Check if special damage effects trigger (like poison DoT)

### Example 5: Test Instant Kill
1. Set damage to 10,000
2. Select "Crush" damage
3. Left-click any entity
4. Verify instant death/destruction

### Example 6: Test Player Damage Effects
1. Set damage to 50
2. Select "Shock" damage
3. Press Reload (R) to damage yourself
4. Observe screen effects and damage indicators

### Example 7: Test Multiple Damage Types
1. Set damage to 100
2. Select "Slash", "Burn", and "Poison"
3. Left-click a nextbot
4. Verify all damage types are processed correctly

## Tips

1. **Start Small**: Begin with low damage values (10-25) when testing to avoid accidentally killing test subjects
2. **Damage Type Matters**: Different entities may resist or be vulnerable to specific damage types - test multiple types
3. **Visual Feedback**: Many damage types trigger visual effects (fire for burn, sparks for shock) - watch for these
4. **Combine Types**: You can select multiple damage types to simulate complex damage (e.g., explosive fire = blast + burn)
5. **Self-Test**: Use Reload to damage yourself to test screen effects, pain sounds, and damage indicators
6. **Reset Selection**: If damage isn't working as expected, clear all damage types and reselect them
7. **Zero Damage**: Setting damage to 0 is useful for testing damage type effects without actually dealing damage
8. **Direction Matters**: The damage force is applied based on where you hit - aim at specific angles to test knockback
9. **Trace-Based**: Damage is applied exactly where your crosshair hits on the entity
10. **Rapid Testing**: You can repeatedly click to quickly apply damage and test damage-over-time effects
11. **Use with Info Tool**: Monitor the nextbot with the Info tool on Page 1 (Status) to see real-time health changes
12. **Godmode Testing**: Use the Godmode tool to make an entity invincible, then test if damage is properly blocked
13. **Default Type**: When the tool first loads, no damage types are selected - you must select at least one type
14. **Stacking**: Multiple damage types don't multiply damage - they just apply different effects and flags
