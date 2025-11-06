# Performance Optimization

Learn how to optimize DrGBase NPCs for better server and client performance. This guide covers think rates, AI optimization, pathfinding, detection systems, and profiling techniques.

## Table of Contents

- [Introduction](#introduction)
- [Think Rate and Update Frequency](#think-rate-and-update-frequency)
- [AI Optimization Techniques](#ai-optimization-techniques)
- [Pathfinding Optimization](#pathfinding-optimization)
- [Detection and Awareness Optimization](#detection-and-awareness-optimization)
- [Animation and Model Optimization](#animation-and-model-optimization)
- [Network Optimization](#network-optimization)
- [Memory Management](#memory-management)
- [Performance Testing and Profiling](#performance-testing-and-profiling)
- [Best Practices and Benchmarks](#best-practices-and-benchmarks)

## Introduction

Performance optimization is crucial when working with NPCs, especially when spawning multiple entities. DrGBase provides built-in optimization features and allows you to fine-tune performance based on your needs.

### Performance Impact Factors

The main factors affecting NPC performance:

1. **Think rate** - How often NPCs update their logic
2. **AI complexity** - Enemy detection and decision-making
3. **Pathfinding** - Navigation mesh calculations
4. **Detection systems** - Sight and hearing checks
5. **Animation updates** - Model animation frequency
6. **Network traffic** - Data replication to clients
7. **Memory usage** - Entity and ragdoll cleanup

### Performance Goals

- **Server FPS**: Maintain 60+ tickrate with multiple NPCs
- **Client FPS**: Minimize client-side impact from NPCs
- **Network Usage**: Reduce bandwidth consumption
- **Memory**: Prevent memory leaks and bloat

## Think Rate and Update Frequency

DrGBase uses a tiered think system with three different update frequencies to balance performance and responsiveness.

### Think Delay Tiers

The base nextbot implements three think delay levels:

```lua
-- Long delays (1 second)
if CurTime() > self._DrGBaseThinkDelayLong then
    self._DrGBaseThinkDelayLong = CurTime() + 1
    self:_RegenHealth()      -- Health regeneration
    self:UpdateAI()          -- AI state updates
end

-- Medium delays (0.1 seconds / 10 Hz)
if CurTime() > self._DrGBaseThinkDelayMedium then
    self._DrGBaseThinkDelayMedium = CurTime() + 0.1
    self:UpdateAnimation()   -- Animation updates
    self:UpdateSpeed()       -- Speed calculations
    -- Physics object updates
end

-- Short delays (0.05 seconds / 20 Hz)
if CurTime() > self._DrGBaseThinkDelayShort then
    self._DrGBaseThinkDelayShort = CurTime() + 0.05
    -- Water level checks
    -- Fire state checks
    -- Fall speed tracking
end
```

### Custom Think Implementation

You can implement custom think logic with adjustable delays:

```lua
function ENT:CustomThink()
    -- This runs based on the delay returned
    -- Perform your custom logic here

    -- Return delay in seconds until next think
    return 0.5  -- Think every 0.5 seconds
end

function ENT:_BaseThink()
    -- Internal framework think
    -- Return delay if needed
end
```

### Think Rate Recommendations

| Update Type | Frequency | Use Case |
|-------------|-----------|----------|
| Critical updates | 0.05s (20 Hz) | Player tracking, combat |
| Standard updates | 0.1s (10 Hz) | Movement, animations |
| Slow updates | 0.5-1.0s | Idle behaviors, status effects |
| Very slow updates | 2-5s | Ambient sounds, rare checks |

**Example: Optimized Think Pattern**

```lua
function ENT:CustomThink()
    if self:HasEnemy() then
        -- Combat mode: faster updates
        self:UpdateCombatLogic()
        return 0.1
    elseif self:IsMoving() then
        -- Moving: medium updates
        self:UpdatePatrolLogic()
        return 0.3
    else
        -- Idle: slow updates
        self:UpdateIdleLogic()
        return 1.0
    end
end
```

### Performance Impact

- Each NPC thinking at 0.05s = 20 function calls per second
- 100 NPCs at 0.05s = 2,000 calls/second
- 100 NPCs at 0.5s = 200 calls/second (10x reduction)

## AI Optimization Techniques

AI systems are often the most expensive part of NPC logic. DrGBase provides several optimization features.

### Enemy Detection Radius

Limit the range at which NPCs detect enemies:

```lua
-- Server console or autoexec.cfg
drgbase_ai_radius 3000  -- Default is 5000
```

**In code:**

```lua
function ENT:OnUpdateEnemy()
    local enemy = self:FetchEnemy()

    -- Only consider enemies within custom range
    if IsValid(enemy) and self:GetRangeSquaredTo(enemy) > 2000^2 then
        return NULL
    end

    return enemy
end
```

### Omniscient Mode

For performance testing or specific game modes:

```lua
-- Makes all NPCs aware of all entities without checks
drgbase_ai_omniscient 1

-- In entity code
ENT.Omniscient = true  -- This NPC always knows enemy positions
```

**Performance Note:** Omniscient mode reduces detection overhead but can make gameplay too easy.

### Conditional AI Updates

Only update AI when necessary:

```lua
function ENT:UpdateAI()
    -- Skip AI updates if possessed
    if self:IsPossessed() then return end

    -- Skip if AI is disabled
    if self:IsAIDisabled() then return end

    -- Only update hostile sight checks
    self:UpdateHostilesSight()
    self:UpdateEnemy()
end
```

### Disable AI Globally

For cutscenes or specific scenarios:

```lua
-- Disable all AI
ai_disabled 1

-- In code
self:SetAIDisabled(true)  -- Disable this NPC's AI
self:EnableAI()           -- Re-enable later
```

### Optimized Enemy Fetching

Prioritize enemies efficiently:

```lua
function ENT:OnFetchEnemy(enemy1, enemy2)
    -- Custom priority logic
    local prio1 = self:GetPriority(enemy1)
    local prio2 = self:GetPriority(enemy2)

    if prio1 > prio2 then
        return true
    elseif prio2 > prio1 then
        return false
    end

    -- Default: choose closer enemy
    return self:GetRangeSquaredTo(enemy1) < self:GetRangeSquaredTo(enemy2)
end
```

### Spot Duration Optimization

Control how long NPCs remember spotted entities:

```lua
ENT.SpotDuration = 30  -- Remember for 30 seconds (default)

-- For faster-paced gameplay
ENT.SpotDuration = 10  -- Forget after 10 seconds

-- For stealth games
ENT.SpotDuration = 60  -- Long memory
```

**Performance Impact:**
- Shorter spot durations reduce timer overhead
- Each spotted entity creates a timer
- 100 NPCs × 5 spotted enemies = 500 timers

## Pathfinding Optimization

Pathfinding is computationally expensive. Optimize it carefully.

### Path Computation Delay

Control how often paths are recomputed:

```lua
-- Server console
drgbase_compute_delay 0.1  -- Default: recompute every 0.1s

-- For slower NPCs or less precision
drgbase_compute_delay 0.5  -- Recompute every 0.5s
```

### Minimize Look-Ahead Distance

The path system uses look-ahead for smoother navigation:

```lua
function ENT:_InitPath()
    self._DrGBaseNavAreaBlacklist = {}
end

function ENT:GetPath()
    if not self._DrGBasePath then
        self._DrGBasePath = Path("Follow")
        -- Default is 300, reduce for simpler paths
        self._DrGBasePath:SetMinLookAheadDistance(200)
        return self._DrGBasePath
    else
        return self._DrGBasePath
    end
end
```

**Impact:**
- Lower values = simpler paths, less computation
- Higher values = smoother paths, more computation

### Smart Path Recomputation

Only recompute when necessary:

```lua
-- DrGBase already implements this optimization
local function ShouldCompute(self, path, pos)
    if not IsValid(path) then return true end

    local path_length = path:GetLength()
    local cursor_pos = path:GetCursorPosition()
    local remaining_length = path_length - cursor_pos

    -- Only recompute if target moved significantly
    return self._DrGBasePathLastSegment.pos:Distance(pos)
        > remaining_length / 2
end
```

### Path Generator Optimization

Customize path cost calculations:

```lua
function ENT:OnComputePath(from, to)
    -- Return 0 for normal cost
    -- Return higher values to make path less desirable
    -- Return -1 to make path impossible

    if to:IsUnderwater() and not self:CanSwim() then
        return -1  -- Don't path through water
    end

    return 0  -- Normal cost
end

function ENT:OnComputePathLadderUp(from, to, ladder)
    -- Increase cost for ladders (NPCs will prefer other routes)
    return 2  -- 2x cost multiplier
end
```

### Disable Expensive Features

Turn off features you don't need:

```lua
ENT.ClimbLedges = false       -- Disable ledge climbing checks
ENT.ClimbLadders = false      -- Disable ladder climbing checks
ENT.ClimbProps = false        -- Disable prop climbing checks

-- Disable obstacle avoidance if not needed
drgbase_avoid_obstacles 0
```

### NavMesh Area Blacklisting

Prevent NPCs from using expensive paths:

```lua
function ENT:OnComputePath(from, to)
    -- Blacklist dangerous or expensive areas
    if to:HasAttributes(NAV_MESH_AVOID) then
        self:BlacklistNavArea(to, true)
        return -1
    end
    return 0
end
```

## Detection and Awareness Optimization

Detection systems (sight and hearing) can be very expensive with many NPCs.

### Sight Optimization

Configure sight parameters for performance:

```lua
-- Reduce sight range
ENT.SightFOV = 120         -- Default: 150 degrees
ENT.SightRange = 3000      -- Default: 15000 units

-- Luminosity constraints
ENT.MinLuminosity = 0.2    -- Don't see in darkness
ENT.MaxLuminosity = 1.0
```

**Performance Calculation:**

```lua
-- Sight check per NPC per frame:
-- 1. Distance check (fast)
-- 2. FOV check (fast)
-- 3. Trace check (expensive)
-- Reducing SightRange eliminates expensive traces
```

### Hearing Optimization

Adjust hearing sensitivity:

```lua
ENT.HearingCoefficient = 0.5  -- 50% hearing range

-- Disable hearing entirely for performance
ENT.HearingCoefficient = 0

-- Global hearing disable
drgbase_ai_hearing 0
```

### Disable Sight Globally

For fog-of-war or blind scenarios:

```lua
drgbase_ai_sight 0  -- Disable all NPC sight
```

### Optimize UpdateSight Calls

Control when sight updates occur:

```lua
function ENT:UpdateAI()
    -- Only update sight for relevant entities
    if self:HasEnemy() then
        -- Only check enemies when we already have one
        self:UpdateEnemiesSight()
    else
        -- Check all hostiles when searching
        self:UpdateHostilesSight()
    end
end
```

### Spot Duration vs. Update Frequency

Balance memory and responsiveness:

```lua
-- High spot duration, low update frequency
ENT.SpotDuration = 60     -- Remember for 1 minute
-- Update AI every 1 second (in CustomThink)

-- Low spot duration, high update frequency
ENT.SpotDuration = 5      -- Remember for 5 seconds
-- Update AI every 0.1 seconds
```

### Visibility Caching

Implement custom visibility caching:

```lua
function ENT:CustomInitialize()
    self.VisibilityCache = {}
    self.VisibilityCacheTime = {}
end

function ENT:IsVisibleCached(ent, duration)
    duration = duration or 0.5

    if self.VisibilityCacheTime[ent] and
       CurTime() < self.VisibilityCacheTime[ent] then
        return self.VisibilityCache[ent]
    end

    local visible = self:Visible(ent)
    self.VisibilityCache[ent] = visible
    self.VisibilityCacheTime[ent] = CurTime() + duration
    return visible
end
```

## Animation and Model Optimization

Animation updates can be expensive, especially with many NPCs.

### Use Walk Frames

Let the engine handle movement animations:

```lua
ENT.UseWalkframes = true  -- Engine handles walk/run cycles
```

**Benefits:**
- Reduces custom animation logic
- Engine-optimized
- Automatic speed matching

### Animation Update Rate

Animations update at medium frequency (0.1s):

```lua
-- In shared.lua Think() function
if CurTime() > self._DrGBaseThinkDelayMedium then
    self._DrGBaseThinkDelayMedium = CurTime() + 0.1
    self:UpdateAnimation()
    self:UpdateSpeed()
end
```

### Model Complexity

Choose appropriate model detail:

```lua
-- Simple models for background NPCs
ENT.Models = {"models/player/kleiner.mdl"}  -- ~5k triangles

-- Avoid high-poly models for NPCs that spawn in large numbers
-- Bad: models/player/charple.mdl (60k+ triangles)
```

### LOD (Level of Detail)

While DrGBase doesn't have built-in LOD, you can implement it:

```lua
function ENT:CustomDraw()
    local dist = LocalPlayer():GetPos():Distance(self:GetPos())

    if dist > 2000 then
        -- Far away: skip custom drawing
        return
    elseif dist > 1000 then
        -- Medium distance: reduced detail
        self:DrawSimple()
    else
        -- Close: full detail
        self:DrawDetailed()
    end
end
```

### Reduce Bone Updates

If you don't need bone manipulation:

```lua
function ENT:CustomInitialize()
    -- Disable IK if not needed (client-side)
    if CLIENT then
        self:SetIK(false)
    end
end
```

### Shadow Rendering

Disable shadows on sprites or 2D NPCs:

```lua
function ENT:Initialize()
    if self.IsDrGNextbotSprite then
        self:DrawShadow(false)  -- No shadow for sprites
    end
end
```

## Network Optimization

Network traffic can impact both server and client performance.

### Minimize NW2 Variable Usage

DrGBase uses NW2 variables for replication:

```lua
-- These are replicated to all clients:
self:SetNW2Int("DrGBaseSightFOV", angle)
self:SetNW2Int("DrGBaseSightRange", range)
self:SetNW2Float("DrGBaseSpeed", speed)
```

**Optimization:**

```lua
-- Only set when values actually change
function ENT:SetSightFOV(angle)
    if angle == self:GetSightFOV() then return end
    if angle > 360 then angle = 360 end
    if angle < 0 then angle = 0 end
    self:SetNW2Int("DrGBaseSightFOV", angle)
end
```

### Reduce Network Messages

Batch network messages when possible:

```lua
-- Bad: Multiple network messages
for i = 1, 10 do
    net.Start("UpdateNPC")
    net.WriteEntity(self)
    net.WriteInt(i, 8)
    net.Broadcast()
end

-- Good: Single batched message
net.Start("UpdateNPCBatch")
net.WriteEntity(self)
net.WriteInt(10, 8)
for i = 1, 10 do
    net.WriteInt(i, 8)
end
net.Broadcast()
```

### Client-Side Sound Optimization

Control where sounds play:

```lua
ENT.ClientIdleSounds = false  -- Server plays idle sounds
-- vs
ENT.ClientIdleSounds = true   -- Each client plays (uses client resources)
```

**Recommendation:** Use server-side sounds unless client prediction is important.

### Player Awareness Network

DrGBase sends awareness updates to players:

```lua
-- This is sent when awareness changes
net.Start("DrGBaseNextbotPlayerAwareness")
net.WriteEntity(self)
net.WriteBit(true)  -- spotted = true
net.Send(ent)
```

**Optimization:** Spot duration affects network traffic. Longer durations mean fewer updates.

## Memory Management

Proper cleanup prevents memory leaks and crashes.

### Ragdoll Management

Configure ragdoll behavior:

```lua
-- Console commands
drgbase_remove_ragdolls 10      -- Remove after 10 seconds (-1 = never)
drgbase_ragdoll_fadeout 3       -- Fade out over 3 seconds
drgbase_ragdoll_collisions_disabled 1  -- Disable ragdoll collisions
```

**In code:**

```lua
ENT.RagdollOnDeath = false  -- Don't create ragdoll on death

-- Or custom ragdoll logic
function ENT:OnRagdoll(ragdoll, dmg, ent)
    -- Custom cleanup
    timer.Simple(5, function()
        if IsValid(ragdoll) then
            ragdoll:Remove()
        end
    end)
    return true  -- Prevent default fadeout
end
```

### Timer Cleanup

DrGBase automatically cleans up timers, but be aware:

```lua
-- This timer is automatically cleaned when NPC is removed
self:Timer(10, function()
    self:DoSomething()
end)

-- Manual timer cleanup if needed
timer.Remove("DrGBaseNB"..self:GetCreationID().."SpotENT"..ent:GetCreationID())
```

### Entity References

Avoid keeping references to removed entities:

```lua
-- Bad: Storing entity reference
self.MyEnemy = enemy

-- Good: Validate before use
if IsValid(self.MyEnemy) then
    self:AttackEntity(self.MyEnemy)
end

-- Better: Use built-in enemy system
local enemy = self:GetEnemy()
if IsValid(enemy) then
    self:AttackEntity(enemy)
end
```

### Table Cleanup

Clear large tables when done:

```lua
function ENT:OnRemove()
    -- Clear large data structures
    self._DrGBaseSpotted = nil
    self._DrGBaseRelationshipCaches = nil
    self.MyCustomTable = nil
end
```

### Coroutine Cleanup

DrGBase manages coroutines, but avoid memory leaks:

```lua
function ENT:ReactInCoroutine(callback, ...)
    -- Arguments are stored until coroutine executes
    -- Don't pass large tables or entities if not needed
end
```

## Performance Testing and Profiling

Measure before optimizing.

### Basic Performance Testing

Test with multiple NPCs:

```lua
-- Spawn test NPCs
for i = 1, 100 do
    local npc = ents.Create("npc_drg_testnextbot")
    npc:SetPos(Vector(math.random(-1000, 1000), math.random(-1000, 1000), 0))
    npc:Spawn()
end
```

### Monitor Server Performance

```lua
-- Console commands
net_graph 1         -- Show network stats
cl_showfps 1        -- Show FPS
r_speeds 1          -- Show render stats

-- Server-side
sv_showlag 1        -- Show server lag
```

### Profiling Custom Code

Add performance logging:

```lua
function ENT:CustomThink()
    local startTime = SysTime()

    -- Your code here
    self:ExpensiveOperation()

    local duration = SysTime() - startTime
    if duration > 0.001 then
        print(string.format("CustomThink took %.4f seconds", duration))
    end

    return 0.1
end
```

### Benchmark Tool

Create a benchmark tool:

```lua
concommand.Add("drgbase_benchmark", function(ply, cmd, args)
    local count = tonumber(args[1]) or 50
    local className = args[2] or "npc_drg_testnextbot"

    local startTime = SysTime()
    local startFPS = 1 / FrameTime()

    for i = 1, count do
        local npc = ents.Create(className)
        npc:SetPos(ply:GetPos() + Vector(math.random(-500, 500), math.random(-500, 500), 0))
        npc:Spawn()
    end

    timer.Simple(5, function()
        local endFPS = 1 / FrameTime()
        local fpsDrop = startFPS - endFPS
        local duration = SysTime() - startTime

        print(string.format("Spawned %d NPCs in %.2f seconds", count, duration))
        print(string.format("FPS: %.1f -> %.1f (-%..1f)", startFPS, endFPS, fpsDrop))
    end)
end)
```

### ConVar Testing

Test with different settings:

```bash
# Test suite
drgbase_ai_radius 5000
drgbase_benchmark 100

drgbase_ai_radius 3000
drgbase_benchmark 100

drgbase_ai_radius 1000
drgbase_benchmark 100

# Compare results
```

### Memory Profiling

Check memory usage:

```lua
concommand.Add("drgbase_memory", function()
    collectgarbage("collect")
    local memBefore = collectgarbage("count")

    -- Do something
    for i = 1, 100 do
        local npc = ents.Create("npc_drg_testnextbot")
        npc:Spawn()
    end

    timer.Simple(1, function()
        collectgarbage("collect")
        local memAfter = collectgarbage("count")
        print(string.format("Memory: %.2f KB -> %.2f KB (+%.2f KB)",
            memBefore, memAfter, memAfter - memBefore))
    end)
end)
```

## Best Practices and Benchmarks

Guidelines for optimal performance.

### Performance Targets

| Scenario | Target FPS | Max NPCs | Settings |
|----------|-----------|----------|----------|
| Singleplayer | 60+ FPS | 50-100 | Default settings |
| Multiplayer | 60+ tick | 20-50 | Reduced AI radius |
| Large battles | 30+ FPS | 100+ | Disabled features |
| Background NPCs | 60+ FPS | 200+ | Slow think rates |

### Optimization Checklist

**Essential:**
- [ ] Set appropriate think rates for your NPC's behavior
- [ ] Reduce sight range to minimum needed
- [ ] Configure spot duration based on gameplay
- [ ] Disable unused features (climbing, weapons, etc.)
- [ ] Use simple models when spawning many NPCs
- [ ] Configure ragdoll cleanup

**Advanced:**
- [ ] Implement custom visibility caching
- [ ] Optimize enemy fetching logic
- [ ] Reduce path computation frequency
- [ ] Batch network messages
- [ ] Implement LOD for drawing
- [ ] Profile and measure performance

### Common Performance Mistakes

**Mistake 1: Too Frequent Thinking**

```lua
-- Bad: Thinking every frame
function ENT:CustomThink()
    self:ComplexCalculation()
    return 0  -- Think every frame!
end

-- Good: Appropriate think rate
function ENT:CustomThink()
    self:ComplexCalculation()
    return 0.2  -- Think every 0.2 seconds
end
```

**Mistake 2: Not Validating Entities**

```lua
-- Bad: No validation
function ENT:CustomThink()
    local enemy = self.CachedEnemy
    self:Attack(enemy)  -- Crash if enemy was removed!
end

-- Good: Always validate
function ENT:CustomThink()
    if IsValid(self.CachedEnemy) then
        self:Attack(self.CachedEnemy)
    end
end
```

**Mistake 3: Expensive Operations in Loops**

```lua
-- Bad: Traces in loop
for i = 1, 360 do
    local tr = util.TraceLine({
        start = self:GetPos(),
        endpos = self:GetPos() + Vector(math.cos(i), math.sin(i), 0) * 1000
    })
end

-- Good: Cache results or reduce frequency
if not self.CachedSurroundings or CurTime() > self.NextSurroundingCheck then
    self:CheckSurroundings()
    self.NextSurroundingCheck = CurTime() + 1
end
```

**Mistake 4: Not Using Distance Squared**

```lua
-- Bad: Square root calculation
if self:GetPos():Distance(enemy:GetPos()) < 500 then
    -- Attack
end

-- Good: No square root
if self:GetPos():DistToSqr(enemy:GetPos()) < 500^2 then
    -- Attack
end

-- Best: Built-in function
if self:IsInRange(enemy, 500) then
    -- Attack (uses DistToSqr internally)
end
```

### Recommended ConVar Profiles

**Performance Mode (large battles):**

```bash
drgbase_ai_radius 2000
drgbase_compute_delay 0.3
drgbase_ai_patrol 0
drgbase_remove_ragdolls 5
drgbase_ragdoll_collisions_disabled 1
drgbase_avoid_obstacles 0
```

**Balanced Mode (default):**

```bash
drgbase_ai_radius 5000
drgbase_compute_delay 0.1
drgbase_ai_patrol 1
drgbase_remove_ragdolls -1
drgbase_ragdoll_collisions_disabled 0
drgbase_avoid_obstacles 1
```

**Quality Mode (few NPCs):**

```bash
drgbase_ai_radius 10000
drgbase_compute_delay 0.05
drgbase_ai_patrol 1
drgbase_remove_ragdolls -1
drgbase_ragdoll_collisions_disabled 0
drgbase_avoid_obstacles 1
```

### Performance Benchmarks

Typical performance on a mid-range system (i5 processor, 16GB RAM):

| NPC Count | Settings | Server FPS | Notes |
|-----------|----------|------------|-------|
| 10 | Default | 60+ | No impact |
| 50 | Default | 60 | Slight impact |
| 100 | Default | 30-45 | Noticeable lag |
| 100 | Performance | 50-60 | Optimized |
| 200 | Performance | 30-40 | Heavy load |
| 500 | Performance | 10-20 | Extreme load |

**Notes:**
- Benchmarks vary by map size and complexity
- Active combat (pathfinding, attacks) is more expensive than idle NPCs
- Client FPS may differ from server tickrate

### Optimization Priority

When optimizing, focus on these areas in order:

1. **Think Rate** (Highest impact, easiest to change)
   - Adjust CustomThink return values
   - Conditional update frequencies

2. **AI Range** (High impact, easy)
   - Reduce drgbase_ai_radius
   - Limit enemy detection range

3. **Detection** (High impact, moderate effort)
   - Reduce sight range/FOV
   - Optimize UpdateSight calls

4. **Pathfinding** (Moderate impact, moderate effort)
   - Increase compute delay
   - Simplify path generators
   - Disable climbing features

5. **Animations** (Low-moderate impact, easy)
   - Use UseWalkframes
   - Simple models

6. **Network** (Low impact, requires careful coding)
   - Batch messages
   - Reduce NW2 updates

### When to Optimize

**Optimize when:**
- Server tickrate drops below 60
- Client FPS drops noticeably
- Network traffic is high (lag)
- Memory usage keeps growing

**Don't optimize when:**
- Everything runs smoothly
- You're just starting development
- You haven't profiled yet

**Remember:** Premature optimization is the root of all evil. Measure first, optimize second.

---

## Summary

DrGBase NPCs are designed to be efficient, but you can significantly improve performance by:

1. Using appropriate think rates for different behaviors
2. Limiting AI detection ranges
3. Optimizing pathfinding parameters
4. Reducing detection system overhead
5. Managing network traffic
6. Cleaning up ragdolls and timers

Always profile your NPCs under realistic conditions and optimize the areas with the most impact first.

For more information, see:
- [Creating Custom NPCs](creating-npcs.md)
- [Best Practices - Performance](../best-practices/02-performance.md)
- [API Reference - AI System](../api/nextbot/ai.md)
