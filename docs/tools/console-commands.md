# Console Commands & ConVars

## Overview

DrGBase provides console commands and variables for debugging, testing, and configuration. This reference covers all available commands.

## Console Variables (ConVars)

### Debugging

```
drgbase_debug <0|1>
```
Enable/disable debug overlays and messages.
- **Default:** 0
- **Usage:** `drgbase_debug 1`
- **Effect:** Shows debug information, overlays, and verbose logging

```
drgbase_debug_animations <0|1>
```
Show animation debug information.
- **Default:** 0
- **Usage:** `drgbase_debug_animations 1`
- **Effect:** Displays current animations, sequences, and cycles

### AI Configuration

```
drgbase_ai_radius <number>
```
Set AI detection radius globally.
- **Default:** 2000
- **Range:** 100 - 10000
- **Usage:** `drgbase_ai_radius 5000`
- **Effect:** Changes how far NPCs can detect enemies

```
drgbase_ai_fov <number>
```
Set AI field of view globally.
- **Default:** 150
- **Range:** 0 - 360
- **Usage:** `drgbase_ai_fov 180`
- **Effect:** Changes NPC sight cone (degrees)

### Performance

```
drgbase_npc_limit <number>
```
Maximum number of DrGBase NPCs allowed.
- **Default:** 50
- **Range:** 0 - 200
- **Usage:** `drgbase_npc_limit 100`
- **Effect:** Limits spawnable NPCs for performance

```
drgbase_ai_tickrate <number>
```
AI update frequency (ticks per second).
- **Default:** 10
- **Range:** 1 - 66
- **Usage:** `drgbase_ai_tickrate 20`
- **Effect:** Higher = more responsive but more expensive

### Visual Settings

```
drgbase_draw_ranges <0|1>
```
Draw detection ranges for NPCs.
- **Default:** 0
- **Usage:** `drgbase_draw_ranges 1`
- **Effect:** Shows colored spheres for sight/attack ranges

```
drgbase_draw_paths <0|1>
```
Draw pathfinding paths.
- **Default:** 0
- **Usage:** `drgbase_draw_paths 1`
- **Effect:** Shows lines for NPC navigation paths

## Lua Commands

These commands are executed in the Lua console (`lua_run` on server, `lua_run_cl` on client).

### Entity Inspection

```lua
-- Get entity by looking at it
ent = Entity(1) -- or player:GetEyeTrace().Entity

-- Print entity info
PrintTable(ent:GetTable())

-- Check if DrGBase NPC
print(ent.IsDrGNextbot)
```

### Health & Status

```lua
-- Get health
print(ent:Health(), ent:GetMaxHealth())

-- Set health
ent:SetHealth(100)

-- Set max health
ent:SetMaxHealth(200)

-- Enable godmode
ent:SetGodMode(true)

-- Check status
print("Godmode:", ent:GetGodMode())
print("Dead:", ent:IsDead())
print("Down:", ent:IsDown())
```

### AI Control

```lua
-- Disable AI
ent:DisableAI(true)

-- Check AI state
print("AI Disabled:", ent:IsAIDisabled())
print("Has Enemy:", ent:HasEnemy())
print("Enemy:", tostring(ent:GetEnemy()))

-- Set enemy manually
ent:SetEnemy(Player(1))

-- Force omniscient
ent:SetOmniscient(true)
```

### Relationships

```lua
-- Check relationship
print(ent:GetRelationship(Player(1)))

-- Set relationship
ent:SetEntityRelationship(Player(1), D_HT) -- Hate
ent:SetEntityRelationship(Player(1), D_LI) -- Like
ent:SetEntityRelationship(Player(1), D_FR) -- Fear
ent:SetEntityRelationship(Player(1), D_NU) -- Neutral

-- Set default relationship
ent:SetDefaultRelationship(D_HT)

-- Players relationship
ent:SetPlayersRelationship(D_LI)
```

### Factions

```lua
-- Get factions
PrintTable(ent:GetFactions())

-- Join faction
ent:JoinFaction("FACTION_ZOMBIES")

-- Join multiple
ent:JoinFactions({"FACTION_ZOMBIES", "FACTION_PASSIVE"})

-- Leave faction
ent:LeaveFaction("FACTION_ZOMBIES")

-- Leave all
ent:LeaveAllFactions()
```

### Movement & Position

```lua
-- Teleport
ent:SetPos(Vector(0, 0, 100))

-- Set angles
ent:SetAngles(Angle(0, 90, 0))

-- Scale model
ent:SetModelScale(2.0) -- Double size

-- Set speed
ent.WalkSpeed = 100
ent.RunSpeed = 200

-- Force path
ent:FollowPath(Vector(0, 0, 0))
```

### Animations

```lua
-- Play activity
ent:PlayActivity(ACT_MELEE_ATTACK1)

-- Play sequence
ent:PlaySequence("zombie_attack_01")

-- Get current sequence
print(ent:GetSequenceName(ent:GetSequence()))

-- Get available animations
PrintTable(ent:GetAnimInfo())

-- Set playback rate
ent:SetPlaybackRate(1.5) -- 150% speed
```

### Possession

```lua
-- Possess NPC
ent:Possess(Player(1))

-- Dispossess
ent:Dispossess()

-- Check possession
print("Possessed:", ent:IsPossessed())
print("Possessor:", tostring(ent:GetPossessor()))
```

### Weapons

```lua
-- Give weapon
ent:GiveWeapon("weapon_ar2")

-- Remove weapon
ent:RemoveWeapon()

-- Check weapon
print("Has Weapon:", ent:HasWeapon())
print("Weapon:", tostring(ent:GetWeapon()))

-- Fire
ent:PrimaryFire()
ent:SecondaryFire()
ent:Reload()
```

### Spawning

```lua
-- Spawn NPC
local npc = ents.Create("npc_drg_zombie")
npc:SetPos(Vector(0, 0, 100))
npc:Spawn()
npc:Activate()

-- Remove
npc:Remove()

-- Kill
npc:TakeDamage(999)
```

## Utility Commands

### Find All DrGBase NPCs

```lua
lua_run for k,v in ipairs(DrGBase.GetNextbots()) do print(k, v) end
```

### Get Player's Factions

```lua
lua_run PrintTable(Player(1):DrG_GetFactions())
```

### List All Registered NPCs

```lua
lua_run PrintTable(DrGBase.GetRegisteredNextbots())
```

### Print NPC Animations

```lua
lua_run Entity(1):PrintAnimations()
```

### Print NPC Bones

```lua
lua_run Entity(1):PrintBones()
```

### Print Attachments

```lua
lua_run Entity(1):PrintAttachments()
```

### Force NPC Attack

```lua
lua_run Entity(1):OnMeleeAttack(Player(1))
```

### Heal NPC

```lua
lua_run Entity(1):SetHealth(Entity(1):GetMaxHealth())
```

### Test Damage

```lua
local dmg = DamageInfo()
dmg:SetDamage(50)
dmg:SetAttacker(Player(1))
dmg:SetDamageType(DMG_BULLET)
Entity(1):TakeDamageInfo(dmg)
```

## Debugging Workflows

### Debug Detection Issues

```lua
-- Check what NPC can see
ent = Entity(1)
print("Sight FOV:", ent.SightFOV)
print("Sight Range:", ent.SightRange)
print("Can see player:", ent:Visible(Player(1)))
print("Has detected player:", ent:HasSpottedLocalPlayer())

-- Enable omniscient to bypass detection
ent:SetOmniscient(true)
```

### Debug Pathfinding

```lua
-- Enable path drawing
drgbase_draw_paths 1

-- Test path
ent = Entity(1)
result = ent:FollowPath(Vector(0, 0, 0))
print("Path result:", result)
```

### Debug Relationships

```lua
ent = Entity(1)
target = Player(1)

print("Relationship:", ent:GetRelationship(target))
print("Factions:", table.concat(ent:GetFactions(), ", "))
print("Default Relationship:", ent.DefaultRelationship)

-- Force relationship
ent:SetEntityRelationship(target, D_HT)
ent:SetEnemy(target)
```

### Debug Animation

```lua
ent = Entity(1)

-- Current animation
print("Sequence:", ent:GetSequenceName(ent:GetSequence()))
print("Cycle:", ent:GetCycle())
print("Playback Rate:", ent:GetPlaybackRate())
print("Is Attacking:", ent:IsAttacking())

-- Force animation
ent:PlayActivity(ACT_MELEE_ATTACK1)

-- Enable animation debug
drgbase_debug_animations 1
```

### Performance Profiling

```lua
-- Count active NPCs
print("Active NPCs:", #DrGBase.GetNextbots())

-- Check AI tickrate
print("AI Tickrate:", GetConVar("drgbase_ai_tickrate"):GetInt())

-- Disable AI on all NPCs for comparison
for k,v in ipairs(DrGBase.GetNextbots()) do
    v:DisableAI(true)
end
```

## Hotkey Binds

You can bind commands to keys for quick access:

```
bind f9 "drgbase_debug 1"
bind f10 "drgbase_debug 0"
bind f11 "lua_run Entity(Player(1):GetEyeTrace().Entity):SetGodMode(true)"
bind f12 "lua_run Entity(Player(1):GetEyeTrace().Entity):DisableAI(true)"
```

## Recommended Debug Binds

```
// Enable debug
bind kp_enter "drgbase_debug 1"

// Quick godmode on target
bind kp_1 "lua_run local tr = LocalPlayer():GetEyeTrace() if IsValid(tr.Entity) and tr.Entity.IsDrGNextbot then tr.Entity:SetGodMode(true) end"

// Disable AI on target
bind kp_2 "lua_run local tr = LocalPlayer():GetEyeTrace() if IsValid(tr.Entity) and tr.Entity.IsDrGNextbot then tr.Entity:DisableAI(true) end"

// Kill target
bind kp_3 "lua_run local tr = LocalPlayer():GetEyeTrace() if IsValid(tr.Entity) and tr.Entity.IsDrGNextbot then tr.Entity:TakeDamage(999) end"

// Print target info
bind kp_4 "lua_run local tr = LocalPlayer():GetEyeTrace() if IsValid(tr.Entity) and tr.Entity.IsDrGNextbot then PrintTable(tr.Entity:GetTable()) end"
```

## ConVar Reference Table

| ConVar | Type | Default | Description |
|--------|------|---------|-------------|
| drgbase_debug | Bool | 0 | Enable debug mode |
| drgbase_debug_animations | Bool | 0 | Show animation debug |
| drgbase_ai_radius | Int | 2000 | AI detection radius |
| drgbase_ai_fov | Int | 150 | AI field of view |
| drgbase_ai_tickrate | Int | 10 | AI updates per second |
| drgbase_npc_limit | Int | 50 | Max NPCs allowed |
| drgbase_draw_ranges | Bool | 0 | Draw detection ranges |
| drgbase_draw_paths | Bool | 0 | Draw pathfinding paths |

## See Also

- [Info Tool](info-tool.md) - Visual debugging
- [Debug Tools](debug-tools.md) - Testing tools
- [Best Practices](../guides/debugging.md) - Debugging guide
