# Info Tool

The Info Tool displays detailed real-time information about DrGBase NPCs, including health, AI state, relationships, and more.

## Accessing the Tool

**Spawn Menu:**
1. Press `Q`
2. Click Utilities tab
3. Select "DrGBase Info Tool"

**Console:** `drgbase_info`

## Usage

1. Equip the Info Tool from your weapons
2. Aim at any DrGBase NPC
3. Information appears on-screen

## Information Displayed

### Basic Stats
- **Entity Class** - NPC's entity class name
- **Health** - Current / Maximum health
- **Model** - Current model path
- **Position** - World coordinates

### AI State
- **State** - Current AI state (Idle, Combat, Patrol, etc.)
- **Target** - Current movement target
- **Enemy** - Current enemy entity
- **Relationship** - Relationship disposition to player

### Combat Info
- **Weapons** - Current weapon (if armed)
- **Ammo** - Current ammo count (if applicable)
- **Accuracy** - Weapon accuracy setting
- **Last Attack** - Time since last attack

### Faction Info
- **Factions** - All factions the NPC belongs to
- **Allies** - Count of nearby allies
- **Enemies** - Count of detected enemies

## Example Output

```
=== DrGBase Info ===
Class: npc_drg_zombie
Health: 73/100
State: Combat
Enemy: Player1
Target: Vector(123, 456, 0)
Faction: FACTION_ZOMBIES
Speed: 200 units/s
```

## Advanced Usage

### Console Variables

```lua
drgbase_info_range <number>     -- Max info display distance (default: 2000)
drgbase_info_update <number>    -- Update frequency in seconds (default: 0.1)
drgbase_info_detailed <0|1>     -- Show detailed info (default: 1)
```

### Filtering Info

Right-click while holding Info Tool to cycle display modes:
- **Basic** - Essential info only
- **Detailed** - All available info
- **Combat** - Combat-related info only
- **AI** - AI state and targets only

## Troubleshooting

**Info not showing:**
- Make sure you're aiming at a DrGBase NPC
- Check you're within info range (default 2000 units)
- Verify tool is equipped properly

**Info flickering:**
- Increase `drgbase_info_update` value
- Move closer to the NPC

## Tips

1. **Track health changes** - Use during combat testing to verify damage
2. **Debug AI** - Watch state changes in real-time
3. **Verify relationships** - Check faction allegiances quickly
4. **Performance testing** - Monitor update frequency during lag

## Next Steps

- [Damage Tool](./03-damage-tool.md) - Apply precise damage
- [AI Tools](./06-ai-tools.md) - Control AI behavior
- [Faction Tool](./04-faction-tool.md) - Change factions
