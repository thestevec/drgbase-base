# Nextbot Info Tool

## Overview

The Nextbot Info Tool is the most powerful diagnostic tool in DrGBase. It provides real-time monitoring of NPC state, AI behavior, animations, and more directly on your Tool Gun screen.

## Usage

**Controls:**
- **Left Click**: Select/deselect NPC
- **Right Click**: Cycle through information pages
- **Reload (R)**: Clear selection

## Information Pages

The Info Tool displays 6 different pages of information:

### 1. Status Page 📊

Displays health and status information:

```
Health: 75 / 100          (Color-coded: Green=healthy, Red=low)
Regen: 2                  (Health regeneration per second)
Godmode: Disabled         (Invincibility status)
Status: Alive             (Alive/Down/Dead)
```

**What It Shows:**
- Current and maximum health
- Health regeneration rate
- Godmode status (enabled/disabled)
- Current status (Alive, Down, or Dead)

**Visual Feedback:**
- Halo color changes based on health percentage
- Green (100%) → Yellow (50%) → Red (0%)

**Use Cases:**
- Monitor health during combat tests
- Verify regeneration is working
- Check if NPC is in "down" state
- Confirm godmode is active

### 2. AI Page 🧠

Displays AI state and enemy information:

```
Enemy: John Doe           (Current target)
Relationship: Hate        (D_HT - Red)
Spotted: True             (Has detected player)
Omniscient: False         (Normal detection)
```

**What It Shows:**
- Current enemy (name or class)
- Relationship with player (Like/Hate/Afraid/Neutral)
- Detection status (spotted or not)
- Omniscient mode (all-seeing enabled/disabled)

**Visual Feedback:**
- Halo color based on relationship:
  - Green = Like (D_LI)
  - Red = Hate (D_HT)
  - Purple = Afraid (D_FR)
  - Cyan = Neutral (D_NU)

**Use Cases:**
- Debug targeting issues
- Verify relationship system
- Test detection range
- Monitor AI state changes

### 3. Possession Page 🎮

Shows possession status and information:

```
Possessed by: Player123   (Who is controlling NPC)
Locked on: npc_zombie     (Current possession target)
```

Or if not possessed:
```
This nextbot
isn't possessed
```

**What It Shows:**
- Who is possessing the NPC
- What entity the possessor is locked onto
- Possession status (active/inactive)

**Visual Feedback:**
- Green halo = Possessed
- Red halo = Not possessed

**Use Cases:**
- Verify possession system working
- Debug possession controls
- Monitor locked-on targets
- Test possession transitions

### 4. Movement Page 🏃

Displays movement and physics information:

```
Speed: 245.67             (Current speed in units/sec)
Scale: 1.00               (Model scale multiplier)
Movement: (0.85, 0.00)    (Movement vector normalized)
```

**What It Shows:**
- Current movement speed
- Model scale (size multiplier)
- Movement vector (X, Y direction)

**Visual Feedback:**
- Cyan halo on NPC
- Real-time speed updates

**Use Cases:**
- Tune movement speeds
- Debug pathfinding issues
- Monitor movement states
- Verify scale changes

### 5. Animation Page 🎭

Shows current animation state:

**For 3D Models:**
```
Sequence: run_all         (Current animation sequence)
Activity: ACT_RUN         (Current activity)
Attacking? True           (Is in attack animation)
```

**For Sprite NPCs:**
```
Animation: walk           (Sprite animation name)
Frame: 3                  (Current frame number)
Attacking? False
```

**What It Shows:**
- Current animation sequence name
- Activity type (for 3D models)
- Current frame (for sprites)
- Attack state

**Visual Feedback:**
- Cyan halo on NPC
- Real-time animation updates

**Use Cases:**
- Debug animation playback
- Verify correct animations playing
- Monitor attack animations
- Test sprite frame progression

### 6. Viewcam Page 📹

Displays what the NPC sees:

Shows real-time view from NPC's perspective:
- Rendered on Tool Gun screen
- Updates continuously
- Shows NPC's field of view
- Same perspective as NPC eye position

**What It Shows:**
- Real-time camera from NPC eyes
- What NPC can actually see
- Visual representation of FOV
- Line of sight perspective

**Use Cases:**
- Debug detection issues ("Why can't NPC see enemy?")
- Verify line of sight
- Test FOV settings
- Understand NPC perspective

## Navigation

**Page Order:**
1. Status
2. AI
3. Possession
4. Movement
5. Animation
6. Viewcam

**Controls:**
- Right-click cycles: 1 → 2 → 3 → 4 → 5 → 6 → 1
- Current page shown at bottom of screen: "(2) AI"

## Practical Examples

### Example 1: Debugging Detection Issues

**Problem:** "NPC doesn't see player"

**Solution:**
1. Select NPC with Info Tool
2. Go to **AI Page** (page 2)
   - Check "Spotted" status
   - Verify relationship isn't Neutral
3. Go to **Viewcam Page** (page 6)
   - See what NPC actually sees
   - Check if obstacles block view
4. Adjust NPC position or sight settings

### Example 2: Testing Combat Balance

**Problem:** "Need to tune NPC health"

**Solution:**
1. Enable godmode with Godmode Tool
2. Select NPC with Info Tool
3. Stay on **Status Page** (page 1)
4. Attack NPC and watch health bar
5. Monitor regeneration rate
6. Adjust health values in code

### Example 3: Animation Issues

**Problem:** "Attack animation not playing"

**Solution:**
1. Select NPC with Info Tool
2. Go to **Animation Page** (page 5)
3. Watch "Attacking?" status
4. Note sequence name when attacking
5. Verify correct animation is set
6. Check OnMeleeAttack() function

### Example 4: AI State Monitoring

**Problem:** "NPC behavior seems buggy"

**Solution:**
1. Select NPC with Info Tool
2. Stay on **AI Page** (page 2)
3. Monitor enemy changes
4. Watch relationship updates
5. Verify spotted status changes
6. Cross-reference with AI code

## Tips & Tricks

### Multiple NPC Monitoring

To monitor multiple NPCs:
1. Use Info Tool on first NPC
2. Note readings
3. Reload (R) to deselect
4. Select next NPC
5. Compare readings

### Real-Time Testing

Leave Info Tool selected while testing:
1. Select NPC with Info Tool
2. Keep Tool Gun equipped
3. Interact with NPC (spawn enemies, etc.)
4. Watch Tool Gun screen for changes
5. Page through information as needed

### Possession Debugging

To debug possession:
1. Before possessing: Check Possession Page → "Not possessed"
2. Possess NPC
3. Have second player select you with Info Tool
4. They can see possession status
5. Monitor locked-on targets

### Viewcam Tips

Viewcam page is perfect for:
- Understanding NPC FOV
- Debugging sight blockers
- Verifying EyePos offset
- Testing detection range
- Checking luminosity-based detection

## Color Legend

**Halos:**
- 🟢 Green → Red: Health-based (Status page)
- 🟢 Green: Friendly relationship (AI page)
- 🔴 Red: Hostile relationship (AI page)
- 🟣 Purple: Fear relationship (AI page)
- 🔵 Cyan: Neutral relationship (AI page)
- 🟢 Green: Possessed (Possession page)
- 🔴 Red: Not possessed (Possession page)
- 🔵 Cyan: General highlight (Movement/Animation pages)

**Text Colors on Screen:**
- 🟢 Green: Positive/Enabled/High health
- 🔴 Red: Negative/Disabled/Low health/Dead
- 🟠 Orange: Warning/Special state
- 🔵 Cyan: Information/Neutral
- ⚪ Gray: Labels/Static text

## Limitations

- Only one NPC can be selected at a time
- Tool Gun must remain equipped to see updates
- Viewcam page has performance cost
- Some information only available in certain states

## Console Commands

Related commands for debugging:
```
drgbase_debug 1                   -- Enable debug overlays
drgbase_debug_animations 1        -- Show animation info
developer 1                       -- Show developer messages
```

## Source Code

File: `lua/weapons/gmod_tool/stools/drgbase_tool_info.lua`

## Related Tools

- [Godmode Tool](debug-tools.md#godmode-tool) - Test with invincibility
- [Disable AI Tool](debug-tools.md#disable-ai-tool) - Freeze NPC for inspection
- [Omniscient Tool](debug-tools.md#omniscient-tool) - Force enemy detection

## See Also

- [Animation System](../examples/09-animations.md)
- [AI Behavior](../examples/06-custom-ai.md)
- [Possession System](../examples/08-possession.md)
- [Relationship System](../examples/07-relationships.md)
