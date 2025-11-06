# Nextbot Info Tool

## Overview

The Nextbot Info tool is a comprehensive debugging and monitoring tool that displays real-time information about DrGBase nextbots. It provides 6 different information pages, each showing different aspects of the nextbot's state. The tool displays information both on the toolgun's screen and through colored halos on the selected nextbot.

**Tool Name**: `Nextbot Info`

**Primary Use**: Real-time nextbot debugging and monitoring

## How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Nextbot Info**
5. Aim at a DrGBase nextbot
6. Left-click to select it
7. Right-click to cycle through information pages

## Actions

### Left Click
**Select/Deselect Nextbot**
- Click on a DrGBase nextbot to select it and view its information
- Click on the same nextbot again to deselect it
- Only DrGBase nextbots can be selected (not regular NPCs or entities)

### Right Click
**Cycle Information Pages**
- Cycles through the 6 available information pages
- If no nextbot is selected, plays an error sound
- Pages loop back to the first page after the last one
- Plays a light switch sound when cycling pages

### Reload (R Key)
**Clear Selection**
- Deselects the currently selected nextbot
- Clears the tool screen

## Information Pages

The tool displays 6 different pages of information. The current page number and name are shown at the bottom of the tool screen.

### Page 1: Status
Displays health and status information:
- **Health**: Current/Max health with color gradient (green = healthy, red = critical)
- **Regen**: Health regeneration rate
- **Godmode**: Whether godmode is enabled or disabled
- **Status**: Current state (Alive/Down/Dead) with color coding

**Halo Color**: Gradient from green (full health) to red (low health)

### Page 2: AI
Displays AI and relationship information:
- **Enemy**: Current enemy entity (name for players, class for entities, or "None")
- **Relationship**: Your relationship with the nextbot (Like/Hate/Fear/Neutral/Error)
- **Spotted**: Whether the nextbot has spotted the local player (True/False)
- **Omniscient**: Whether omniscience is enabled (True/False)

**Halo Color**:
- Green = Like
- Red = Hate
- Purple = Fear
- Cyan = Neutral
- Orange = Error

**Note**: If AI is disabled, the page name appears in red on the tool screen.

### Page 3: Possession
Displays possession information:
- **Possessed by**: Name of the player possessing the nextbot
- **Locked on**: Entity the possessed nextbot is locked onto
- If not possessed, displays "This nextbot isn't possessed"

**Halo Color**:
- Green = Possessed
- Red = Not possessed

### Page 4: Movement
Displays movement and physics information:
- **Speed**: Current speed (rounded to 2 decimal places)
- **Scale**: Current size scale (rounded to 2 decimal places)
- **Movement**: Current movement vector with detailed formatting

**Halo Color**: Cyan

### Page 5: Animation
Displays animation information:

**For regular nextbots**:
- **Sequence**: Current sequence name
- **Activity**: Current activity name
- **Attacking?**: Whether the current sequence is an attack animation (True/False)

**For sprite nextbots**:
- **Animation**: Current sprite animation name
- **Frame**: Current frame number
- **Attacking?**: Whether the current sequence is an attack animation

**Halo Color**: Cyan

### Page 6: Viewcam
Displays a live view from the nextbot's perspective:
- Shows what the nextbot sees
- Camera positioned at the nextbot's eye position
- Uses the nextbot's eye angles for orientation
- Renders in real-time on the tool screen
- May show the local player if they're in view

**Halo Color**: None (viewcam rendering)

**Note**: This page uses real-time rendering and may impact performance slightly.

## Options/Settings

This tool has no configurable options. All information is displayed automatically based on the selected nextbot's state.

## Examples

### Example 1: Monitor Health During Combat
1. Select a nextbot that's about to enter combat
2. Stay on Page 1 (Status)
3. Watch the health bar change in real-time as it takes damage
4. The halo color will shift from green to red as health decreases

### Example 2: Debug AI Targeting
1. Select a nextbot
2. Cycle to Page 2 (AI)
3. Check if it has spotted you and what its current enemy is
4. Verify the relationship status is correct
5. Check if omniscience is affecting its behavior

### Example 3: Watch Movement Behavior
1. Select a nextbot
2. Cycle to Page 4 (Movement)
3. Watch the speed and movement vector update in real-time
4. Use with the Mover tool to verify pathfinding behavior

### Example 4: Test Possession
1. Possess a nextbot (using possession system)
2. Select it with the Info tool
3. Cycle to Page 3 (Possession)
4. Verify you're listed as the possessor
5. Check what the nextbot is locked onto

### Example 5: Debug Animations
1. Select a nextbot
2. Cycle to Page 5 (Animation)
3. Watch which sequences/activities play during different behaviors
4. Verify attack animations are properly flagged

### Example 6: See Through Nextbot's Eyes
1. Select a nextbot
2. Cycle to Page 6 (Viewcam)
3. View what the nextbot can see from its perspective
4. Useful for debugging line-of-sight and vision problems

## Tips

1. **Quick Health Check**: The halo color on Page 1 provides instant visual feedback about nextbot health
2. **AI Debugging**: Page 2 is essential for debugging relationship and detection issues
3. **Performance**: The Viewcam page (6) is more resource-intensive than others; avoid leaving it on for extended periods with many nextbots
4. **Combining with Other Tools**: Use the Info tool alongside the Disable AI tool to freeze a nextbot and examine its state in detail
5. **Page Navigation**: The ">>" symbol at the bottom-right of the tool screen indicates you can right-click to advance pages
6. **Sound Cues**: Listen for sound feedback - a button click means no nextbot is selected, a light switch means page changed successfully
7. **Color Reference**: Remember the relationship color coding - green (friendly), red (hostile), purple (afraid), cyan (neutral)
8. **Multiple Monitoring**: While you can only select one nextbot at a time, you can quickly switch between nextbots by left-clicking different ones
9. **Status at a Glance**: The page number shown as "(1)" through "(6)" helps you remember which page you're on
10. **Real-time Updates**: All information updates continuously while the nextbot is selected - no need to reselect to refresh
