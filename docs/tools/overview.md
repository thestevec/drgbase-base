# DrGBase Developer Tools Overview

## Introduction

DrGBase includes a comprehensive suite of in-game developer tools accessible through the Tool Gun. These tools help debug, test, and tune NPCs without reloading the game.

## Accessing Tools

1. Open Tool Gun (Q menu → Tools tab)
2. Navigate to **DrGBase → Tools** category
3. Select desired tool
4. Use according to tool instructions

## Available Tools

### 🔍 Information & Debugging

| Tool | Purpose | Quick Description |
|------|---------|-------------------|
| **Nextbot Info** | View NPC stats in real-time | Monitor health, AI state, animations, relationships |

### ⚙️ Configuration Tools

| Tool | Purpose | Quick Description |
|------|---------|-------------------|
| **Faction Tool** | Manage factions | Copy/paste factions between NPCs and players |
| **Relationship Tool** | Edit relationships | Set dispositions between entities |
| **Scale Tool** | Adjust NPC size | Change model scale dynamically |
| **Mover Tool** | Reposition NPCs | Move NPCs without physics |
| **Damage Tool** | Test damage | Apply custom damage to NPCs |

### 🐛 Testing & Debug Tools

| Tool | Purpose | Quick Description |
|------|---------|-------------------|
| **Toggle Godmode** | Invincibility | Enable/disable godmode on NPCs |
| **Disable AI** | Freeze AI | Stop AI processing for testing |
| **No Target** | Invisibility to AI | Make player invisible to NPCs |
| **Omniscient** | All-seeing AI | NPCs detect everything instantly |

## Quick Reference

### Common Workflows

**Testing NPC Combat:**
1. Use **Disable AI** to freeze test dummy
2. Use **Godmode** to prevent death
3. Use **Damage Tool** to test damage values
4. Use **Info Tool** to monitor health/stats

**Debugging AI Issues:**
1. Use **Info Tool** to view AI state
2. Use **Omniscient** to force enemy detection
3. Use **Relationship Tool** to test dispositions
4. Watch real-time state changes on Tool Gun screen

**Setting Up Factions:**
1. Create test NPC with desired factions
2. Use **Faction Tool** (right-click) to copy factions
3. Apply to other NPCs with left-click
4. Use **Info Tool** to verify relationships

## Tool Gun Screen

Most tools display information on the Tool Gun's screen:
- Real-time data updates
- Color-coded status indicators
- Multi-page information displays
- Visual feedback

## Color Coding

Tools use consistent color coding:
- 🟢 **Green**: Enabled/Active/Friendly
- 🔴 **Red**: Disabled/Dead/Hostile
- 🟡 **Orange**: Warning/Special State
- 🔵 **Cyan**: Neutral/Information
- 🟣 **Purple**: Fear/Special

## Console Commands

Some tools have associated console commands:
```
drgbase_debug 1              -- Enable debug overlays
drgbase_ai_radius 2000       -- Set AI detection radius
drgbase_debug_animations 1   -- Show animation debug info
```

## Tool Categories

### Diagnostic Tools
- Nextbot Info Tool
- Viewcam (part of Info Tool)

### Behavior Modifiers
- Disable AI Tool
- Omniscient Tool
- No Target Tool

### Entity Manipulation
- Mover Tool
- Scale Tool
- Damage Tool
- Godmode Tool

### Relationship Management
- Faction Tool
- Relationship Tool

## Best Practices

1. **Info Tool First**: Always start with Info Tool to understand current state
2. **One Tool at a Time**: Focus on one tool per debugging session
3. **Godmode for Testing**: Enable godmode when testing non-lethal features
4. **Disable AI for Setup**: Freeze AI when positioning or configuring NPCs
5. **Use Halos**: Most tools highlight affected NPCs with colored halos

## Troubleshooting Tools

### Tool doesn't work
- Ensure you're targeting a DrGBase NPC
- Check tool is selected in Tool Gun
- Look for console errors
- Verify NPC is spawned and valid

### Tool Gun screen is black
- Try switching tools
- Reload tool (R key)
- Check if NPC is selected

### Halos not showing
- Check graphics settings
- Ensure Tool Gun is equipped
- Verify correct tool is active

## Related Documentation

- [Info Tool](info-tool.md) - Detailed NPC diagnostics
- [Faction Tool](faction-tool.md) - Faction management
- [Debug Tools](debug-tools.md) - Testing utilities
- [Console Commands](console-commands.md) - All available commands

## Source Files

Tools location: `lua/weapons/gmod_tool/stools/drgbase_tool_*.lua`

Available tools:
- `drgbase_tool_info.lua` - Info Tool
- `drgbase_tool_faction.lua` - Faction Tool
- `drgbase_tool_relationship_simple.lua` - Relationship Tool
- `drgbase_tool_godmode.lua` - Godmode Tool
- `drgbase_tool_disableai.lua` - AI Disable Tool
- `drgbase_tool_notarget.lua` - No Target Tool
- `drgbase_tool_omniscient.lua` - Omniscient Tool
- `drgbase_tool_scale.lua` - Scale Tool
- `drgbase_tool_mover.lua` - Mover Tool
- `drgbase_tool_damage.lua` - Damage Tool

## See Also

- [Examples: Getting Started](../examples/01-getting-started.md)
- [Examples: Custom AI](../examples/06-custom-ai.md)
- [Examples: Relationships](../examples/07-relationships.md)
