# Creating Projectiles

This guide teaches you how to create custom projectile entities using DrGBase. Projectiles are physics-based entities that can deal damage, explode, and have various effects when they collide with objects or entities.

## Table of Contents

1. [Introduction to Projectile System](#introduction-to-projectile-system)
2. [Creating a Basic Projectile](#creating-a-basic-projectile)
3. [Projectile Properties](#projectile-properties)
4. [Collision Handling](#collision-handling)
5. [Visual Effects and Trails](#visual-effects-and-trails)
6. [Explosive Projectiles](#explosive-projectiles)
7. [Homing Projectiles](#homing-projectiles)
8. [Complete Examples](#complete-examples)
9. [Best Practices](#best-practices)

---

## Introduction to Projectile System

DrGBase provides a powerful projectile system built on top of the entity base. Projectiles inherit from `proj_drg_default`, which extends `drgbase_entity` with specialized features for projectile-based gameplay.

### Key Features

- **Physics-based movement** with optional gravity
- **Collision detection** with triggers and physics collisions
- **Built-in damage dealing** with radius damage support
- **Automatic cleanup** with configurable delay
- **Effect system** for sounds, particles, and decals
- **Helper functions** for aiming and throwing
- **Network optimization** with configurable tickrate

### Projectile Types

DrGBase supports various projectile types:

- **Direct projectiles** - Straight-line projectiles (rockets, plasma)
- **Ballistic projectiles** - Arc-based projectiles affected by gravity (grenades, arrows)
- **Explosive projectiles** - Projectiles that explode on impact
- **Homing projectiles** - Projectiles that track targets
- **Persistent projectiles** - Projectiles that stay in the world (sticky grenades)

---

## Creating a Basic Projectile

Let's start with a simple projectile entity. Create a new file in `lua/entities/`:

**File:** `lua/entities/proj_my_rocket.lua`

```lua
if not DrGBase then return end -- Safety check

-- Inherit from the projectile base
ENT.Base = "proj_drg_default"

-- Basic Info
ENT.PrintName = "My Rocket"
ENT.Category = "My Addon"
ENT.Spawnable = true
ENT.AdminOnly = false

-- Model
ENT.Models = {"models/weapons/w_missile_closed.mdl"}
ENT.ModelScale = 1

-- Physics
ENT.Gravity = false  -- No gravity for rockets
ENT.Physgun = false  -- Can't be picked up with physgun
ENT.Gravgun = false  -- Can't be picked up with gravgun

-- Contact Behavior
ENT.OnContactDelay = 0.1      -- Delay between contact callbacks
ENT.OnContactDelete = 0       -- Delete immediately on contact (0), after delay (>0), or never (-1)

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Set the initial velocity
        self:SetVelocity(self:GetForward() * 1000)
    end

    function ENT:OnContact(ent)
        -- Deal damage on contact
        if IsValid(ent) and not ent:IsWorld() then
            self:DealDamage(ent, 50, DMG_BLAST)
        end
    end
end
```

### Spawning Your Projectile

To spawn your projectile from code:

```lua
local rocket = ents.Create("proj_my_rocket")
rocket:SetPos(self:GetPos() + self:GetForward() * 50)
rocket:SetAngles(self:GetAngles())
rocket:SetOwner(self)  -- Set owner for damage attribution
rocket:Spawn()
rocket:Activate()
```

---

## Projectile Properties

Projectiles have many configurable properties that control their behavior.

### Model Properties

```lua
-- Single model
ENT.Models = {"models/weapons/w_missile_closed.mdl"}

-- Multiple models (random selection)
ENT.Models = {
    "models/weapons/w_grenade.mdl",
    "models/props_junk/watermelon01.mdl"
}

-- Model scale
ENT.ModelScale = 1.5  -- 150% size
```

### Physics Properties

```lua
-- Gravity
ENT.Gravity = true   -- Affected by gravity (grenades, arrows)
ENT.Gravity = false  -- No gravity (rockets, energy projectiles)

-- Interaction
ENT.Physgun = true   -- Can be picked up with physgun
ENT.Gravgun = true   -- Can be picked up with gravgun
```

### Contact Properties

```lua
-- Contact delay (prevents rapid-fire contact callbacks)
ENT.OnContactDelay = 0.1  -- Wait 0.1 seconds between contacts

-- Auto-delete behavior
ENT.OnContactDelete = 0    -- Delete immediately on contact
ENT.OnContactDelete = 2.5  -- Delete 2.5 seconds after contact
ENT.OnContactDelete = -1   -- Never auto-delete
```

### Decal Properties

```lua
-- Decals left on world geometry
ENT.OnContactDecals = {"Scorch"}           -- Single decal
ENT.OnContactDecals = {"BeerSplash", "Impact.Concrete"}  -- Random selection
ENT.OnContactDecals = {}                   -- No decals
```

### Velocity Control

```lua
function ENT:CustomInitialize()
    -- Set initial velocity
    self:SetVelocity(self:GetForward() * 1500)

    -- Or aim at a target
    local dir = self:AimAt(target:GetPos(), 1000)
    self:SetVelocity(dir)

    -- Or throw with gravity
    local dir = self:ThrowAt(target:GetPos(), {magnitude = 800})
    self:SetVelocity(dir)
end
```

---

## Collision Handling

Projectiles detect collisions through two mechanisms: physics collisions and trigger touches.

### Contact Callback

The `OnContact` function is called when the projectile touches something:

```lua
function ENT:OnContact(ent)
    -- ent is the entity we hit (or World entity for world geometry)

    -- Check if we hit the world
    if ent:IsWorld() then
        print("Hit a wall!")
        return
    end

    -- Check if we hit a specific entity type
    if ent:IsPlayer() then
        -- Hit a player
        self:DealDamage(ent, 75, DMG_BULLET)
    elseif ent:IsNPC() then
        -- Hit an NPC
        self:DealDamage(ent, 50, DMG_BLAST)
    end

    -- Return false to prevent default behavior
    -- (sounds, effects, auto-delete)
    -- return false
end
```

### Dealing Damage

DrGBase provides convenient damage functions:

```lua
-- Deal direct damage to a single entity
self:DealDamage(ent, damage, damageType)

-- Parameters:
-- ent: The entity to damage
-- damage: Amount of damage (number)
-- damageType: Damage type constant (DMG_BULLET, DMG_BLAST, etc.)
```

**Examples:**

```lua
-- Bullet damage
self:DealDamage(ent, 25, DMG_BULLET)

-- Explosive damage
self:DealDamage(ent, 100, DMG_BLAST)

-- Fire damage
self:DealDamage(ent, 10, DMG_BURN)

-- Multiple damage types
self:DealDamage(ent, 50, DMG_SHOCK + DMG_DISSOLVE)

-- Simple direct damage
self:DealDamage(ent, 30, DMG_DIRECT)
```

### Radius Damage

For area-of-effect damage:

```lua
-- Deal damage to all entities in a radius
self:RadiusDamage(damage, damageType, range, filter)

-- Parameters:
-- damage: Maximum damage at center
-- damageType: Damage type constant
-- range: Damage radius
-- filter: Optional function to filter targets
```

**Examples:**

```lua
-- Simple radius damage
function ENT:OnContact(ent)
    -- 100 damage within 250 units
    self:RadiusDamage(100, DMG_BLAST, 250)
end

-- Custom filter
function ENT:OnContact(ent)
    self:RadiusDamage(150, DMG_BLAST, 300, function(target)
        -- Only damage players
        return target:IsPlayer()
    end)
end

-- Damage falls off with distance (automatic)
-- At center: 100 damage
-- At 50% range: 50 damage
-- At edge: 0 damage
```

### Preventing Self-Damage

The default filter automatically prevents damaging the projectile's owner:

```lua
function ENT:SpawnFunction(ply, tr, class)
    local proj = ents.Create(class)
    proj:SetOwner(ply)  -- Owner won't be damaged by this projectile
    proj:SetPos(tr.HitPos)
    proj:Spawn()
    return proj
end
```

---

## Visual Effects and Trails

DrGBase projectiles support various visual effects.

### Sound Effects

```lua
-- Loop sounds (play continuously)
ENT.LoopSounds = {"weapons/rpg/rocket1.wav"}

-- Contact sounds (play on collision)
ENT.OnContactSounds = {
    "weapons/explode3.wav",
    "weapons/explode4.wav"
}

-- Remove sounds (play when deleted)
ENT.OnRemoveSounds = {"ambient/explosions/exp1.wav"}
```

### Particle Effects

```lua
-- Attached effects (follow the projectile)
ENT.AttachEffects = {"fire_small"}

-- Contact effects (spawn on collision)
ENT.OnContactEffects = {"explosion_huge"}

-- Remove effects (spawn when deleted)
ENT.OnRemoveEffects = {"drg_plasma_explosion"}
```

### Complete Effect Example

```lua
ENT.Base = "proj_drg_default"
ENT.PrintName = "Plasma Bolt"

-- Model
ENT.Models = {}  -- Invisible model
ENT.Gravity = false

-- Effects
ENT.AttachEffects = {"drg_plasma_ball"}  -- Glowing plasma trail
ENT.OnContactDecals = {"Scorch"}         -- Burn mark on walls
ENT.OnContactSounds = {"weapons/stunstick/stunstick_fleshhit1.wav"}
ENT.OnContactEffects = {"cball_explode"}  -- Explosion effect

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Add a dynamic light
        self:DynamicLight(Color(0, 150, 255), 300, 1)

        -- Set velocity
        self:SetVelocity(self:GetForward() * 2000)
    end

    function ENT:OnContact(ent)
        self:DealDamage(ent, 75, DMG_SHOCK + DMG_DISSOLVE)
    end
end
```

### Custom Client Effects

You can add custom effects on the client:

```lua
if CLIENT then
    function ENT:CustomDraw()
        -- Custom rendering
        render.SetMaterial(Material("sprites/light_glow02_add"))
        render.DrawSprite(self:GetPos(), 64, 64, Color(255, 100, 0))
    end

    function ENT:CustomThink()
        -- Spawn client-side particles
        local emitter = ParticleEmitter(self:GetPos())
        if emitter then
            local particle = emitter:Add("effects/spark", self:GetPos())
            if particle then
                particle:SetVelocity(VectorRand() * 50)
                particle:SetLifeTime(0)
                particle:SetDieTime(0.5)
                particle:SetStartAlpha(255)
                particle:SetEndAlpha(0)
                particle:SetStartSize(5)
                particle:SetEndSize(0)
                particle:SetColor(255, 150, 0)
            end
            emitter:Finish()
        end
    end
end
```

---

## Explosive Projectiles

Create projectiles that explode on impact.

### Using the Explosion Helper

```lua
ENT.Base = "proj_drg_default"
ENT.PrintName = "Explosive Rocket"

ENT.Models = {"models/weapons/w_missile_closed.mdl"}
ENT.Gravity = false
ENT.OnContactDelete = 0  -- Delete immediately

-- Effects
ENT.OnContactSounds = {"weapons/explode3.wav"}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        self:SetVelocity(self:GetForward() * 1200)
    end

    function ENT:OnContact(ent)
        -- Create explosion
        -- Parameters: damage, radius, filter
        self:Explosion(150, 300)
    end
end
```

### The Explosion Function

```lua
-- Full signature
self:Explosion(damage, range, filter)
```

The `Explosion` function:
1. Creates a visual explosion effect (env_explosion)
2. Deals radius damage that falls off with distance
3. Automatically filters the owner by default
4. Plays explosion sounds from `OnContactSounds`

### Custom Explosion Effects

```lua
function ENT:OnContact(ent)
    -- Custom visual effect
    ParticleEffect("explosion_huge", self:GetPos(), Angle(0, 0, 0))

    -- Play sound
    self:EmitSound("ambient/explosions/explode_4.wav", 100, 100)

    -- Deal radius damage with custom filter
    self:RadiusDamage(200, DMG_BLAST, 400, function(target)
        -- Don't damage friendly NPCs
        if target.IsDrGNextbot then
            local owner = self:GetOwner()
            if IsValid(owner) and owner.IsDrGNextbot then
                return not owner:IsAlly(target)
            end
        end
        return true
    end)

    -- Create screen shake
    util.ScreenShake(self:GetPos(), 10, 5, 1, 500)

    -- Push physics props
    for _, prop in ipairs(ents.FindInSphere(self:GetPos(), 400)) do
        if IsValid(prop) and prop:GetPhysicsObject():IsValid() then
            local phys = prop:GetPhysicsObject()
            local dir = (prop:GetPos() - self:GetPos()):GetNormalized()
            phys:ApplyForceCenter(dir * 50000)
        end
    end
end
```

### Grenade-Style Projectiles

For grenades with delayed detonation, inherit from `proj_drg_grenade`:

```lua
ENT.Base = "proj_drg_grenade"
ENT.PrintName = "My Grenade"

ENT.Models = {"models/weapons/w_eq_fraggrenade.mdl"}

-- Grenade-specific properties
ENT.Bounce = 0.75  -- Velocity multiplier on bounce (0-1)
ENT.OnBounceSounds = {"weapons/hegrenade/he_bounce-1.wav"}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Set explosion parameters
        self:SetDamage(150)   -- Explosion damage
        self:SetRange(300)    -- Explosion radius
        self:SetDelay(3)      -- Time until detonation (seconds)

        -- Auto-unpin when spawned
        self:Unpin()
    end

    function ENT:OnDetonate()
        -- Custom explosion behavior
        self:Explosion(self:GetDamage(), self:GetRange())

        -- Or custom effects
        ParticleEffect("explosion_huge", self:GetPos(), Angle(0, 0, 0))
    end
end
```

---

## Homing Projectiles

Create projectiles that track and follow targets.

### Basic Homing Projectile

```lua
ENT.Base = "proj_drg_default"
ENT.PrintName = "Homing Missile"

ENT.Models = {"models/weapons/w_missile_closed.mdl"}
ENT.Gravity = false
ENT.OnContactDelete = 0

-- Effects
ENT.AttachEffects = {"smoke_trail"}
ENT.LoopSounds = {"weapons/rpg/rocket1.wav"}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Initial velocity
        self:SetVelocity(self:GetForward() * 1000)

        -- Store target
        self.Target = self:GetOwner():GetEyeTrace().Entity
    end

    function ENT:CustomThink()
        -- Update every tick
        self:HomeToTarget()
        return 0
    end

    function ENT:HomeToTarget()
        local target = self.Target

        -- Validate target
        if not IsValid(target) then return end
        if target:Health() <= 0 then return end

        -- Get current velocity
        local currentVel = self:GetVelocity()
        local currentSpeed = currentVel:Length()

        -- Calculate direction to target
        local targetPos = target:GetPos() + target:OBBCenter()
        local dirToTarget = (targetPos - self:GetPos()):GetNormalized()

        -- Smoothly turn toward target
        local newDir = LerpVector(0.1, currentVel:GetNormalized(), dirToTarget)

        -- Apply new velocity
        self:SetVelocity(newDir * currentSpeed)

        -- Face movement direction
        self:SetAngles(newDir:Angle())
    end

    function ENT:OnContact(ent)
        self:Explosion(100, 250)
    end
end
```

### Advanced Homing with Speed Control

```lua
ENT.Base = "proj_drg_default"
ENT.PrintName = "Smart Missile"

ENT.Models = {"models/weapons/w_missile_launch.mdl"}
ENT.Gravity = false

-- Homing parameters
ENT.HomingSpeed = 1500        -- Maximum speed
ENT.HomingTurnRate = 0.15     -- How fast it turns (0-1)
ENT.HomingAcceleration = 100  -- Speed increase per second

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        self:SetVelocity(self:GetForward() * 500)  -- Start slow
        self.CurrentSpeed = 500
        self.Target = self:FindNearestTarget()
    end

    function ENT:FindNearestTarget()
        local owner = self:GetOwner()
        local bestTarget = nil
        local bestDist = math.huge

        for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 2000)) do
            if ent == owner then continue end
            if not ent:IsNPC() and not ent:IsPlayer() then continue end
            if ent:Health() <= 0 then continue end

            local dist = self:GetPos():Distance(ent:GetPos())
            if dist < bestDist then
                bestDist = dist
                bestTarget = ent
            end
        end

        return bestTarget
    end

    function ENT:CustomThink()
        local target = self.Target

        -- Re-acquire target if lost
        if not IsValid(target) or target:Health() <= 0 then
            self.Target = self:FindNearestTarget()
            target = self.Target
        end

        if not IsValid(target) then
            -- No target, fly straight
            return 0.1
        end

        -- Accelerate
        self.CurrentSpeed = math.min(self.CurrentSpeed + self.HomingAcceleration * engine.TickInterval(), self.HomingSpeed)

        -- Calculate aim direction
        local targetPos = target:GetPos() + target:OBBCenter()
        local dirToTarget = (targetPos - self:GetPos()):GetNormalized()

        -- Smoothly turn
        local currentDir = self:GetVelocity():GetNormalized()
        local newDir = LerpVector(self.HomingTurnRate, currentDir, dirToTarget)

        -- Apply velocity
        self:SetVelocity(newDir * self.CurrentSpeed)
        self:SetAngles(newDir:Angle())

        return 0
    end

    function ENT:OnContact(ent)
        -- Bigger explosion for direct hit on target
        if ent == self.Target then
            self:Explosion(200, 350)
        else
            self:Explosion(100, 250)
        end
    end
end
```

### Predictive Homing

For more realistic homing, aim ahead of moving targets:

```lua
function ENT:CustomThink()
    local target = self.Target
    if not IsValid(target) then return end

    -- Get target velocity
    local targetVel = target:GetVelocity()

    -- Calculate interception point
    local timeToImpact = self:GetPos():Distance(target:GetPos()) / self.CurrentSpeed
    local predictedPos = target:GetPos() + targetVel * timeToImpact

    -- Aim at predicted position
    local dirToTarget = (predictedPos - self:GetPos()):GetNormalized()
    local currentDir = self:GetVelocity():GetNormalized()
    local newDir = LerpVector(self.HomingTurnRate, currentDir, dirToTarget)

    self:SetVelocity(newDir * self.CurrentSpeed)
    self:SetAngles(newDir:Angle())

    return 0
end
```

---

## Complete Examples

Here are complete, working projectile examples you can use and modify.

### Example 1: Fireball

A magical fireball that ignites targets.

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Fireball"
ENT.Category = "Magic"
ENT.Spawnable = true

ENT.Models = {"models/props_junk/watermelon01.mdl"}
ENT.ModelScale = 0.5
ENT.Gravity = false
ENT.OnContactDelete = 0

ENT.AttachEffects = {"fire_small"}
ENT.OnContactDecals = {"Scorch"}
ENT.OnContactSounds = {"ambient/fire/ignite.wav"}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Invisible model with fire effect
        self:SetNoDraw(true)
        self:SetVelocity(self:GetForward() * 1200)

        -- Orange glow
        self:DynamicLight(Color(255, 150, 0), 200, 2)
    end

    function ENT:CustomThink()
        -- Slowly fall due to "magical gravity"
        local vel = self:GetVelocity()
        self:SetVelocity(vel + Vector(0, 0, -10))
        return 0
    end

    function ENT:OnContact(ent)
        -- Deal fire damage
        if IsValid(ent) and not ent:IsWorld() then
            self:DealDamage(ent, 40, DMG_BURN)
            ent:Ignite(5)  -- Set on fire for 5 seconds
        end

        -- Small area burn
        self:RadiusDamage(20, DMG_BURN, 100)

        -- Fire effect
        ParticleEffect("fire_medium_01", self:GetPos(), Angle(0, 0, 0))
    end
end
```

### Example 2: Freeze Ray Projectile

A freezing projectile that slows targets.

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Freeze Ray"
ENT.Category = "Special Weapons"
ENT.Spawnable = true

ENT.Models = {}
ENT.Gravity = false
ENT.OnContactDelete = 0

ENT.AttachEffects = {"drg_plasma_ball"}  -- Use plasma effect with blue tint
ENT.OnContactDecals = {"BeerSplash"}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        self:SetNoDraw(true)
        self:SetVelocity(self:GetForward() * 2000)

        -- Blue glow
        self:DynamicLight(Color(100, 150, 255), 250, 1)

        -- Set model color to blue
        self:SetColor(Color(100, 150, 255))
    end

    function ENT:OnContact(ent)
        if not IsValid(ent) or ent:IsWorld() then return end

        -- Deal cold damage
        self:DealDamage(ent, 15, DMG_GENERIC)

        -- Freeze effect
        if ent:IsPlayer() or ent:IsNPC() then
            -- Slow down
            local currentSpeed = ent:GetVelocity():Length()
            ent:SetVelocity(ent:GetVelocity() * 0.3)

            -- Apply frost effect
            local freezeDuration = 3
            self:Timer(0, function()
                -- Make entity bluish
                local oldColor = ent:GetColor()
                ent:SetColor(Color(100, 150, 255, oldColor.a))

                -- Restore after duration
                self:Timer(freezeDuration, function()
                    ent:SetColor(oldColor)
                end)
            end)
        end

        -- Ice particles
        local effectdata = EffectData()
        effectdata:SetOrigin(self:GetPos())
        util.Effect("ManhackSparks", effectdata)
    end
end
```

### Example 3: Bouncing Plasma Ball

A plasma projectile that bounces off surfaces.

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Bouncing Plasma"
ENT.Category = "Energy Weapons"
ENT.Spawnable = true

ENT.Models = {"models/hunter/misc/sphere025x025.mdl"}
ENT.ModelScale = 0.5
ENT.Gravity = false
ENT.OnContactDelete = -1  -- Never auto-delete

ENT.AttachEffects = {"drg_plasma_ball"}
ENT.OnContactDecals = {"Scorch"}
ENT.OnContactSounds = {"ambient/energy/zap5.wav"}

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        -- Set material
        self:SetMaterial("models/debug/debugwhite")
        self:SetColor(Color(0, 255, 150))

        self:SetVelocity(self:GetForward() * 1500)
        self:DynamicLight(Color(0, 255, 150), 300, 1)

        -- Bounce limit
        self.BounceCount = 0
        self.MaxBounces = 5

        -- Lifetime
        self:Timer(10, function()
            self:Remove()
        end)
    end

    function ENT:OnContact(ent)
        self.BounceCount = self.BounceCount + 1

        -- Deal damage
        if IsValid(ent) and not ent:IsWorld() then
            self:DealDamage(ent, 35, DMG_SHOCK)
            self:Remove()
            return
        end

        -- Bounce off walls
        if ent:IsWorld() then
            if self.BounceCount >= self.MaxBounces then
                -- Explode after max bounces
                ParticleEffect("cball_explode", self:GetPos(), Angle(0, 0, 0))
                self:RadiusDamage(50, DMG_SHOCK, 200)
                self:Remove()
                return
            end

            -- Calculate bounce
            local vel = self:GetVelocity()
            local normal = (self:GetPos() - self:GetOwner():GetPos()):GetNormalized()

            -- Reflect velocity
            local reflected = vel - 2 * vel:Dot(normal) * normal

            self:Timer(0, function()
                self:SetVelocity(reflected * 0.9)  -- Lose 10% energy per bounce
            end)
        end
    end
end
```

### Example 4: Sticky Grenade

A grenade that sticks to surfaces before exploding.

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_grenade"

ENT.PrintName = "Sticky Grenade"
ENT.Category = "Explosives"
ENT.Spawnable = true

ENT.Models = {"models/weapons/w_eq_fraggrenade.mdl"}
ENT.Bounce = 0  -- Don't bounce

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        self:SetDamage(120)
        self:SetRange(300)
        self:SetDelay(2)  -- 2 second fuse

        self.HasStuck = false
    end

    function ENT:OnContact(ent)
        if self.HasStuck then return end

        -- Stick to surface
        self.HasStuck = true

        -- Stop moving
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            phys:EnableMotion(false)
            phys:EnableGravity(false)
        end

        -- Unpin and start countdown
        self:Unpin()

        -- Beep sound
        self:Timer(0, function()
            for i = 1, 10 do
                self:Timer(i * 0.2, function()
                    self:EmitSound("buttons/button17.wav", 75, 100 + i * 10)
                end)
            end
        end)

        -- Parent to entity if not world
        if not ent:IsWorld() and IsValid(ent) then
            self:SetParent(ent)
        end

        return false  -- Prevent default bounce behavior
    end

    function ENT:OnDetonate()
        -- Create directional explosion
        ParticleEffect("explosion_huge", self:GetPos(), self:GetAngles())
        self:Explosion(self:GetDamage(), self:GetRange())
    end
end
```

### Example 5: Lightning Bolt

A fast electrical projectile that chains to nearby targets.

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Lightning Bolt"
ENT.Category = "Energy Weapons"
ENT.Spawnable = true

ENT.Models = {}
ENT.Gravity = false
ENT.OnContactDelete = 0

if SERVER then
    AddCSLuaFile()

    function ENT:CustomInitialize()
        self:SetNoDraw(true)
        self:SetVelocity(self:GetForward() * 3000)  -- Very fast

        -- Lightning trail
        self.TrailData = util.SpriteTrail(self, 0, Color(150, 200, 255), false, 15, 1, 0.2,
            1 / (15 + 1) * 0.5, "trails/laser.vmt")

        self:DynamicLight(Color(150, 200, 255), 400, 3)
    end

    function ENT:OnContact(ent)
        if not IsValid(ent) then return end

        -- Lightning effect
        local effectdata = EffectData()
        effectdata:SetOrigin(self:GetPos())
        effectdata:SetMagnitude(2)
        effectdata:SetScale(1)
        effectdata:SetRadius(10)
        util.Effect("TeslaHitboxes", effectdata)

        -- Shock sound
        self:EmitSound("ambient/energy/zap" .. math.random(1, 9) .. ".wav", 90, math.random(90, 110))

        if ent:IsWorld() then return end

        -- Primary target damage
        self:DealDamage(ent, 50, DMG_SHOCK)

        -- Chain lightning to nearby targets
        self:ChainLightning(ent, 3, 30, 300)
    end

    function ENT:ChainLightning(fromEnt, remainingChains, damage, range)
        if remainingChains <= 0 then return end

        local owner = self:GetOwner()
        local hitTargets = {fromEnt}

        -- Find nearby target
        for _, target in ipairs(ents.FindInSphere(fromEnt:GetPos(), range)) do
            if target == owner then continue end
            if target == self then continue end
            if table.HasValue(hitTargets, target) then continue end
            if not target:IsNPC() and not target:IsPlayer() then continue end
            if target:Health() <= 0 then continue end

            -- Lightning beam effect
            local effectdata = EffectData()
            effectdata:SetOrigin(fromEnt:GetPos())
            effectdata:SetStart(target:GetPos())
            effectdata:SetEntity(target)
            effectdata:SetScale(0.5)
            util.Effect("TeslaHitboxes", effectdata)

            -- Damage
            self:DealDamage(target, damage, DMG_SHOCK)

            -- Sound
            target:EmitSound("ambient/energy/zap" .. math.random(1, 9) .. ".wav", 80, math.random(90, 110))

            -- Continue chain
            table.insert(hitTargets, target)
            self:Timer(0.1, function()
                self:ChainLightning(target, remainingChains - 1, damage * 0.7, range)
            end)

            break  -- Only chain to one target per iteration
        end
    end
end
```

---

## Best Practices

### Performance

1. **Use appropriate tick rates**
   ```lua
   -- For most projectiles, default tick rate is fine
   -- For homing projectiles, use CustomThink with return 0 for every tick
   -- For simple projectiles, use longer intervals

   function ENT:CustomThink()
       self:DoSimpleUpdate()
       return 0.1  -- Update every 0.1 seconds
   end
   ```

2. **Clean up effects**
   ```lua
   -- Use the built-in effect system
   ENT.AttachEffects = {"my_trail"}

   -- Instead of manually managing particles
   -- The framework handles cleanup automatically
   ```

3. **Limit expensive operations**
   ```lua
   function ENT:CustomThink()
       -- Bad: Search every tick
       -- self:FindNearestTarget()

       -- Good: Search periodically
       if not self.NextTargetSearch or CurTime() > self.NextTargetSearch then
           self:FindNearestTarget()
           self.NextTargetSearch = CurTime() + 0.5
       end
       return 0
   end
   ```

### Safety

1. **Always check validity**
   ```lua
   function ENT:OnContact(ent)
       if not IsValid(ent) then return end
       if ent:IsWorld() then return end

       -- Now safe to use ent
       self:DealDamage(ent, 50)
   end
   ```

2. **Set owners properly**
   ```lua
   local proj = ents.Create("proj_my_rocket")
   proj:SetOwner(attacker)  -- For damage attribution and filtering
   proj:Spawn()
   ```

3. **Use SafeRemove for delayed cleanup**
   ```lua
   function ENT:OnContact(ent)
       self:Explosion(100, 250)

       -- Bad: self:Remove() might be called during effect processing

       -- Good: Delay removal
       self:Timer(0.1, function()
           self:Remove()
       end)
   end
   ```

### Network Optimization

1. **Use NW2 vars for replicated data**
   ```lua
   -- Set on server
   self:SetNW2Int("BounceCount", self.BounceCount)

   -- Read on client
   local bounces = self:GetNW2Int("BounceCount", 0)
   ```

2. **Minimize network messages**
   ```lua
   -- Bad: Sending position every tick
   -- net.Start("UpdatePos")
   -- net.WriteVector(self:GetPos())
   -- net.Broadcast()

   -- Good: Use entity networking (automatic)
   -- Position is automatically networked
   ```

### Code Organization

1. **Separate concerns**
   ```lua
   -- Initialize once
   function ENT:CustomInitialize()
       self:SetupEffects()
       self:SetupPhysics()
       self:SetupLifetime()
   end

   -- Update per tick
   function ENT:CustomThink()
       self:UpdateHoming()
       self:UpdateEffects()
       return 0
   end

   -- Handle collision
   function ENT:OnContact(ent)
       self:HandleImpact(ent)
   end
   ```

2. **Use configuration variables**
   ```lua
   -- At the top of the file
   ENT.ProjectileSpeed = 1500
   ENT.ProjectileDamage = 75
   ENT.ExplosionRadius = 300

   -- Use in functions
   function ENT:CustomInitialize()
       self:SetVelocity(self:GetForward() * self.ProjectileSpeed)
   end

   function ENT:OnContact(ent)
       self:Explosion(self.ProjectileDamage, self.ExplosionRadius)
   end
   ```

### Debugging

1. **Use debug visualization**
   ```lua
   if GetConVar("developer"):GetBool() then
       debugoverlay.Line(self:GetPos(), target:GetPos(), 0.1, Color(255, 0, 0))
       debugoverlay.Sphere(self:GetPos(), 50, 0.1, Color(0, 255, 0))
   end
   ```

2. **Add console output for testing**
   ```lua
   function ENT:OnContact(ent)
       print(string.format("[%s] Hit: %s", self:GetClass(), tostring(ent)))
       self:DealDamage(ent, 50)
   end
   ```

3. **Use the trajectory debugger**
   ```lua
   -- Enable trajectory visualization
   -- drgbase_debug_trajectories 1

   -- Now AimAt and ThrowAt will show predicted paths
   local dir = self:AimAt(target:GetPos(), 1000)
   ```

---

## Summary

DrGBase projectiles provide a flexible foundation for creating custom projectile-based weapons and abilities. Key features include:

- Easy inheritance from `proj_drg_default` or `proj_drg_grenade`
- Built-in damage dealing with `DealDamage` and `RadiusDamage`
- Automatic physics and collision handling
- Integrated sound, particle, and light effects
- Helper functions for aiming and throwing
- Support for homing, explosive, and specialty projectiles

Start with the basic examples and gradually add complexity as needed. Remember to test thoroughly and follow best practices for performance and network efficiency.

For more information, see:
- [Entity Metatable Extensions](../api/meta/entity.md)
- [Physics Metatable Extensions](../api/meta/physobj.md)
- [Creating NPCs](./creating-npcs.md)
