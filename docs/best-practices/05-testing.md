# Testing Strategies

Ensuring your DrGBase NPCs work correctly through systematic testing.

## Unit Testing Basics

### Test Individual Functions
```lua
-- Test helper function
function TestMeleeDamage()
    local npc = ents.Create("npc_your_zombie")
    npc:Spawn()

    local initialHealth = npc:Health()
    npc:TakeDamage(10)

    assert(npc:Health() == initialHealth - 10, "Damage not applied correctly")

    npc:Remove()
    print("✓ Melee damage test passed")
end
```

## Integration Testing

### Test NPC Interactions
```lua
function TestFactionSystem()
    local rebel = ents.Create("npc_your_rebel")
    local combine = ents.Create("npc_your_soldier")

    rebel:Spawn()
    combine:Spawn()

    -- Position them close
    combine:SetPos(rebel:GetPos() + Vector(100, 0, 0))

    -- Wait for detection
    timer.Simple(2, function()
        assert(IsValid(rebel:GetEnemy()), "Rebel should detect enemy")
        assert(IsValid(combine:GetEnemy()), "Combine should detect enemy")
        print("✓ Faction detection test passed")

        rebel:Remove()
        combine:Remove()
    end)
end
```

## Automated Testing

### Test Suite
```lua
-- lua/autorun/server/sv_tests.lua
if not game.SinglePlayer() then return end  -- Only in singleplayer

local tests = {}

function tests.TestSpawning()
    local npc = ents.Create("npc_your_zombie")
    assert(IsValid(npc), "NPC failed to create")
    npc:Spawn()
    assert(npc:Health() > 0, "NPC has no health")
    npc:Remove()
    return true
end

function tests.TestMovement()
    local npc = ents.Create("npc_your_zombie")
    npc:Spawn()

    local startPos = npc:GetPos()
    npc:SetTarget(startPos + Vector(500, 0, 0))

    timer.Simple(3, function()
        local moved = npc:GetPos():Distance(startPos) > 100
        assert(moved, "NPC didn't move")
        npc:Remove()
    end)

    return true
end

concommand.Add("yourmod_runtests", function()
    print("Running tests...")
    for name, test in pairs(tests) do
        local success, err = pcall(test)
        if success then
            print("✓", name, "passed")
        else
            print("✗", name, "failed:", err)
        end
    end
end)
```

## Manual Testing Checklist

### Basic Functionality
- [ ] NPC spawns without errors
- [ ] Model displays correctly
- [ ] Collision works properly
- [ ] Health system works
- [ ] Death triggers correctly

### AI Testing
- [ ] NPC detects enemies
- [ ] Pathfinding works on various terrain
- [ ] Combat AI engages properly
- [ ] Patrol behavior works
- [ ] Idle behavior is appropriate

### Relationship Testing
- [ ] Faction relationships work
- [ ] Allied NPCs don't fight
- [ ] Enemy NPCs attack appropriately
- [ ] Player relationships correct

### Combat Testing
- [ ] Melee attacks deal damage
- [ ] Ranged attacks hit targets
- [ ] Attack animations play
- [ ] Attack sounds play
- [ ] Damage types work correctly

### Performance Testing
- [ ] No lag with 10+ NPCs
- [ ] No console errors
- [ ] Memory doesn't leak
- [ ] FPS remains stable

## Debugging Tools

### Use Developer Console
```
developer 1          // Enable developer console
drgbase_ai_debug 1   // Show AI visualization
drgbase_info         // Use info tool
```

### Add Debug Prints
```lua
function ENT:OnMeleeAttack(enemy)
    if GetConVar("yourmod_debug"):GetBool() then
        print("Attacking:", enemy, "at distance:", self:GetPos():Distance(enemy:GetPos()))
    end

    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### Error Handling
```lua
function ENT:SafeCustomFunction()
    local success, err = pcall(function()
        -- Your code
    end)

    if not success then
        ErrorNoHalt("[YourMod] Error in CustomFunction: " .. tostring(err) .. "\n")
    end
end
```

## Regression Testing

### Test After Changes
```lua
-- Keep a test save
-- Before releasing update:
1. Load test save
2. Run automated tests
3. Manually verify core features
4. Check for new console errors
```

## Beta Testing

### Community Testing
1. Release beta version to trusted users
2. Collect feedback and logs
3. Fix reported issues
4. Repeat until stable

### What to Test in Beta
- Different game modes
- Various maps
- Multiplayer scenarios
- Edge cases
- Performance on different hardware

For detailed testing with DrGBase tools, see [Developer Tools](../tools/01-overview.md).
