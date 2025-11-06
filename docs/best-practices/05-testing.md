# Testing Strategies

Testing is essential for creating reliable DrGBase addons. Proper testing catches bugs early, ensures NPCs behave correctly in various scenarios, and prevents regressions when making changes. This guide covers effective testing strategies for DrGBase development.

## Best Practice 1: Create Test Commands for Development

Build console commands to quickly spawn and test your NPCs in various configurations.

### ✅ DO

```lua
if SERVER then
    -- Test command for spawning NPC
    concommand.Add("mymod_test_spawn", function(ply, cmd, args)
        if not IsValid(ply) then return end

        -- Get player's aim position
        local tr = ply:GetEyeTrace()

        -- Spawn NPC
        local npc = ents.Create("npc_mymod_soldier")
        npc:SetPos(tr.HitPos + Vector(0, 0, 10))
        npc:SetAngles(Angle(0, ply:EyeAngles().y + 180, 0))
        npc:Spawn()

        -- Set creator for cleanup
        npc:SetCreator(ply)
        ply:DrG_AddUndo(npc, "NPC", "Undone Test NPC")

        print(string.format("Spawned %s at %s", npc:GetClass(), tostring(tr.HitPos)))
    end)

    -- Test command for spawning waves
    concommand.Add("mymod_test_wave", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local count = tonumber(args[1]) or 5
        count = math.Clamp(count, 1, 20)

        local tr = ply:GetEyeTrace()
        local pos = tr.HitPos

        for i = 1, count do
            local npc = ents.Create("npc_mymod_soldier")

            -- Arrange in a circle
            local angle = (i / count) * 360
            local rad = math.rad(angle)
            local offset = Vector(math.cos(rad) * 200, math.sin(rad) * 200, 10)

            npc:SetPos(pos + offset)
            npc:SetAngles(Angle(0, angle, 0))
            npc:Spawn()
            npc:SetCreator(ply)

            ply:DrG_AddUndo(npc, "NPCs", "Undone Test Wave")
        end

        print(string.format("Spawned %d NPCs", count))
    end)

    -- Test command for forcing behaviors
    concommand.Add("mymod_test_behavior", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local behavior = args[1] or "idle"
        local npc = ply:GetEyeTrace().Entity

        if not IsValid(npc) or not npc.IsDrGNextbot then
            print("Not looking at a DrGBase NPC")
            return
        end

        if behavior == "idle" then
            npc:SetEnemy(NULL)
            print("NPC set to idle")
        elseif behavior == "combat" then
            npc:SetEnemy(ply)
            print("NPC set to attack player")
        elseif behavior == "patrol" then
            npc:SetEnemy(NULL)
            for i = 1, 5 do
                npc:AddPatrolPos(npc:GetPos() + Vector(math.random(-500, 500), math.random(-500, 500), 0))
            end
            print("NPC set to patrol")
        end
    end)
end
```

### ❌ DON'T

```lua
-- BAD: No test commands, must manually spawn from menu every time
-- BAD: Can't quickly test specific scenarios
-- BAD: Wasting time on repetitive manual testing
```

### Why This Matters

Test commands save enormous amounts of time during development. Instead of clicking through spawn menus, you can instantly spawn NPCs in specific configurations, test edge cases, and iterate quickly.

## Best Practice 2: Test in Realistic Conditions

Always test with multiple NPCs and in multiplayer environments.

### ✅ DO

```lua
if SERVER then
    -- Spawn multiple NPCs for stress testing
    concommand.Add("mymod_stress_test", function(ply, cmd, args)
        if not IsValid(ply) then return end
        if not ply:IsSuperAdmin() then return end

        local count = tonumber(args[1]) or 10
        local tr = ply:GetEyeTrace()
        local spawned = 0

        for i = 1, count do
            local npc = ents.Create("npc_mymod_soldier")

            local offset = Vector(
                math.random(-300, 300),
                math.random(-300, 300),
                10
            )

            npc:SetPos(tr.HitPos + offset)
            npc:SetAngles(Angle(0, math.random(0, 360), 0))
            npc:Spawn()
            npc:SetCreator(ply)

            spawned = spawned + 1

            -- Spawn in batches to avoid lag spike
            if spawned % 5 == 0 then
                coroutine.yield()
            end
        end

        print(string.format("Stress test: Spawned %d NPCs", count))
        print(string.format("Monitor sv_fps and ping for performance issues"))
    end)

    -- Test with different faction setups
    concommand.Add("mymod_test_factions", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local tr = ply:GetEyeTrace()
        local pos = tr.HitPos

        -- Spawn friendly faction
        for i = 1, 3 do
            local npc = ents.Create("npc_mymod_soldier")
            npc:SetPos(pos + Vector(i * 100, 0, 10))
            npc:Spawn()
            npc:SetCreator(ply)
        end

        -- Spawn enemy faction
        for i = 1, 3 do
            local npc = ents.Create("npc_mymod_zombie")
            npc:SetPos(pos + Vector(i * 100, 300, 10))
            npc:Spawn()
            npc:SetCreator(ply)
        end

        print("Faction test: Spawned soldiers vs zombies")
    end)
end
```

### ❌ DON'T

```lua
-- BAD: Only testing with 1-2 NPCs
-- BAD: Only testing in singleplayer
-- BAD: Never testing faction interactions
-- BAD: Not testing performance under load
```

### Why This Matters

NPCs that work fine alone may break with multiple instances. Singleplayer testing misses networking issues. Testing in realistic conditions catches problems before players encounter them.

## Best Practice 3: Use Debug Visualization

Implement debug drawing to visualize NPC behavior and decision-making.

### ✅ DO

```lua
-- Shared debug convar
CreateConVar("mymod_debug", "0", {FCVAR_ARCHIVE, FCVAR_REPLICATED}, "Enable debug visualization")

if SERVER then
    function ENT:CustomInitialize()
        self._debugTargetPos = nil
        self._debugPathPoints = {}
    end

    function ENT:CustomThink()
        -- Track debug data
        if GetConVar("mymod_debug"):GetBool() then
            if self:HasEnemy() then
                self._debugTargetPos = self:GetEnemy():GetPos()
            else
                self._debugTargetPos = nil
            end

            -- Track path
            if self:GetPath() and self:GetPath():IsValid() then
                self._debugPathPoints = {}
                for i = 1, self:GetPath():GetLength() do
                    table.insert(self._debugPathPoints, self:GetPath():GetPositionOnPath(i * 50))
                end
                self:SetNW2String("DebugPath", util.TableToJSON(self._debugPathPoints))
            end
        end

        return 0.5
    end
end

if CLIENT then
    function ENT:CustomDraw()
        if not GetConVar("mymod_debug"):GetBool() then return end
        if not GetConVar("developer"):GetBool() then return end

        local myPos = self:GetPos()

        -- Draw sight range
        local sightColor = self:HasEnemy() and Color(255, 0, 0) or Color(0, 255, 0)
        render.DrawWireframeSphere(myPos, self.SightRange, 20, 20, sightColor, false)

        -- Draw attack range
        if self:HasEnemy() then
            render.DrawWireframeSphere(myPos, self.MeleeAttackRange, 10, 10, Color(255, 255, 0), false)
        end

        -- Draw target line
        if self:HasEnemy() then
            local enemy = self:GetEnemy()
            if IsValid(enemy) then
                render.DrawLine(myPos + Vector(0, 0, 50), enemy:GetPos() + Vector(0, 0, 50), Color(255, 0, 0), false)
            end
        end

        -- Draw path
        local pathJSON = self:GetNW2String("DebugPath", "")
        if pathJSON ~= "" then
            local pathPoints = util.JSONToTable(pathJSON)
            if pathPoints then
                for i = 1, #pathPoints - 1 do
                    render.DrawLine(pathPoints[i], pathPoints[i + 1], Color(0, 255, 255), false)
                end
            end
        end

        -- Draw state text
        local stateText = "State: " .. (self:HasEnemy() and "Combat" or "Idle")
        if self:GetEnemy() and IsValid(self:GetEnemy()) then
            stateText = stateText .. "\nTarget: " .. self:GetEnemy():GetClass()
        end

        local pos = myPos + Vector(0, 0, 80)
        local ang = LocalPlayer():EyeAngles()
        ang:RotateAroundAxis(ang:Forward(), 90)
        ang:RotateAroundAxis(ang:Right(), 90)

        cam.Start3D2D(pos, ang, 0.1)
            draw.SimpleText(stateText, "DermaDefault", 0, 0, Color(255, 255, 255), TEXT_ALIGN_CENTER)
        cam.End3D2D()
    end
end
```

### ❌ DON'T

```lua
-- BAD: No debug visualization
-- BAD: Can't see what NPC is thinking
-- BAD: Can't see why pathing fails
-- BAD: Guessing at why behavior is wrong
```

### Why This Matters

Debug visualization makes invisible behavior visible. You can see detection ranges, paths, target selection, and state changes in real-time, making debugging exponentially faster.

## Best Practice 4: Log Important Events

Add logging for debugging complex behaviors.

### ✅ DO

```lua
-- Debug logging helper
local function DebugLog(npc, category, message)
    if not GetConVar("developer"):GetBool() then return end
    if not GetConVar("mymod_debug"):GetBool() then return end

    local prefix = string.format("[%s][%s]", npc:GetClass(), category)
    print(prefix, message)
end

if SERVER then
    function ENT:OnNewEnemy(enemy)
        DebugLog(self, "AI", string.format("New enemy: %s at distance %.0f", enemy:GetClass(), self:GetRangeTo(enemy)))

        -- Call original behavior
        self:EmitSound("npc/soldier/soldier_alert.wav")
    end

    function ENT:OnEnemyUnreachable(enemy)
        DebugLog(self, "Path", string.format("Enemy unreachable: %s", enemy:GetClass()))

        -- Log why pathing failed
        if not navmesh.IsLoaded() then
            DebugLog(self, "Path", "No navmesh loaded!")
        end
    end

    function ENT:OnMeleeAttack(enemy)
        DebugLog(self, "Combat", string.format("Melee attack on %s", enemy:GetClass()))

        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnTakeDamage(dmg)
        if GetConVar("mymod_debug"):GetBool() then
            local attacker = dmg:GetAttacker()
            local damage = dmg:GetDamage()
            local damageType = dmg:GetDamageType()

            DebugLog(self, "Damage", string.format(
                "Took %.1f damage from %s (Type: %d, Health: %d/%d)",
                damage,
                IsValid(attacker) and attacker:GetClass() or "unknown",
                damageType,
                self:Health(),
                self:GetMaxHealth()
            ))
        end
    end

    function ENT:OnPatrolUnreachable(pos, patrol)
        DebugLog(self, "Patrol", string.format("Patrol point unreachable at %s", tostring(pos)))
    end
end
```

### ❌ DON'T

```lua
-- BAD: No logging
-- BAD: Can't trace what happened
-- BAD: Can't debug complex sequences
-- BAD: No visibility into AI decisions
```

### Why This Matters

Logs provide a timeline of what happened, essential for debugging complex AI sequences, understanding why NPCs made certain decisions, and tracking down intermittent bugs.

## Best Practice 5: Test Edge Cases

Deliberately test unusual scenarios that players might encounter.

### ✅ DO

```lua
if SERVER then
    -- Test NPC with no navmesh
    concommand.Add("mymod_test_no_navmesh", function(ply, cmd, args)
        if not IsValid(ply) then return end

        if navmesh.IsLoaded() then
            print("WARNING: Navmesh is loaded. NPCs should handle missing navmesh gracefully.")
        else
            print("Testing with no navmesh (expected scenario)")
        end

        local npc = ents.Create("npc_mymod_soldier")
        npc:SetPos(ply:GetEyeTrace().HitPos)
        npc:Spawn()

        print("Watch for errors or crashes")
    end)

    -- Test NPC stuck in wall
    concommand.Add("mymod_test_stuck", function(ply, cmd, args)
        if not IsValid(ply) then return end

        -- Spawn inside a wall deliberately
        local npc = ents.Create("npc_mymod_soldier")
        npc:SetPos(ply:GetPos())  -- Spawn at player position (likely invalid)
        npc:Spawn()

        print("Testing stuck detection - NPC may be stuck in geometry")
        print("NPC should detect and handle being stuck")
    end)

    -- Test with extreme health values
    concommand.Add("mymod_test_health", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local npc = ply:GetEyeTrace().Entity
        if not IsValid(npc) or not npc.IsDrGNextbot then
            print("Not looking at NPC")
            return
        end

        local testType = args[1] or "zero"

        if testType == "zero" then
            npc:SetHealth(0)
            print("Set health to 0")
        elseif testType == "negative" then
            npc:SetHealth(-100)
            print("Set health to -100")
        elseif testType == "huge" then
            npc:SetHealth(999999)
            print("Set health to 999999")
        end

        print("Testing health edge case - check for errors")
    end)

    -- Test rapid spawning/removal
    concommand.Add("mymod_test_rapid_spawn", function(ply, cmd, args)
        if not IsValid(ply) then return end
        if not ply:IsSuperAdmin() then return end

        local pos = ply:GetEyeTrace().HitPos

        for i = 1, 100 do
            local npc = ents.Create("npc_mymod_soldier")
            npc:SetPos(pos)
            npc:Spawn()

            timer.Simple(0.1, function()
                if IsValid(npc) then
                    npc:Remove()
                end
            end)
        end

        print("Rapid spawn/remove test - check for memory leaks or errors")
    end)
end
```

### ❌ DON'T

```lua
-- BAD: Only testing ideal scenarios
-- BAD: Assuming navmesh always exists
-- BAD: Not testing extreme values
-- BAD: Not testing rapid spawn/remove
```

### Why This Matters

Edge cases expose bugs that normal testing misses. Players will encounter these scenarios (missing navmesh, stuck NPCs, unusual health values), and your addon should handle them gracefully.

## Best Practice 6: Create Test Scenarios

Build complete test scenarios that simulate real gameplay.

### ✅ DO

```lua
if SERVER then
    -- Create a full combat scenario
    concommand.Add("mymod_test_combat_scenario", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local center = ply:GetEyeTrace().HitPos

        print("=== Combat Scenario Test ===")

        -- Spawn defensive team
        print("Spawning defenders...")
        for i = 1, 3 do
            local npc = ents.Create("npc_mymod_soldier")
            npc:SetPos(center + Vector(i * 100, 0, 10))
            npc:SetAngles(Angle(0, 180, 0))
            npc:Spawn()
            npc:SetCreator(ply)
        end

        -- Spawn attacking team after delay
        timer.Simple(2, function()
            print("Spawning attackers...")
            for i = 1, 5 do
                local npc = ents.Create("npc_mymod_zombie")
                npc:SetPos(center + Vector(i * 80, 500, 10))
                npc:SetAngles(Angle(0, 0, 0))
                npc:Spawn()
                npc:SetCreator(ply)
            end

            print("Scenario started - observe NPC behavior")
            print("- Do soldiers detect zombies?")
            print("- Do they engage at correct range?")
            print("- Do they work together?")
            print("- How's the performance?")
        end)
    end)

    -- Test patrol and detection
    concommand.Add("mymod_test_patrol_scenario", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local center = ply:GetEyeTrace().HitPos

        print("=== Patrol Scenario Test ===")

        -- Spawn patrolling NPC
        local npc = ents.Create("npc_mymod_soldier")
        npc:SetPos(center)
        npc:Spawn()
        npc:SetCreator(ply)

        -- Set up patrol route
        local patrolPoints = {
            center + Vector(200, 0, 0),
            center + Vector(200, 200, 0),
            center + Vector(0, 200, 0),
            center
        }

        for _, point in ipairs(patrolPoints) do
            npc:AddPatrolPos(point)
        end

        -- Spawn target that wanders into patrol
        timer.Simple(10, function()
            local target = ents.Create("npc_mymod_zombie")
            target:SetPos(center + Vector(300, 100, 10))
            target:Spawn()
            target:SetCreator(ply)

            print("Target spawned - observe:")
            print("- Does patroller detect target?")
            print("- Does it break patrol to engage?")
            print("- Does it return to patrol after?")
        end)
    end)
end
```

### ❌ DON'T

```lua
-- BAD: Only spawning one NPC at a time
-- BAD: Not testing interactions
-- BAD: No realistic scenarios
-- BAD: Can't verify complete behaviors
```

### Why This Matters

Individual tests verify isolated features, but scenario tests verify that everything works together correctly in realistic situations. They catch integration issues and unexpected interactions.

## Best Practice 7: Validate Network Synchronization

Test that client and server stay synchronized.

### ✅ DO

```lua
if SERVER then
    concommand.Add("mymod_test_network", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local npc = ply:GetEyeTrace().Entity
        if not IsValid(npc) or not npc.IsDrGNextbot then
            print("Not looking at NPC")
            return
        end

        print("=== Network Sync Test ===")
        print("Server view:")
        print(string.format("  Health: %d/%d", npc:Health(), npc:GetMaxHealth()))
        print(string.format("  HasEnemy: %s", tostring(npc:HasEnemy())))
        print(string.format("  Position: %s", tostring(npc:GetPos())))

        if npc.GetPhase then
            print(string.format("  Phase: %d", npc:GetPhase()))
        end

        -- Send client check request
        net.Start("MyModClientStateCheck")
        net.WriteEntity(npc)
        net.Send(ply)
    end)

    util.AddNetworkString("MyModClientStateCheck")
    util.AddNetworkString("MyModClientStateResponse")

    net.Receive("MyModClientStateResponse", function(len, ply)
        local data = net.ReadTable()

        print("\nClient view:")
        for k, v in pairs(data) do
            print(string.format("  %s: %s", k, tostring(v)))
        end

        print("\nCompare server and client values - they should match!")
    end)
end

if CLIENT then
    net.Receive("MyModClientStateCheck", function()
        local npc = net.ReadEntity()

        if not IsValid(npc) then
            print("NPC not valid on client")
            return
        end

        -- Gather client-side state
        local data = {
            Health = npc:Health(),
            MaxHealth = npc:GetMaxHealth(),
            HasEnemy = npc:HasEnemy(),
            Position = tostring(npc:GetPos())
        }

        if npc.GetPhase then
            data.Phase = npc:GetNW2Int("CustomPhase", 1)
        end

        -- Send back to server
        net.Start("MyModClientStateResponse")
        net.WriteTable(data)
        net.SendToServer()
    end)
end
```

### ❌ DON'T

```lua
-- BAD: Assuming client and server always sync
-- BAD: Not testing network edge cases
-- BAD: No verification of networked vars
```

### Why This Matters

Network desync causes confusing bugs where behavior differs between server and clients. Testing synchronization catches these issues before they affect players.

## Best Practice 8: Automate Regression Testing

Create tests that can be run quickly to verify nothing broke after changes.

### ✅ DO

```lua
if SERVER then
    -- Quick smoke test
    concommand.Add("mymod_test_all", function(ply, cmd, args)
        if not IsValid(ply) then return end

        local passedTests = 0
        local failedTests = 0

        local function Test(name, testFunc)
            print(string.format("Running: %s", name))

            local success, err = pcall(testFunc)

            if success then
                print(string.format("  ✓ PASSED"))
                passedTests = passedTests + 1
            else
                print(string.format("  ✗ FAILED: %s", err))
                failedTests = failedTests + 1
            end
        end

        print("=== Running Test Suite ===\n")

        -- Test 1: Basic spawning
        Test("NPC spawns without errors", function()
            local npc = ents.Create("npc_mymod_soldier")
            assert(IsValid(npc), "NPC failed to create")

            npc:SetPos(ply:GetPos() + Vector(100, 0, 0))
            npc:Spawn()

            assert(IsValid(npc), "NPC not valid after spawn")
            assert(npc:Health() > 0, "NPC has no health")

            npc:Remove()
        end)

        -- Test 2: Enemy detection
        Test("NPC detects enemy", function()
            local npc = ents.Create("npc_mymod_soldier")
            npc:SetPos(ply:GetPos() + Vector(100, 0, 0))
            npc:Spawn()

            local enemy = ents.Create("npc_mymod_zombie")
            enemy:SetPos(ply:GetPos() + Vector(200, 0, 0))
            enemy:Spawn()

            -- Wait for detection
            timer.Simple(2, function()
                assert(npc:HasEnemy() or enemy:HasEnemy(), "NPCs didn't detect each other")

                npc:Remove()
                enemy:Remove()
            end)
        end)

        -- Test 3: Damage handling
        Test("NPC handles damage correctly", function()
            local npc = ents.Create("npc_mymod_soldier")
            npc:SetPos(ply:GetPos() + Vector(100, 0, 0))
            npc:Spawn()

            local startHealth = npc:Health()

            local dmg = DamageInfo()
            dmg:SetDamage(10)
            dmg:SetAttacker(ply)
            npc:TakeDamage(dmg)

            assert(npc:Health() < startHealth, "NPC didn't take damage")
            assert(npc:Health() == startHealth - 10, "Damage calculation wrong")

            npc:Remove()
        end)

        -- Print summary
        timer.Simple(3, function()
            print(string.format("\n=== Test Results ==="))
            print(string.format("Passed: %d", passedTests))
            print(string.format("Failed: %d", failedTests))

            if failedTests == 0 then
                print("All tests passed!")
            else
                print("Some tests failed - check output above")
            end
        end)
    end)
end
```

### ❌ DON'T

```lua
-- BAD: Manual testing only
-- BAD: No regression tests
-- BAD: Re-testing everything manually after each change
```

### Why This Matters

Automated tests let you quickly verify that changes didn't break existing functionality. They save time and catch regressions immediately.

## Tips and Recommendations

1. **Test Early, Test Often**: Don't wait until development is complete. Test as you build.

2. **Test on Dedicated Server**: Singleplayer and listen servers behave differently from dedicated servers.

3. **Use Multiple Maps**: Test on different maps with different nav meshes to catch environment-specific issues.

4. **Get Others to Test**: Fresh eyes catch issues you've become blind to.

5. **Keep Test Commands**: Don't delete test commands after release. Keep them for future development.

6. **Document Test Procedures**: Write down how to test each feature so others (or future you) can repeat tests.

7. **Monitor Performance**: Use sv_fps, net_graph, and developer console during testing.

8. **Test with Addons**: Test with other popular addons to catch compatibility issues.

9. **Test Error Recovery**: Intentionally cause errors and verify your addon handles them gracefully.

10. **Version Control**: Commit working versions before making changes, so you can revert if tests fail.
