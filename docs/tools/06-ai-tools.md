# AI Control Tools

## Overview

DrGBase provides three tools for controlling AI behavior and perception. These tools allow you to pause AI processing, modify vision capabilities, and make yourself invisible to specific nextbots. They are essential for debugging, testing, and creating specific gameplay scenarios.

This document covers:
- **Disable AI** - Pause/resume AI processing
- **Toggle Omniscient** - Enable/disable wall-hacking vision
- **Toggle Notarget** - Make yourself invisible to specific nextbots

---

## Disable AI Tool

### Overview

The Disable AI tool allows you to freeze or unfreeze a nextbot's AI processing. When AI is disabled, the nextbot stops thinking, moving, and reacting to its environment. This is invaluable for debugging behavior, taking screenshots, or examining a nextbot's state.

**Tool Name**: `Disable AI`

**Primary Use**: Debugging and pausing nextbot AI

### How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Disable AI**
5. Left-click on any nextbot to toggle its AI on/off
6. All nextbots are highlighted with halos:
   - Green = AI enabled (active)
   - Red = AI disabled (frozen)

### Actions

#### Left Click
**Toggle AI State**
- Switches the nextbot's AI between enabled and disabled
- Works on any DrGBase nextbot
- Effect is immediate
- The nextbot's halo color changes to reflect the new state

#### Right Click
**No Action**

#### Reload (R Key)
**No Action**

### Visual Feedback

**All DrGBase nextbots** are highlighted with halos when this tool is active:
- **Green Halo**: AI is enabled - nextbot is active and thinking
- **Red Halo**: AI is disabled - nextbot is frozen

This makes it easy to see the AI state of all nextbots at a glance.

### Examples

#### Example 1: Freeze a Nextbot for Inspection
1. Select the Disable AI tool
2. Left-click on a moving nextbot
3. The nextbot freezes in place (red halo)
4. Switch to the Info tool to examine its state
5. Return to Disable AI tool and click again to resume

#### Example 2: Pause Combat for Screenshots
1. During a battle, select the Disable AI tool
2. Click all combatant nextbots to freeze them
3. Take your screenshots
4. Click each nextbot again to resume the battle

#### Example 3: Debug Pathfinding
1. Disable AI on a nextbot
2. Use the Mover tool to set a destination
3. Re-enable AI
4. Watch how it pathfinds to the destination

### Tips

1. **Quick Visual Check**: The green/red halos show all nextbots' AI states at once
2. **Combine with Info Tool**: Freeze a nextbot, then use Info tool to examine its AI state on Page 2
3. **Performance Testing**: Disable AI on many nextbots to test performance without AI processing
4. **Animation Continues**: Disabled AI doesn't freeze animations - the model may still animate
5. **Page 2 Indicator**: In the Info tool, Page 2 (AI) shows in red when AI is disabled
6. **Not Physics**: Disabling AI doesn't disable physics - nextbots can still be pushed or fall
7. **Persistent**: AI state persists until you toggle it again
8. **All Visible**: Unlike other tools, this one shows halos for ALL nextbots, not just selected ones
9. **Quick Toggle**: You can rapidly click different nextbots to toggle their states

---

## Toggle Omniscient Tool

### Overview

The Toggle Omniscient tool grants or removes omniscience from nextbots. An omniscient nextbot can detect enemies anywhere in the map, regardless of walls, distance, or line of sight. This is useful for testing behaviors, creating challenging encounters, or debugging detection issues.

**Tool Name**: `Toggle Omniscient`

**Primary Use**: Controlling nextbot vision and enemy detection capabilities

### How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Toggle Omniscient**
5. Left-click on any nextbot to toggle omniscience on/off
6. All nextbots are highlighted with halos:
   - Green = Omniscient (can see through walls)
   - Red = Normal vision

### Actions

#### Left Click
**Toggle Omniscience**
- Switches the nextbot between omniscient and normal vision
- Works on any DrGBase nextbot
- Effect is immediate
- The nextbot can now detect enemies through walls and at any distance (if enabled)

#### Right Click
**No Action**

#### Reload (R Key)
**No Action**

### Visual Feedback

**All DrGBase nextbots** are highlighted with halos when this tool is active:
- **Green Halo**: Omniscient - can see through walls and detect enemies anywhere
- **Red Halo**: Normal vision - requires line of sight and range

### How Omniscience Works

When a nextbot is omniscient:
- It can detect enemies through walls and solid objects
- Distance limitations may still apply depending on implementation
- It doesn't need line of sight to track enemies
- It can "sense" enemies it has never seen before
- Perfect for creating "psychic" or "all-knowing" enemies

### Examples

#### Example 1: Create a Relentless Pursuer
1. Select Toggle Omniscient tool
2. Left-click a nextbot (turns green)
3. The nextbot can now track you through walls
4. Hide behind objects - it will still pursue

#### Example 2: Test Detection Logic
1. Enable omniscience on a nextbot
2. Move far away or behind walls
3. Verify the nextbot still detects you
4. Disable omniscience and test normal vision

#### Example 3: Create Different Difficulty Levels
1. Make some nextbots omniscient (hard mode)
2. Leave others with normal vision (normal mode)
3. Players must handle both types differently

### Tips

1. **Info Tool Page 2**: Check the Info tool's AI page to see if omniscience is enabled
2. **Not Instant Aggro**: Omniscience affects detection, but relationships still matter
3. **Performance**: Omniscient nextbots may use more CPU since they can detect more entities
4. **Strategic Placement**: Use omniscient nextbots as "sentries" that can detect intruders anywhere
5. **Debugging**: Enable omniscience to verify a nextbot's attack/pursuit behavior works correctly
6. **Disable for Stealth**: Turn off omniscience to create stealth gameplay scenarios
7. **All Visible**: Like Disable AI, this tool shows halos for all nextbots
8. **Combine with Notarget**: Use Notarget to test if omniscience is working correctly

---

## Toggle Notarget Tool

### Overview

The Toggle Notarget tool makes you invisible to specific nextbots. When notarget is enabled on a nextbot, it cannot detect, target, or react to you. This is perfect for testing, observing behavior without interference, or creating stealth mechanics.

**Tool Name**: `Toggle Notarget`

**Primary Use**: Making yourself invisible to specific nextbots

### How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Toggle Notarget**
5. Aim at a nextbot
6. Left-click to toggle whether it can see you
7. The nextbot you're aiming at is highlighted with a halo:
   - Green = Cannot see you (you're ignored)
   - Red = Can see you (normal)

### Actions

#### Left Click
**Toggle Notarget on Nextbot**
- Toggles whether the targeted nextbot ignores you
- Only affects the nextbot you click on
- Each nextbot has independent notarget state for you
- Effect is immediate

#### Right Click
**No Action**

#### Reload (R Key)
**No Action**

### Visual Feedback

**Only the nextbot you're aiming at** is highlighted:
- **Green Halo**: Notarget enabled - this nextbot ignores you
- **Red Halo**: Notarget disabled - this nextbot can detect you normally

Unlike the other AI tools, this only shows a halo for the nextbot you're currently looking at.

### How Notarget Works

When notarget is enabled for a nextbot:
- The nextbot cannot detect you as a target
- It won't attack you or react to your presence
- It will still react to other entities normally
- It's as if you don't exist to that specific nextbot
- Other players and entities are unaffected

### Examples

#### Example 1: Observe Natural Behavior
1. Enable notarget on all nextbots you want to observe
2. Watch them interact with other entities without interference
3. Perfect for recording or studying AI behavior

#### Example 2: Test Combat Against Specific Enemies
1. Set up multiple nextbots
2. Enable notarget on some, leave it off on others
3. Only the ones without notarget will attack you
4. Test specific combat scenarios

#### Example 3: Create Safe Zones
1. Enable notarget on nextbots in certain areas
2. Players can safely pass through those areas
3. Useful for tutorials or safe zones in complex maps

#### Example 4: Debug Without Interference
1. Enable notarget on aggressive nextbots
2. Get close to inspect them without being attacked
3. Disable notarget when done testing

### Tips

1. **Per-Nextbot**: Notarget is set individually for each nextbot - you need to click each one
2. **Only You**: Notarget only affects YOU (the player using the tool) - other players/entities are unaffected
3. **Not Relationships**: Notarget overrides relationships - even a nextbot that hates you won't attack if notarget is enabled
4. **Hover to Check**: Aim at a nextbot to see its notarget state via the halo color
5. **Quick Toggle**: Click again to disable notarget and become visible again
6. **Async Check**: The tool uses asynchronous checking for notarget state, so there may be a brief delay in halo display
7. **Not Global**: Unlike Disable AI and Omniscient, this only shows one halo at a time (the nextbot you're aiming at)
8. **Multiplayer**: In multiplayer, each player has independent notarget states for each nextbot
9. **Combine with Omniscient**: Test if an omniscient nextbot can be blocked by notarget
10. **Careful Management**: Keep track of which nextbots have notarget enabled - there's no "show all" view

---

## AI Tools Comparison

| Tool | Affects | Visual | Scope | Use Case |
|------|---------|--------|-------|----------|
| **Disable AI** | AI processing | All nextbots | Global view | Freeze/unfreeze nextbots |
| **Toggle Omniscient** | Vision/detection | All nextbots | Global view | Wall-hacking vision |
| **Toggle Notarget** | Target detection | Current nextbot only | Single entity | Stealth/invisibility |

## Combined Usage Tips

1. **Disable AI + Info Tool**: Freeze a nextbot, then examine its AI state
2. **Omniscient + Notarget**: Test if notarget blocks omniscient detection
3. **All Three + Relationship Tool**: Create complex AI scenarios with precise control
4. **Disable AI + Mover Tool**: Freeze nextbots, position them, then re-enable AI
5. **Omniscient + Faction Tool**: Create all-seeing guards for specific factions
6. **Notarget + Damage Tool**: Attack nextbots without retaliation for testing
7. **Visual Debugging**: Use the halo colors to quickly assess the state of all nextbots
