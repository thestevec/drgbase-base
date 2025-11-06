# Boss NPC Example

This example demonstrates creating a complex boss encounter with multiple phases, special attacks, and dynamic behavior.

## Overview

A boss NPC typically features:
- Multiple health phases with different behaviors
- Varied attack patterns
- Minion spawning
- Environmental effects
- Health bars and announcements

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc --
ENT.PrintName = "Overseer Boss"
ENT.Category = "Boss NPCs"
ENT.Models = {"models/Combine_Strider.mdl"}
ENT.ModelScale = 0.5
ENT.CollisionBounds = Vector(40, 40, 100)
ENT.BloodColor = BLOOD_COLOR_MECH

-- Stats --
ENT.SpawnHealth = 1000  -- High health
ENT.HealthRegen = 2  -- Regenerates slowly

-- AI --
ENT.RangeAttackRange = 1500
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 800
ENT.AvoidEnemyRange = 400

-- Relationships --
ENT.Factions = {FACTION_COMBINE}

-- Movements --
ENT.RunSpeed = 180
ENT.WalkSpeed = 90
ENT.Acceleration = 300

-- Detection --
ENT.SightRange = 3000
ENT.SightFOV = 180

if SERVER then

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- Boss state
        self.BossPhase = 1
        self.NextSpecialAttack = CurTime() + 10
        self.NextMinionSpawn = CurTime() + 15
        self.InvulnerableUntil = 0

        -- Create shield effect
        self:CreateShield()

        -- Announce boss spawn
        self:AnnouncePhase("The Overseer has awakened!")
    end

    function ENT:CreateShield()
        local sprite = ents.Create("env_sprite")
        if IsValid(sprite) then
            sprite:SetKeyValue("model", "sprites/light_glow02.vmt")
            sprite:SetKeyValue("scale", "2")
            sprite:SetKeyValue("rendermode", "5")
            sprite:SetKeyValue("rendercolor", "0 150 255")
            sprite:SetPos(self:GetPos())
            sprite:SetParent(self)
            sprite:Spawn()
            self:DeleteOnRemove(sprite)
            self.ShieldSprite = sprite
        end
    end

    function ENT:AnnouncePhase(message)
        for _, ply in ipairs(player.GetAll()) do
            ply:SendLua([[
                notification.AddLegacy("]]..message..[[", NOTIFY_ERROR, 5)
                surface.PlaySound("ambient/alarms/warningbell1.wav")
            ]])
        end
    end

    -- Phase Management --

    function ENT:CustomThink()
        local health = self:Health()
        local maxHealth = self:GetMaxHealth()
        local healthPercent = (health / maxHealth) * 100

        -- Phase transitions
        if healthPercent <= 50 and self.BossPhase == 1 then
            self:EnterPhase2()
        elseif healthPercent <= 25 and self.BossPhase == 2 then
            self:EnterPhase3()
        end

        -- Special attacks
        if CurTime() >= self.NextSpecialAttack and IsValid(self:GetEnemy()) then
            self:SpecialAttack()
            self.NextSpecialAttack = CurTime() + math.random(8, 12)
        end

        -- Spawn minions
        if CurTime() >= self.NextMinionSpawn then
            self:SpawnMinions()
            self.NextMinionSpawn = CurTime() + 20
        end
    end

    function ENT:EnterPhase2()
        self.BossPhase = 2
        self:AnnouncePhase("The Overseer enters Phase 2!")

        -- Heal partially
        self:SetHealth(self:Health() + 200)

        -- Speed boost
        self.RunSpeed = 250
        self.RangeAttackRange = 2000

        -- Temporary invulnerability
        self.InvulnerableUntil = CurTime() + 3

        -- Visual effect
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        effectData:SetMagnitude(5)
        effectData:SetScale(3)
        util.Effect("Explosion", effectData)
    end

    function ENT:EnterPhase3()
        self.BossPhase = 3
        self:AnnouncePhase("The Overseer is enraged! Final Phase!")

        -- Massive heal
        self:SetHealth(self:Health() + 300)

        -- Maximum aggression
        self.RunSpeed = 300
        self.RangeAttackRange = 2500
        self.HealthRegen = 5

        -- Remove shield, add rage aura
        if IsValid(self.ShieldSprite) then
            self.ShieldSprite:Remove()
        end

        local sprite = ents.Create("env_sprite")
        if IsValid(sprite) then
            sprite:SetKeyValue("model", "sprites/redglow2.vmt")
            sprite:SetKeyValue("scale", "3")
            sprite:SetKeyValue("rendermode", "5")
            sprite:SetPos(self:GetPos())
            sprite:SetParent(self)
            sprite:Spawn()
            self:DeleteOnRemove(sprite)
        end
    end

    -- Attacks --

    function ENT:OnRangeAttack(enemy)
        if self.BossPhase == 3 then
            -- Phase 3: Rapid fire
            for i = 1, 3 do
                self:FireProjectile(enemy)
                self:PauseCoroutine(0.2)
            end
            self:PauseCoroutine(0.8)
        else
            -- Normal attack
            self:FireProjectile(enemy)
            self:PauseCoroutine(1.2)
        end
    end

    function ENT:FireProjectile(enemy)
        self:FaceTo(enemy:EyePos())

        local proj = ents.Create("proj_drg_plasma")
        if IsValid(proj) then
            local startPos = self:GetPos() + Vector(0, 0, 50)
            local dir = (enemy:EyePos() - startPos):GetNormalized()

            proj:SetPos(startPos)
            proj:SetAngles(dir:Angle())
            proj:Spawn()
            proj:SetOwner(self)
            proj:SetModelScale(1.5)  -- Larger projectiles

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(dir * 800)
            end
        end

        self:EmitSound("weapons/physcannon/energy_sing_explosion2.wav", 85)
    end

    function ENT:SpecialAttack()
        local enemy = self:GetEnemy()
        if not IsValid(enemy) then return end

        if self.BossPhase == 1 then
            -- Phase 1: Shockwave
            self:Shockwave()
        elseif self.BossPhase == 2 then
            -- Phase 2: Orbital strike
            self:OrbitalStrike(enemy)
        else
            -- Phase 3: AOE barrage
            self:AOEBarrage(enemy)
        end
    end

    function ENT:Shockwave()
        self:EmitSound("ambient/energy/weld1.wav")

        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        effectData:SetScale(500)
        util.Effect("Explosion", effectData)

        -- Damage and push nearby entities
        for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
            if ent ~= self and (ent:IsPlayer() or ent:IsNPC()) then
                local dmg = DamageInfo()
                dmg:SetDamage(30)
                dmg:SetAttacker(self)
                dmg:SetInflictor(self)
                dmg:SetDamageType(DMG_BLAST)
                ent:TakeDamageInfo(dmg)

                -- Push away
                if IsValid(ent:GetPhysicsObject()) then
                    local dir = (ent:GetPos() - self:GetPos()):GetNormalized()
                    ent:SetVelocity(dir * 500 + Vector(0, 0, 200))
                end
            end
        end
    end

    function ENT:OrbitalStrike(target)
        local targetPos = target:GetPos()

        -- Warning indicator (would need custom effect in real use)
        self:EmitSound("npc/scanner/combat_scan5.wav")

        -- Delay before strike
        timer.Simple(1.5, function()
            if not IsValid(self) then return end

            -- Create explosion at target
            local effectData = EffectData()
            effectData:SetOrigin(targetPos)
            effectData:SetScale(3)
            util.Effect("Explosion", effectData)

            util.BlastDamage(self, self, targetPos, 300, 80)
            self:EmitSound("weapons/stinger_fire1.wav")
        end)
    end

    function ENT:AOEBarrage(target)
        for i = 1, 5 do
            timer.Simple(i * 0.3, function()
                if not IsValid(self) or not IsValid(target) then return end

                local randomOffset = Vector(
                    math.random(-200, 200),
                    math.random(-200, 200),
                    0
                )

                local proj = ents.Create("proj_drg_grenade")
                if IsValid(proj) then
                    local startPos = self:GetPos() + Vector(0, 0, 50)
                    local targetPos = target:GetPos() + randomOffset

                    proj:SetPos(startPos)
                    proj:Spawn()
                    proj:SetOwner(self)

                    local phys = proj:GetPhysicsObject()
                    if IsValid(phys) then
                        local dir = (targetPos - startPos):GetNormalized()
                        phys:SetVelocity(dir * 600 + Vector(0, 0, 300))
                    end

                    if proj.Unpin then proj:Unpin() end
                end
            end)
        end
    end

    function ENT:SpawnMinions()
        if self.BossPhase < 2 then return end  -- Only phase 2+

        local count = self.BossPhase  -- More minions in later phases
        for i = 1, count do
            local minion = ents.Create("npc_drg_zombie")  -- or custom minion
            if IsValid(minion) then
                local offset = Vector(math.random(-100, 100), math.random(-100, 100), 0)
                minion:SetPos(self:GetPos() + offset)
                minion:Spawn()

                -- Mark as boss minion
                minion.IsBossMinion = true
            end
        end

        self:EmitSound("npc/combine_gunship/gunship_weapon_fire_loop6.wav")
    end

    -- Damage --

    function ENT:OnTakeDamage(dmg)
        -- Invulnerability during phase transitions
        if CurTime() < self.InvulnerableUntil then
            return false
        end

        -- Take reduced damage in phase 1 (shield)
        if self.BossPhase == 1 then
            return 0.7  -- 70% damage
        end

        return true
    end

    function ENT:OnDeath(dmg, hitgroup)
        -- Epic death
        self:AnnouncePhase("The Overseer has been defeated!")

        -- Multiple explosions
        for i = 1, 5 do
            timer.Simple(i * 0.2, function()
                if not IsValid(self) then return end

                local effectData = EffectData()
                effectData:SetOrigin(self:GetPos() + VectorRand() * 50)
                effectData:SetScale(2)
                util.Effect("Explosion", effectData)
                self:EmitSound("ambient/explosions/explode_" .. math.random(1, 9) .. ".wav")
            end)
        end

        -- Remove minions
        for _, ent in ipairs(ents.GetAll()) do
            if ent.IsBossMinion then
                timer.Simple(1, function()
                    if IsValid(ent) then ent:Remove() end
                end)
            end
        end
    end

end

-- Register --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Key Boss Features

### Health Phases
```lua
if healthPercent <= 50 and self.BossPhase == 1 then
    self:EnterPhase2()
end
```

### Special Abilities
- Shockwave AOE
- Orbital strikes
- Minion spawning
- Phase-specific attacks

### Dynamic Difficulty
```lua
-- Stats change per phase
self.RunSpeed = 250  -- Faster in phase 2
self.HealthRegen = 5  -- More regen in phase 3
```

## Customization Examples

### Add Healing Phase
```lua
function ENT:EnterPhase2()
    -- Become invulnerable and heal
    self.InvulnerableUntil = CurTime() + 5

    timer.Create("BossHeal_"..self:EntIndex(), 0.1, 50, function()
        if not IsValid(self) then return end
        self:SetHealth(math.min(self:Health() + 20, self:GetMaxHealth()))
    end)
end
```

### Enrage Timer
```lua
function ENT:CustomInitialize()
    self.EnrageTime = CurTime() + 180  -- 3 minutes
end

function ENT:CustomThink()
    if CurTime() >= self.EnrageTime and not self.Enraged then
        self.Enraged = true
        self:AnnouncePhase("Time's up! The boss is enraged!")
        self.RunSpeed = self.RunSpeed * 2
        self.HealthRegen = 10
    end
end
```

## Performance Tips

1. **Limit minions** - Don't spawn too many at once
2. **Use timers carefully** - Clean up on death
3. **Optimize effects** - Don't spam particle effects

## Next Steps

- Learn [Advanced AI](./advanced-ai.md) for complex behaviors
- Try [Faction System](./faction-system.md) for boss allies
- Explore [Custom Weapons](./custom-weapon.md) for unique boss weapons
