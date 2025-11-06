# Custom Projectile Example

This example demonstrates creating custom projectile entities for use with DrGBase NPCs. Projectiles can have custom models, behaviors, effects, and damage.

## Overview

DrGBase projectiles inherit from `proj_drg_default` and support:
- Custom models and effects
- Physics-based or hitscan behavior
- Impact events and callbacks
- Visual and sound effects
- Custom damage logic

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

-- Misc --
ENT.PrintName = "Fire Bolt"
ENT.Category = "Custom Projectiles"
ENT.Models = {"models/Items/AR2_Grenade.mdl"}
ENT.Spawnable = false  -- Don't show in spawn menu

-- Physics --
ENT.Gravity = false  -- Disable gravity
ENT.Physgun = false
ENT.Gravgun = false

-- Visual Effects --
ENT.AttachEffects = {"fire_medium_01"}  -- Particle effect
ENT.OnContactEffects = {"fire_large_01"}
ENT.OnRemoveEffects = {}

-- Decals --
ENT.OnContactDecals = {"Scorch"}

-- Sounds --
ENT.LoopSounds = {"ambient/fire/fire_small_loop1.wav"}
ENT.OnContactSounds = {"ambient/fire/ignite.wav"}
ENT.OnRemoveSounds = {}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Set color/material
        self:SetColor(Color(255, 100, 0))
        self:SetMaterial("models/debug/debugwhite")

        -- Add dynamic light
        self:DynamicLight(Color(255, 100, 0), 200, 0.1)

        -- Auto-remove after 10 seconds
        timer.Simple(10, function()
            if IsValid(self) then self:Remove() end
        end)
    end

    function ENT:CustomThink()
        -- Trail effect
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        effectData:SetStart(self:GetPos())
        effectData:SetScale(0.5)
        util.Effect("VortDispel", effectData)
    end

    function ENT:OnContact(ent)
        -- Deal damage on impact
        if IsValid(ent) and (ent:IsPlayer() or ent:IsNPC()) then
            self:DealDamage(ent, 40, DMG_BURN)
        end

        -- Ignite target
        if IsValid(ent) and ent:IsOnFire() == false then
            ent:Ignite(5)
        end

        -- Small AOE damage
        util.BlastDamage(self, self:GetOwner(), self:GetPos(), 100, 20)

        -- Remove projectile
        self:Remove()
    end

end
```

## Key Properties

### Physics Behavior
```lua
ENT.Gravity = false  -- Projectile flies straight
ENT.Gravity = true   -- Projectile arcs like grenade
```

### Effects
```lua
ENT.AttachEffects = {"fire_medium_01"}      -- While flying
ENT.OnContactEffects = {"explosion_huge"}   -- On impact
ENT.OnRemoveEffects = {"explosion_small"}   -- When removed
```

### Sounds
```lua
ENT.LoopSounds = {"sound.wav"}          -- Continuous sound
ENT.OnContactSounds = {"impact.wav"}    -- Impact sound
ENT.OnRemoveSounds = {"remove.wav"}     -- Removal sound
```

## Customization Examples

### Homing Missile
```lua
function ENT:CustomInitialize()
    self.HomingTarget = self:GetOwner():GetEnemy()
end

function ENT:CustomThink()
    if not IsValid(self.HomingTarget) then return end

    local phys = self:GetPhysicsObject()
    if not IsValid(phys) then return end

    -- Turn towards target
    local dir = (self.HomingTarget:EyePos() - self:GetPos()):GetNormalized()
    local currentVel = phys:GetVelocity()
    local newVel = LerpVector(0.05, currentVel, dir * 600)

    phys:SetVelocity(newVel)
    self:SetAngles(newVel:Angle())
end
```

### Bouncing Projectile
```lua
ENT.Bounces = 3

function ENT:CustomInitialize()
    self.BouncesLeft = self.Bounces
end

function ENT:OnContact(ent)
    if self.BouncesLeft > 0 then
        -- Bounce off surfaces
        self.BouncesLeft = self.BouncesLeft - 1

        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            local vel = phys:GetVelocity()
            phys:SetVelocity(vel * 0.7)  -- Lose some velocity
        end

        self:EmitSound("weapons/physcannon/energy_bounce1.wav")
    else
        -- Explode after last bounce
        self:Explode()
        self:Remove()
    end
end

function ENT:Explode()
    util.BlastDamage(self, self:GetOwner(), self:GetPos(), 200, 50)

    local effectData = EffectData()
    effectData:SetOrigin(self:GetPos())
    util.Effect("Explosion", effectData)
end
```

### Sticky Projectile
```lua
function ENT:OnContact(ent)
    -- Stick to surface
    self:SetMoveType(MOVETYPE_NONE)
    self:SetParent(ent)

    self:EmitSound("weapons/crossbow/hit1.wav")

    -- Explode after delay
    timer.Simple(2, function()
        if not IsValid(self) then return end

        util.BlastDamage(self, self:GetOwner(), self:GetPos(), 150, 60)

        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        util.Effect("Explosion", effectData)

        self:Remove()
    end)
end
```

### Piercing Projectile
```lua
ENT.MaxPierces = 3

function ENT:CustomInitialize()
    self.PierceCount = 0
    self.HitEntities = {}
end

function ENT:OnContact(ent)
    -- Don't hit same entity twice
    if self.HitEntities[ent] then return end
    self.HitEntities[ent] = true

    if IsValid(ent) and (ent:IsPlayer() or ent:IsNPC()) then
        self:DealDamage(ent, 30, DMG_BULLET)
        self.PierceCount = self.PierceCount + 1

        if self.PierceCount >= self.MaxPierces then
            self:Remove()
        end
    else
        -- Hit world, remove
        self:Remove()
    end
end
```

### Cluster Projectile
```lua
function ENT:OnContact(ent)
    -- Spawn cluster projectiles
    for i = 1, 8 do
        timer.Simple(i * 0.05, function()
            if not IsValid(self) then return end

            local cluster = ents.Create("proj_drg_plasma")
            if IsValid(cluster) then
                cluster:SetPos(self:GetPos())
                cluster:Spawn()
                cluster:SetOwner(self:GetOwner())

                local phys = cluster:GetPhysicsObject()
                if IsValid(phys) then
                    local angle = (360 / 8) * i
                    local dir = Vector(
                        math.cos(math.rad(angle)),
                        math.sin(math.rad(angle)),
                        0.2
                    ):GetNormalized()

                    phys:SetVelocity(dir * 400)
                end
            end
        end)
    end

    self:Remove()
end
```

### Lingering Effect
```lua
function ENT:OnContact(ent)
    -- Create damage field
    local damageField = ents.Create("env_fire")
    if IsValid(damageField) then
        damageField:SetPos(self:GetPos())
        damageField:SetKeyValue("health", "30")  -- Duration
        damageField:SetKeyValue("firesize", "128")
        damageField:SetKeyValue("fireattack", "10")
        damageField:SetKeyValue("damagescale", "1.0")
        damageField:Spawn()
        damageField:Activate()
    end

    self:Remove()
end
```

## Common Issues and Solutions

### Issue: Projectile Doesn't Move

**Solutions:**
1. Check physics object is valid
2. Verify velocity is set: `phys:SetVelocity(dir * 500)`
3. Make sure `ENT.Gravity` setting is appropriate

### Issue: Effects Don't Show

**Solutions:**
1. Verify effect names are correct (case-sensitive)
2. Check AddCSLuaFile() is called
3. Effect names: `"fire_medium_01"`, `"explosion_huge"`, etc.

### Issue: Projectile Hits Owner

**Solution:**
```lua
function ENT:OnContact(ent)
    if ent == self:GetOwner() then return end  -- Ignore owner
    -- Damage logic
end
```

### Issue: Projectile Disappears Immediately

**Solutions:**
1. Don't call `self:Remove()` in `CustomInitialize`
2. Check `OnContact` isn't removing too early
3. Add timer to prevent instant removal:
```lua
function ENT:CustomInitialize()
    self.SpawnTime = CurTime()
end

function ENT:OnContact(ent)
    if CurTime() - self.SpawnTime < 0.1 then return end  -- Grace period
    -- Contact logic
end
```

## Performance Tips

1. **Set lifetime** - Always remove projectiles eventually
```lua
timer.Simple(10, function()
    if IsValid(self) then self:Remove() end
end)
```

2. **Limit particles** - Don't use too many AttachEffects
3. **Clean up timers** - Remove timers when projectile is removed
```lua
function ENT:OnRemove()
    timer.Remove("ProjectileTimer_"..self:EntIndex())
end
```

4. **Optimize CustomThink** - Don't run expensive code every tick

## Next Steps

- Try [Ranged NPC Example](./ranged-npc.md) to use your projectiles
- Learn [Boss NPC](./boss-npc.md) for complex projectile patterns
- Explore [Advanced AI](./advanced-ai.md) for smarter projectile aiming
