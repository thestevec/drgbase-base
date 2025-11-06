# Common Pitfalls

This guide covers frequent mistakes DrGBase developers make and how to avoid them. Learning from these common pitfalls will save you hours of debugging and frustration.

## Pitfall 1: Forgetting the DrGBase Check

**Problem**: Entity code runs when DrGBase isn't installed, causing errors.

### ❌ Problem Code

```lua
-- No check if DrGBase exists
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "My NPC"
ENT.SpawnHealth = 100

-- Error: DrGBase not found!
-- ENT.Base doesn't exist, addon breaks
```

### ✅ Solution

```lua
-- Always check if DrGBase is loaded
if not DrGBase then return end

ENT.Base = "drgbase_nextbot"

ENT.PrintName = "My NPC"
ENT.SpawnHealth = 100

-- Now gracefully handles missing DrGBase
```

### Why This Happens

Players may not have DrGBase installed, or it may fail to load. Without the check, your entire addon breaks with cryptic errors.

### How to Avoid

Always add `if not DrGBase then return end` at the very top of every DrGBase entity file.

## Pitfall 2: Setting Properties in Initialize

**Problem**: Properties set in CustomInitialize() don't work because DrGBase reads them before CustomInitialize() runs.

### ❌ Problem Code

```lua
ENT.Base = "drgbase_nextbot"

if SERVER then
    function ENT:CustomInitialize()
        -- TOO LATE! DrGBase already read these properties
        self.SpawnHealth = 100
        self.WalkSpeed = 150
        self.Models = {"models/player.mdl"}
    end
end

-- NPC spawns with default values, not custom ones
```

### ✅ Solution

```lua
ENT.Base = "drgbase_nextbot"

-- Set properties at class level
ENT.SpawnHealth = 100
ENT.WalkSpeed = 150
ENT.Models = {"models/player.mdl"}

if SERVER then
    function ENT:CustomInitialize()
        -- Use CustomInitialize for runtime configuration
        self:SetDefaultRelationship(D_HT)

        -- Or conditional initialization
        if math.random(2) == 1 then
            self:SetModelScale(1.5)
        end
    end
end
```

### Why This Happens

DrGBase's base Initialize() function runs before CustomInitialize() and reads entity properties to configure the NPC. Setting them too late means they're ignored.

### How to Avoid

Set all ENT properties at the class level (outside any functions). Use CustomInitialize() only for conditional or runtime-specific initialization.

## Pitfall 3: Not Including AddCSLuaFile()

**Problem**: Entity doesn't show up on clients or causes missing model errors in multiplayer.

### ❌ Problem Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "My NPC"
ENT.Models = {"models/player.mdl"}

-- Missing AddCSLuaFile()!
DrGBase.AddNextbot(ENT)

-- Clients can't see the entity properly
```

### ✅ Solution

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "My NPC"
ENT.Models = {"models/player.mdl"}

-- ALWAYS include this
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### Why This Happens

AddCSLuaFile() tells the server to send the entity's code to clients. Without it, clients don't receive the entity definition and can't render it properly.

### How to Avoid

Always include `AddCSLuaFile()` before `DrGBase.AddNextbot(ENT)` at the end of every entity file. Consider it mandatory boilerplate.

## Pitfall 4: Using CLIENT/SERVER Checks Incorrectly

**Problem**: Code runs on wrong realm or doesn't run at all.

### ❌ Problem Code

```lua
ENT.Base = "drgbase_nextbot"

-- Shared properties are fine
ENT.PrintName = "My NPC"

-- BAD: Server function with no SERVER check
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)  -- Errors on client!
end

-- BAD: CLIENT code but no CLIENT check
function ENT:CustomDraw()
    -- Doesn't run because no CLIENT guard
    self:DrawModel()
end
```

### ✅ Solution

```lua
ENT.Base = "drgbase_nextbot"

-- Shared properties
ENT.PrintName = "My NPC"
ENT.SpawnHealth = 100

-- Server-only code
if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    function ENT:OnMeleeAttack(enemy)
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end
end

-- Client-only code
if CLIENT then
    function ENT:CustomDraw()
        self:DrawModel()

        -- Client-specific rendering
        local phase = self:GetNW2Int("CustomPhase", 1)
        if phase >= 2 then
            -- Draw special effects
        end
    end
end

-- Shared code (no guards needed)
function ENT:GetDisplayName()
    return self.PrintName
end
```

### Why This Happens

Garry's Mod runs code on both server and client. Without realm checks, code meant for one realm runs on both, causing errors or unexpected behavior.

### How to Avoid

Always wrap realm-specific code in `if SERVER then` or `if CLIENT then` blocks. When in doubt, check if the function uses server-only or client-only APIs.

## Pitfall 5: Infinite Loops in Think Functions

**Problem**: Think function never returns, freezing the NPC or server.

### ❌ Problem Code

```lua
if SERVER then
    function ENT:CustomThink()
        -- BAD: Infinite loop
        while self:HasEnemy() do
            local enemy = self:GetEnemy()

            if not IsValid(enemy) then
                break
            end

            -- Never yields, freezes server!
            self:AttackEnemy(enemy)
        end

        return 0.1
    end
end
```

### ✅ Solution

```lua
if SERVER then
    function ENT:CustomThink()
        -- Think functions should be quick, single-frame operations
        if self:HasEnemy() then
            local enemy = self:GetEnemy()

            if IsValid(enemy) then
                -- Do one frame's worth of work
                if self:IsInRange(enemy, self.MeleeAttackRange) then
                    self:AttackEnemy(enemy)
                end
            end
        end

        -- Return controls how often this runs
        return 0.5  -- Run every 0.5 seconds
    end

    -- For complex sequences, use coroutines in hooks like OnMeleeAttack
    function ENT:OnMeleeAttack(enemy)
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
        -- DrGBase handles the coroutine yielding
    end
end
```

### Why This Happens

Think functions are called repeatedly by the game engine. Loops that don't yield will freeze execution until they complete, hanging the server.

### How to Avoid

Never use loops in Think functions. Do one frame's worth of work and return. For sequences that take multiple frames, use DrGBase's coroutine system via hooks like OnMeleeAttack.

## Pitfall 6: Not Cleaning Up Timers

**Problem**: Timers continue running after entity is removed, causing errors and memory leaks.

### ❌ Problem Code

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- BAD: Generic timer name
        timer.Create("AbilityTimer", 5, 0, function()
            self:UseAbility()  -- Error if entity removed!
        end)

        -- No cleanup when entity removed
    end
end
```

### ✅ Solution

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- GOOD: Use ENT:Timer() for automatic cleanup
        self:Timer(5, function()
            self:UseAbility()
        end, true)  -- true = repeat

        -- Or use unique timer names with cleanup
        local timerID = "Ability_" .. self:EntIndex()

        timer.Create(timerID, 5, 0, function()
            if not IsValid(self) then
                timer.Remove(timerID)
                return
            end

            self:UseAbility()
        end)

        -- Clean up when removed
        self:CallOnRemove("CleanupAbilityTimer", function()
            timer.Remove(timerID)
        end)
    end
end
```

### Why This Happens

Regular timers persist even after the entity that created them is removed. They continue trying to call methods on invalid entities, causing errors.

### How to Avoid

Use DrGBase's `ENT:Timer()` function which automatically cleans up. If using regular timers, always use unique names and clean them up in CallOnRemove().

## Pitfall 7: Excessive Network Traffic

**Problem**: Too much data networked to clients causes lag.

### ❌ Problem Code

```lua
if SERVER then
    function ENT:CustomThink()
        -- BAD: Networking large amounts of data every think
        self:SetNW2String("PathData", util.TableToJSON(self._pathPoints))
        self:SetNW2String("TargetHistory", util.TableToJSON(self._targets))
        self:SetNW2Float("CurrentSpeed", self:GetVelocity():Length())
        self:SetNW2Vector("LookTarget", self:GetLookTarget())

        -- BAD: Updating even when unchanged
        self:SetNW2Int("Phase", self:CalculatePhase())

        return 0.1  -- 10 times per second!
    end
end
```

### ✅ Solution

```lua
if SERVER then
    function ENT:CustomInitialize()
        self._lastNetworkedPhase = 1
    end

    function ENT:CustomThink()
        -- GOOD: Only network when changed
        local phase = self:CalculatePhase()
        if phase ~= self._lastNetworkedPhase then
            self:SetNW2Int("Phase", phase)
            self._lastNetworkedPhase = phase
        end

        -- GOOD: Don't network server-only data
        -- Keep path data, targets, etc. server-side

        return 0.5  -- Less frequent updates
    end

    function ENT:SetPhase(phase)
        if self:GetNW2Int("Phase") ~= phase then
            self:SetNW2Int("Phase", phase)

            -- Send one-time event for visual effects
            net.Start("NPCPhaseChange")
            net.WriteEntity(self)
            net.WriteUInt(phase, 8)
            net.Broadcast()
        end
    end
end
```

### Why This Matters

Every SetNW2* call sends a network message to all clients. Excessive networking wastes bandwidth and causes lag for all players.

### How to Avoid

Only network what clients need to see. Check if values changed before networking. Use net messages for one-time events instead of constantly updating NW2 vars.

## Pitfall 8: Missing Navmesh Handling

**Problem**: NPC breaks or throws errors when no navmesh is loaded.

### ❌ Problem Code

```lua
if SERVER then
    function ENT:OnIdle()
        -- Assumes navmesh always exists
        local randomPos = self:RandomPos(1000)
        self:FollowPath(randomPos)  -- Crashes if no navmesh!
    end
end
```

### ✅ Solution

```lua
if SERVER then
    function ENT:OnIdle()
        -- Check if navmesh exists
        if not navmesh.IsLoaded() then
            -- Graceful fallback behavior
            return
        end

        local randomPos = self:RandomPos(1000)
        if randomPos then
            self:FollowPath(randomPos)
        end
    end

    function ENT:OnEnemyUnreachable(enemy)
        if not navmesh.IsLoaded() then
            -- Can't path without navmesh - try direct approach
            self:MoveTowards(enemy:GetPos())
            return
        end

        -- Handle unreachable enemy
        self:OnLostEnemy(enemy)
    end
end
```

### Why This Happens

Not all maps have navmeshes, and some maps have incomplete or broken navmeshes. Assuming navmesh exists causes crashes.

### How to Avoid

Always check `navmesh.IsLoaded()` before using pathfinding. Implement fallback behavior for maps without navmeshes.

## Pitfall 9: Hardcoding Values Instead of Configuration

**Problem**: Values scattered throughout code make balancing difficult.

### ❌ Problem Code

```lua
if SERVER then
    function ENT:UseFireball()
        if self:GetRangeTo(self:GetEnemy()) > 1000 then
            return false
        end

        self:DealDamage(self:GetEnemy(), 50, 200)
        self._fireballCooldown = CurTime() + 5
    end

    function ENT:UseLightning()
        if self:GetRangeTo(self:GetEnemy()) > 800 then
            return false
        end

        self:ChainLightning(self:GetEnemy(), 30, 3)
        self._lightningCooldown = CurTime() + 8
    end
end
```

### ✅ Solution

```lua
ENT.Base = "drgbase_nextbot"

-- Configuration at the top
ENT.Abilities = {
    fireball = {
        range = 1000,
        damage = 50,
        radius = 200,
        cooldown = 5
    },
    lightning = {
        range = 800,
        damage = 30,
        chains = 3,
        cooldown = 8
    }
}

if SERVER then
    function ENT:UseAbility(name)
        local config = self.Abilities[name]
        if not config then return false end

        local enemy = self:GetEnemy()

        if self:GetRangeTo(enemy) > config.range then
            return false
        end

        if name == "fireball" then
            self:DealDamage(enemy, config.damage, config.radius)
        elseif name == "lightning" then
            self:ChainLightning(enemy, config.damage, config.chains)
        end

        self._cooldowns[name] = CurTime() + config.cooldown
        return true
    end
end
```

### Why This Happens

Developers often hardcode values during initial development and forget to refactor them into configuration.

### How to Avoid

Define configuration values at the top of the file in tables. Keep implementation separate from configuration for easy balancing.

## Pitfall 10: Not Testing in Multiplayer

**Problem**: Addon works in singleplayer but breaks in multiplayer.

### Common Issues

```lua
-- Works in singleplayer, breaks in multiplayer:

-- 1. Using LocalPlayer() on server
if SERVER then
    function ENT:OnSpawn()
        local ply = LocalPlayer()  -- ERROR: Server has no LocalPlayer()
        self:SetEnemy(ply)
    end
end

-- 2. Assuming immediate network sync
if SERVER then
    function ENT:SetPhase(phase)
        self:SetNW2Int("Phase", phase)
    end
end

if CLIENT then
    -- This might not see the phase change immediately!
    function ENT:Initialize()
        local phase = self:GetNW2Int("Phase", 1)
        -- Might be 0 or default value due to network delay
    end
end

-- 3. Not testing with latency
-- Code that works on localhost may break with 50-100ms ping
```

### Solutions

```lua
-- 1. Don't use LocalPlayer() on server
if SERVER then
    function ENT:OnSpawn()
        -- Find players a different way
        for _, ply in ipairs(player.GetAll()) do
            if self:ShouldTargetPlayer(ply) then
                self:SetEnemy(ply)
                break
            end
        end
    end
end

-- 2. Handle network delays
if CLIENT then
    function ENT:Initialize()
        -- Wait for network sync
        timer.Simple(0.1, function()
            if not IsValid(self) then return end
            local phase = self:GetNW2Int("Phase", 1)
            self:OnPhaseReceived(phase)
        end)
    end
end

-- 3. Test on dedicated server
-- Always test with actual network latency
```

### How to Avoid

Always test on a dedicated server with multiple clients. Test with simulated latency (`net_fakelag` console command).

## Tips for Avoiding Pitfalls

1. **Read Error Messages**: Lua errors tell you exactly what's wrong. Read them carefully.

2. **Use Developer Console**: Enable `developer 1` to see warnings and additional information.

3. **Check the Wiki**: DrGBase and GMod wikis document correct usage of functions.

4. **Look at Examples**: Study the example NPCs included with DrGBase (zombie, headcrab).

5. **Test Incrementally**: Test after each change, not after implementing everything.

6. **Use Version Control**: Commit working code before making changes so you can revert.

7. **Ask for Help**: DrGBase community can help if you're stuck.

8. **Keep It Simple**: Start simple and add complexity gradually.

9. **Read Other Addons**: See how established addons solve common problems.

10. **Stay Updated**: Follow DrGBase updates as APIs and best practices evolve.

## Debugging Checklist

When something doesn't work, check these common issues:

- [ ] Is DrGBase installed and loaded?
- [ ] Did you include `if not DrGBase then return end`?
- [ ] Did you include `AddCSLuaFile()`?
- [ ] Are properties set at class level, not in Initialize?
- [ ] Are SERVER/CLIENT guards in the right places?
- [ ] Are you using the correct realm for the function?
- [ ] Did you check if navmesh is loaded before pathfinding?
- [ ] Are timer names unique per entity?
- [ ] Are you cleaning up timers on remove?
- [ ] Did you test in multiplayer?
- [ ] Is `developer 1` enabled to see warnings?
- [ ] Have you checked the console for errors?

## Real-World Example: Mistake-Free NPC

```lua
-- ✅ All common pitfalls avoided

-- Check DrGBase exists
if not DrGBase then return end

-- Set base
ENT.Base = "drgbase_nextbot"

-- Configuration at class level
ENT.PrintName = "Proper NPC"
ENT.Category = "My Addon"
ENT.SpawnHealth = 100
ENT.Models = {"models/player.mdl"}

ENT.Abilities = {
    fireball = {damage = 50, cooldown = 5, range = 1000}
}

-- Server-only code
if SERVER then
    function ENT:CustomInitialize()
        -- Runtime initialization only
        self:SetDefaultRelationship(D_HT)

        -- Use ENT:Timer for automatic cleanup
        self:Timer(5, function()
            self:UseAbility("fireball")
        end, true)
    end

    function ENT:CustomThink()
        -- Quick, single-frame operations only
        if self:HasEnemy() then
            self:FaceTowards(self:GetEnemy())
        end

        -- Return delay to control frequency
        return 0.5
    end

    function ENT:OnIdle()
        -- Check navmesh exists
        if not navmesh.IsLoaded() then
            return
        end

        local pos = self:RandomPos(1000)
        if pos then
            self:AddPatrolPos(pos)
        end
    end

    function ENT:UseAbility(name)
        local config = self.Abilities[name]
        if not config then return false end

        -- Ability logic using configuration
        local enemy = self:GetEnemy()
        if not IsValid(enemy) then return false end

        if self:GetRangeTo(enemy) > config.range then
            return false
        end

        self:DealDamage(enemy, config.damage)
        return true
    end
end

-- Client-only code
if CLIENT then
    function ENT:CustomDraw()
        -- Client rendering
        self:DrawModel()
    end
end

-- Required boilerplate
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

This example avoids all common pitfalls and follows all best practices outlined in this guide.
