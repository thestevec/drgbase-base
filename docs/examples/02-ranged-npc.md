# Creating a Ranged Attack NPC

## Overview

This guide shows how to create an NPC with ranged attacks, using the Headcrab leap attack as an example. Ranged attacks allow NPCs to engage enemies from a distance.

## The Headcrab Example

The Headcrab is a classic example of a ranged attacker that leaps at enemies.

### Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Basic Info
ENT.PrintName = "Headcrab"
ENT.Category = "DrGBase"
ENT.Models = {"models/headcrabclassic.mdl"}
ENT.CollisionBounds = Vector(12, 12, 24)
ENT.BloodColor = BLOOD_COLOR_GREEN

-- Stats
ENT.SpawnHealth = 40

-- AI Configuration
ENT.RangeAttackRange = 150    -- Attack from up to 150 units away
ENT.MeleeAttackRange = 0      -- No melee attacks
ENT.ReachEnemyRange = 125     -- Get within 125 units
ENT.AvoidEnemyRange = 100     -- Back off to 100 units

-- Relationships
ENT.Factions = {FACTION_ZOMBIES}

-- Animations
ENT.WalkAnimation = ACT_RUN
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE
ENT.JumpAnimation = ACT_IDLE

-- Movement
ENT.UseWalkframes = true

-- Sounds
ENT.OnIdleSounds = {"NPC_HeadCrab.Idle"}
ENT.OnDamageSounds = {"NPC_HeadCrab.Pain"}
ENT.OnDeathSounds = {"NPC_HeadCrab.Die"}

-- Possession
ENT.PossessionEnabled = true
ENT.PossessionCrosshair = true
ENT.PossessionViews = {
    {
        offset = Vector(0, 10, 10),
        distance = 50
    },
    {
        offset = Vector(7.5, 0, 0),
        distance = 0,
        eyepos = true
    }
}
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:HeadcrabLeap(self:PossessorTrace().HitPos)
        end
    }}
}

if SERVER then
    -- Custom leap attack function
    function ENT:HeadcrabLeap(pos)
        self:FaceTo(pos)                                    -- Face target
        self.OnIdleSounds = {}                              -- Silence during leap
        self:PlaySequence("jumpattack_broadcast")           -- Wind-up animation
        self:PauseCoroutine(0.5)                            -- Wait half second
        self.CanBite = true                                 -- Enable contact damage
        self:EmitSound("NPC_Headcrab.Attack")               -- Attack sound
        self:Leap(pos, 400)                                 -- Launch at 400 units/sec
        self.CanBite = false                                -- Disable contact damage
        self.OnIdleSounds = {"NPC_HeadCrab.Idle"}           -- Re-enable idle sounds
    end

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    -- Called when in range to attack
    function ENT:OnRangeAttack(enemy)
        self:HeadcrabLeap(enemy:EyePos() - Vector(0, 0, 10))
    end

    -- Called when physically touching another entity
    function ENT:OnContact(ent)
        if not IsValid(ent) then return end
        -- Only damage during leap and if touching enemy
        if self.CanBite and
           (self:IsPossessed() or ent == self:GetEnemy()) then
            self:EmitSound("NPC_HeadCrab.Bite")
            self.CanBite = false

            local dmg = DamageInfo()
            dmg:SetDamage(20)
            dmg:SetAttacker(self)
            dmg:SetInflictor(self)
            dmg:SetDamageType(DMG_SLASH)
            ent:TakeDamageInfo(dmg)
        end
    end

    -- Idle behavior
    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end

    function ENT:OnReachedPatrol(pos)
        self:Wait(math.random(3, 7))
    end

    -- Damage modifier
    function ENT:OnTakeDamage(dmg)
        local attacker = dmg:GetAttacker()
        if IsValid(attacker) and attacker:IsPlayer() then
            local weapon = attacker:GetActiveWeapon()
            if IsValid(weapon) and weapon:GetClass() == "weapon_crowbar" then
                return 2 -- Take 2x damage from crowbar
            end
        end
    end

    function ENT:OnNewEnemy()
        self:EmitSound("NPC_HeadCrab.Alert")
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Key Concepts

### 1. Range Attack Configuration

```lua
ENT.RangeAttackRange = 150    -- Maximum attack distance
ENT.MeleeAttackRange = 0      -- Disable melee (set to 0)
ENT.ReachEnemyRange = 125     -- Optimal positioning distance
ENT.AvoidEnemyRange = 100     -- Minimum safe distance
```

**How it works:**
- NPC approaches enemy until within `RangeAttackRange`
- Stays between `AvoidEnemyRange` and `ReachEnemyRange`
- Calls `OnRangeAttack()` when in position

### 2. The Leap Mechanic

```lua
function ENT:HeadcrabLeap(pos)
    self:FaceTo(pos)                          -- Turn to face target
    self:PlaySequence("jumpattack_broadcast") -- Play wind-up animation
    self:PauseCoroutine(0.5)                  -- Wait for wind-up
    self:Leap(pos, 400)                       -- Launch toward position
end
```

**self:Leap(position, speed)**
- Launches NPC toward position
- Speed in units per second
- NPC becomes airborne
- Affected by gravity

### 3. Contact Damage

```lua
function ENT:OnContact(ent)
    if self.CanBite and ent == self:GetEnemy() then
        -- Deal damage directly
        local dmg = DamageInfo()
        dmg:SetDamage(20)
        dmg:SetAttacker(self)
        ent:TakeDamageInfo(dmg)
    end
end
```

**OnContact(entity)**
- Called when NPC physically touches entity
- Perfect for leap attacks, charges, trampling
- Check state flags (like `self.CanBite`) to control when damage applies

### 4. Ranged Attack Hook

```lua
function ENT:OnRangeAttack(enemy)
    self:HeadcrabLeap(enemy:EyePos() - Vector(0, 0, 10))
end
```

**When it's called:**
- Enemy is within `RangeAttackRange`
- NPC has line of sight
- Not on cooldown

## Other Ranged Attack Types

### Projectile Attacks

```lua
ENT.RangeAttackRange = 1000
ENT.MeleeAttackRange = 0

function ENT:OnRangeAttack(enemy)
    -- Fire a projectile
    local proj = ents.Create("proj_drg_grenade")
    if not IsValid(proj) then return end

    proj:SetPos(self:GetAttachment(self:LookupAttachment("anim_attachment_RH")).Pos)
    proj:SetAngles(self:GetAngles())
    proj:SetOwner(self)
    proj:Spawn()
    proj:Activate()

    -- Launch toward enemy
    local phys = proj:GetPhysicsObject()
    if IsValid(phys) then
        local dir = (enemy:GetPos() - proj:GetPos()):GetNormalized()
        phys:SetVelocity(dir * 1000)
    end

    self:PlayActivityAndMove(ACT_RANGE_ATTACK1, 1, self.FaceEnemy)
    self:Wait(1.5) -- Cooldown between shots
end
```

### Hitscan Attacks (Instant)

```lua
function ENT:OnRangeAttack(enemy)
    self:EmitSound("Weapon_AR2.Single")

    -- Trace from eye to enemy
    local tr = util.TraceLine({
        start = self:EyePos(),
        endpos = enemy:EyePos(),
        filter = self
    })

    if tr.Hit and IsValid(tr.Entity) then
        local dmg = DamageInfo()
        dmg:SetDamage(10)
        dmg:SetAttacker(self)
        dmg:SetInflictor(self)
        dmg:SetDamageType(DMG_BULLET)
        dmg:SetDamagePosition(tr.HitPos)
        tr.Entity:TakeDamageInfo(dmg)

        -- Bullet effect
        local effectdata = EffectData()
        effectdata:SetOrigin(tr.HitPos)
        effectdata:SetNormal(tr.HitNormal)
        util.Effect("Impact", effectdata)
    end

    self:PlayActivityAndMove(ACT_RANGE_ATTACK1, 1, self.FaceEnemy)
    self:Wait(0.5)
end
```

### Charged Attacks

```lua
function ENT:OnRangeAttack(enemy)
    -- Charge up
    self:PlaySequence("charging")
    self:EmitSound("ChargeUp.Sound")
    self:Wait(2) -- 2 second charge

    -- Release
    self:EmitSound("ChargeRelease.Sound")

    -- Powerful explosion
    util.BlastDamage(self, self, enemy:GetPos(), 200, 100)

    local effectdata = EffectData()
    effectdata:SetOrigin(enemy:GetPos())
    util.Effect("Explosion", effectdata)

    self:Wait(5) -- Long cooldown
end
```

### Area Denial (Gas, Fire, etc.)

```lua
function ENT:OnRangeAttack(enemy)
    -- Create gas cloud
    local cloud = ents.Create("env_fire")
    if not IsValid(cloud) then return end

    cloud:SetPos(enemy:GetPos())
    cloud:SetKeyValue("health", "30")
    cloud:SetKeyValue("firesize", "128")
    cloud:SetKeyValue("fireattack", "5")
    cloud:SetKeyValue("damagescale", "1.0")
    cloud:Spawn()
    cloud:Fire("StartFire", "", 0)

    self:PlayActivityAndMove(ACT_RANGE_ATTACK2, 1, self.FaceEnemy)
    self:Wait(10) -- Long cooldown
end
```

## Advanced Techniques

### Predictive Aiming

```lua
function ENT:PredictPosition(target, projectile_speed)
    local target_pos = target:GetPos()
    local target_vel = target:GetVelocity()
    local distance = self:GetPos():Distance(target_pos)
    local time = distance / projectile_speed

    -- Lead target
    return target_pos + (target_vel * time)
end

function ENT:OnRangeAttack(enemy)
    local predicted_pos = self:PredictPosition(enemy, 1000)
    -- Fire at predicted position
    self:FireProjectile(predicted_pos)
end
```

### Burst Fire

```lua
function ENT:OnRangeAttack(enemy)
    for i = 1, 3 do
        self:FireBullet(enemy)
        self:Wait(0.1)
    end
    self:Wait(2) -- Cooldown between bursts
end
```

### Range-Based Behavior

```lua
function ENT:OnRangeAttack(enemy)
    local distance = self:GetPos():Distance(enemy:GetPos())

    if distance > 500 then
        self:SniperShot(enemy) -- Long range: accurate single shot
    elseif distance > 200 then
        self:BurstFire(enemy)  -- Medium range: burst fire
    else
        self:LeapAttack(enemy) -- Close range: leap
    end
end
```

## Troubleshooting

### NPC doesn't use ranged attack
- Ensure `RangeAttackRange` > 0
- Check `MeleeAttackRange` is 0 (if no melee)
- Verify `OnRangeAttack()` function exists
- Check for line of sight blockage

### Leap doesn't work
- Ensure navmesh exists
- Check ground is below NPC
- Verify collision bounds are correct
- Test on flat terrain first

### Contact damage not working
- Verify `OnContact()` function exists
- Check state flags (like `self.CanBite`)
- Ensure collision bounds overlap
- Debug with `print()` statements

## Source Reference

Source file: `lua/entities/npc_drg_headcrab.lua`

## Related Documentation

- [Animation System](09-animations.md)
- [Custom AI Behavior](06-custom-ai.md)
- [Projectile System](../reference/projectiles.md)
