# Faction Tool

## Overview

The Faction Tool allows you to manage faction memberships for DrGBase nextbots and players. Factions are groups that entities can belong to, which affect their relationships and behaviors. You can copy factions from one entity, create custom faction lists, and apply them to nextbots or yourself.

**Tool Name**: `Faction Tool`

**Primary Use**: Managing faction memberships for nextbots and players

## How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Faction Tool**
5. Open the context menu (C key) to build your faction list:
   - Type faction names in the text entry field
   - Click "Insert faction" to add them to the list
   - Click on factions in the list to remove them
   - Right-click factions to copy them to the text entry
6. Use the tool to copy and apply factions

## Actions

### Left Click
**Apply Factions**
- **On a DrGBase Nextbot**: Removes all of the nextbot's current factions and applies the factions from your list
- **On the Ground/World**: Removes all of your (the player's) current factions and applies the factions from your list
- **On Other Entities**: No effect

The tool completely replaces existing factions - it doesn't add to them.

### Right Click
**Copy Factions**
- **On a DrGBase Nextbot**: Copies all of the nextbot's current factions to your faction list
- **On a Player**: Copies all of the player's current factions to your faction list
- **On the Ground/World**: Copies all of your own factions to your faction list
- **On Other Entities**: No effect

This overwrites your current faction list with the copied factions.

### Reload (R Key)
**Clear Faction List**
- Clears all factions from your faction list in the context menu
- Does not affect any entity's factions - only clears the tool's list
- Useful for starting fresh when building a new faction list

## Options/Settings

### Faction List
**ConVar**: `drgbase_tool_faction_list`

A dynamic list showing all factions you've added. This list determines what factions will be applied when you left-click.

**Managing the List**:
- **Add Faction**: Type a name in the text field and click "Insert faction"
- **Remove Faction**: Left-click on a faction in the list
- **Copy to Entry**: Right-click on a faction to copy it to the text entry field
- **Clear All**: Click "Clear factions" button to remove all factions

### Custom Faction Text Entry
A text field where you can type custom faction names.

**Faction Naming**:
- Faction names are automatically converted to UPPERCASE
- Can be any string (letters, numbers, special characters)
- Common faction names: TEAM_RED, TEAM_BLUE, ALLIES, AXIS, FRIENDLY, HOSTILE
- Names are case-insensitive when checking membership (automatically uppercased)

### Insert Faction Button
Adds the faction name from the text entry to the faction list.

### Clear Factions Button
Removes all factions from the list at once.

## Examples

### Example 1: Make Nextbots Friendly to Each Other
1. Type "FRIENDLY_TEAM" in the text entry
2. Click "Insert faction"
3. Left-click on the first nextbot to apply the faction
4. Left-click on the second nextbot to apply the faction
5. Both nextbots now share the FRIENDLY_TEAM faction

### Example 2: Copy and Apply Factions
1. Right-click on a nextbot to copy its factions
2. The faction list updates with that nextbot's factions
3. Left-click on other nextbots to apply the same factions

### Example 3: Create Multiple Faction Membership
1. Type "TEAM_RED" and click "Insert faction"
2. Type "SOLDIERS" and click "Insert faction"
3. Type "GUARDS" and click "Insert faction"
4. Left-click a nextbot to make it a member of all three factions
5. Any other entity in TEAM_RED, SOLDIERS, or GUARDS will recognize this nextbot

### Example 4: Remove All Factions from a Nextbot
1. Click "Clear factions" to empty the list
2. Left-click a nextbot
3. The nextbot is now removed from all factions

### Example 5: Join Factions as a Player
1. Build a faction list (e.g., "TEAM_BLUE")
2. Aim at the ground or world brush
3. Left-click to join those factions as a player
4. Nextbots in the same faction will recognize you

### Example 6: Copy Your Own Factions
1. Right-click on the ground
2. Your current factions are copied to the list
3. You can now apply these to nextbots

### Example 7: Create Team-Based Setup
1. Create a "TEAM_RED" faction list
2. Apply it to half your nextbots
3. Clear the list and create a "TEAM_BLUE" faction list
4. Apply it to the other half
5. Use the Relationship tool to make teams hostile to each other

## Tips

1. **Uppercase Automatic**: All faction names are converted to uppercase, so "team_red", "TEAM_RED", and "Team_Red" are all the same
2. **Complete Replacement**: Left-clicking completely replaces all existing factions - if you want to add to existing factions, copy them first, then add more
3. **Quick Copy**: Right-click to quickly copy factions from a well-configured nextbot to apply to others
4. **World Click for Self**: Aiming at the ground and clicking applies factions to yourself as a player
5. **List Management**: Right-click on a faction in the list to copy it to the text entry for easy modification
6. **Testing Relationships**: Use factions in combination with the Relationship tool for complex team dynamics
7. **Multiple Membership**: Entities can belong to multiple factions simultaneously - this allows for complex hierarchies
8. **Faction Relationships**: Factions themselves don't define relationships - use the Relationship tool to define how faction members interact
9. **Persistent Storage**: The faction list persists while you have the tool selected, making it easy to apply the same factions to multiple nextbots
10. **Empty List Effect**: Applying an empty faction list removes all factions from the target
11. **Debugging**: Use the console command to check the current faction list value if needed
12. **Naming Convention**: Use descriptive names like "TEAM_name", "FACTION_name", or "GROUP_name" for clarity
13. **Player Factions**: Players can join factions too - this affects how faction-aware nextbots treat them
14. **No Visual Indicator**: There's no visual indicator of which factions an entity belongs to - use right-click to copy and check
15. **Combining with Info Tool**: While the Info tool doesn't show factions directly, you can use the Relationship page to infer faction effects
