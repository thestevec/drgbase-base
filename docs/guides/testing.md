# Testing and Quality Assurance Guide

## Overview

Thorough testing ensures your DrGBase NPCs work correctly across different scenarios. This guide covers testing strategies and quality assurance practices.

## Testing Checklist

### Basic Functionality Tests

#### ✅ Spawning
- [ ] NPC appears in spawn menu
- [ ] NPC spawns without errors
- [ ] Model loads correctly
- [ ] Collision bounds are correct
- [ ] Multiple NPCs can spawn

#### ✅ AI & Detection
- [ ] Detects player within range
- [ ] Doesn't detect player outside range
- [ ] Respects FOV settings
- [ ] Line of sight works correctly
- [ ] Loses track of hidden enemies

#### ✅ Movement & Pathfinding
- [ ] Walks/runs at correct speeds
- [ ] Follows players smoothly
- [ ] Doesn't get stuck on props
- [ ] Navigates stairs/ramps
- [ ] Jumps over obstacles
- [ ] Respects navmesh boundaries

#### ✅ Combat
- [ ] Attacks when in range
- [ ] Attack animations play
- [ ] Damage is dealt correctly
- [ ] Doesn't attack friendly NPCs
- [ ] Attack cooldowns work

#### ✅ Health & Damage
- [ ] Takes damage from weapons
- [ ] Takes damage from physics
- [ ] Health regeneration works (if enabled)
- [ ] Dies at 0 health
- [ ] Death animation/ragdoll works

#### ✅ Relationships
- [ ] Attacks hostile entities
- [ ] Ignores friendly entities
- [ ] Faction system works
- [ ] Relationship changes apply correctly

#### ✅ Sounds
- [ ] Idle sounds play
- [ ] Attack sounds play
- [ ] Damage sounds play
- [ ] Death sounds play
- [ ] Sounds don't spam

#### ✅ Performance
- [ ] 30+ NPCs don't cause lag
- [ ] No console errors
- [ ] No infinite loops
- [ ] Memory doesn't leak

## Testing Scenarios

### Scenario 1: Basic Combat

**Setup:**
1. Spawn test NPC
2. Enable godmode (for player)
3. Let NPC detect and attack

**Test:**
- Does NPC detect player?
- Does NPC approach player?
- Does attack animation play?
- Is damage dealt?
- Do sounds play?

**Console commands:**
```
god                                  // Player godmode
impulse 101                          // Give weapons
drgbase_debug 1                     // Debug info
notarget                            // Toggle detection
```

### Scenario 2: Pathfinding Stress Test

**Setup:**
1. Spawn NPC in complex area
2. Place obstacles
3. Move around environment

**Test:**
- Does NPC navigate around props?
- Does NPC handle stairs/ramps?
- Does NPC jump over obstacles?
- Does NPC recover if stuck?

**Test locations:**
- `gm_construct`: Complex geometry
- `gm_flatgrass`: Open area
- `de_dust2`: Multi-level

### Scenario 3: Multi-NPC Test

**Setup:**
1. Spawn 30-50 NPCs
2. Monitor performance

**Test:**
- FPS stays above 30?
- NPCs all function correctly?
- No console errors?
- AI updates properly?

**Commands:**
```lua
// Spawn 50 NPCs
for i = 1, 50 do
    local npc = ents.Create("npc_drg_zombie")
    npc:SetPos(Vector(i * 100, 0, 100))
    npc:Spawn()
end
```

### Scenario 4: Faction Warfare

**Setup:**
1. Spawn NPCs from Faction A
2. Spawn NPCs from Faction B
3. Set hostile relationships
4. Observe combat

**Test:**
- Do factions attack each other?
- Do same-faction NPCs cooperate?
- Do relationships persist?

### Scenario 5: Possession Test

**Setup:**
1. Spawn NPC with possession enabled
2. Possess NPC
3. Test all controls

**Test:**
- All keys work?
- Camera views work?
- Abilities function correctly?
- Exit possession works?

### Scenario 6: Damage Types

**Setup:**
1. Spawn NPC with godmode OFF
2. Test different damage types

**Test damage from:**
- Bullets (weapon_pistol)
- Explosions (weapon_rpg)
- Fire (weapon_flamethrower or env_fire)
- Physics (props)
- Fall damage (drop from height)
- Melee (crowbar)

**Verify:**
- Correct damage taken
- Appropriate reactions
- Death behavior consistent

### Scenario 7: Edge Cases

**Test unusual situations:**

**No navmesh:**
```
nav_delete
// Does NPC handle gracefully?
```

**Underwater:**
- Spawn NPC in water
- Does it behave correctly?

**Very high/low positions:**
- Spawn NPC on tall building
- Does pathfinding work?

**Confined spaces:**
- Spawn in small room
- Does NPC get stuck?

**Rapid spawning/removal:**
```lua
// Spawn and remove rapidly
for i = 1, 100 do
    local npc = ents.Create("npc_drg_zombie")
    npc:SetPos(Vector(0, 0, 100))
    npc:Spawn()
    timer.Simple(0.1, function() npc:Remove() end)
end
```

## Automated Testing

### Test Script Template

```lua
-- Place in lua/autorun/server/npc_tests.lua

local NPCTests = {}

function NPCTests:TestSpawning()
    local npc = ents.Create("npc_drg_zombie")
    if not IsValid(npc) then
        print("❌ FAIL: NPC didn't spawn")
        return false
    end

    npc:SetPos(Vector(0, 0, 100))
    npc:Spawn()

    if not IsValid(npc) then
        print("❌ FAIL: NPC invalid after spawn")
        return false
    end

    print("✅ PASS: Spawning test")
    npc:Remove()
    return true
end

function NPCTests:TestDetection()
    local npc = ents.Create("npc_drg_zombie")
    npc:SetPos(Vector(0, 0, 100))
    npc:Spawn()

    local ply = player.GetAll()[1]
    if not IsValid(ply) then
        print("⚠️ SKIP: No player for detection test")
        npc:Remove()
        return nil
    end

    -- Force detection
    npc:SetOmniscient(true)
    npc:SetEntityRelationship(ply, D_HT)

    timer.Simple(1, function()
        if npc:GetEnemy() == ply then
            print("✅ PASS: Detection test")
        else
            print("❌ FAIL: NPC didn't detect player")
        end
        npc:Remove()
    end)

    return true
end

function NPCTests:TestCombat()
    local npc = ents.Create("npc_drg_zombie")
    npc:SetPos(Vector(0, 0, 100))
    npc:Spawn()

    local dummy = ents.Create("prop_physics")
    dummy:SetModel("models/props_c17/oildrum001.mdl")
    dummy:SetPos(Vector(50, 0, 100))
    dummy:Spawn()

    npc:SetEnemy(dummy)
    npc:SetEntityRelationship(dummy, D_HT)

    timer.Simple(3, function()
        if not IsValid(dummy) or dummy:Health() < 100 then
            print("✅ PASS: Combat test (damage dealt)")
        else
            print("❌ FAIL: No damage dealt")
        end

        if IsValid(npc) then npc:Remove() end
        if IsValid(dummy) then dummy:Remove() end
    end)

    return true
end

function NPCTests:RunAll()
    print("========== NPC TESTS ==========")
    self:TestSpawning()
    self:TestDetection()
    self:TestCombat()
    print("===============================")
end

-- Run tests
concommand.Add("npc_test", function()
    NPCTests:RunAll()
end)
```

## Quality Assurance Checklist

### Code Quality

#### ✅ Before Testing
- [ ] No syntax errors
- [ ] No Lua errors in console
- [ ] All hooks implemented correctly
- [ ] All configurations set
- [ ] AddCSLuaFile() at end
- [ ] DrGBase.AddNextbot(ENT) at end

#### ✅ Code Review
- [ ] Functions have clear names
- [ ] Complex logic is commented
- [ ] No magic numbers (use constants)
- [ ] No repeated code (use functions)
- [ ] Performance considerations applied

### Gameplay Quality

#### ✅ Balance
- [ ] Health appropriate for difficulty
- [ ] Damage balanced
- [ ] Speed feels right
- [ ] Detection range reasonable
- [ ] Attack frequency balanced

#### ✅ Polish
- [ ] Animations smooth
- [ ] Sounds appropriate
- [ ] Visual effects work
- [ ] Feels responsive
- [ ] No jarring behaviors

### Performance Quality

#### ✅ Optimization
- [ ] AI updates throttled
- [ ] Expensive operations cached
- [ ] LOD system implemented (if needed)
- [ ] No entity searches every frame
- [ ] Path updates reasonable

## Debugging During Tests

### Print Debug Information

```lua
function ENT:CustomInitialize()
    if GetConVar("drgbase_debug"):GetBool() then
        print("=== NPC Spawned ===")
        print("Class:", self:GetClass())
        print("Health:", self:Health())
        print("Position:", self:GetPos())
        print("===================")
    end
end

function ENT:OnMeleeAttack(enemy)
    if self.DebugMode then
        print("Attack:", tostring(enemy))
        print("Range:", self.MeleeAttackRange)
        print("Distance:", self:GetPos():Distance(enemy:GetPos()))
    end

    -- Attack code
end
```

### Visual Debug Overlays

```lua
function ENT:CustomThink()
    if not GetConVar("drgbase_debug"):GetBool() then return end

    -- Draw detection range
    debugoverlay.Sphere(
        self:GetPos(),
        self.SightRange,
        0.1,
        Color(0, 255, 0, 10),
        true
    )

    -- Draw to enemy
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        debugoverlay.Line(
            self:EyePos(),
            enemy:EyePos(),
            0.1,
            Color(255, 0, 0),
            true
        )
    end
end
```

## Test Environment Setup

### Recommended Test Maps

1. **gm_flatgrass** - Basic testing, open area
2. **gm_construct** - Complex geometry, multiple levels
3. **gm_bigcity** - Performance testing, large area
4. **de_dust2** - Navigation testing, combat areas

### Test Configuration

**Create test config file:**
`garrysmod/cfg/npc_test.cfg`

```
// NPC Testing Configuration

// Debug settings
drgbase_debug 1
developer 1

// Convenience binds
bind f9 "noclip"
bind f10 "god"
bind f11 "notarget"
bind f12 "impulse 101"

// Quick spawn
alias spawn_test "lua_run local n = ents.Create('npc_drg_zombie') n:SetPos(LocalPlayer():GetEyeTrace().HitPos) n:Spawn()"
bind kp_enter "spawn_test"

// Quick remove all
alias remove_all "lua_run for k,v in ipairs(ents.FindByClass('npc_drg_*')) do v:Remove() end"
bind kp_del "remove_all"
```

**Load config:**
```
exec npc_test
```

## Bug Reporting Template

When you find bugs, document them:

```
Bug Report: Zombie Attack Animation Doesn't Play

Reproduction Steps:
1. Spawn npc_drg_zombie
2. Approach zombie
3. Wait for attack
4. Observe: No attack animation

Expected Behavior:
- Attack animation should play
- Damage should be dealt

Actual Behavior:
- No animation plays
- Damage IS dealt
- Console shows: "Activity ACT_MELEE_ATTACK1 not found"

Environment:
- Map: gm_construct
- DrGBase Version: 1.0
- Garry's Mod: Latest
- Singleplayer

Additional Info:
- Works on other models
- Only affects this specific model
- Error appears in console

Fix Attempted:
- Changed to sequence name: "zombie_attack_01"
- Result: Fixed
```

## Pre-Release Checklist

Before releasing your NPC pack:

### Functionality
- [ ] All features work as intended
- [ ] No console errors
- [ ] No Lua errors
- [ ] Tested on multiple maps
- [ ] Tested with 30+ NPCs
- [ ] Tested in singleplayer
- [ ] Tested in multiplayer (if applicable)

### Polish
- [ ] All sounds present
- [ ] All animations work
- [ ] Visual effects complete
- [ ] Spawn menu category correct
- [ ] PrintName descriptive

### Documentation
- [ ] Code commented
- [ ] README created (if pack)
- [ ] Workshop description written
- [ ] Known issues documented

### Performance
- [ ] No lag with 30+ NPCs
- [ ] LOD system (if needed)
- [ ] No memory leaks
- [ ] Optimized for servers

## See Also

- [Debugging Guide](debugging.md)
- [Performance Guide](performance.md)
- [Common Pitfalls](common-pitfalls.md)
- [Console Commands](../tools/console-commands.md)
