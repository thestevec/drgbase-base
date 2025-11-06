# Performance Optimization Guide

## Overview

Proper optimization ensures your DrGBase NPCs run smoothly even with many active at once. This guide covers performance best practices and optimization techniques.

## Key Performance Factors

### 1. AI Update Rate

**Problem:** AI updates every frame = expensive
**Solution:** Use coroutines and Wait()

```lua
-- ❌ Bad: Runs every frame
function ENT:CustomThink()
    self:CheckForEnemies()
    self:UpdateTargeting()
    self:ProcessBehavior()
end

-- ✅ Good: Runs periodically
function ENT:CustomThink()
    -- Fast checks only
end

function ENT:AIBehaviour()
    while true do
        self:CheckForEnemies()
        self:UpdateTargeting()
        self:ProcessBehavior()
        self:Wait(0.5) -- Half-second intervals
    end
end
```

**Best Practices:**
- Heavy AI logic in `AIBehaviour()` with `Wait()`
- Light checks in `CustomThink()` (runs every frame)
- Use 0.1-0.5 second intervals for AI updates

### 2. Detection Range

**Problem:** Large detection ranges = expensive traces
**Solution:** Use appropriate ranges

```lua
-- ❌ Bad: Unnecessarily large
ENT.SightRange = 50000 -- Can see across entire map
ENT.RangeAttackRange = 10000

-- ✅ Good: Reasonable ranges
ENT.SightRange = 2000  -- ~40 meters
ENT.RangeAttackRange = 500  -- ~10 meters
```

**Guidelines:**
- **Close Combat NPC**: SightRange = 1000-1500
- **Ranged NPC**: SightRange = 2000-3000
- **Sniper NPC**: SightRange = 4000-5000
- **Boss NPC**: SightRange = 3000-5000

### 3. Entity Searches

**Problem:** Frequent `ents.FindInSphere()` = laggy
**Solution:** Cache and throttle searches

```lua
-- ❌ Bad: Every frame
function ENT:CustomThink()
    local nearby = ents.FindInSphere(self:GetPos(), 500)
    -- Process nearby entities
end

-- ✅ Good: Cached with refresh interval
function ENT:CustomInitialize()
    self.NearbyEntitiesCache = {}
    self.NextCacheUpdate = 0
end

function ENT:GetNearbyEntities()
    if CurTime() < self.NextCacheUpdate then
        return self.NearbyEntitiesCache
    end

    self.NearbyEntitiesCache = ents.FindInSphere(self:GetPos(), 500)
    self.NextCacheUpdate = CurTime() + 1 -- Update every second
    return self.NearbyEntitiesCache
end
```

### 4. Pathfinding

**Problem:** Path recalculation every frame = expensive
**Solution:** Use built-in path caching

```lua
-- ❌ Bad: Recalculates constantly
function ENT:CustomThink()
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        self:ComputePath(enemy:GetPos())
        self:FollowPath(enemy:GetPos())
    end
end

-- ✅ Good: Uses cached paths
function ENT:HandleEnemy(enemy)
    -- DrGBase automatically caches and refreshes paths
    self:ChaseEnemy(enemy)
end

-- Or in custom AI:
function ENT:AIBehaviour()
    while true do
        local enemy = self:GetEnemy()
        if IsValid(enemy) then
            self:FollowPath(enemy:GetPos())
            self:Wait(0.5) -- Update path every 0.5s
        else
            self:Wait(1)
        end
    end
end
```

### 5. Sound Management

**Problem:** Too many sounds = audio lag
**Solution:** Use sound slots

```lua
-- ❌ Bad: Unlimited concurrent sounds
function ENT:CustomThink()
    self:EmitSound("Idle.Loop")  -- Plays every frame!
end

-- ✅ Good: Use sound slots
function ENT:CustomThink()
    -- EmitSlotSound prevents overlapping
    self:EmitSlotSound("idle", 5, "Idle.Loop")
    -- Only plays if 5 seconds passed since last
end

-- Or use timers
function ENT:CustomInitialize()
    self:LoopTimer(5, function(self)
        self:EmitSound("Idle.Sound")
    end)
end
```

### 6. Network Traffic

**Problem:** Sending too much data to clients
**Solution:** Only network what's necessary

```lua
-- ❌ Bad: Networking everything
function ENT:CustomThink()
    self:SetNWFloat("MyVariable", someValue) -- Every frame!
end

-- ✅ Good: Network only when changed
function ENT:SetMyVariable(value)
    if self.MyVariable != value then
        self.MyVariable = value
        self:SetNWFloat("MyVariable", value)
    end
end

-- Or use NW2 for better performance
self:SetNW2Float("MyVariable", value)
```

## Optimization Techniques

### LOD (Level of Detail)

Reduce update rate for distant NPCs:

```lua
function ENT:CustomInitialize()
    self.UpdateInterval = 0.1
end

function ENT:CustomThink()
    -- Adjust update interval based on distance to players
    local closestDist = math.huge
    for _, ply in ipairs(player.GetAll()) do
        local dist = self:GetPos():Distance(ply:GetPos())
        closestDist = math.min(closestDist, dist)
    end

    if closestDist > 2000 then
        self.UpdateInterval = 1.0 -- Far: update every second
    elseif closestDist > 1000 then
        self.UpdateInterval = 0.5 -- Medium: half second
    else
        self.UpdateInterval = 0.1 -- Close: tenth second
    end
end

function ENT:AIBehaviour()
    while true do
        self:UpdateAI()
        self:Wait(self.UpdateInterval)
    end
end
```

### Culling

Disable expensive features when not needed:

```lua
function ENT:CustomThink()
    local anyPlayerNearby = false
    for _, ply in ipairs(player.GetAll()) do
        if self:GetPos():Distance(ply:GetPos()) < 3000 then
            anyPlayerNearby = true
            break
        end
    end

    if not anyPlayerNearby then
        -- No players nearby, reduce processing
        self:DisableAI(true)
    else
        if self:IsAIDisabled() then
            self:DisableAI(false)
        end
    end
end
```

### Batch Operations

Process multiple NPCs together:

```lua
-- Server-side batch processing
hook.Add("Think", "DrGBase_BatchUpdate", function()
    if (DrGBase.NextBatchUpdate or 0) > CurTime() then return end
    DrGBase.NextBatchUpdate = CurTime() + 0.5

    -- Update all NPCs in batch
    for _, npc in ipairs(DrGBase.GetNextbots()) do
        if IsValid(npc) then
            npc:BatchUpdate()
        end
    end
end)

function ENT:BatchUpdate()
    -- Expensive operations here
    -- Called every 0.5s for ALL NPCs at once
end
```

### Lazy Initialization

Delay expensive setup:

```lua
function ENT:CustomInitialize()
    -- Don't do everything immediately
    self:Timer(0.5, function(self)
        self:CompleteSetup()
    end)
end

function ENT:CompleteSetup()
    -- Expensive initialization
    self:BuildNavigationCache()
    self:PrecomputeAnimations()
    self:LoadResources()
end
```

## Performance Checklist

### ✅ AI Optimization
- [ ] Use `Wait()` in `AIBehaviour()` (0.1-0.5s intervals)
- [ ] Keep `CustomThink()` lightweight
- [ ] Use appropriate detection ranges
- [ ] Cache enemy/target references
- [ ] Implement LOD system for distant NPCs

### ✅ Detection Optimization
- [ ] Reasonable `SightRange` (< 5000)
- [ ] Appropriate `SightFOV` (60-180 degrees)
- [ ] Cache entity searches
- [ ] Use DrGBase's built-in awareness caching

### ✅ Pathfinding Optimization
- [ ] Don't recalculate paths every frame
- [ ] Use 0.5-1s path refresh intervals
- [ ] Check path validity before following
- [ ] Use nav_generate for proper navmesh

### ✅ Sound Optimization
- [ ] Use `EmitSlotSound()` for repeated sounds
- [ ] Limit concurrent sounds per NPC
- [ ] Use sound delays/cooldowns
- [ ] Clean up sounds on death

### ✅ Animation Optimization
- [ ] Don't call `PlayActivity()` every frame
- [ ] Cache animation sequences
- [ ] Use appropriate playback rates
- [ ] Disable animations on distant NPCs (if possible)

### ✅ Network Optimization
- [ ] Only network changed values
- [ ] Use NW2 instead of NW
- [ ] Minimize networked variables
- [ ] Batch network updates

## Common Performance Issues

### Issue 1: Lag Spikes

**Symptom:** Game freezes periodically

**Causes:**
- Too many NPCs spawned at once
- Large `ents.FindInSphere()` radius
- Path calculation on every frame
- No LOD system

**Solutions:**
```lua
-- Limit spawning rate
function SpawnNPCs()
    for i = 1, 100 do
        timer.Simple(i * 0.1, function()
            -- Spawn one NPC every 0.1s
            local npc = ents.Create("npc_drg_zombie")
            npc:Spawn()
        end)
    end
end

-- Reduce search radius
-- Old: ents.FindInSphere(pos, 5000)
-- New: ents.FindInSphere(pos, 1000)

-- Add LOD system (see above)
```

### Issue 2: Low FPS with Many NPCs

**Symptom:** FPS drops with 20+ NPCs

**Causes:**
- No AI throttling
- All NPCs updating every frame
- Expensive CustomThink()

**Solutions:**
```lua
-- Stagger AI updates
function ENT:CustomInitialize()
    self.AIOffset = math.random() * 0.5
end

function ENT:AIBehaviour()
    self:Wait(self.AIOffset) -- Stagger start times
    while true do
        self:UpdateAI()
        self:Wait(0.5)
    end
end

-- Use ConVar to limit NPCs
CreateConVar("mypack_npc_limit", "30", FCVAR_ARCHIVE)

hook.Add("PlayerSpawnedNPC", "LimitNPCs", function(ply, ent)
    if #DrGBase.GetNextbots() > GetConVar("mypack_npc_limit"):GetInt() then
        ent:Remove()
        ply:ChatPrint("NPC limit reached!")
        return false
    end
end)
```

### Issue 3: Stuttering Movement

**Symptom:** NPCs move jerkily

**Causes:**
- Pathfinding too infrequent
- Network lag
- Animation issues

**Solutions:**
```lua
-- Smooth movement with interpolation
function ENT:CustomThink()
    -- Let DrGBase handle movement smoothing
    -- Don't manually set positions every frame
end

-- Use appropriate path update rate
function ENT:AIBehaviour()
    while true do
        self:FollowPath(targetPos)
        self:Wait(0.3) -- 300ms is good balance
    end
end
```

## Profiling Tools

### Built-in Profiling

```lua
-- Time specific functions
local startTime = SysTime()
self:ExpensiveFunction()
local endTime = SysTime()
print("Function took:", (endTime - startTime) * 1000, "ms")

-- Count function calls
self.CallCount = (self.CallCount or 0) + 1
if self.CallCount % 100 == 0 then
    print("Function called", self.CallCount, "times")
end
```

### ConVars for Testing

```
// Disable AI to test non-AI performance
drgbase_ai_tickrate 0

// Reduce detection range
drgbase_ai_radius 500

// Enable debug to see update frequency
drgbase_debug 1
```

## Performance Metrics

**Target Performance:**
- 60 FPS with 20-30 NPCs
- <1ms per NPC per frame
- Path updates: 0.5-1s intervals
- AI updates: 0.1-0.5s intervals

**Red Flags:**
- ❌ Any function taking >5ms
- ❌ CustomThink() doing heavy operations
- ❌ Entity searches every frame
- ❌ Paths calculated every frame
- ❌ Network updates every frame

## See Also

- [Debugging Guide](debugging.md)
- [Code Organization](code-organization.md)
- [Common Pitfalls](common-pitfalls.md)
