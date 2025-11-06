# Common Pitfalls and How to Avoid Them

## Overview

This guide covers frequent mistakes developers make when creating DrGBase NPCs and how to avoid them.

---

## 1. Animation Issues

### ❌ Pitfall: Calling PlayActivity() Every Frame

```lua
function ENT:CustomThink()
    if self:IsMoving() then
        self:PlayActivity(ACT_WALK) -- Restarts animation every frame!
    end
end
```

**Problem:** Animation constantly restarts, looks stuttery

**✅ Solution:**
```lua
function ENT:CustomThink()
    if self:IsMoving() and self:GetSequence() != self:LookupSequence(ACT_WALK) then
        self:PlayActivity(ACT_WALK) -- Only play if not already playing
    end
end

-- Or better: Let DrGBase handle it automatically
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
-- DrGBase will play these automatically
```

### ❌ Pitfall: Wrong Activity Name

```lua
function ENT:OnMeleeAttack(enemy)
    self:PlayActivity(ACT_MELEE) -- This activity doesn't exist!
end
```

**Problem:** Lua error or no animation plays

**✅ Solution:**
```lua
function ENT:OnMeleeAttack(enemy)
    -- Check if activity exists
    local seq = self:SelectWeightedSequence(ACT_MELEE_ATTACK1)
    if seq == -1 then
        -- Activity doesn't exist, use sequence name
        self:PlaySequence("zombie_attack_01")
    else
        self:PlayActivity(ACT_MELEE_ATTACK1)
    end
end
```

### ❌ Pitfall: Attack Damage Fires Every Frame

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({damage = 10}) -- Fires multiple times!
    end
end
```

**Problem:** Deals damage multiple times per attack

**✅ Solution:**
```lua
function ENT:OnMeleeAttack(enemy)
    self.AttackExecuted = false
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and not self.AttackExecuted then
        if self:GetCycle() >= 0.3 then
            self.AttackExecuted = true
            self:Attack({damage = 10})
        end
    end
end
```

---

## 2. AI and Behavior Issues

### ❌ Pitfall: Blocking CustomThink()

```lua
function ENT:CustomThink()
    self:Wait(1) -- ERROR: Can't use Wait() here!
end
```

**Problem:** Lua error, Wait() only works in coroutines

**✅ Solution:**
```lua
-- Use AIBehaviour() for coroutines
function ENT:AIBehaviour()
    while true do
        self:MyBehavior()
        self:Wait(1) -- OK here
    end
end

-- Or use timers
function ENT:CustomInitialize()
    self:LoopTimer(1, function(self)
        self:MyBehavior()
    end)
end
```

### ❌ Pitfall: Not Checking IsValid()

```lua
function ENT:CustomThink()
    local enemy = self:GetEnemy()
    enemy:GetPos() -- Crashes if enemy is nil!
end
```

**Problem:** Lua error when enemy doesn't exist

**✅ Solution:**
```lua
function ENT:CustomThink()
    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        local pos = enemy:GetPos()
        -- Safe to use pos
    end
end
```

### ❌ Pitfall: Forgetting Default Relationship

```lua
ENT.Factions = {FACTION_ZOMBIES}

function ENT:CustomInitialize()
    -- Nothing here
end
```

**Problem:** NPC ignores players (default is D_NU)

**✅ Solution:**
```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT) -- Hate everything
    -- Or specifically hate players
    self:SetPlayersRelationship(D_HT)
end
```

### ❌ Pitfall: AIBehaviour() Without Loop

```lua
function ENT:AIBehaviour()
    self:DoSomething()
    self:Wait(1)
    -- Function ends, AI stops!
end
```

**Problem:** AI runs once then stops

**✅ Solution:**
```lua
function ENT:AIBehaviour()
    while true do -- Must have infinite loop!
        self:DoSomething()
        self:Wait(1)
    end
end
```

---

## 3. Performance Issues

### ❌ Pitfall: Entity Search Every Frame

```lua
function ENT:CustomThink()
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
        -- Check entities every frame = laggy!
    end
end
```

**Problem:** Very expensive, causes lag

**✅ Solution:**
```lua
function ENT:CustomInitialize()
    self.NearbyCache = {}
    self:LoopTimer(1, function(self)
        -- Update cache every second
        self.NearbyCache = ents.FindInSphere(self:GetPos(), 500)
    end)
end

function ENT:CustomThink()
    for _, ent in ipairs(self.NearbyCache) do
        -- Use cached list
    end
end
```

### ❌ Pitfall: Continuous Sound Spam

```lua
function ENT:CustomThink()
    if self:Health() < 30 then
        self:EmitSound("Pain") -- Plays every frame!
    end
end
```

**Problem:** Audio spam, performance hit

**✅ Solution:**
```lua
function ENT:CustomThink()
    if self:Health() < 30 then
        self:EmitSlotSound("pain", 2, "Pain") -- Only plays every 2 seconds
    end
end

-- Or use flag
function ENT:CustomThink()
    if self:Health() < 30 and not self.PlayedPainSound then
        self:EmitSound("Pain")
        self.PlayedPainSound = true
    end
end
```

### ❌ Pitfall: No LOD System

```lua
-- All NPCs update at full rate, regardless of distance
```

**Problem:** Wastes performance on distant NPCs

**✅ Solution:**
```lua
function ENT:AIBehaviour()
    while true do
        self:UpdateAI()

        -- Adjust interval based on distance to players
        local closestDist = math.huge
        for _, ply in ipairs(player.GetAll()) do
            closestDist = math.min(closestDist, self:GetPos():Distance(ply:GetPos()))
        end

        if closestDist > 2000 then
            self:Wait(1.0) -- Far away: slow updates
        else
            self:Wait(0.1) -- Close: fast updates
        end
    end
end
```

---

## 4. Relationship and Faction Issues

### ❌ Pitfall: Wrong Relationship Constant

```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HATE) -- Doesn't exist!
end
```

**Problem:** Lua error

**✅ Solution:**
```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)  -- Correct: D_HT, D_LI, D_FR, D_NU
end
```

### ❌ Pitfall: Faction Without Relationship Rules

```lua
ENT.Factions = {FACTION_GUARDS}

function ENT:CustomInitialize()
    -- No relationship rules set!
end
```

**Problem:** NPCs in same faction still attack each other

**✅ Solution:**
```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)
    self:SetSelfFactionRelationship(D_LI) -- Like same faction
end
```

---

## 5. Pathfinding Issues

### ❌ Pitfall: No Navmesh

```lua
-- NPC doesn't move
```

**Problem:** No navigation mesh generated

**✅ Solution:**
```
// In console:
nav_generate

// Or for large maps:
nav_generate_incremental
```

### ❌ Pitfall: Path Calculated Every Frame

```lua
function ENT:CustomThink()
    self:ComputePath(targetPos) -- Recalculates every frame!
    self:FollowPath(targetPos)
end
```

**Problem:** Very expensive, causes lag

**✅ Solution:**
```lua
function ENT:AIBehaviour()
    while true do
        self:FollowPath(targetPos) -- DrGBase caches automatically
        self:Wait(0.5) -- Update every half second
    end
end
```

---

## 6. Damage and Health Issues

### ❌ Pitfall: OnTakeDamage() Returns 0

```lua
function ENT:OnTakeDamage(dmg)
    print("Took damage:", dmg:GetDamage())
    return 0 -- NPC takes no damage!
end
```

**Problem:** NPC invincible

**✅ Solution:**
```lua
function ENT:OnTakeDamage(dmg)
    -- Don't return 0 unless you want to cancel damage
    -- return 1.0 -- Normal damage
    -- return 0.5 -- Half damage
    -- return 2.0 -- Double damage
end
```

### ❌ Pitfall: Setting Health in CustomThink()

```lua
function ENT:CustomThink()
    if self:Health() < 100 then
        self:SetHealth(self:Health() + 1) -- Every frame!
    end
end
```

**Problem:** Regenerates too fast

**✅ Solution:**
```lua
-- Use built-in regen
ENT.HealthRegen = 1 -- 1 HP per second

-- Or use timer
function ENT:CustomInitialize()
    self:LoopTimer(1, function(self)
        if self:Health() < self:GetMaxHealth() then
            self:SetHealth(self:Health() + 1)
        end
    end)
end
```

---

## 7. Weapon Issues

### ❌ Pitfall: Wrong Base Class

```lua
ENT.Base = "drgbase_nextbot"
ENT.Weapons = {"weapon_ar2"} -- Doesn't work!
```

**Problem:** Weapons only work with human base

**✅ Solution:**
```lua
ENT.Base = "drgbase_nextbot_human" -- Correct base for weapons
ENT.Weapons = {"weapon_ar2"}
```

---

## 8. Initialization Issues

### ❌ Pitfall: Heavy Work in CustomInitialize()

```lua
function ENT:CustomInitialize()
    -- 1000 expensive operations at spawn
    for i = 1, 1000 do
        self:ExpensiveCalculation()
    end
end
```

**Problem:** Spawn lag, game freezes

**✅ Solution:**
```lua
function ENT:CustomInitialize()
    -- Delay heavy work
    self:Timer(0.5, function(self)
        for i = 1, 1000 do
            self:ExpensiveCalculation()
        end
    end)
end
```

### ❌ Pitfall: Missing AddCSLuaFile()

```lua
-- At end of file:
DrGBase.AddNextbot(ENT)
-- Missing AddCSLuaFile()!
```

**Problem:** Clients get errors

**✅ Solution:**
```lua
-- Always include both at end:
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

---

## 9. Networking Issues

### ❌ Pitfall: Setting NW Vars Every Frame

```lua
function ENT:CustomThink()
    self:SetNWFloat("MyVar", self.MyValue) -- Every frame!
end
```

**Problem:** Network spam, lag

**✅ Solution:**
```lua
function ENT:SetMyValue(value)
    if self.MyValue != value then
        self.MyValue = value
        self:SetNWFloat("MyVar", value) -- Only when changed
    end
end
```

---

## 10. Possession Issues

### ❌ Pitfall: Coroutine Flag Wrong

```lua
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:PrimaryFire() -- Instant action, doesn't need coroutine!
        end
    }}
}
```

**Problem:** Unnecessary coroutine overhead

**✅ Solution:**
```lua
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = false, -- No waiting, use false
        onkeydown = function(self)
            self:PrimaryFire()
        end
    }}
}

-- Use coroutine = true only when using Wait():
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true, -- Needs Wait()
        onkeydown = function(self)
            self:PlayAnimation()
            self:Wait(0.5)
            self:Attack()
        end
    }}
}
```

---

## Quick Checklist

### ✅ Before Testing
- [ ] AddCSLuaFile() at end of file
- [ ] DrGBase.AddNextbot(ENT) at end of file
- [ ] Navmesh generated (nav_generate)
- [ ] Default relationship set
- [ ] Base class correct for features used

### ✅ During Development
- [ ] IsValid() checks before using entities
- [ ] while true loops in AIBehaviour()
- [ ] Wait() only in coroutines
- [ ] Animation flags to prevent duplicate triggers
- [ ] Proper LOD/throttling for performance

### ✅ Before Release
- [ ] Remove debug prints
- [ ] Test with multiple NPCs
- [ ] Check for Lua errors
- [ ] Verify pathfinding works
- [ ] Test all animations
- [ ] Check relationships work
- [ ] Performance test (30+ NPCs)

## See Also

- [Debugging Guide](debugging.md)
- [Performance Guide](performance.md)
- [Code Organization](code-organization.md)
