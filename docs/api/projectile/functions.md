# Projectile Functions & Hooks

Functions and hooks for DrGBase projectiles.

**File:** `entities/proj_drg_default/shared.lua`

---

## Initialization

### ENT:Initialize()
**Realm:** Server/Client

Main initialization function. Called automatically when the projectile spawns.

**Server Behavior:**
- Sets model from `ENT.Models` (random selection)
- Applies `ModelScale`
- Sets up physics (VPhysics)
- Initializes sounds and effects
- Calls `_BaseInitialize()` and `CustomInitialize()`

**Client Behavior:**
- Calls `_BaseInitialize()` and `CustomInitialize()`

Do not override this function. Use `CustomInitialize()` instead.

### ENT:_BaseInitialize()
**Realm:** Shared

Internal hook for base projectile initialization. Reserved for future use by DrGBase.

### ENT:CustomInitialize()
**Realm:** Shared

Custom initialization hook for your projectile. Called after base initialization.

```lua
function ENT:CustomInitialize()
    if SERVER then
        -- Initialize projectile state
        self.Damage = 100
        self.Range = 200

        -- Network to clients
        self:SetNW2Int("Damage", self.Damage)

        -- Create dynamic light
        self:DynamicLight(Color(150, 255, 0), 300, 0.1)
    end
end
```

**Example - Homing Projectile:**
```lua
function ENT:CustomInitialize()
    if SERVER then
        self.Target = nil
        self.HomingForce = 500

        -- Find nearest enemy
        for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
            if IsValid(ent) and ent:IsNPC() then
                self.Target = ent
                break
            end
        end
    end
end
```

---

## Think

### ENT:Think()
**Realm:** Shared

Main think function. Calls think hooks and handles tickrate.

Do not override this function. Use `CustomThink()` instead.

### ENT:_BaseThink()
**Realm:** Shared
**Returns:** `number` or `nil` - Delay until next base think

Internal think hook. Reserved for future use by DrGBase.

### ENT:CustomThink()
**Realm:** Shared
**Returns:** `number` or `nil` - Delay until next custom think (seconds)

Custom think hook. Called every frame (or at tickrate if set).

```lua
function ENT:CustomThink()
    if SERVER then
        -- Homing behavior
        if IsValid(self.Target) then
            local phys = self:GetPhysicsObject()
            if IsValid(phys) then
                local dir = (self.Target:GetPos() - self:GetPos()):GetNormalized()
                phys:ApplyForceCenter(dir * self.HomingForce)
            end
        end
    end

    -- Think every 0.1 seconds
    return 0.1
end
```

**Example - Projectile Trail:**
```lua
function ENT:CustomThink()
    if CLIENT then
        -- Create trail effect
        local emitter = ParticleEmitter(self:GetPos())
        if emitter then
            local particle = emitter:Add("effects/spark", self:GetPos())
            particle:SetVelocity(VectorRand() * 10)
            particle:SetLifeTime(0)
            particle:SetDieTime(0.5)
            particle:SetStartAlpha(255)
            particle:SetEndAlpha(0)
            particle:SetStartSize(5)
            particle:SetEndSize(0)
            particle:SetColor(255, 100, 0)
            emitter:Finish()
        end
    end

    return 0.05 -- Think every 0.05 seconds
end
```

---

## Collision & Contact

### ENT:PhysicsCollide(data)
**Realm:** Server
**Parameters:**
- `data` (table) - Physics collision data from Garry's Mod

Called when the projectile physically collides with something. Calls `Contact()` internally.

Do not override this function. Use `OnContact()` instead.

### ENT:Touch(ent)
**Realm:** Server
**Parameters:**
- `ent` (Entity) - The entity that was touched

Called when the projectile touches a trigger. Calls `Contact()` internally.

Do not override this function. Use `OnContact()` instead.

### ENT:Contact(ent)
**Realm:** Server
**Parameters:**
- `ent` (Entity) - The entity contacted (can be world)

Internal contact handler. Manages contact delay, calls `OnContact()`, plays effects/sounds, and handles auto-removal.

Do not override this function. Use `OnContact()` instead.

### ENT:OnContact(ent)
**Realm:** Server
**Parameters:**
- `ent` (Entity) - The entity contacted
**Returns:** `boolean` or `nil` - Return `false` to prevent default contact behavior

Custom contact hook. Called when the projectile hits something (respecting `OnContactDelay`).

```lua
function ENT:OnContact(ent)
    -- Deal damage to entities
    if IsValid(ent) and ent:Health() > 0 then
        self:DealDamage(ent, 50, DMG_BLAST)
    end

    -- Explode on any contact
    self:Explosion(100, 200)

    -- Remove immediately (can also use OnContactDelete property)
    self:Remove()
end
```

**Example - Bounce and Damage:**
```lua
function ENT:OnContact(ent)
    -- Bounce off entities
    if IsValid(ent) and not ent:IsWorld() then
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            local vel = phys:GetVelocity()
            phys:SetVelocity(vel * 0.8) -- Lose some energy
        end

        -- Deal damage
        self:DealDamage(ent, 25, DMG_CRUSH)
    end

    -- Don't auto-remove
    return false
end
```

**Example - Stick to Surface:**
```lua
function ENT:OnContact(ent)
    -- Stop moving and stick
    local phys = self:GetPhysicsObject()
    if IsValid(phys) then
        phys:EnableMotion(false)
        phys:Sleep()
    end

    -- Set contact delay very high so we don't trigger again
    self.OnContactDelay = 999999

    -- Start timer for detonation
    timer.Simple(3, function()
        if IsValid(self) then
            self:Explosion(150, 250)
            self:Remove()
        end
    end)

    return false
end
```

---

## Damage

### ENT:DealDamage(ent, value, type)
**Realm:** Server
**Parameters:**
- `ent` (Entity) - Target entity
- `value` (number) - Damage amount
- `type` (number) - Damage type (default: `DMG_DIRECT`)

Deals damage to a single entity. Sets attacker to projectile's owner.

```lua
function ENT:OnContact(ent)
    if IsValid(ent) and ent:Health() > 0 then
        self:DealDamage(ent, 75, DMG_BULLET)
    end
end
```

**Common Damage Types:**
- `DMG_GENERIC` - Generic damage
- `DMG_CRUSH` - Crushing damage
- `DMG_BULLET` - Bullet damage
- `DMG_SLASH` - Slashing damage
- `DMG_BURN` - Fire damage
- `DMG_BLAST` - Explosive damage
- `DMG_CLUB` - Blunt damage
- `DMG_SHOCK` - Electric damage
- `DMG_SONIC` - Sound damage
- `DMG_ENERGYBEAM` - Energy beam
- `DMG_DISSOLVE` - Dissolve effect
- `DMG_POISON` - Poison damage
- `DMG_ACID` - Acid damage

### ENT:RadiusDamage(damage, type, range, filter)
**Realm:** Server
**Parameters:**
- `damage` (number) - Maximum damage at epicenter
- `type` (number) - Damage type
- `range` (number) - Damage radius in units
- `filter` (function, optional) - Filter function to exclude entities

Deals damage to all entities in a radius. Damage falls off with distance.

**Default Filter:**
```lua
function(ent)
    if ent == owner then return false end
    if not IsValid(owner) or not owner.IsDrGNextbot then return true end
    return not owner:IsAlly(ent)
end
```

**Example - Simple Explosion:**
```lua
function ENT:OnContact(ent)
    -- 100 damage at center, 0 at 300 units
    self:RadiusDamage(100, DMG_BLAST, 300)
    self:Remove()
end
```

**Example - Custom Filter:**
```lua
function ENT:OnContact(ent)
    -- Only damage NPCs
    local filter = function(ent)
        return ent:IsNPC()
    end

    self:RadiusDamage(150, DMG_BLAST, 250, filter)
    self:Remove()
end
```

### ENT:Explosion(damage, range, filter)
**Realm:** Server
**Parameters:**
- `damage` (number) - Explosion damage
- `range` (number) - Damage radius
- `filter` (function, optional) - Entity filter

Creates explosion effect and deals radius damage.

```lua
function ENT:OnContact(ent)
    -- Explosion with effect
    self:Explosion(200, 400)
    self:Remove()
end
```

### ENT:OnDealtDamage(ent, dmg)
**Realm:** Server
**Parameters:**
- `ent` (Entity) - Entity being damaged
- `dmg` (CTakeDamageInfo) - Damage info object
**Returns:** `boolean` or `nil` - Return `true` to prevent damage

Hook called when projectile deals damage. Use to modify or prevent damage.

```lua
function ENT:OnDealtDamage(ent, dmg)
    -- Prevent crush damage (from physics)
    if dmg:IsDamageType(DMG_CRUSH) then
        return true -- Cancel damage
    end

    -- Double damage to players
    if ent:IsPlayer() then
        dmg:ScaleDamage(2)
    end
end
```

---

## Utility Functions

### ENT:AimAt(target, speed, feet)
**Realm:** Server
**Parameters:**
- `target` (Entity or Vector) - Target to aim at
- `speed` (number, optional) - Velocity magnitude
- `feet` (boolean, optional) - Aim at target's feet instead of center

Aims the projectile at a target with optional speed. Uses `DrG_AimAt` helper.

```lua
function ENT:CustomInitialize()
    if SERVER then
        -- Find and aim at player
        local ply = player.GetAll()[1]
        if IsValid(ply) then
            self:AimAt(ply, 1000) -- Fly at player at 1000 units/sec
        end
    end
end
```

### ENT:ThrowAt(target, options, feet)
**Realm:** Server
**Parameters:**
- `target` (Entity or Vector) - Target to throw at
- `options` (table, optional) - Throw options (arc, speed, etc.)
- `feet` (boolean, optional) - Aim at target's feet

Throws the projectile in an arc toward a target. Uses `DrG_ThrowAt` helper.

```lua
function ENT:CustomInitialize()
    if SERVER then
        local enemy = ents.FindByClass("npc_*")[1]
        if IsValid(enemy) then
            -- Lob at enemy with arc
            self:ThrowAt(enemy, {speed = 800, arc = 0.5})
        end
    end
end
```

---

## Drawing (Client)

### ENT:Draw()
**Realm:** Client

Draws the projectile. Calls model draw and custom draw hooks.

Do not override this function. Use `CustomDraw()` instead.

### ENT:_BaseDraw()
**Realm:** Client

Internal draw hook. Reserved for future use by DrGBase.

### ENT:CustomDraw()
**Realm:** Client

Custom drawing hook. Called every frame on client.

```lua
function ENT:CustomDraw()
    -- Draw glow effect
    render.SetColorModulation(0, 1, 0)
    render.SetBlend(0.8)
    self:DrawModel()
    render.SetColorModulation(1, 1, 1)
    render.SetBlend(1)
end
```

---

## Complete Example

Example projectile with full functionality:

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Sticky Grenade"
ENT.Category = "My Projectiles"
ENT.Spawnable = true

-- Appearance
ENT.Models = {"models/weapons/w_eq_fraggrenade.mdl"}
ENT.ModelScale = 0.8

-- Physics
ENT.Physgun = true
ENT.Gravgun = true

-- Effects
ENT.LoopSounds = {"weapons/c4/c4_beep1.wav"}
ENT.OnContactSounds = {"physics/metal/metal_grenade_impact_hard1.wav"}
ENT.OnRemoveEffects = {"explosion"}

-- Never auto-remove
ENT.OnContactDelete = -1
ENT.OnContactDelay = 0.1

-- Custom properties
ENT.StickDamage = 200
ENT.StickRange = 300
ENT.FuseTime = 3

function ENT:CustomInitialize()
    if SERVER then
        self.HasStuck = false
        self.DetonateTime = nil
    end
end

function ENT:OnContact(ent)
    if SERVER and not self.HasStuck then
        -- Stick to surface
        self.HasStuck = true

        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            phys:EnableMotion(false)
            phys:Sleep()
        end

        -- Weld to entity if not world
        if IsValid(ent) and not ent:IsWorld() then
            constraint.Weld(self, ent, 0, 0, 0, true, false)
        end

        -- Start fuse
        self.DetonateTime = CurTime() + self.FuseTime

        -- Speed up beeping
        self:StopLoopingSound(self._DrGBaseLoopingSound)
        self._DrGBaseLoopingSound = self:StartLoopingSound("weapons/c4/c4_beep1.wav", 1, 150)

        -- Prevent further contact triggers
        self.OnContactDelay = 999999

        return false
    end
end

function ENT:CustomThink()
    if SERVER and self.HasStuck and self.DetonateTime then
        -- Check if time to detonate
        if CurTime() >= self.DetonateTime then
            self:Explosion(self.StickDamage, self.StickRange)
            self:Remove()
        end
    end

    return 0.1
end

-- Allow manual detonation
function ENT:Use(activator)
    if SERVER and self.HasStuck then
        self:Explosion(self.StickDamage, self.StickRange)
        self:Remove()
    end
end

AddCSLuaFile()
DrGBase.AddEntity(ENT)
```

---

## Hook Summary

| Hook | Realm | Purpose |
|------|-------|---------|
| `CustomInitialize()` | Shared | Initialize projectile |
| `CustomThink()` | Shared | Per-frame/tick logic |
| `OnContact(ent)` | Server | When hitting something |
| `OnDealtDamage(ent, dmg)` | Server | When dealing damage |
| `CustomDraw()` | Client | Custom rendering |

---

## Function Summary

| Function | Realm | Purpose |
|----------|-------|---------|
| `DealDamage(ent, value, type)` | Server | Damage single entity |
| `RadiusDamage(damage, type, range, filter)` | Server | Damage in radius |
| `Explosion(damage, range, filter)` | Server | Create explosion |
| `AimAt(target, speed, feet)` | Server | Aim at target |
| `ThrowAt(target, options, feet)` | Server | Throw in arc |

---

## See Also

- [Projectile Base Configuration](base-config.md) - All projectile properties
- [Creating Projectiles Guide](../../guides/creating-projectiles.md) - Step-by-step projectile creation tutorial
