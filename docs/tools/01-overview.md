# DrGBase Developer Tools Overview

## Introduction

DrGBase provides a comprehensive suite of developer tools accessible through the Garry's Mod Toolgun. These tools help you debug, test, and manipulate DrGBase nextbots during development and gameplay.

## Accessing the Tools

1. Equip the **Toolgun** (default key: Q)
2. Right-click to open the tool menu
3. Navigate to the **DrGBase** tab
4. Select the desired tool from the **Tools** category

## Available Tools

### Information & Debugging
- **Nextbot Info** - View detailed real-time information about a nextbot across 6 different pages (Status, AI, Possession, Movement, Animation, Viewcam)

### Combat & Damage
- **Inflict Damage** - Apply customizable damage to entities with various damage types (crush, slash, blast, burn, etc.)

### AI Behavior Control
- **Disable AI** - Toggle AI processing on/off for nextbots
- **Toggle Omniscient** - Enable/disable omniscience (nextbots can see through walls and detect enemies anywhere)
- **Toggle Notarget** - Make yourself invisible to specific nextbots

### Relationships & Factions
- **Faction Tool** - Manage faction membership for nextbots and players
- **Set Relationship** - Define how nextbots/NPCs relate to other entities (like, hate, fear, ignore)

### Entity Manipulation
- **Toggle Godmode** - Enable/disable invincibility for nextbots
- **Nextbot Mover** - Command nextbots to move to specific locations
- **Change Scale** - Resize nextbots dynamically

## Common Features

### Entity Selection
Many tools use a selection system:
- **Left Click** - Select/deselect an entity
- **Shift + Left Click** - Select multiple entities
- **Reload (R)** - Clear all selected entities

Selected entities are highlighted with colored halos indicating their state.

### Visual Feedback
Tools provide visual feedback through:
- **Colored Halos** - Entities are outlined in different colors based on their state
  - Green: Enabled/Active/Friendly
  - Red: Disabled/Inactive/Hostile
  - Cyan: Neutral/Selected
  - Purple: Fear relationship
  - Orange: Error/Warning states
- **Tool Screen** - The toolgun's screen displays relevant information
- **Sound Feedback** - Audio cues confirm actions

### Color Coding
- **Green**: Positive states (enabled, friendly, healthy)
- **Red**: Negative states (disabled, hostile, low health)
- **Cyan**: Neutral states (selected, no relationship)
- **Purple**: Fear relationships
- **Orange**: Warnings or special states

## General Usage Tips

1. **Testing AI**: Use the Info tool to monitor AI behavior in real-time
2. **Setting Up Scenarios**: Combine Faction, Relationship, and Mover tools to create complex scenarios
3. **Debugging**: Use Disable AI and Godmode to pause and inspect nextbot behavior
4. **Batch Operations**: Use Shift+Click with relationship and mover tools to affect multiple entities
5. **Quick Reset**: The Reload key (R) clears selections in most tools

## Tool Categories

### Quick Reference

| Tool | Primary Use | Left Click | Right Click | Reload |
|------|-------------|------------|-------------|--------|
| Nextbot Info | View details | Select nextbot | Cycle pages | Clear selection |
| Inflict Damage | Testing damage | Damage entity | - | Damage self |
| Faction Tool | Manage factions | Apply factions | Copy factions | Clear list |
| Set Relationship | AI relationships | Select entity | Set relationship | Clear selection |
| Disable AI | Debug AI | Toggle AI | - | - |
| Toggle Omniscient | AI perception | Toggle omniscience | - | - |
| Toggle Notarget | Stealth testing | Toggle notarget | - | - |
| Toggle Godmode | Invincibility | Toggle godmode | - | - |
| Nextbot Mover | Movement testing | Select nextbot | Set destination | Clear selection |
| Change Scale | Resize nextbots | Scale up (+10%) | Scale down (-10%) | Reset to 1.0 |

## Next Steps

- Read individual tool documentation for detailed usage instructions
- Experiment with combining tools for complex testing scenarios
- Use the Info tool to understand how other tools affect nextbots
