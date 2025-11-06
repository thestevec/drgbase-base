# Melee Combat System

## Overview

This guide covers advanced melee combat techniques including attack animations, damage timing, combos, and special effects.

## Basic Melee Attack

### Simple Implementation

```lua
ENT.Base = "drgbase_nextbot"
ENT.MeleeAttackRange = 50
ENT.ReachEnemyRange = 50

function ENT:OnMeleeAttack(enemy)
    self:EmitSound("NPC.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({
            damage = 10,
            type = DMG_SLASH
        })
    end
end
```

### Key Components

**OnMeleeAttack(enemy)**
- Called when NPC is in melee range
- Play attack animation
- Emit attack sound
- Setup attack state

**OnAnimEvent()**
- Called on animation events
- Check if attacking
- Check animation timing
- Execute damage

**self:Attack({...})**
- Area damage in front of NPC
- Parameters: damage, type, range, callback

## Advanced Melee Techniques

### 1. Precise Attack Timing

```lua
function ENT:OnMeleeAttack(enemy)
    self:EmitSound("Zombie.Attack")
    self.AttackFrame = 0.3 -- Damage at 30% through animation
    self.AttackExecuted = false
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and not self.AttackExecuted then
        local cycle = self:GetCycle()
        if cycle >= self.AttackFrame then
            self.AttackExecuted = true
            self:Attack({
                damage = 10,
                type = DMG_SLASH,
                range = 50
            }, function(self, hit)
                if #hit > 0 then
                    self:EmitSound("Zombie.AttackHit")
                    -- Hit feedback
                    self:SetPlaybackRate(1.2) -- Speed up slightly
                else
                    self:EmitSound("Zombie.AttackMiss")
                end
            end)
        end
    end
end
```

### 2. Multi-Hit Combos

```lua
function ENT:CustomInitialize()
    self.ComboAttacks = {
        {anim = ACT_MELEE_ATTACK1, damage = 10, frame = 0.3},
        {anim = ACT_MELEE_ATTACK2, damage = 15, frame = 0.4},
        {anim = "swing_special", damage = 25, frame = 0.5}
    }
    self.ComboIndex = 1
end

function ENT:OnMeleeAttack(enemy)
    local attack = self.ComboAttacks[self.ComboIndex]

    self:EmitSound("Swing" .. self.ComboIndex)
    self.AttackFrame = attack.frame
    self.AttackDamage = attack.damage
    self.AttackExecuted = false

    self:PlayActivityAndMove(attack.anim, 1, self.FaceEnemy)

    -- Advance combo
    self.ComboIndex = self.ComboIndex + 1
    if self.ComboIndex > #self.ComboAttacks then
        self.ComboIndex = 1
    end

    -- Reset combo after delay
    self:Timer(2, function(self)
        self.ComboIndex = 1
    end)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and not self.AttackExecuted then
        if self:GetCycle() >= self.AttackFrame then
            self.AttackExecuted = true
            self:Attack({
                damage = self.AttackDamage,
                type = DMG_SLASH,
                range = 50
            })
        end
    end
end
```

### 3. Attack Variations

```lua
function ENT:OnMeleeAttack(enemy)
    local attacks = {
        {anim = ACT_MELEE_ATTACK1, type = DMG_SLASH, sound = "Slash"},
        {anim = ACT_MELEE_ATTACK2, type = DMG_CLUB, sound = "Punch"},
        {anim = "swing_heavy", type = DMG_CRUSH, sound = "Smash"}
    }

    local attack = attacks[math.random(#attacks)]

    self:EmitSound(attack.sound)
    self.CurrentAttackType = attack.type
    self:PlayActivityAndMove(attack.anim, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({
            damage = 15,
            type = self.CurrentAttackType or DMG_SLASH
        })
    end
end
```

### 4. Directional Attacks

```lua
function ENT:OnMeleeAttack(enemy)
    local mypos = self:GetPos()
    local enemypos = enemy:GetPos()
    local toEnemy = (enemypos - mypos):GetNormalized()
    local myAngles = self:GetAngles()
    local forward = myAngles:Forward()
    local right = myAngles:Right()

    -- Calculate attack direction
    local dotForward = toEnemy:Dot(forward)
    local dotRight = toEnemy:Dot(right)

    local anim = ACT_MELEE_ATTACK1
    if math.abs(dotRight) > math.abs(dotForward) then
        if dotRight > 0 then
            anim = "swing_right"
        else
            anim = "swing_left"
        end
    end

    self:PlayActivityAndMove(anim, 1, self.FaceEnemy)
end
```

### 5. Charge Attacks

```lua
ENT.ChargeTime = 0

function ENT:OnMeleeAttack(enemy)
    -- Start charging
    self.IsCharging = true
    self.ChargeTime = 0
    self:PlaySequence("charge_start")

    self:Wait(1.5) -- Hold for 1.5 seconds

    -- Release charge
    self.IsCharging = false
    local damage = 10 + (self.ChargeTime * 20) -- 10-40 damage

    self:EmitSound("ChargeRelease")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK_SWING, 1.5, self.FaceEnemy)

    self:Attack({
        damage = damage,
        type = DMG_CRUSH,
        range = 80 -- Longer range for charged attack
    })
end

function ENT:CustomThink()
    if self.IsCharging then
        self.ChargeTime = self.ChargeTime + 0.1
    end
end
```

### 6. Area Cleave Attacks

```lua
function ENT:OnMeleeAttack(enemy)
    self:EmitSound("GreatSword.Swing")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK_SWING, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        -- Hit multiple enemies in front
        local enemies = self:FindEnemiesInRange(100)

        for _, enemy in ipairs(enemies) do
            if self:IsInFront(enemy, 0.5) then -- 90 degree cone
                local dmg = DamageInfo()
                dmg:SetDamage(20)
                dmg:SetAttacker(self)
                dmg:SetInflictor(self)
                dmg:SetDamageType(DMG_SLASH)
                enemy:TakeDamageInfo(dmg)

                -- Knockback
                local force = (enemy:GetPos() - self:GetPos()):GetNormalized()
                force.z = 0.5 -- Add upward component
                enemy:SetVelocity(force * 500)
            end
        end
    end
end

-- Helper function
function ENT:FindEnemiesInRange(range)
    local enemies = {}
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), range)) do
        if self:GetRelationship(ent) == D_HT and ent:Health() > 0 then
            table.insert(enemies, ent)
        end
    end
    return enemies
end

function ENT:IsInFront(ent, threshold)
    threshold = threshold or 0.7 -- cosine of angle
    local toEnt = (ent:GetPos() - self:GetPos()):GetNormalized()
    local dot = toEnt:Dot(self:GetForward())
    return dot > threshold
end
```

### 7. Grab Attacks

```lua
function ENT:OnMeleeAttack(enemy)
    if self.HasGrabbedEnemy then
        -- Already grabbed, now throw
        self:ThrowEnemy()
    else
        -- Try to grab
        self:AttemptGrab(enemy)
    end
end

function ENT:AttemptGrab(enemy)
    self:EmitSound("Grab.Attempt")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)

    self:Timer(0.3, function(self)
        if self:IsInRange(enemy:GetPos(), 60) then
            -- Successful grab
            self.GrabbedEnemy = enemy
            self.HasGrabbedEnemy = true

            if enemy:IsPlayer() then
                enemy:Freeze(true)
            end

            self:PlaySequence("grab_hold")
            self:EmitSound("Grab.Success")

            -- Auto-throw after 3 seconds
            self:Timer(3, function(self)
                if self.HasGrabbedEnemy then
                    self:ThrowEnemy()
                end
            end)
        else
            self:EmitSound("Grab.Miss")
        end
    end)
end

function ENT:ThrowEnemy()
    if not IsValid(self.GrabbedEnemy) then
        self.HasGrabbedEnemy = false
        return
    end

    self:EmitSound("Throw")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK_SWING, 1.2)

    local enemy = self.GrabbedEnemy
    local forward = self:GetForward()

    -- Damage
    local dmg = DamageInfo()
    dmg:SetDamage(30)
    dmg:SetAttacker(self)
    dmg:SetDamageType(DMG_CRUSH)
    enemy:TakeDamageInfo(dmg)

    -- Launch
    if enemy:IsPlayer() then
        enemy:Freeze(false)
    end

    enemy:SetVelocity(forward * 600 + Vector(0, 0, 300))

    self.GrabbedEnemy = nil
    self.HasGrabbedEnemy = false
end

function ENT:OnDeath()
    -- Release grabbed enemy on death
    if self.HasGrabbedEnemy and IsValid(self.GrabbedEnemy) then
        if self.GrabbedEnemy:IsPlayer() then
            self.GrabbedEnemy:Freeze(false)
        end
    end
end
```

## Special Effects

### Blood Effects

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({
            damage = 10,
            type = DMG_SLASH
        }, function(self, hit)
            if #hit > 0 then
                for _, ent in ipairs(hit) do
                    -- Blood effect
                    local effectdata = EffectData()
                    effectdata:SetOrigin(ent:GetPos() + Vector(0, 0, 40))
                    effectdata:SetNormal(self:GetForward())
                    effectdata:SetColor(BLOOD_COLOR_RED)
                    util.Effect("BloodImpact", effectdata)

                    -- Decal
                    util.Decal("Blood", ent:GetPos(), ent:GetPos() - Vector(0, 0, 100))
                end
            end
        end)
    end
end
```

### Screen Shake

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.5 then
        self:Attack({
            damage = 25,
            type = DMG_CRUSH
        }, function(self, hit)
            -- Shake for nearby players
            util.ScreenShake(self:GetPos(), 10, 5, 0.5, 300)

            -- Ground effect
            local effectdata = EffectData()
            effectdata:SetOrigin(self:GetPos())
            util.Effect("cball_explode", effectdata)
        end)
    end
end
```

### Particle Effects

```lua
function ENT:OnMeleeAttack(enemy)
    -- Fire weapon trail
    ParticleEffectAttach("fire_trail", PATTACH_POINT_FOLLOW, self, self:LookupAttachment("anim_attachment_RH"))

    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)

    -- Remove after attack
    self:Timer(1, function(self)
        self:StopParticles()
    end)
end
```

## Attack Modifiers

### Critical Hits

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        local isCrit = math.random(100) <= 15 -- 15% crit chance

        self:Attack({
            damage = isCrit and 30 or 10,
            type = DMG_SLASH
        }, function(self, hit)
            if isCrit and #hit > 0 then
                self:EmitSound("CriticalHit")
                -- Crit effect
                for _, ent in ipairs(hit) do
                    local effectdata = EffectData()
                    effectdata:SetOrigin(ent:GetPos() + Vector(0, 0, 50))
                    util.Effect("crit_effect", effectdata)
                end
            end
        end)
    end
end
```

### Backstab Bonus

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        local enemy = self:GetEnemy()
        local damage = 10

        if IsValid(enemy) then
            -- Check if attacking from behind
            local toBehind = (self:GetPos() - enemy:GetPos()):GetNormalized()
            local dot = toBehind:Dot(enemy:GetForward())

            if dot > 0.7 then -- ~45 degree cone behind
                damage = damage * 3 -- Triple damage from behind
                self:EmitSound("Backstab")
            end
        end

        self:Attack({
            damage = damage,
            type = DMG_SLASH
        })
    end
end
```

## Troubleshooting

### Attack happens too early/late
- Adjust `self:GetCycle()` threshold in `OnAnimEvent()`
- Use `print(self:GetCycle())` to debug timing
- Check animation speed with `self:SetPlaybackRate()`

### Multiple hits per attack
- Use flag like `self.AttackExecuted` to prevent duplicate hits
- Reset flag when attack animation ends

### Attack misses when it shouldn't
- Increase `range` parameter in `self:Attack()`
- Check collision bounds with `ent_bbox`
- Verify NPC is facing target

### Animation doesn't play
- Check activity name with `self:GetAnimInfo()`
- Use sequence name if activity doesn't exist
- Verify model has the animation

## Source Reference

Source: `lua/entities/npc_drg_zombie.lua` (lines 69-117)

## Related Documentation

- [Animation System](09-animations.md)
- [Damage System](../reference/damage.md)
- [Effects and Particles](../reference/effects.md)
