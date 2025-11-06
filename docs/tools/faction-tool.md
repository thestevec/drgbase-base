# Faction Tool

## Overview

The Faction Tool allows you to view, copy, and modify factions for NPCs and players without editing code. Perfect for testing faction relationships and quickly setting up teams.

## Usage

**Controls:**
- **Left Click on NPC**: Apply current faction list to NPC
- **Left Click on Ground**: Apply faction list to yourself (player)
- **Right Click on NPC**: Copy NPC's factions to tool
- **Right Click on Player**: Copy player's factions to tool
- **Right Click on Ground**: Copy your own factions to tool
- **Reload (R)**: Clear faction list

## Tool Panel

The Faction Tool has a configuration panel in the Tool Gun menu:

### Faction List Display

Shows all currently selected factions:
```
Factions:
FACTION_ZOMBIES
FACTION_PASSIVE
FACTION_CUSTOM
```

**Actions:**
- **Left click** on faction → Remove from list
- **Right click** on faction → Copy to text entry field

### Adding Factions

1. Type faction name in text entry
2. Click "Insert faction" button
3. Faction added to list

### Managing List

- **Clear factions** button → Remove all factions
- List persists between sessions
- Saved to console variable: `drgbase_tool_faction_list`

## Workflows

### Workflow 1: Copy Factions Between NPCs

**Goal:** Make NPC2 have same factions as NPC1

**Steps:**
1. Equip Faction Tool
2. Right-click on NPC1 (copies factions)
3. Left-click on NPC2 (applies factions)
4. Done!

**Example:**
```
NPC1 (Zombie): FACTION_ZOMBIES
→ Right-click on NPC1
→ Left-click on NPC2
NPC2 now has: FACTION_ZOMBIES
```

### Workflow 2: Create Custom Faction Team

**Goal:** Create team of NPCs in custom faction

**Steps:**
1. Open Faction Tool panel
2. Type "FACTION_MYTEAM" in text entry
3. Click "Insert faction"
4. Left-click each NPC to apply
5. All NPCs now in same faction

### Workflow 3: Set Player Factions

**Goal:** Join specific faction as player

**Steps:**
1. Right-click on NPC with desired factions (or configure manually)
2. Aim at ground
3. Left-click
4. You now have those factions!

**Use case:** Test NPC relationships with players in different factions

### Workflow 4: Multi-Faction Setup

**Goal:** Add NPCs to multiple factions

**Steps:**
1. Clear faction list (Reload key or Clear button)
2. Add first faction: "FACTION_GUARDS"
3. Add second faction: "FACTION_REBELS"
4. Add third faction: "FACTION_TRADERS"
5. Apply to NPCs

Result: NPCs belong to all three factions simultaneously

## Built-in Factions

DrGBase includes these factions by default:

```lua
FACTION_PLAYERS      -- All players
FACTION_ZOMBIES      -- Zombie faction
FACTION_ANTLIONS     -- Antlion faction
FACTION_COMBINE      -- Combine soldiers
FACTION_REBELS       -- Rebel forces
FACTION_PASSIVE      -- Peaceful NPCs
```

## Creating Custom Factions

### Method 1: Use Faction Tool

Simply type any faction name:
```
FACTION_BANDITS
FACTION_GUARDS
FACTION_MERCHANTS
FACTION_MYSTIC_ORDER
```

**Note:** Faction is created automatically when first used

### Method 2: In Code (Permanent)

```lua
-- In autorun file or addon init:
FACTION_BANDITS = DrGBase.CreateFaction("Bandits")
FACTION_GUARDS = DrGBase.CreateFaction("Guards")

-- Then use in NPC code:
ENT.Factions = {FACTION_BANDITS}
```

## Faction Naming Convention

**Best practices:**
- Use ALL_CAPS
- Prefix with "FACTION_"
- Use underscores for spaces
- Be descriptive

**Good examples:**
```
FACTION_CITY_GUARDS
FACTION_DARK_WIZARDS
FACTION_FRIENDLY_ANIMALS
FACTION_BOSS_MINIONS
```

**Bad examples:**
```
guards           -- Not ALL_CAPS
MYFAC            -- Not descriptive
faction guards   -- Has space, no underscore
GUARDS           -- Missing FACTION_ prefix
```

## Practical Examples

### Example 1: Team Deathmatch Setup

**Create 2 teams:**

**Red Team:**
1. Open Faction Tool panel
2. Clear factions
3. Add "FACTION_RED_TEAM"
4. Apply to red NPCs
5. Set relationship: Hate Blue Team

**Blue Team:**
1. Clear factions
2. Add "FACTION_BLUE_TEAM"
3. Apply to blue NPCs
4. Set relationship: Hate Red Team

**Result:** Two opposing teams

### Example 2: Guards Protecting Traders

**Traders:**
1. Add to tool: "FACTION_TRADERS"
2. Apply to merchant NPCs

**Guards:**
1. Clear faction list
2. Add "FACTION_GUARDS"
3. Apply to guard NPCs

**In NPC code:**
```lua
-- Guards like traders
function ENT:CustomInitialize()
    self:SetFactionRelationship(FACTION_TRADERS, D_LI)
end
```

### Example 3: Free-For-All

**Remove all factions:**
1. Right-click NPC to copy factions
2. Clear all factions from list
3. Left-click NPCs to remove factions

**Result:** NPCs with no factions (use default relationships)

### Example 4: Player as NPC Ally

**Join NPC faction:**
1. Right-click friendly NPC
2. Aim at ground
3. Left-click

**Result:** You're now in same faction as NPC

**Then in NPC code:**
```lua
function ENT:CustomInitialize()
    -- NPCs like their own faction
    self:SetSelfFactionRelationship(D_LI)
end
```

## Faction Relationships

Factions work with the Relationship system:

```lua
-- NPC code example:
function ENT:CustomInitialize()
    -- Like own faction
    self:SetSelfFactionRelationship(D_LI)

    -- Hate other factions
    self:SetFactionRelationship(FACTION_BANDITS, D_HT)
    self:SetFactionRelationship(FACTION_ZOMBIES, D_HT)

    -- Fear powerful faction
    self:SetFactionRelationship(FACTION_BOSSES, D_FR)

    -- Neutral to civilians
    self:SetFactionRelationship(FACTION_CIVILIANS, D_NU)
end
```

## Viewing Current Factions

**For NPCs:**
Use Info Tool (AI page) to see relationships

**In Console:**
```lua
-- For entity
lua_run PrintTable(Entity(1):GetFactions())

-- For player
lua_run PrintTable(Player(1):DrG_GetFactions())
```

## Troubleshooting

### Faction not working
- Ensure faction name is correct (case-sensitive)
- Check if NPC has relationship rules for that faction
- Verify faction was applied (right-click to copy and check)
- Use Info Tool to view actual relationships

### Can't add faction
- Check for typos in faction name
- Ensure using valid characters (letters, numbers, underscores)
- Try adding manually in code first

### Tool doesn't apply factions
- Make sure you're clicking DrGBase NPCs
- Check console for errors
- Verify NPC is valid and spawned

### Factions reset on spawn
- Factions applied with tool are temporary
- For permanent factions, set in NPC code:
  ```lua
  ENT.Factions = {FACTION_MYTEAM}
  ```

## Advanced Usage

### Lua Commands

**Add faction manually:**
```lua
-- For NPC
Entity(1):JoinFaction("FACTION_CUSTOM")

-- For player
Player(1):DrG_JoinFaction("FACTION_CUSTOM")
```

**Remove factions:**
```lua
-- Remove all
Entity(1):LeaveAllFactions()

-- Remove specific
Entity(1):LeaveFaction("FACTION_ZOMBIES")
```

**Get factions:**
```lua
local factions = Entity(1):GetFactions()
PrintTable(factions)
```

## Console Variable

Faction list stored in:
```
drgbase_tool_faction_list
```

**View current value:**
```
drgbase_tool_faction_list
```

**Manually set (JSON format):**
```
drgbase_tool_faction_list [["FACTION_TEST"],["FACTION_CUSTOM"]]
```

## Source Code

File: `lua/weapons/gmod_tool/stools/drgbase_tool_faction.lua`

## Related Tools

- [Relationship Tool](debug-tools.md#relationship-tool) - Set specific entity relationships
- [Info Tool](info-tool.md) - View current relationships

## See Also

- [Relationship System](../examples/07-relationships.md)
- [Faction Examples](../examples/07-relationships.md#faction-system)
- [Multi-Faction Setup](../examples/07-relationships.md#complex-faction-systems)
