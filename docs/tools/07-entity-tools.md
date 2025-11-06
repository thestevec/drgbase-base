# Entity Manipulation Tools

## Overview

DrGBase provides three tools for directly manipulating nextbot entities. These tools allow you to make nextbots invincible, force them to move to specific locations, and change their physical size. They are essential for testing, creating scenarios, and debugging physical behaviors.

This document covers:
- **Toggle Godmode** - Enable/disable invincibility
- **Nextbot Mover** - Force nextbots to move to specific positions
- **Change Scale** - Resize nextbots dynamically

---

## Toggle Godmode Tool

### Overview

The Toggle Godmode tool enables or disables invincibility for nextbots. When godmode is active, the nextbot cannot take damage from any source. This is useful for testing behaviors without worrying about death, creating invincible enemies, or protecting nextbots during setup.

**Tool Name**: `Toggle Godmode`

**Primary Use**: Making nextbots invincible or vulnerable

### How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Toggle Godmode**
5. Left-click on any nextbot to toggle godmode on/off
6. All nextbots are highlighted with halos:
   - Green = Godmode enabled (invincible)
   - Red = Godmode disabled (vulnerable)

### Actions

#### Left Click
**Toggle Godmode State**
- Switches the nextbot's godmode between enabled and disabled
- Works on any DrGBase nextbot
- Effect is immediate
- When enabled, the nextbot takes no damage from any source
- When disabled, the nextbot takes damage normally

#### Right Click
**No Action**

#### Reload (R Key)
**No Action**

### Visual Feedback

**All DrGBase nextbots** are highlighted with halos when this tool is active:
- **Green Halo**: Godmode is enabled - nextbot is invincible
- **Red Halo**: Godmode is disabled - nextbot is vulnerable

This makes it easy to see which nextbots are protected at a glance.

### How Godmode Works

When godmode is enabled:
- The nextbot takes zero damage from all sources
- Weapons, explosions, falls, and environmental hazards have no effect
- The nextbot can still be pushed by physics
- Animations and reactions to damage may still play (but no actual damage occurs)
- Health remains unchanged when attacked
- The nextbot can still die from direct :Kill() calls in code

### Examples

#### Example 1: Test Behavior Without Death
1. Select Toggle Godmode tool
2. Left-click a nextbot (turns green)
3. The nextbot is now invincible
4. Test combat behaviors without worrying about the nextbot dying

#### Example 2: Create an Invincible Boss
1. Enable godmode on a boss nextbot
2. Players must complete objectives to defeat it (rather than just dealing damage)
3. Creates unique gameplay mechanics

#### Example 3: Protect Setup Nextbots
1. Enable godmode on nextbots while setting up a scenario
2. Configure relationships, factions, and positions without damage
3. Disable godmode when ready to start the scenario

#### Example 4: Test Damage Display
1. Enable godmode on a nextbot
2. Attack it with various weapons
3. Verify damage effects display correctly even when no damage is dealt
4. Use Info tool Page 1 (Status) to confirm health doesn't decrease

#### Example 5: Create Immortal Allies
1. Enable godmode on friendly nextbots
2. They can assist players without risk of death
3. Useful for escort missions or support characters

### Tips

1. **Info Tool Integration**: Page 1 (Status) of the Info tool shows "Godmode: Enabled" or "Disabled"
2. **Visual Check**: Green halos make it obvious which nextbots are protected
3. **Still Moveable**: Godmode nextbots can still be pushed by physics and explosions
4. **Persistent**: Godmode state persists until you toggle it again
5. **All Visible**: Shows halos for ALL nextbots, making it easy to see protection status
6. **Test Damage Systems**: Use godmode to test if damage detection and effects work correctly
7. **Combine with Damage Tool**: Attack godmode nextbots to verify godmode is working
8. **Not Immortal**: Godmode only prevents damage - nextbots can still be killed by code
9. **Quick Toggle**: Click again to disable godmode and restore vulnerability
10. **Performance**: Godmode has no performance impact - it simply blocks damage

---

## Nextbot Mover Tool

### Overview

The Nextbot Mover tool forces nextbots to move to specific locations. You select one or more nextbots, then specify where you want them to go. The nextbots will pathfind to that location using their normal movement system. This is perfect for testing pathfinding, positioning nextbots, or creating scripted movement.

**Tool Name**: `Nextbot Mover`

**Primary Use**: Commanding nextbots to move to specific positions

### How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Nextbot Mover**
5. Left-click to select one or more nextbots (hold Shift for multiple)
6. Right-click where you want them to go
7. The selected nextbots will pathfind to that location
8. Press Reload (R) to clear your selection

### Actions

#### Left Click
**Select/Deselect Nextbot**
- Click on a DrGBase nextbot to add it to your selection
- Click on an already-selected nextbot to remove it from selection
- **Hold Shift + Left Click** to select multiple nextbots without deselecting previous ones
- Selected nextbots are highlighted with cyan halos

#### Right Click
**Set Movement Destination**
- All selected nextbots will attempt to move to the position you're aiming at
- Uses the nextbot's normal pathfinding and movement systems
- Nextbots will use the GoTo behavior internally
- The movement command can be interrupted by issuing a new movement command
- Each right-click sends the nextbots to a new destination

#### Reload (R Key)
**Clear Selection**
- Deselects all currently selected nextbots
- Clears the cyan halos
- Useful for starting a new movement command

### Visual Feedback

Selected nextbots are highlighted with **cyan halos**, making it easy to see which nextbots will move when you right-click.

### How Movement Works

When you right-click to set a destination:
- Each selected nextbot runs a GoTo behavior to pathfind to the target position
- The nextbot uses the normal pathfinding system (NavMesh if available)
- The nextbot will navigate around obstacles automatically
- Movement continues until the nextbot reaches the destination or is interrupted
- Issuing a new movement command cancels the previous one
- The nextbot's AI behavior may affect the movement

### Examples

#### Example 1: Position a Nextbot
1. Left-click on a nextbot to select it (cyan halo)
2. Right-click where you want it to go
3. The nextbot walks/runs to that position
4. Press Reload to clear the selection

#### Example 2: Group Movement
1. Hold Shift and left-click multiple nextbots
2. All selected nextbots show cyan halos
3. Right-click at a destination
4. All nextbots pathfind to that location

#### Example 3: Test Pathfinding
1. Select a nextbot
2. Right-click on the other side of an obstacle
3. Watch how the nextbot navigates around it
4. Useful for debugging NavMesh and pathfinding

#### Example 4: Create a Patrol Path
1. Select a nextbot
2. Right-click at position 1
3. Wait for it to arrive
4. Right-click at position 2
5. Repeat to move the nextbot through multiple waypoints

#### Example 5: Rapid Repositioning
1. Select multiple nextbots
2. Right-click various positions in sequence
3. Nextbots will continuously update their destination
4. Useful for dynamic scenario setup

#### Example 6: Test Movement Systems
1. Select a nextbot with custom movement
2. Use the Mover tool to test if pathfinding works correctly
3. Monitor with Info tool Page 4 (Movement) to see movement vectors

### Tips

1. **Shift for Multiple**: Hold Shift when selecting to build a group of nextbots
2. **Cyan = Selected**: Only cyan-highlighted nextbots will move on right-click
3. **Clear Often**: Press Reload to clear selection and start fresh
4. **Interrupt Anytime**: Right-click again to send nextbots to a new destination
5. **AI Interference**: Normal AI behavior may override or interfere with forced movement
6. **Use Disable AI**: Combine with Disable AI tool to ensure nextbots only follow your movement commands
7. **NavMesh Required**: Complex pathfinding requires a properly generated NavMesh
8. **Watch Movement**: Use Info tool Page 4 to monitor movement vectors and speed
9. **Position Before Combat**: Use this tool to position nextbots before enabling AI or starting combat
10. **Test Accessibility**: Try to move nextbots to inaccessible areas to verify pathfinding limits
11. **Rapid Commands**: You can quickly issue multiple movement commands without waiting
12. **GoTo Behavior**: Internally uses the nextbot's GoTo method with a cancellation check
13. **Flag Management**: The tool sets an internal flag to allow cancellation of movement commands

---

## Change Scale Tool

### Overview

The Change Scale tool allows you to resize DrGBase nextbots dynamically. You can make them larger or smaller, affecting their visual size, collision, and physics. This is useful for creating variety, testing size-based mechanics, or making bosses/minions.

**Tool Name**: `Change Scale`

**Primary Use**: Resizing nextbots up or down

### How to Use

1. Equip the Toolgun (default: Q key)
2. Open the tool menu (right-click)
3. Navigate to **DrGBase > Tools**
4. Select **Change Scale**
5. Aim at a DrGBase nextbot
6. **Left-click** to increase size by 10%
7. **Right-click** to decrease size by 10%
8. **Reload (R)** to reset to default size (scale 1.0)

### Actions

#### Left Click
**Scale Up**
- Increases the nextbot's scale by 10% (multiplies current scale by 1.1)
- The change happens smoothly over 0.1 seconds
- Works on any DrGBase nextbot
- Can be used repeatedly to make the nextbot very large
- Affects visual size, collision box, and physics

#### Right Click
**Scale Down**
- Decreases the nextbot's scale by 10% (multiplies current scale by 0.9)
- The change happens smoothly over 0.1 seconds
- Works on any DrGBase nextbot
- Can be used repeatedly to make the nextbot very small
- Affects visual size, collision box, and physics

#### Reload (R Key)
**Reset to Default Scale**
- Sets the nextbot's scale back to 1.0 (normal size)
- The change happens smoothly over 0.1 seconds
- Useful for quickly returning a nextbot to its original size

### Visual Feedback

This tool does not use halos. The visual feedback is the nextbot's changing size itself. You can also check the current scale using the Info tool Page 4 (Movement), which displays the scale value.

### How Scaling Works

When you change a nextbot's scale:
- **Visual Size**: The model becomes larger or smaller
- **Collision**: The collision box scales proportionally
- **Physics**: Physics interactions scale accordingly
- **Smooth Transition**: Changes happen over 0.1 seconds for smooth animation
- **Multiplicative**: Each click multiplies the current scale (not additive)
- **Health/Damage**: Typically not affected (depends on implementation)
- **Speed**: May or may not scale (depends on implementation)

**Scale Values**:
- 1.0 = Normal size (default)
- 2.0 = Double size
- 0.5 = Half size
- Values can go very small or very large

### Examples

#### Example 1: Create a Giant Boss
1. Aim at a nextbot
2. Left-click multiple times (e.g., 5-10 times)
3. The nextbot becomes much larger
4. Creates an imposing boss enemy

#### Example 2: Create Tiny Minions
1. Aim at a nextbot
2. Right-click multiple times (e.g., 5 times)
3. The nextbot becomes much smaller
4. Creates swarm-style enemies

#### Example 3: Precise Sizing
1. Aim at a nextbot
2. Left-click 3 times for 1.331x size (1.1^3)
3. Right-click 1 time for ~1.2x size (1.331 * 0.9)
4. Fine-tune to desired size

#### Example 4: Quick Reset
1. A nextbot has been scaled up or down
2. Aim at it and press Reload
3. It returns to normal size (1.0)
4. Useful for reverting experimental changes

#### Example 5: Size-Based Difficulty
1. Scale enemies down for easy mode
2. Keep normal size for normal mode
3. Scale enemies up for hard mode
4. Creates visual and gameplay variety

#### Example 6: Test Collision at Different Sizes
1. Scale a nextbot very large
2. Test if it can fit through doorways
3. Scale it very small
4. Test if it can navigate tight spaces

### Tips

1. **10% Per Click**: Each click changes size by 10% of the CURRENT size, not the original size
2. **Multiplicative Stacking**: 10 left-clicks = 1.1^10 ≈ 2.59x size (not 2.0x)
3. **Quick Reset**: Use Reload to quickly return to default size if you scale too far
4. **Info Tool Check**: Page 4 (Movement) shows the exact scale value
5. **Smooth Animation**: The 0.1 second transition makes scaling look smooth and natural
6. **No Minimum**: You can scale down to extremely tiny sizes (be careful!)
7. **No Maximum**: You can scale up to extremely large sizes (may cause issues)
8. **Collision Updates**: Collision boxes update automatically with scale
9. **Performance**: Very large nextbots may impact performance
10. **Tiny Nextbots**: Very small nextbots may be hard to click - use Reload from a distance
11. **Works Per-Nextbot**: Each nextbot maintains its own scale independently
12. **Persistent**: Scale persists until changed or the nextbot is removed
13. **No Visual Halo**: Unlike other tools, this doesn't show halos - just aim and click
14. **Combine with Info**: Monitor scale changes in real-time with Info tool Page 4
15. **Test Physics**: Scaled nextbots interact with physics differently - test this with various sizes

---

## Entity Tools Comparison

| Tool | Affects | Controls | Visual Feedback | Use Case |
|------|---------|----------|-----------------|----------|
| **Toggle Godmode** | Damage resistance | Single click toggle | Green/Red halos (all) | Make invincible/vulnerable |
| **Nextbot Mover** | Position/movement | Select → Destination | Cyan halos (selected) | Force movement to positions |
| **Change Scale** | Size/collision | Click to adjust | No halos (size changes) | Resize larger or smaller |

## Combined Usage Tips

1. **Godmode + Damage Tool**: Test damage effects without killing the nextbot
2. **Godmode + Mover**: Position nextbots in dangerous areas without risk
3. **Scale + Mover**: Test if differently-sized nextbots can pathfind correctly
4. **Scale + Info Tool**: Monitor exact scale value on Page 4 (Movement)
5. **All Three + Disable AI**: Create complex setups with protected, positioned, and sized nextbots
6. **Godmode + Scale**: Create huge invincible bosses or tiny invincible swarms
7. **Mover + Disable AI**: Position frozen nextbots, then re-enable AI to see behavior
8. **Scale + Collision**: Test how different sizes affect collision and pathfinding

## General Tips for Entity Manipulation

1. **Use Info Tool**: Monitor changes in real-time with the Info tool
2. **Combine Tools**: These tools work great together for complex setups
3. **Save States**: Set up nextbots with these tools, then save the game to preserve configurations
4. **Test Thoroughly**: After manipulating entities, test their behaviors to ensure everything works
5. **Visual Clarity**: Godmode and Mover use halos for clarity; Scale uses direct visual feedback
6. **Persistence**: All changes persist until manually reverted or the nextbot is removed
7. **No Undo**: There's no undo button - use Reload (for Scale) or toggle back (for Godmode)
8. **Multiplayer**: Changes affect the nextbot for all players on the server
