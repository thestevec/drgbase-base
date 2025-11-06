# Set Relationship Tool

## Overview

The Set Relationship tool allows you to define how DrGBase nextbots and NPCs perceive and react to other entities. You can make nextbots like, hate, fear, or ignore specific entities or groups of entities. This tool is essential for setting up complex AI behaviors, team dynamics, and custom scenarios.

**Tool Name**: `Set Relationship`

**Primary Use**: Defining AI relationships between entities

## How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Set Relationship**
5. Open the context menu (C key) to configure:
   - Select the relationship type (Like, Hate, Fear, Ignore)
   - Optionally enable "Both ways" for mutual relationships
6. Use left-click to select source entities (nextbots/NPCs)
7. Right-click on a target entity to set the relationship
8. Press Reload (R) to clear your selection

## Actions

### Left Click
**Select/Deselect Entity**
- Click on a DrGBase nextbot or NPC to add it to your selection
- Click on an already-selected entity to remove it from selection
- **Hold Shift + Left Click** to select multiple entities without deselecting previous ones
- Selected entities are highlighted with colored halos based on their relationship with you

### Right Click
**Set Relationship**

**Normal Mode** (not holding Sprint):
- All selected entities will gain the configured relationship toward the entity you right-click
- **On an Entity**: Selected entities will react to that entity based on the relationship type
- **On the Ground/World**: Selected entities will react to YOU (the player) based on the relationship type

**Group Mode** (Hold Sprint/Shift while right-clicking):
- All selected entities will gain the configured relationship toward EACH OTHER
- Creates relationships between every pair of selected entities
- Useful for making groups of entities friendly or hostile to each other

### Reload (R Key)
**Clear Selection**
- Deselects all currently selected entities
- Clears the colored halos
- Useful for starting a new relationship configuration

## Options/Settings

### Relationship Dropdown
**ConVar**: `drgbase_tool_relationship_simple_disposition`

Choose how the selected entities will perceive the target:

| Relationship | Value | Effect |
|--------------|-------|--------|
| **Like** | 3 (D_LI) | Entities treat the target as friendly/ally |
| **Hate** | 1 (D_HT) | Entities treat the target as hostile/enemy |
| **Fear** | 2 (D_FR) | Entities are afraid of the target and may flee |
| **Ignore** | 4 (D_NU) | Entities treat the target as neutral/uninteresting |

### Both Ways Checkbox
**ConVar**: `drgbase_tool_relationship_simple_bothways`
**Default**: Unchecked (off)

When enabled, the relationship is applied in both directions:
- The selected entity gets the relationship toward the target
- The target gets the same relationship toward the selected entity
- Creates mutual relationships (mutual friendship, mutual hostility, etc.)

**Note**: Despite the name and help text, this option is not fully implemented in the current version. Relationships are one-directional.

## Visual Feedback

Selected entities are highlighted with colored halos based on their current relationship toward YOU (the local player):

- **Green**: The entity likes you
- **Red**: The entity hates you
- **Purple**: The entity fears you
- **Cyan**: The entity is neutral toward you
- **Orange**: Error or unknown relationship

This helps you see at a glance how your selected entities currently feel about you.

## Examples

### Example 1: Make a Nextbot Friendly to You
1. Left-click on a nextbot to select it
2. Set relationship to "Like" in the context menu
3. Aim at the ground (or yourself)
4. Right-click to apply
5. The nextbot now likes you and won't attack

### Example 2: Make a Nextbot Hostile to Another Entity
1. Left-click a nextbot to select it
2. Set relationship to "Hate"
3. Aim at a different nextbot or NPC
4. Right-click to apply
5. The selected nextbot will now attack the target

### Example 3: Create a Friendly Group
1. Hold Shift and left-click multiple nextbots to select them
2. Set relationship to "Like"
3. Hold Sprint (Shift) and right-click anywhere
4. All selected nextbots now like each other and won't fight

### Example 4: Make an Entity Afraid of You
1. Left-click a nextbot to select it
2. Set relationship to "Fear"
3. Aim at the ground
4. Right-click to apply
5. The nextbot will now run away from you

### Example 5: Set Up Two Opposing Teams
1. Select all members of Team A (Shift + left-click)
2. Set relationship to "Like"
3. Hold Sprint and right-click (Team A members like each other)
4. Press Reload to clear selection
5. Select all members of Team B
6. Set relationship to "Like"
7. Hold Sprint and right-click (Team B members like each other)
8. Press Reload to clear selection
9. Select all Team A members
10. Set relationship to "Hate"
11. Right-click on a Team B member (Team A hates Team B)
12. Press Reload and repeat for Team B hating Team A

### Example 6: Make All Nextbots Ignore an Entity
1. Hold Shift and select multiple nextbots
2. Set relationship to "Ignore"
3. Right-click on a specific entity
4. All selected nextbots will now ignore that entity

### Example 7: Quick Hostile Setup
1. Left-click a single nextbot
2. Set relationship to "Hate"
3. Aim at the ground and right-click
4. The nextbot is now hostile to you

## Tips

1. **Shift for Multiple**: Always hold Shift when left-clicking if you want to select multiple entities - otherwise you'll deselect previous selections
2. **Color Coding**: Watch the halo colors to understand current relationships before making changes
3. **Clear Often**: Use Reload frequently to clear your selection and start fresh - this prevents accidentally modifying the wrong entities
4. **Sprint for Groups**: Hold Sprint (default Shift) while right-clicking to set relationships between all selected entities
5. **Ground = You**: Right-clicking the ground is shorthand for targeting yourself as a player
6. **One-Way Streets**: Remember that relationships are one-directional - if you want mutual relationships, set them twice (once in each direction)
7. **Test with Info Tool**: Use the Nextbot Info tool (Page 2: AI) to verify relationship changes took effect
8. **NPCs Too**: This tool works with regular Garry's Mod NPCs, not just DrGBase nextbots
9. **Relationship Priority**: New relationships override old ones - you can change a nextbot's feelings by applying a different relationship
10. **Fear Behavior**: Fear relationship causes fleeing behavior - useful for prey-predator scenarios
11. **Ignore is Not Neutral**: Ignore (D_NU) means the entity won't care about the target at all - they won't help or attack
12. **Visual Selection Feedback**: Before right-clicking, look at your selected entities' halos to see their current state
13. **Batch Operations**: Select many entities at once to efficiently set up large-scale scenarios
14. **Combining with Factions**: Use factions first to group entities, then use relationships to define inter-faction dynamics
15. **Persistence**: Relationships persist until changed or the entity is removed - they don't reset automatically
