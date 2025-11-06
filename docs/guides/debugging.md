# Debugging Guide

## Overview

Effective debugging is essential for developing DrGBase NPCs. This guide covers strategies, tools, and techniques for finding and fixing issues.

## Debugging Workflow

### 1. Identify the Problem

**Ask these questions:**
- What is happening? (observed behavior)
- What should happen? (expected behavior)
- When does it occur? (always, sometimes, specific conditions)
- Which NPCs are affected? (all, specific class, random)

**Example:**
```
Problem: "Zombie doesn't attack player"

Specifics:
- Happens: Always
- Expected: Attack when player nearby
- Affected: Only npc_drg_zombie
```

### 2. Isolate the System

**Narrow down to specific system:**
- Detection issue?
- AI behavior issue?
- Animation issue?
- Relationship issue?

**Use Info Tool to check:**
```
1. AI Page → Check if zombie detects player ("Spotted: True")
2. AI Page → Check relationship ("Relationship: Hate")
3. Animation Page → Check if attack animation plays
```

### 3. Add Debug Output

**Strategic print statements:**

```lua
function ENT:OnMeleeAttack(enemy)
    print("OnMeleeAttack called!")
    print("Enemy:", tostring(enemy))
    print("Enemy valid:", IsValid(enemy))
    print("In range:", self:IsInRange(enemy:GetPos(), self.MeleeAttackRange))

    self:EmitSound("Zombie.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### 4. Test Incrementally

**Test each component:**

```lua
-- Test 1: Check if function is called
function ENT:OnMeleeAttack(enemy)
    print("TEST 1: Function called") -- If this doesn't print, function never runs
    self:EmitSound("Zombie.Attack")
end

-- Test 2: Check animation
function ENT:OnMeleeAttack(enemy)
    print("TEST 2: Playing animation")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    print("Animation started")
end

-- Test 3: Check damage
function ENT:OnAnimEvent()
    print("TEST 3: Animation event")
    if self:IsAttacking() then
        print("Is attacking, applying damage")
        self:Attack({damage = 10})
    end
end
```

### 5. Use Debugging Tools

**Tool chain:**
1. **Info Tool** → Check NPC state
2. **Godmode Tool** → Prevent death during testing
3. **Disable AI Tool** → Isolate AI issues
4. **Omniscient Tool** → Force enemy detection
5. **Console Commands** → Direct manipulation

## Common Debugging Scenarios

### Scenario 1: NPC Doesn't Detect Player

**Symptoms:**
- NPC doesn't react to player
- No enemy on Info Tool AI page
- "Spotted: False"

**Debug steps:**

```lua
-- Step 1: Check detection settings
print("SightRange:", self.SightRange)
print("SightFOV:", self.SightFOV)
print("HearingCoefficient:", self.HearingCoefficient)

-- Step 2: Check line of sight
print("Can see player:", self:Visible(Player(1)))

-- Step 3: Check relationship
print("Relationship with player:", self:GetRelationship(Player(1)))

-- Step 4: Force omniscient mode (bypass detection)
self:SetOmniscient(true) -- If this fixes it, it's a detection issue
```

**Common causes:**
- SightRange too small
- SightFOV too narrow
- Relationship is Neutral (D_NU)
- Player has No Target enabled
- Obstacles blocking view

**Solutions:**
```lua
-- Increase range
ENT.SightRange = 3000

-- Widen FOV
ENT.SightFOV = 180

-- Set hostile relationship
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)
end
```

### Scenario 2: Attack Animation Doesn't Play

**Symptoms:**
- NPC tries to attack but animation doesn't play
- OnMeleeAttack() is called but no animation

**Debug steps:**

```lua
function ENT:OnMeleeAttack(enemy)
    print("=== ATTACK DEBUG ===")
    print("Activity:", ACT_MELEE_ATTACK1)

    -- Check if activity exists
    local seq = self:SelectWeightedSequence(ACT_MELEE_ATTACK1)
    print("Sequence ID:", seq)

    if seq == -1 then
        print("ERROR: Activity not found!")
        print("Available animations:")
        PrintTable(self:GetAnimInfo())
    else
        print("Playing animation")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end
end
```

**Common causes:**
- Model doesn't have animation
- Wrong activity name
- Animation interrupted

**Solutions:**
```lua
-- Use sequence name instead
function ENT:OnMeleeAttack(enemy)
    local anims = self:GetAnimInfo()
    -- Find attack animation
    for activity, sequences in pairs(anims.activities) do
        if string.find(string.lower(activity), "attack") then
            print("Found attack activity:", activity)
        end
    end

    -- Use specific sequence
    self:PlaySequenceAndMove("zombie_attack_01", 1, self.FaceEnemy)
end
```

### Scenario 3: NPC Takes No Damage

**Symptoms:**
- Shooting NPC does nothing
- Health doesn't decrease

**Debug steps:**

```lua
function ENT:OnTakeDamage(dmg)
    print("=== DAMAGE DEBUG ===")
    print("Damage amount:", dmg:GetDamage())
    print("Damage type:", dmg:GetDamageType())
    print("Attacker:", tostring(dmg:GetAttacker()))
    print("Current health:", self:Health())
    print("Godmode:", self:GetGodMode())

    -- Return value affects damage
    local multiplier = 1.0
    print("Damage multiplier:", multiplier)
    return multiplier
end
```

**Common causes:**
- Godmode enabled
- OnTakeDamage() returns 0
- Health set incorrectly

**Solutions:**
```lua
-- Check godmode
if self:GetGodMode() then
    print("WARNING: Godmode is enabled!")
    self:SetGodMode(false)
end

-- Don't return 0 unless intended
function ENT:OnTakeDamage(dmg)
    -- return 0 -- DON'T DO THIS (prevents damage)
    -- return 1 -- Normal damage
    -- return 2 -- Double damage
end
```

### Scenario 4: Pathfinding Fails

**Symptoms:**
- NPC doesn't move toward target
- NPC stuck in place
- "Path failed" messages

**Debug steps:**

```lua
function ENT:CustomThink()
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        local result = self:FollowPath(enemy:GetPos())
        print("Path result:", result)
        -- Results: "complete", "failed", "stuck", "timeout", "invalid"

        if result == "failed" then
            print("Path failed! Checking navmesh...")
            print("Navmesh exists:", navmesh.IsLoaded())
            print("Target position:", enemy:GetPos())
            print("Can reach:", self:ComputePath(enemy:GetPos()))
        end
    end
end
```

**Common causes:**
- No navmesh (run `nav_generate`)
- Target unreachable
- NPC stuck in geometry

**Solutions:**
```
// Generate navmesh
nav_generate

// Check navmesh
nav_edit 1

// Teleport NPC to test
lua_run Entity(1):SetPos(Vector(0, 0, 100))
```

### Scenario 5: Lua Errors

**Reading error messages:**

```
[ERROR] lua/entities/npc_drg_mypack_zombie.lua:45: attempt to index field 'Enemy' (a nil value)
    1. OnMeleeAttack - lua/entities/npc_drg_mypack_zombie.lua:45
    2. unknown - lua/entities/drgbase_nextbot/ai.lua:123
```

**Breakdown:**
- **File:** `npc_drg_mypack_zombie.lua`
- **Line:** 45
- **Error:** `attempt to index field 'Enemy' (a nil value)`
- **Meaning:** Trying to use `self.Enemy.something` when `self.Enemy` is nil

**Fix:**
```lua
-- Bad (line 45)
function ENT:OnMeleeAttack(enemy)
    print(self.Enemy:GetName()) -- self.Enemy is nil!
end

-- Good
function ENT:OnMeleeAttack(enemy)
    if IsValid(enemy) then
        print(enemy:GetName())
    end
end
```

## Debugging Tools & Commands

### Essential Console Commands

```
// Enable debug mode
drgbase_debug 1

// Show animations
drgbase_debug_animations 1

// Draw paths
drgbase_draw_paths 1

// Draw ranges
drgbase_draw_ranges 1

// Reload entity (after code changes)
lua_run Entity(1):Remove()
(then respawn)

// Print entity table
lua_run PrintTable(Entity(1):GetTable())
```

### Debug Visualization

```lua
function ENT:CustomThink()
    if not GetConVar("drgbase_debug"):GetBool() then return end

    -- Draw detection range
    debugoverlay.Sphere(self:GetPos(), self.SightRange, 0.1, Color(0, 255, 0, 10), true)

    -- Draw to enemy
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        debugoverlay.Line(self:GetPos(), enemy:GetPos(), 0.1, Color(255, 0, 0), true)
    end

    -- Draw current position
    debugoverlay.Cross(self:GetPos(), 20, 0.1, Color(255, 255, 0), true)

    -- Draw text info
    debugoverlay.EntityTextAtPosition(self:GetPos(), 0, tostring(self:GetEnemy()), 0.1)
    debugoverlay.EntityTextAtPosition(self:GetPos(), 1, "HP: " .. self:Health(), 0.1)
end
```

### Custom Debug Mode

```lua
ENT.DebugMode = false

function ENT:CustomInitialize()
    -- Enable debug for testing
    self.DebugMode = GetConVar("drgbase_debug"):GetBool()
end

function ENT:DebugPrint(...)
    if not self.DebugMode then return end
    print("[" .. self:GetClass() .. "]", ...)
end

function ENT:OnMeleeAttack(enemy)
    self:DebugPrint("Attack started on", tostring(enemy))
    -- Rest of function
end
```

## Debugging Checklist

### Before Debugging

- [ ] Read error message carefully
- [ ] Note when problem occurs
- [ ] Check if problem is consistent
- [ ] Try with fresh NPC spawn

### During Debugging

- [ ] Add print statements
- [ ] Use Info Tool
- [ ] Enable debug ConVars
- [ ] Test in isolation
- [ ] Check one system at a time

### After Fixing

- [ ] Remove debug prints
- [ ] Test edge cases
- [ ] Verify other NPCs still work
- [ ] Document the fix

## Advanced Debugging

### Breakpoint-Style Debugging

```lua
function ENT:OnMeleeAttack(enemy)
    -- Pause execution and examine state
    if self.Debug_BreakOnAttack then
        print("=== BREAKPOINT ===")
        print("Self:", self)
        print("Enemy:", enemy)
        print("Health:", self:Health())
        print("Position:", self:GetPos())
        PrintTable(self:GetTable())
        self.Debug_BreakOnAttack = false
    end

    -- Normal function continues
end

-- Trigger breakpoint
lua_run Entity(1).Debug_BreakOnAttack = true
```

### Stack Trace Debugging

```lua
function ENT:MyFunction()
    -- Print call stack
    debug.Trace()

    -- Or get stack as string
    local trace = debug.traceback()
    print(trace)
end
```

### Conditional Debugging

```lua
function ENT:CustomThink()
    -- Only debug specific NPCs
    if self:EntIndex() == 123 then
        print("NPC 123 thinking")
    end

    -- Only debug when health is low
    if self:Health() < 20 then
        print("Low health! Current:", self:Health())
    end

    -- Only debug near player
    if self:GetPos():Distance(Player(1):GetPos()) < 500 then
        print("Near player!")
    end
end
```

## Debugging Resources

### Useful Hooks for Debugging

```lua
-- Track when NPC is created
hook.Add("OnEntityCreated", "DebugNPCSpawn", function(ent)
    if ent.IsDrGNextbot then
        print("NPC spawned:", ent:GetClass())
    end
end)

-- Track when NPC is removed
function ENT:OnRemove()
    print("NPC removed:", self:GetClass(), "at", self:GetPos())
end

-- Track errors
function ENT:OnError(err)
    print("NPC ERROR:", self:GetClass())
    print("Error message:", err)
    print("Position:", self:GetPos())
    print("Health:", self:Health())
    debug.Trace()
end
```

### Performance Profiling

```lua
function ENT:CustomInitialize()
    self.PerformanceData = {}
end

function ENT:ProfileFunction(name, func)
    local startTime = SysTime()
    func()
    local endTime = SysTime()

    local duration = (endTime - startTime) * 1000
    self.PerformanceData[name] = (self.PerformanceData[name] or 0) + duration

    if self.PerformanceData[name] > 100 then
        print("WARNING: " .. name .. " is slow:", self.PerformanceData[name] .. "ms")
    end
end

-- Usage
self:ProfileFunction("Pathfinding", function()
    self:FollowPath(targetPos)
end)
```

## Troubleshooting Tips

1. **Isolate the problem**: Test with minimal code
2. **Check the basics**: Navmesh, relationships, ranges
3. **Use tools first**: Info Tool shows most issues
4. **Add prints strategically**: Not everywhere, just key points
5. **Test incrementally**: One change at a time
6. **Read errors carefully**: They tell you exactly what's wrong
7. **Compare working code**: Look at example NPCs
8. **Ask for help**: With error messages and what you've tried

## See Also

- [Console Commands](../tools/console-commands.md)
- [Info Tool](../tools/info-tool.md)
- [Debug Tools](../tools/debug-tools.md)
- [Common Pitfalls](common-pitfalls.md)
