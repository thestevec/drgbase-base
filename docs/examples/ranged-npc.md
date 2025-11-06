# Ranged NPC Example

This example demonstrates creating an NPC that uses ranged projectile attacks. We'll create a turret-style enemy that shoots energy balls at targets from a distance.

## Overview

This example creates a "Plasma Turret" NPC that:
- Detects enemies at range
- Fires projectile attacks
- Keeps distance from enemies
- Uses energy weapon sounds and effects
- Has proper ranged combat AI

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc --
ENT.PrintName = "Plasma Turret"
ENT.Category = "Custom NPCs"
ENT.Models = {"models/Combine_Helicopter/helicopter_bomb01.mdl"}
ENT.CollisionBounds = Vector(20, 20, 30)
ENT.BloodColor = BLOOD_COLOR_MECH

-- Stats --
ENT.SpawnHealth = 80
ENT.HealthRegen = 0

-- AI --
ENT.MeleeAttackRange = 0  -- No melee
ENT.RangeAttackRange = 1500  -- Long range
ENT.ReachEnemyRange = 800  -- Stop before getting too close
ENT.AvoidEnemyRange = 400  -- Back up if enemy gets close

-- Relationships --
ENT.Factions = {FACTION_COMBINE}

-- Movements --
ENT.RunSpeed = 150
ENT.WalkSpeed = 80
ENT.Acceleration = 300
ENT.StepHeight = 24
ENT.JumpHeight = 64

-- Detection --
ENT.SightRange = 2000
ENT.SightFOV = 160

-- Animations --
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE

-- Sounds --
ENT.OnDamageSounds = {"npc/scanner/scanner_pain1.wav", "npc/scanner/scanner_pain2.wav"}
ENT.OnDeathSounds = {"npc/scanner/scanner_explode_crash2.wav"}

if SERVER then

    -- Init --

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- Start with a small charging effect
        self:CreateLight(Color(0, 150, 255), 100)
    end

    function ENT:CreateLight(color, brightness)
        local light = ents.Create("light_dynamic")
        if IsValid(light) then
            light:SetKeyValue("brightness", brightness)
            light:SetKeyValue("distance", 200)
            light:SetPos(self:GetPos() + Vector(0, 0, 20))
            light:SetParent(self)
            light:SetColor(color)
            light:Spawn()
            light:Activate()
            light:Fire("TurnOn", "", 0)
            self:DeleteOnRemove(light)
        end
    end

    -- AI --

    function ENT:OnRangeAttack(enemy)
        -- Face enemy before shooting
        self:FaceTo(enemy:EyePos())

        -- Wait briefly to aim
        self:PauseCoroutine(0.2)

        -- Play attack sound
        self:EmitSound("weapons/physcannon/energy_sing_loop4.wav", 75, math.random(90, 110))

        -- Spawn projectile
        local proj = ents.Create("proj_drg_plasma")
        if IsValid(proj) then
            local startPos = self:GetPos() + Vector(0, 0, 20)
            local targetPos = enemy:EyePos()

            -- Calculate direction with slight prediction
            local enemyVel = enemy:GetVelocity()
            local distance = startPos:Distance(targetPos)
            local travelTime = distance / 500
            local predictedPos = targetPos + (enemyVel * travelTime * 0.5)

            local dir = (predictedPos - startPos):GetNormalized()

            proj:SetPos(startPos)
            proj:SetAngles(dir:Angle())
            proj:Spawn()
            proj:SetOwner(self)
            proj:SetPhysicsAttacker(self, 10)

            -- Set velocity
            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(dir * 500)
            end
        end

        -- Cooldown before next attack
        self:PauseCoroutine(0.8)
    end

    -- Patrol behavior --

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1000))
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(3, 6))
    end

    -- Combat behavior --

    function ENT:OnChaseEnemy(enemy)
        -- If enemy is too close, back away
        local dist = self:GetPos():Distance(enemy:GetPos())
        if dist < self.AvoidEnemyRange then
            -- Move away from enemy
            local awayDir = (self:GetPos() - enemy:GetPos()):GetNormalized()
            local awayPos = self:GetPos() + awayDir * 300
            self:SetTarget(awayPos)
        end
    end

    -- Events --

    function ENT:OnNewEnemy(enemy)
        self:EmitSound("npc/scanner/scanner_talk1.wav")
    end

    function ENT:OnDeath(dmg, hitgroup)
        -- Explosion effect
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        util.Effect("Explosion", effectData)

        -- Small explosion damage
        util.BlastDamage(self, self, self:GetPos(), 100, 25)
    end

end

-- Register --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Step-by-Step Explanation

### 1. Ranged Combat Configuration

```lua
ENT.MeleeAttackRange = 0  -- Disable melee
ENT.RangeAttackRange = 1500  -- Attack from far away
ENT.ReachEnemyRange = 800  -- Optimal combat distance
ENT.AvoidEnemyRange = 400  -- Back up if too close
```

Key differences from melee NPCs:
- `MeleeAttackRange = 0` - Completely disables melee attacks
- `RangeAttackRange` - Maximum range to shoot at enemies (in units)
- `ReachEnemyRange` - NPC stops moving when this close to enemy
- `AvoidEnemyRange` - NPC backs away when enemy gets within this range

**Range Strategy:**
- `RangeAttackRange > ReachEnemyRange > AvoidEnemyRange`
- This creates a "comfort zone" where the NPC prefers to fight

### 2. Projectile Creation

```lua
function ENT:OnRangeAttack(enemy)
    -- Face enemy
    self:FaceTo(enemy:EyePos())
    self:PauseCoroutine(0.2)

    -- Create projectile
    local proj = ents.Create("proj_drg_plasma")
    if IsValid(proj) then
        local startPos = self:GetPos() + Vector(0, 0, 20)
        proj:SetPos(startPos)
        proj:SetAngles((enemy:EyePos() - startPos):Angle())
        proj:Spawn()
        proj:SetOwner(self)

        -- Launch it
        local phys = proj:GetPhysicsObject()
        if IsValid(phys) then
            local dir = (enemy:EyePos() - startPos):GetNormalized()
            phys:SetVelocity(dir * 500)
        end
    end

    self:PauseCoroutine(0.8)  -- Attack cooldown
end
```

**Steps:**
1. Face the target using `FaceTo()`
2. Pause briefly to "aim"
3. Create projectile entity
4. Position it at firing point
5. Set owner (for damage attribution)
6. Calculate direction to target
7. Apply velocity to physics object
8. Add cooldown delay

### 3. Target Prediction

For more accurate shots against moving targets:

```lua
-- Calculate direction with prediction
local enemyVel = enemy:GetVelocity()
local distance = startPos:Distance(targetPos)
local travelTime = distance / 500  -- 500 is projectile speed
local predictedPos = targetPos + (enemyVel * travelTime * 0.5)
local dir = (predictedPos - startPos):GetNormalized()
```

This predicts where the enemy will be by the time the projectile arrives.

### 4. Avoid Behavior

```lua
function ENT:OnChaseEnemy(enemy)
    local dist = self:GetPos():Distance(enemy:GetPos())
    if dist < self.AvoidEnemyRange then
        -- Move away from enemy
        local awayDir = (self:GetPos() - enemy:GetPos()):GetNormalized()
        local awayPos = self:GetPos() + awayDir * 300
        self:SetTarget(awayPos)
    end
end
```

This makes the NPC kite/retreat when enemies get too close, maintaining optimal range.

## Customization Examples

### Rapid Fire Weapon

```lua
function ENT:OnRangeAttack(enemy)
    -- Fire burst of 3 shots
    for i = 1, 3 do
        self:FaceTo(enemy:EyePos())

        local proj = ents.Create("proj_drg_plasma")
        if IsValid(proj) then
            local startPos = self:GetPos() + Vector(0, 0, 20)
            local dir = (enemy:EyePos() - startPos):GetNormalized()

            proj:SetPos(startPos)
            proj:SetAngles(dir:Angle())
            proj:Spawn()
            proj:SetOwner(self)

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(dir * 600)
            end
        end

        self:EmitSound("weapons/ar2/fire1.wav")
        self:PauseCoroutine(0.15)  -- Delay between shots
    end

    self:PauseCoroutine(1.5)  -- Cooldown after burst
end
```

### Spread Shot (Shotgun Pattern)

```lua
function ENT:OnRangeAttack(enemy)
    self:FaceTo(enemy:EyePos())
    self:EmitSound("weapons/shotgun/shotgun_fire6.wav")

    -- Fire 5 projectiles in a spread
    for i = 1, 5 do
        local proj = ents.Create("proj_drg_plasma")
        if IsValid(proj) then
            local startPos = self:GetPos() + Vector(0, 0, 20)
            local baseDir = (enemy:EyePos() - startPos):GetNormalized()

            -- Add random spread
            local spread = Angle(math.Rand(-5, 5), math.Rand(-5, 5), 0)
            local dir = spread:Forward()
            dir = (baseDir + dir * 0.2):GetNormalized()

            proj:SetPos(startPos)
            proj:SetAngles(dir:Angle())
            proj:Spawn()
            proj:SetOwner(self)

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(dir * 800)
            end
        end
    end

    self:PauseCoroutine(2.0)
end
```

### Arc Projectile (Grenade-like)

```lua
function ENT:OnRangeAttack(enemy)
    local proj = ents.Create("proj_drg_grenade")
    if IsValid(proj) then
        local startPos = self:GetPos() + Vector(0, 0, 30)
        local targetPos = enemy:GetPos()

        proj:SetPos(startPos)
        proj:Spawn()
        proj:SetOwner(self)

        -- Calculate arc trajectory
        local phys = proj:GetPhysicsObject()
        if IsValid(phys) then
            local dist = startPos:Distance(targetPos)
            local dir = (targetPos - startPos):GetNormalized()
            local velocity = dir * (dist * 0.5)
            velocity.z = 400  -- Add upward velocity for arc

            phys:SetVelocity(velocity)
        end

        -- Unpin grenade
        if proj.Unpin then proj:Unpin() end
    end

    self:PauseCoroutine(3.0)
end
```

### Charging Attack

```lua
function ENT:OnRangeAttack(enemy)
    -- Charging phase
    self:FaceTo(enemy:EyePos())
    self:EmitSound("weapons/physcannon/superphys_launch1.wav", 70, 150)

    -- Visual charging effect
    for i = 1, 3 do
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos() + Vector(0, 0, 20))
        effectData:SetScale(i)
        util.Effect("ElectricSpark", effectData)
        self:PauseCoroutine(0.3)
    end

    -- Fire powerful shot
    self:EmitSound("weapons/physcannon/superphys_launch4.wav")

    local proj = ents.Create("proj_drg_plasma")
    if IsValid(proj) then
        local startPos = self:GetPos() + Vector(0, 0, 20)
        local dir = (enemy:EyePos() - startPos):GetNormalized()

        proj:SetPos(startPos)
        proj:SetAngles(dir:Angle())
        proj:Spawn()
        proj:SetOwner(self)
        proj:SetModelScale(2)  -- Larger projectile

        local phys = proj:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(dir * 1000)
        end
    end

    self:PauseCoroutine(2.5)  -- Longer cooldown
end
```

### Homing Missiles

```lua
function ENT:OnRangeAttack(enemy)
    local proj = ents.Create("proj_drg_plasma")
    if IsValid(proj) then
        local startPos = self:GetPos() + Vector(0, 0, 20)
        proj:SetPos(startPos)
        proj:Spawn()
        proj:SetOwner(self)

        -- Store target for homing behavior
        proj.HomingTarget = enemy

        -- Homing logic
        timer.Create("HomingMissile_"..proj:EntIndex(), 0.05, 0, function()
            if not IsValid(proj) or not IsValid(enemy) then return end

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                local dir = (enemy:EyePos() - proj:GetPos()):GetNormalized()
                local currentVel = phys:GetVelocity()
                local newVel = LerpVector(0.1, currentVel, dir * 600)
                phys:SetVelocity(newVel)
                proj:SetAngles(newVel:Angle())
            end
        end)
    end

    self:PauseCoroutine(1.5)
end
```

### Alternate Between Attack Types

```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)
    self.AttackType = 1  -- Track which attack to use
end

function ENT:OnRangeAttack(enemy)
    if self.AttackType == 1 then
        -- Fast single shot
        self:FireSingleShot(enemy)
        self:PauseCoroutine(0.5)
    elseif self.AttackType == 2 then
        -- Spread shot
        self:FireSpread(enemy)
        self:PauseCoroutine(1.5)
    else
        -- Charged shot
        self:FireCharged(enemy)
        self:PauseCoroutine(2.0)
    end

    -- Cycle through attack types
    self.AttackType = (self.AttackType % 3) + 1
end
```

## Common Issues and Solutions

### Issue: Projectiles Don't Spawn

**Problem:** Attack animation plays but no projectile appears

**Solutions:**
1. Check entity class name is correct: `ents.Create("proj_drg_plasma")`
2. Verify projectile entity exists in your addon
3. Make sure `proj:Spawn()` is called
4. Check console for errors about missing entities

**Debug:**
```lua
function ENT:OnRangeAttack(enemy)
    local proj = ents.Create("proj_drg_plasma")
    print("Created projectile:", proj, IsValid(proj))
    if IsValid(proj) then
        proj:SetPos(self:GetPos() + Vector(0, 0, 20))
        proj:Spawn()
        print("Spawned at:", proj:GetPos())
    end
end
```

### Issue: Projectiles Fall to Ground

**Problem:** Projectiles spawn but drop immediately

**Solutions:**
1. Make sure physics object exists and is valid
2. Verify velocity is set correctly
3. Check if projectile has gravity disabled in its entity definition

```lua
local phys = proj:GetPhysicsObject()
if IsValid(phys) then
    print("Physics valid, setting velocity")
    phys:SetVelocity(dir * 500)
    phys:EnableGravity(false)  -- Disable gravity if needed
end
```

### Issue: Shots Always Miss

**Problem:** Projectiles fire but go in wrong direction

**Solutions:**
1. Verify direction calculation: `(target - start):GetNormalized()`
2. Check spawn position is in front of NPC, not behind
3. Make sure NPC is facing target before shooting
4. Add prediction for moving targets (see examples above)

**Debug:**
```lua
local dir = (enemy:EyePos() - startPos):GetNormalized()
print("Firing direction:", dir)
print("From:", startPos, "To:", enemy:EyePos())
```

### Issue: NPC Won't Keep Distance

**Problem:** NPC runs right up to enemies instead of keeping range

**Solutions:**
1. Verify `AvoidEnemyRange` is set
2. Check `ReachEnemyRange` is less than `RangeAttackRange`
3. Implement custom `OnChaseEnemy` to enforce distance
4. Make sure ranged attack is working (NPC stops when it can shoot)

**Proper setup:**
```lua
ENT.RangeAttackRange = 1500  -- Can shoot from this far
ENT.ReachEnemyRange = 800    -- Stop moving here
ENT.AvoidEnemyRange = 400    -- Back up if closer than this
```

### Issue: Attacks Too Fast/Slow

**Problem:** NPC spam fires or takes too long between shots

**Solutions:**
1. Adjust `PauseCoroutine` duration at end of `OnRangeAttack`
2. Balance attack delay with projectile speed and damage
3. Consider adding minimum/maximum fire rate limits

```lua
function ENT:OnRangeAttack(enemy)
    -- Your attack code here

    -- Adjust this delay for fire rate
    self:PauseCoroutine(1.0)  -- 1 second between attacks
end
```

### Issue: Projectiles Hurt Self or Allies

**Problem:** Projectiles damage the NPC that fired them or friendly NPCs

**Solutions:**
1. Set projectile owner: `proj:SetOwner(self)`
2. Use `SetPhysicsAttacker`: `proj:SetPhysicsAttacker(self, 10)`
3. Implement damage filtering in projectile entity
4. Add brief delay before projectile becomes active

```lua
proj:SetOwner(self)
proj:SetPhysicsAttacker(self, 10)

-- In projectile entity:
function ENT:OnContact(ent)
    if ent == self:GetOwner() then return end  -- Ignore owner
    -- Damage logic here
end
```

## Performance Tips

1. **Limit projectile lifetime**
```lua
-- In projectile entity
function ENT:CustomInitialize()
    timer.Simple(10, function()
        if IsValid(self) then self:Remove() end
    end)
end
```

2. **Reduce fire rate**
```lua
-- Longer delays = less server load
self:PauseCoroutine(1.0)  -- Good
self:PauseCoroutine(0.1)  -- Causes spam
```

3. **Optimize homing missiles**
```lua
-- Update every 0.1s instead of 0.05s
timer.Create("Homing_"..proj:EntIndex(), 0.1, 0, function()
    -- Homing logic
end)
```

4. **Clean up timers**
```lua
-- Always clean up projectile timers
function ENT:OnRemove()
    timer.Remove("HomingMissile_"..self:EntIndex())
end
```

## Next Steps

- Learn about [Custom Projectiles](./custom-projectile.md) for advanced projectile behavior
- Try the [Boss NPC Example](./boss-npc.md) for complex multi-phase combat
- Explore [Advanced AI](./advanced-ai.md) for smarter targeting and tactics
