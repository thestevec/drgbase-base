# Performance Guidelines

Optimizing DrGBase NPCs for smooth gameplay and server performance.

## NPC Count Management

### Limit Active NPCs
```lua
-- Good: Respect server limits
local maxNPCs = GetConVar("drgbase_max_npcs"):GetInt()
local currentNPCs = #ents.FindByClass("drgbase_nextbot")

if currentNPCs < maxNPCs then
    -- Spawn NPC
end

-- Bad: Unlimited spawning
for i = 1, 100 do
    local npc = ents.Create("npc_drg_zombie")
    npc:Spawn()
end
```

### Despawn Distant NPCs
```lua
function ENT:CustomThink()
    -- Despawn if too far from players
    local closestPlayer = self:GetClosestPlayer()
    if IsValid(closestPlayer) then
        local dist = self:GetPos():Distance(closestPlayer:GetPos())
        if dist > 5000 then  -- 5000 units
            self:Remove()
        end
    end
end
```

## Sight Range Optimization

```lua
-- Good: Reasonable sight range
ENT.SightRange = 2000  -- 2000 units is usually sufficient

-- Bad: Excessive sight range
ENT.SightRange = 50000  -- Checks too many entities, causes lag
```

### Dynamic Sight Range
```lua
function ENT:CustomThink()
    -- Reduce sight range when no enemy
    if not IsValid(self:GetEnemy()) then
        self.SightRange = 1000  -- Low when idle
    else
        self.SightRange = 3000  -- Higher when in combat
    end
end
```

## Think Rate Optimization

### Don't Think Every Tick
```lua
-- Good: Throttle expensive checks
function ENT:CustomThink()
    if CurTime() > (self.NextExpensiveCheck or 0) then
        self:DoExpensiveCalculation()
        self.NextExpensiveCheck = CurTime() + 0.5  -- Every 0.5s
    end
end

-- Bad: Every tick
function ENT:CustomThink()
    self:DoExpensiveCalculation()  -- Called 66 times per second!
end
```

## Pathfinding

### Cache Path Results
```lua
-- Good: Cache paths
function ENT:SetPathTarget(pos)
    if self.LastPathTarget and self.LastPathTarget:Distance(pos) < 100 then
        return  -- Target hasn't changed much, don't recalculate
    end

    self.LastPathTarget = pos
    self:SetTarget(pos)
end

-- Bad: Constant recalculation
function ENT:CustomThink()
    self:SetTarget(enemy:GetPos())  -- Recalculates every think!
end
```

## Entity Finding

### Limit Search Radius
```lua
-- Good: Reasonable radius
local nearby = ents.FindInSphere(self:GetPos(), 500)

-- Bad: Huge radius
local nearby = ents.FindInSphere(self:GetPos(), 10000)  -- Checks too many entities
```

### Cache Entity Lists
```lua
-- Good: Cache and refresh periodically
function ENT:GetNearbyEnemies()
    if CurTime() < (self.NextEnemyRefresh or 0) then
        return self.CachedEnemies or {}
    end

    self.CachedEnemies = {}
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
        if self:GetRelationship(ent) == D_HT then
            table.insert(self.CachedEnemies, ent)
        end
    end

    self.NextEnemyRefresh = CurTime() + 1  -- Refresh every second
    return self.CachedEnemies
end
```

## Sound Performance

### Limit Sound Spam
```lua
-- Good: Cooldown on sounds
function ENT:PlayFootstep()
    if CurTime() < (self.NextFootstep or 0) then return end

    self:EmitSound("npc/zombie/foot" .. math.random(1, 3) .. ".wav")
    self.NextFootstep = CurTime() + 0.3
end

-- Bad: No cooldown
function ENT:OnAnimEvent()
    self:EmitSound("npc/zombie/foot1.wav")  -- Can spam rapidly
end
```

## Effect Performance

### Limit Particle Effects
```lua
-- Good: Conditional effects
function ENT:CreateEffect()
    -- Only create if players nearby
    if #player.GetHumans() == 0 then return end
    if not self:IsInPlayerPVS() then return end

    local effectData = EffectData()
    effectData:SetOrigin(self:GetPos())
    util.Effect("BloodImpact", effectData)
end
```

## Timer Management

### Clean Up Timers
```lua
-- Good: Always clean up
function ENT:CustomInitialize()
    timer.Create("NPCAbility_"..self:EntIndex(), 5, 0, function()
        if not IsValid(self) then return end
        self:SpecialAbility()
    end)
end

function ENT:OnRemove()
    timer.Remove("NPCAbility_"..self:EntIndex())
end

-- Bad: Timer leaks
function ENT:CustomInitialize()
    timer.Create("NPCAbility", 5, 0, function()
        self:SpecialAbility()  -- Will error after NPC removed
    end)
end
```

## Network Optimization

### Minimize Network Updates
```lua
-- Good: Only update when changed
function ENT:SetCustomHealth(health)
    if self:GetNW2Int("CustomHealth") == health then return end
    self:SetNW2Int("CustomHealth", health)
end

-- Bad: Constant updates
function ENT:CustomThink()
    self:SetNW2Int("CustomHealth", self:Health())  -- Updates every tick!
end
```

## Convars for Performance

```lua
-- Allow server owners to tune performance
CreateConVar("yourmod_ai_thinkrate", "30", FCVAR_ARCHIVE, "AI think rate (Hz)")
CreateConVar("yourmod_max_npcs", "50", FCVAR_ARCHIVE, "Maximum NPC count")
CreateConVar("yourmod_sight_range", "2000", FCVAR_ARCHIVE, "Default sight range")
```

## Profiling

### Find Performance Issues
```lua
-- Use profiling to find bottlenecks
function ENT:CustomThink()
    local startTime = SysTime()

    -- Your code here

    local endTime = SysTime()
    if endTime - startTime > 0.001 then  -- More than 1ms
        print("CustomThink took", (endTime - startTime) * 1000, "ms")
    end
end
```

## Performance Checklist

- [ ] Sight range under 3000 units
- [ ] Think rate throttled (not every tick for expensive ops)
- [ ] Entity searches limited to reasonable radius
- [ ] Timers properly cleaned up on remove
- [ ] Sounds have cooldowns
- [ ] Effects only created when visible
- [ ] Network updates minimized
- [ ] Paths cached when possible
- [ ] NPC count limited
