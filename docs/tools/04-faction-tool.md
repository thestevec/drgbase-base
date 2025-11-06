# Faction Tool

The Faction Tool allows dynamic modification of NPC factions for testing relationship systems and alliance mechanics.

## Accessing the Tool

**Spawn Menu:**
1. Press `Q`
2. Click Utilities tab
3. Select "DrGBase Faction Tool"

**Console:** `drgbase_faction <faction_name>`

## Usage

### Basic Usage
1. Equip the Faction Tool
2. Aim at target NPC
3. **Left Click** - Apply selected faction
4. **Right Click** - View current factions
5. **Reload** - Open faction selection menu

### Setting Faction

**Method 1: Console**
```
drgbase_faction FACTION_REBELS
drgbase_faction FACTION_COMBINE
drgbase_faction FACTION_ZOMBIES
```

**Method 2: Selection Menu**
1. Press Reload while holding tool
2. Select faction from list
3. Click Apply

## Available Factions

### Built-in Factions
- `FACTION_PLAYERS` - Friendly to players
- `FACTION_REBELS` - Half-Life 2 Rebels
- `FACTION_COMBINE` - Half-Life 2 Combine
- `FACTION_ZOMBIES` - Zombies and headcrabs
- `FACTION_ANTLIONS` - Antlion faction
- `FACTION_ANIMALS` - Wildlife
- `FACTION_BARNACLES` - Barnacles
- `FACTION_GMAN` - G-Man faction

### Custom Factions
Any custom factions defined in your addon will also appear in the menu.

## Testing Scenarios

### Test Faction Switching
```
-- Spawn two hostile NPCs
-- Use faction tool to make them allies
-- Verify they stop fighting
```

### Test Player Relationships
```
drgbase_faction FACTION_PLAYERS
-- NPC should become friendly to player
-- Verify it doesn't attack
```

### Test Multi-Faction
```
-- Add NPC to multiple factions
-- Right click with tool to view all factions
-- Verify relationships with all factions work
```

## Advanced Features

### Add to Faction (Don't Replace)
Hold **Shift + Left Click** to add faction without removing existing ones.

### Remove from Faction
Hold **Ctrl + Left Click** to remove NPC from selected faction.

### Clear All Factions
Hold **Ctrl + Shift + Left Click** to remove from all factions.

## Console Commands

```lua
drgbase_faction_add <faction>         -- Add to faction
drgbase_faction_remove <faction>      -- Remove from faction
drgbase_faction_clear                 -- Clear all factions
drgbase_faction_list                  -- List all available factions
```

## Testing Scenarios

### Alliance Testing
1. Spawn NPCs from different factions (Rebels vs Combine)
2. Use tool to change one to match the other
3. Verify they become allies

### Betrayal Testing
1. Spawn allied NPCs
2. Change one to enemy faction
3. Verify they become hostile

### Neutral Testing
1. Use `drgbase_faction_clear` to remove all factions
2. NPC should be neutral to everyone
3. Verify no auto-targeting occurs

## Console Variables

```lua
drgbase_faction_showfeedback <0|1>    -- Show faction change messages (default: 1)
drgbase_faction_playscfx <0|1>        -- Play sound effect on change (default: 1)
drgbase_faction_updaterel <0|1>       -- Auto-update relationships (default: 1)
```

## Troubleshooting

**Faction not changing:**
- Verify NPC is a DrGBase NPC
- Check console for error messages
- Try using console command directly

**Relationships not updating:**
- Set `drgbase_faction_updaterel 1`
- Manually update with relationship tool
- Restart map for full reset

**Custom factions not showing:**
- Verify faction is defined in shared realm
- Check faction string is correct
- Reload map/restart server

## Tips

1. **Quick testing** - Use console commands for rapid faction changes
2. **Visual feedback** - Enable showfeedback to see changes
3. **Multi-faction NPCs** - Use Shift+Click to test complex allegiances
4. **Reset easily** - Use clear command to start fresh

## Example Workflow

```lua
-- Test faction system
1. Spawn npc_drg_zombie (FACTION_ZOMBIES)
2. Spawn npc_drg_rebel (FACTION_REBELS)
3. They fight (as expected)
4. drgbase_faction FACTION_REBELS
5. Click on zombie
6. Zombie joins rebels, they become allies
7. Success!
```

## Next Steps

- [Relationship Tool](./05-relationship-tool.md) - Fine-tune relationships
- [Info Tool](./02-info-tool.md) - View faction membership
- [Advanced AI Example](../examples/advanced-ai.md) - Implement faction behavior
