# DrGBase Developer Tools Overview

DrGBase provides a comprehensive suite of developer tools for testing, debugging, and manipulating NPCs during development.

## Available Tools

1. **Info Tool** - View detailed NPC information and stats
2. **Damage Tool** - Apply precise damage to NPCs for testing
3. **Faction Tool** - Change NPC factions dynamically
4. **Relationship Tool** - Modify entity relationships on-the-fly
5. **AI Control Tools** - Control AI behavior (disable, omniscient, notarget)
6. **Entity Manipulation Tools** - Move, scale, and modify entities

## Accessing Tools

### From Spawn Menu
1. Press `Q` to open spawn menu
2. Click **Utilities** tab (wrench icon)
3. Find DrGBase tools in the list

### From Console
Most tools can be accessed via console commands with the `drgbase_` prefix.

## Common Tool Commands

```
drgbase_info                 -- Enable info display tool
drgbase_damage <amount>      -- Set damage tool amount
drgbase_faction <faction>    -- Set faction tool faction
drgbase_ai_enabled <0|1>     -- Toggle AI globally
drgbase_notarget <0|1>       -- Toggle notarget mode
```

## Quick Start

1. **Testing NPC Combat:**
   - Use Info Tool to view health/stats in real-time
   - Use Damage Tool to test death animations
   - Verify damage calculations are correct

2. **Testing Relationships:**
   - Spawn multiple NPCs
   - Use Faction Tool to change allegiances
   - Use Relationship Tool to test specific interactions

3. **Testing AI:**
   - Use AI Control Tools to freeze/unfreeze NPCs
   - Test behavior with omniscient mode
   - Use notarget to observe without interference

## Tool Safety

- Tools are **admin-only** by default
- Changes are temporary (reset on map change)
- Use `drgbase_tools_enabled 0` to disable all tools

## Next Steps

- [Info Tool](./02-info-tool.md) - View NPC information
- [Damage Tool](./03-damage-tool.md) - Apply damage for testing
- [AI Tools](./06-ai-tools.md) - Control AI behavior
