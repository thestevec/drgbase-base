# AI Control Tools

AI Control Tools allow you to manipulate NPC artificial intelligence for testing, debugging, and content creation.

## Available AI Tools

### 1. AI Enable/Disable
Toggle AI processing globally or per-NPC

### 2. Omniscient Mode
NPCs know all enemy positions

### 3. Notarget Mode
NPCs ignore specific entities

### 4. AI Freeze
Pause individual NPC AI

## Global AI Control

### Disable All AI
```
drgbase_ai_enabled 0    -- Disable all NPC AI
drgbase_ai_enabled 1    -- Re-enable all NPC AI
```

All DrGBase NPCs will freeze in place. Use for:
- Taking screenshots
- Setting up scenes
- Performance testing
- Debugging without interference

### Omniscient Mode
```
drgbase_ai_omniscient 1    -- NPCs know all enemy locations
drgbase_ai_omniscient 0    -- Normal detection
```

NPCs can "see" enemies through walls and at any distance. Use for:
- Testing combat without line-of-sight issues
- Creating challenging encounters
- Debugging AI targeting

### Notarget Mode
```
drgbase_ai_notarget 1    -- NPCs ignore you
drgbase_ai_notarget 0    -- Normal targeting
```

NPCs won't detect or attack you. Use for:
- Observing AI behavior
- Setting up scenes
- Taking screenshots
- Debugging without being attacked

## Per-NPC Control

### AI Freeze Tool
**Accessing:**
1. Spawn Menu > Utilities > "DrGBase AI Freeze"
2. Console: `drgbase_ai_freeze`

**Usage:**
1. Equip tool
2. **Left Click** NPC - Toggle AI freeze
3. **Right Click** NPC - View AI state
4. **Reload** - Freeze all nearby NPCs

**Console:**
```
drgbase_ai_freeze_radius <radius>    -- Freeze all within radius
drgbase_ai_freeze_all                -- Freeze all NPCs
drgbase_ai_unfreeze_all              -- Unfreeze all NPCs
```

### AI Debug Visualizer
```
drgbase_ai_debug 1    -- Show AI debug info
drgbase_ai_debug 0    -- Hide debug info
```

Shows:
- Current target (line drawn to target)
- Vision cone
- Attack range circle
- Path visualization
- Current AI state (above NPC)

## Testing Scenarios

### Test Detection Range
```
1. drgbase_ai_notarget 1
2. Spawn NPC
3. Approach slowly
4. drgbase_ai_notarget 0 at specific distance
5. Note when NPC detects you
6. Verify detection range settings
```

### Test Patrol Behavior
```
1. drgbase_ai_notarget 1
2. Spawn NPC
3. Observe patrol paths without interference
4. Verify OnIdle/OnReachedPatrol work correctly
```

### Test Combat AI
```
1. drgbase_ai_omniscient 1
2. Spawn multiple NPCs
3. They immediately engage (no detection delay)
4. Test pure combat behavior
```

### Debug Pathfinding
```
1. drgbase_ai_debug 1
2. Spawn NPC
3. Give it a target (spawn enemy)
4. Watch path visualization
5. Identify pathfinding issues
```

## Console Variables

```lua
-- Global AI
drgbase_ai_enabled <0|1>         -- Toggle all AI (default: 1)
drgbase_ai_omniscient <0|1>      -- Omniscient mode (default: 0)
drgbase_ai_notarget <0|1>        -- Notarget mode (default: 0)

-- Visualization
drgbase_ai_debug <0|1>           -- Show AI debug (default: 0)
drgbase_ai_debug_range <number>  -- Debug draw distance (default: 2000)

-- Performance
drgbase_ai_thinkrate <number>    -- AI think rate in Hz (default: 30)
drgbase_ai_maxthink <number>     -- Max AI thinks per frame (default: 10)
```

## Advanced Features

### AI Profile Switching
```
drgbase_ai_profile <profile>
-- Profiles: default, aggressive, defensive, passive
```

Changes AI behavior globally:
- **aggressive**: Higher attack ranges, less retreat
- **defensive**: More cover usage, earlier retreat
- **passive**: Reduced aggression, more patrol

### AI Recording
```
drgbase_ai_record 1              -- Start recording AI actions
drgbase_ai_record_stop           -- Stop recording
drgbase_ai_record_play           -- Replay recorded actions
```

Records NPC movements and actions for playback. Use for:
- Creating scripted sequences
- Testing reproducibility
- Bug reporting

## Troubleshooting

**AI not responding to commands:**
- Verify NPC is a DrGBase NPC
- Check NPC isn't frozen individually
- Try toggling command off and on

**Omniscient mode not working:**
- Verify enemies exist on map
- Check NPC has valid detection settings
- Ensure AI is enabled

**Debug visualization not showing:**
- Check `drgbase_ai_debug_range` value
- Move closer to NPC
- Verify `developer 1` in console

## Tips

1. **Combine tools** - Use notarget + debug to observe AI without interfering
2. **Test incrementally** - Test one AI feature at a time
3. **Document behavior** - Note console commands used when finding bugs
4. **Performance testing** - Disable AI to isolate performance issues

## Example Workflows

### Debug Combat AI
```
1. drgbase_ai_notarget 1 (observe without interference)
2. drgbase_ai_debug 1 (show AI visualization)
3. Spawn NPC and enemy
4. Watch how NPC engages
5. Identify issues in behavior
```

### Test Stealth Mechanic
```
1. drgbase_ai_debug 1 (show vision cone)
2. Spawn guard NPC
3. Try sneaking past
4. See exactly when you enter vision
5. Adjust detection settings
```

### Create Scripted Scene
```
1. drgbase_ai_enabled 0 (freeze all AI)
2. Position NPCs
3. Set up perfect scene
4. drgbase_ai_enabled 1 (animate scene)
5. Take screenshot/video
```

## Next Steps

- [Info Tool](./02-info-tool.md) - Monitor AI state changes
- [Entity Tools](./07-entity-tools.md) - Position NPCs precisely
- [Advanced AI Example](../examples/advanced-ai.md) - Implement complex AI
