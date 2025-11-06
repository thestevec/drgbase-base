# Damage Tool

The Damage Tool allows precise application of damage to NPCs for testing combat mechanics, death animations, and damage resistance.

## Accessing the Tool

**Spawn Menu:**
1. Press `Q`
2. Click Utilities tab
3. Select "DrGBase Damage Tool"

**Console:** `drgbase_damage <amount>`

## Usage

### Basic Usage
1. Equip the Damage Tool
2. Aim at target NPC
3. **Left Click** - Apply damage
4. **Right Click** - Heal target
5. **Reload** - Open damage settings menu

### Setting Damage Amount

**Method 1: Console**
```
drgbase_damage 50    -- Set damage to 50
drgbase_damage 100   -- Set damage to 100
```

**Method 2: Settings Menu**
1. Press Reload while holding tool
2. Use slider to adjust damage
3. Click Apply

## Damage Types

Use console to set damage type:
```lua
drgbase_damage_type DMG_BULLET    -- Bullet damage
drgbase_damage_type DMG_SLASH     -- Melee/slash damage
drgbase_damage_type DMG_CLUB      -- Blunt damage
drgbase_damage_type DMG_BURN      -- Fire damage
drgbase_damage_type DMG_BLAST     -- Explosion damage
drgbase_damage_type DMG_SHOCK     -- Electric damage
```

## Testing Scenarios

### Test Death Animation
```
drgbase_damage 10000    -- Set lethal damage
-- Left click on NPC to kill instantly
```

### Test Damage Resistance
```
drgbase_damage 25       -- Set moderate damage
-- Click multiple times
-- Verify OnTakeDamage modifiers work
```

### Test Healing
```
drgbase_damage 50       -- Damage NPC first
-- Right click to heal
-- Verify health regeneration
```

### Test Damage Types
```
drgbase_damage_type DMG_BURN
drgbase_damage 30
-- Left click
-- Verify NPC responds correctly to fire damage
```

## Advanced Features

### Damage Over Time
```lua
drgbase_damage_dot <damage> <duration>
-- Example: drgbase_damage_dot 5 10
-- Deals 5 damage per second for 10 seconds
```

### Damage to Multiple Targets
```lua
drgbase_damage_radius <damage> <radius>
-- Example: drgbase_damage_radius 50 200
-- Damages all NPCs within 200 units
```

### Headshot Multiplier
```lua
drgbase_damage_headshot <multiplier>
-- Example: drgbase_damage_headshot 2.0
-- Doubles damage if aiming at head hitbox
```

## Console Variables

```lua
drgbase_damage_default <number>      -- Default damage amount (default: 10)
drgbase_damage_showfeedback <0|1>    -- Show damage numbers (default: 1)
drgbase_damage_allowheal <0|1>       -- Allow healing with right click (default: 1)
drgbase_damage_soundeffect <0|1>     -- Play sound on damage (default: 1)
```

## Testing Checklist

- [ ] Test normal damage application
- [ ] Verify damage types affect NPCs differently
- [ ] Check death triggers at 0 health
- [ ] Test damage resistance/multipliers
- [ ] Verify headshot detection
- [ ] Test healing functionality
- [ ] Check damage feedback appears

## Troubleshooting

**Damage not applying:**
- Verify NPC has OnTakeDamage implemented
- Check NPC isn't in godmode
- Ensure damage amount is > 0

**Wrong damage amount:**
- Check `drgbase_damage_default` value
- Verify OnTakeDamage isn't modifying damage
- Test with different damage types

## Tips

1. **Quick testing** - Set damage to 1 to test multiple hits
2. **One-shot testing** - Set damage to 10000 for instant kills
3. **Healing tests** - Damage then heal to test regeneration
4. **Type testing** - Test all damage types for full coverage

## Next Steps

- [Info Tool](./02-info-tool.md) - Monitor health changes
- [AI Tools](./06-ai-tools.md) - Test AI responses to damage
- [Entity Tools](./07-entity-tools.md) - Combine with godmode testing
