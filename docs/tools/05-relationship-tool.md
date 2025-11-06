# Relationship Tool

The Relationship Tool allows precise control over entity relationships for testing faction dynamics and AI targeting behavior.

## Accessing the Tool

**Spawn Menu:**
1. Press `Q`
2. Click Utilities tab
3. Select "DrGBase Relationship Tool"

**Console:** `drgbase_relationship <disposition>`

## Usage

### Basic Usage
1. Equip the Relationship Tool
2. Aim at first NPC (source)
3. **Left Click** - Set source NPC
4. Aim at second NPC (target)
5. **Left Click** - Apply relationship
6. **Reload** - Open relationship menu

## Relationship Dispositions

```lua
D_HT    -- Hate (hostile, will attack)
D_FR    -- Fear (flee from target)
D_LI    -- Like (friendly, will not attack)
D_NU    -- Neutral (ignore)
```

## Setting Relationships

**Method 1: Console**
```
drgbase_relationship D_HT    -- Set to hostile
drgbase_relationship D_LI    -- Set to friendly
drgbase_relationship D_NU    -- Set to neutral
```

**Method 2: Selection Menu**
1. Press Reload while holding tool
2. Select disposition
3. Set source and target NPCs

## Testing Scenarios

### Make Enemies Fight
```
1. Spawn two neutral NPCs
2. Use tool: NPC1 -> NPC2 = D_HT
3. NPC1 should attack NPC2
```

### Make Enemy Friendly
```
1. Spawn hostile NPC
2. Use tool: NPC -> Player = D_LI
3. NPC should stop attacking player
```

### Create One-Way Relationships
```
1. NPC1 -> NPC2 = D_HT (NPC1 hates NPC2)
2. NPC2 -> NPC1 = D_LI (NPC2 likes NPC1)
3. Only NPC1 attacks
```

## Advanced Features

### Mutual Relationships
Hold **Shift + Left Click** to create bidirectional relationship (both directions same).

### Entity Classes
Hold **Ctrl** when selecting target to apply relationship to entire entity class.

### Player Relationships
Aim at yourself (look down) to set relationships with player.

## Console Commands

```lua
drgbase_rel_set <source> <target> <disp>    -- Set specific relationship
drgbase_rel_clear <entity>                  -- Clear all relationships
drgbase_rel_list <entity>                   -- List entity's relationships
drgbase_rel_default <disposition>           -- Set default for new NPCs
```

## Testing Scenarios

### Alliance Testing
```
-- Create temporary alliance
drgbase_relationship D_LI
-- Click: Combine -> Rebels
-- Verify they don't fight
```

### Fear Response Testing
```
drgbase_relationship D_FR
-- Click: Weak NPC -> Strong NPC
-- Weak NPC should flee
```

### Faction Override Testing
```
-- Override faction relationships
1. Two same-faction NPCs
2. Set one to hate the other
3. Verify relationship overrides faction
```

## Console Variables

```lua
drgbase_rel_showfeedback <0|1>    -- Show relationship changes (default: 1)
drgbase_rel_persistent <0|1>      -- Persist across spawns (default: 0)
drgbase_rel_override <0|1>        -- Override faction relationships (default: 1)
```

## Troubleshooting

**Relationship not applying:**
- Verify both NPCs are valid DrGBase NPCs
- Check console for errors
- Try clearing relationships first

**NPC still attacks despite D_LI:**
- Check faction relationships override
- Verify OnNewEnemy isn't forcing hostility
- Clear and reapply relationship

**Bidirectional not working:**
- Make sure Shift is held during click
- Apply each direction separately if needed

## Tips

1. **Test AI targeting** - Use to verify NPCs correctly identify threats
2. **Debug factions** - Override faction relationships temporarily
3. **Create scenarios** - Set up complex relationship dynamics
4. **Performance testing** - Test with many NPCs with different relationships

## Example Workflows

### Test Betrayal Mechanic
```
1. Spawn allied NPCs (same faction)
2. Use tool: NPC1 -> NPC2 = D_HT
3. NPC1 betrays and attacks NPC2
4. NPC2 retaliates (faction relationship)
```

### Test Guard Behavior
```
1. Spawn guard NPC
2. Spawn merchant NPC
3. Set: Guard -> Merchant = D_LI
4. Spawn enemy
5. Guard protects merchant
```

## Next Steps

- [Faction Tool](./04-faction-tool.md) - Manage faction membership
- [AI Tools](./06-ai-tools.md) - Control AI decision-making
- [Info Tool](./02-info-tool.md) - View current relationships
