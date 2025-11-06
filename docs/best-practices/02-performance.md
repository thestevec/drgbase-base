# Performance Guidelines

Performance optimization is critical for DrGBase addons, especially when dealing with multiple NPCs simultaneously. A poorly optimized NPC might run fine alone but cause severe lag when spawned in groups. This guide covers essential performance best practices for creating efficient DrGBase NPCs.

## Best Practice 1: Use Think Delays Effectively

Avoid running expensive operations every frame. Use return values in CustomThink() to control update frequency.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        self._nextPathUpdate = 0
        self._nextEnemyCheck = 0
        self._nearbyAllies = {}
    end

    function ENT:CustomThink()
        -- Expensive operations get their own timers
        if CurTime() > self._nextEnemyCheck then
            self._nextEnemyCheck = CurTime() + 1  -- Check every 1 second
            self:SearchForEnemies()
        end

        if CurTime() > self._nextPathUpdate then
            self._nextPathUpdate = CurTime() + 0.5  -- Update path every 0.5 seconds
            self:UpdateCurrentPath()
        end

        -- Return delay for this think function
        return 0.1  -- Run CustomThink every 0.1 seconds
    end

    function ENT:SearchForEnemies()
        -- Expensive operation: find entities in radius
        local enemies = ents.FindInSphere(self:GetPos(), 2000)
        -- Process enemies...
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomThink()
        -- BAD: Running expensive operations every tick
        local enemies = ents.FindInSphere(self:GetPos(), 2000)

        for _, ent in ipairs(enemies) do
            -- Processing every frame causes lag
            if self:ShouldAttack(ent) then
                self:SetEnemy(ent)
            end
        end

        local allies = ents.FindInSphere(self:GetPos(), 3000)
        -- More expensive operations every frame...

        -- No return value = runs every tick (0.015s)
    end
end
```

### Why This Matters

Garry's Mod runs at 66 ticks per second. Without think delays, CustomThink() runs 66 times per second. With 10 NPCs, that's 660 calls per second. Expensive operations like ents.FindInSphere() can cause severe lag when called this frequently.

## Best Practice 2: Cache Expensive Lookups

Store the results of expensive operations and reuse them instead of recalculating.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        self._cachedAllies = {}
        self._cachedEnemies = {}
        self._cacheUpdateTime = 0
    end

    function ENT:GetCachedAllies()
        -- Update cache every 2 seconds
        if CurTime() > self._cacheUpdateTime then
            self._cacheUpdateTime = CurTime() + 2

            self._cachedAllies = {}
            for _, npc in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
                if npc.IsDrGNextbot and self:GetRelationship(npc) == D_LI then
                    table.insert(self._cachedAllies, npc)
                end
            end
        end

        return self._cachedAllies
    end

    function ENT:CustomThink()
        -- Use cached data multiple times without recalculating
        local allies = self:GetCachedAllies()

        if #allies > 3 then
            self:CoordinateWithAllies(allies)
        end

        if #allies == 0 then
            self:CallForBackup()
        end

        return 0.5
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:GetAllies()
        -- Recalculates every single time it's called
        local allies = {}
        for _, npc in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
            if npc.IsDrGNextbot and self:GetRelationship(npc) == D_LI then
                table.insert(allies, npc)
            end
        end
        return allies
    end

    function ENT:CustomThink()
        -- BAD: Calling the same expensive function multiple times
        if #self:GetAllies() > 3 then
            self:CoordinateWithAllies(self:GetAllies())  -- Calculated again!
        end

        if #self:GetAllies() == 0 then  -- And again!
            self:CallForBackup()
        end

        return 0.5
    end
end
```

### Why This Matters

Each call to ents.FindInSphere() checks every entity in the game. With multiple NPCs calling this multiple times per second, performance degrades quickly. Caching reduces redundant calculations dramatically.

## Best Practice 3: Optimize Detection and Vision Checks

Use DrGBase's built-in detection systems efficiently and avoid redundant visibility checks.

### ✅ DO

```lua
-- Configure efficient detection settings
ENT.SightRange = 2000      -- Reasonable range
ENT.SightFOV = 120         -- Limited field of view
ENT.HearingCoefficient = 1 -- Balanced hearing

if SERVER then
    function ENT:OnMeleeAttack(enemy)
        -- Use the enemy parameter - it's already validated
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:CustomThink()
        -- Don't duplicate DrGBase's detection - it's already optimized
        -- Just use the provided enemy system

        if self:HasEnemy() then
            local enemy = self:GetEnemy()

            -- Single visibility check when needed
            if self:Visible(enemy) then
                self:SetLastSeenPosition(enemy:GetPos())
            end
        end

        return 0.5
    end
end
```

### ❌ DON'T

```lua
-- Inefficient detection settings
ENT.SightRange = 50000     -- Unreasonably large
ENT.SightFOV = 360         -- Full sphere (expensive)

if SERVER then
    function ENT:CustomThink()
        -- BAD: Manual detection duplicating DrGBase's work
        for _, ply in ipairs(player.GetAll()) do
            -- Redundant visibility checks
            if self:Visible(ply) then
                if self:IsInSight(ply) then  -- Already checked by Visible()
                    if self:GetRangeTo(ply) < 2000 then  -- More redundant checks
                        self:SetEnemy(ply)
                    end
                end
            end
        end

        -- Multiple visibility checks for the same entity
        if self:HasEnemy() then
            local enemy = self:GetEnemy()

            if self:Visible(enemy) then
                -- Some code
            end

            if self:Visible(enemy) then  -- Checked again!
                -- More code
            end

            if self:Visible(enemy) then  -- And again!
                -- Even more code
            end
        end

        return 0  -- Every tick is overkill
    end
end
```

### Why This Matters

Visibility checks involve expensive trace operations. DrGBase's detection system is already optimized with appropriate delays and caching. Duplicating this work wastes CPU cycles and can cause noticeable lag.

## Best Practice 4: Minimize Network Traffic

Only network data that clients actually need. Excessive networking causes lag and bandwidth issues.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- Only network essential client-visible state
        self:SetNW2Int("CustomPhase", 1)

        -- Keep server-only data local
        self._internalCooldowns = {}
        self._pathingData = {}
        self._targetPriorities = {}
    end

    function ENT:SetPhase(phase)
        -- Network only when changed
        if self:GetNW2Int("CustomPhase") ~= phase then
            self:SetNW2Int("CustomPhase", phase)
        end
    end

    function ENT:OnHealthChange(old, new)
        -- Don't network health percentage every change
        -- DrGBase already networks health appropriately
    end
end

if CLIENT then
    function ENT:CustomDraw()
        -- Read networked data on client
        local phase = self:GetNW2Int("CustomPhase", 1)

        if phase >= 2 then
            -- Draw special effects based on phase
        end
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- BAD: Networking everything unnecessarily
        self:SetNW2Int("InternalCooldown1", 0)
        self:SetNW2Int("InternalCooldown2", 0)
        self:SetNW2Int("InternalCooldown3", 0)
        self:SetNW2Float("PathingProgress", 0)
        self:SetNW2Vector("InternalTargetPos", Vector())
        self:SetNW2String("DebugState", "idle")
    end

    function ENT:CustomThink()
        -- BAD: Updating networked vars every think
        self:SetNW2Float("PathingProgress", self:GetPathProgress())
        self:SetNW2String("DebugState", self:GetCurrentState())

        -- BAD: Networking large data structures
        self:SetNW2String("AllCooldowns", util.TableToJSON(self._cooldowns))

        return 0.1
    end
end
```

### Why This Matters

Every SetNW2* call sends data to all clients. With 20 NPCs each networking 10 variables every 0.1 seconds, you're sending 2000 network messages per second. This wastes bandwidth and causes lag, especially on servers with many players.

## Best Practice 5: Optimize Pathfinding

Use pathfinding efficiently and avoid recalculating paths unnecessarily.

### ✅ DO

```lua
if SERVER then
    function ENT:OnReachedPatrol(pos, patrol)
        -- Wait between patrol points to reduce pathfinding load
        self:Wait(math.random(3, 7))
    end

    function ENT:OnChaseEnemy(enemy)
        -- Don't recalculate path if enemy hasn't moved much
        if self._lastEnemyPos and self._lastEnemyPos:Distance(enemy:GetPos()) < 200 then
            return  -- Keep using current path
        end

        self._lastEnemyPos = enemy:GetPos()
        -- FollowPath is called by DrGBase - we just validate we need a new one
    end

    function ENT:OnIdle()
        -- Spread out patrol point additions over time
        if not self._nextIdlePatrol or CurTime() > self._nextIdlePatrol then
            self._nextIdlePatrol = CurTime() + math.random(5, 10)
            self:AddPatrolPos(self:RandomPos(1500))
        end
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomThink()
        -- BAD: Constantly recalculating paths
        if self:HasEnemy() then
            self:FollowPath(self:GetEnemy())  -- Every 0.1 seconds!
        else
            -- BAD: Random patrol every think
            self:FollowPath(self:RandomPos(2000))
        end

        return 0.1
    end

    function ENT:OnChaseEnemy(enemy)
        -- BAD: Recalculating even when enemy barely moved
        self:FollowPath(enemy:GetPos())  -- Expensive!
        return true
    end

    function ENT:OnIdle()
        -- BAD: Constant patrol point spam
        self:AddPatrolPos(self:RandomPos(1500))
        self:AddPatrolPos(self:RandomPos(1500))
        self:AddPatrolPos(self:RandomPos(1500))
    end
end
```

### Why This Matters

Pathfinding is one of the most expensive operations in Source engine. Calculating a path can take several milliseconds. With many NPCs recalculating paths constantly, this quickly becomes the primary source of lag.

## Best Practice 6: Use Local Variables in Loops

Store frequently accessed properties and methods in local variables to improve performance.

### ✅ DO

```lua
if SERVER then
    function ENT:ProcessNearbyEntities()
        -- Cache frequently used methods and properties
        local myPos = self:GetPos()
        local myFaction = self.Factions[1]

        local nearbyEnts = ents.FindInSphere(myPos, 1000)

        -- Process in batches to spread load
        local processed = 0
        local maxPerFrame = 10

        for i, ent in ipairs(nearbyEnts) do
            if processed >= maxPerFrame then
                break  -- Don't process everything in one frame
            end

            -- Use local variables instead of repeated method calls
            if ent.IsDrGNextbot then
                local entPos = ent:GetPos()
                local distance = myPos:Distance(entPos)

                if distance < 500 then
                    self:ReactToNearbyNPC(ent, distance)
                    processed = processed + 1
                end
            end
        end
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:ProcessNearbyEntities()
        -- BAD: Repeatedly calling the same methods
        local nearbyEnts = ents.FindInSphere(self:GetPos(), 1000)

        for i, ent in ipairs(nearbyEnts) do
            if ent.IsDrGNextbot then
                -- BAD: Calculating distance multiple times
                if self:GetPos():Distance(ent:GetPos()) < 500 then
                    -- Calculated again!
                    local dist = self:GetPos():Distance(ent:GetPos())

                    -- And again!
                    if self:GetPos():Distance(ent:GetPos()) < 300 then
                        self:ReactToNearbyNPC(ent, dist)
                    end
                end
            end
        end

        -- No batch limiting - processes all entities every frame
    end
end
```

### Why This Matters

Method calls have overhead. In a loop processing 100 entities, calling self:GetPos() 300 times wastes CPU cycles. Local variables are faster to access and make code more readable.

## Best Practice 7: Clean Up Timers and Hooks

Remove timers and hooks when entities are removed to prevent memory leaks and errors.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        local uniqueID = "CustomAbility_" .. self:EntIndex()

        -- Use entity-specific timer names
        timer.Create(uniqueID, 5, 0, function()
            if not IsValid(self) then
                timer.Remove(uniqueID)
                return
            end
            self:UseSpecialAbility()
        end)

        -- Clean up on remove
        self:CallOnRemove("CleanupCustomTimers", function()
            timer.Remove(uniqueID)
        end)
    end

    -- Use ENT:Timer() for automatic cleanup
    function ENT:StartRegeneration()
        self:Timer(1, function()
            if self:Health() < self:GetMaxHealth() then
                self:SetHealth(self:Health() + 5)
                self:StartRegeneration()  -- Restart timer
            end
        end)
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- BAD: Generic timer names cause conflicts
        timer.Create("CustomAbility", 5, 0, function()
            -- What if multiple NPCs create this timer?
            self:UseSpecialAbility()  -- self might be invalid!
        end)

        -- BAD: No cleanup - timer runs forever even after removal
    end

    function ENT:StartRegeneration()
        -- BAD: No validation, no cleanup
        timer.Create("Regen", 1, 0, function()
            self:SetHealth(self:Health() + 5)  -- Error if self is removed
        end)
    end
end
```

### Why This Matters

Timers that aren't cleaned up continue running after the entity is removed, causing errors and memory leaks. With multiple NPCs spawning and dying, orphaned timers accumulate and degrade performance.

## Best Practice 8: Optimize Animation and Model Updates

Avoid unnecessary animation updates and model manipulations.

### ✅ DO

```lua
-- Use appropriate animation rate
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1
ENT.IdleAnimRate = 1

if SERVER then
    function ENT:CustomInitialize()
        -- Set bone manipulations once
        if self:LookupBone("ValveBiped.Bip01_Head1") then
            self:ManipulateBoneScale(self:LookupBone("ValveBiped.Bip01_Head1"), Vector(1.2, 1.2, 1.2))
        end
    end

    function ENT:OnMeleeAttack(enemy)
        -- Play animation only when needed
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end
end

if CLIENT then
    function ENT:CustomDraw()
        -- Only manipulate rendering when actually needed
        local phase = self:GetNW2Int("CustomPhase", 1)

        if phase >= 2 then
            -- Apply effect only in specific phases
            render.SetColorModulation(1, 0.5, 0.5)
            self:DrawModel()
            render.SetColorModulation(1, 1, 1)
        end
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomThink()
        -- BAD: Constantly manipulating bones
        if self:LookupBone("ValveBiped.Bip01_Head1") then
            self:ManipulateBoneScale(self:LookupBone("ValveBiped.Bip01_Head1"), Vector(1.2, 1.2, 1.2))
        end

        -- BAD: Setting pose parameters every think
        self:SetPoseParameter("aim_yaw", math.random(-180, 180))
        self:SetPoseParameter("aim_pitch", math.random(-90, 90))

        return 0.1
    end
end

if CLIENT then
    function ENT:Draw()
        -- BAD: Complex rendering every frame for every NPC
        for i = 1, 10 do
            render.SetColorModulation(math.random(), math.random(), math.random())
            self:DrawModel()
        end

        -- BAD: Looking up bones every frame
        local headBone = self:LookupBone("ValveBiped.Bip01_Head1")
        local spineBone = self:LookupBone("ValveBiped.Bip01_Spine")
        -- More expensive lookups...
    end
end
```

### Why This Matters

Bone lookups and manipulations are expensive, especially when done every frame for multiple NPCs. Rendering calls have significant overhead. Minimizing these operations improves frame rate noticeably.

## Performance Testing and Profiling

### Testing with Multiple NPCs

Always test your NPCs in realistic conditions:

```lua
-- Create a test map with stress testing
concommand.Add("test_npc_performance", function()
    -- Spawn multiple NPCs to test performance
    for i = 1, 20 do
        local npc = ents.Create("npc_mymod_soldier")
        npc:SetPos(Vector(i * 100, 0, 0))
        npc:Spawn()
    end

    print("Spawned 20 NPCs for performance testing")
end)
```

### Using the Performance Profiler

```lua
if SERVER then
    function ENT:CustomThink()
        if GetConVar("developer"):GetInt() >= 2 then
            local startTime = SysTime()

            -- Your expensive code here
            self:ProcessComplexLogic()

            local endTime = SysTime()
            local duration = (endTime - startTime) * 1000  -- Convert to ms

            if duration > 1 then  -- Warn if over 1ms
                print(string.format("[%s] CustomThink took %.2fms", self:GetClass(), duration))
            end
        end

        return 0.5
    end
end
```

## Tips and Recommendations

1. **Profile First**: Don't optimize blindly. Use developer 2 and measure actual performance bottlenecks.

2. **Test with Quantity**: Always test with 10-20 NPCs spawned simultaneously. Performance issues multiply with scale.

3. **Monitor Server FPS**: Keep an eye on sv_fps. It should stay above 60. Anything below 40 indicates serious performance issues.

4. **Use Source's Developer Console**: Enable developer 1 and watch for warnings about expensive operations.

5. **Batch Operations**: When processing multiple entities, consider spreading the work across multiple frames.

6. **Consider LOD**: For visual effects, reduce complexity for distant NPCs that players can barely see.

7. **Network Wisely**: Remember that networking has both CPU and bandwidth costs. Only network what's necessary.

8. **Avoid String Operations in Loops**: String concatenation and formatting in hot paths adds up quickly.

## Real-World Example: Optimized Boss NPC

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Optimized Boss"
ENT.SpawnHealth = 5000

-- Efficient detection settings
ENT.SightRange = 3000
ENT.SightFOV = 120

-- Configuration separate from logic
ENT.Abilities = {
    slam = {damage = 100, radius = 300, cooldown = 8},
    roar = {radius = 500, duration = 3, cooldown = 15}
}

if SERVER then
    function ENT:CustomInitialize()
        -- Initialize with reasonable defaults
        self._abilityCooldowns = {}
        self._nearbyEnemies = {}
        self._cacheTime = 0

        -- Use ENT:Timer for automatic cleanup
        self:Timer(5, function()
            self:CheckForMinions()
        end)
    end

    function ENT:CustomThink()
        -- Update enemy cache every 1 second
        if CurTime() > self._cacheTime then
            self._cacheTime = CurTime() + 1
            self:UpdateEnemyCache()
        end

        -- Use cached data
        if #self._nearbyEnemies >= 3 then
            self:UseAbility("slam")
        end

        return 0.5  -- Don't run every tick
    end

    function ENT:UpdateEnemyCache()
        -- Cache expensive lookup
        self._nearbyEnemies = {}
        local myPos = self:GetPos()

        for _, ent in ipairs(ents.FindInSphere(myPos, 500)) do
            if IsValid(ent) and ent:IsPlayer() or ent:IsNPC() then
                if self:GetRelationship(ent) == D_HT then
                    table.insert(self._nearbyEnemies, ent)
                end
            end
        end
    end

    function ENT:UseAbility(name)
        -- Check cooldown (cached in table)
        if self._abilityCooldowns[name] and CurTime() < self._abilityCooldowns[name] then
            return false
        end

        local config = self.Abilities[name]
        if not config then return false end

        -- Execute ability
        self:EmitSound("boss/ability_" .. name .. ".wav")
        -- Ability effects...

        -- Set cooldown
        self._abilityCooldowns[name] = CurTime() + config.cooldown
        return true
    end

    function ENT:CheckForMinions()
        if not IsValid(self) then return end

        -- Efficient counting
        local minionCount = 0
        for _, ent in ipairs(ents.FindByClass("npc_mymod_minion")) do
            if IsValid(ent) then
                minionCount = minionCount + 1
            end
        end

        if minionCount < 3 then
            self:SpawnMinion()
        end

        -- Restart timer
        self:Timer(5, function()
            self:CheckForMinions()
        end)
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

This optimized boss uses caching, appropriate think delays, efficient data structures, and DrGBase's timer system to minimize performance impact while maintaining rich behavior.
