# Projectile Base API Reference

DrGBase provides a projectile base (`proj_drg_default`) for creating physics-based projectiles like grenades, rockets, and thrown objects.

**Source Files:**
- `lua/entities/proj_drg_default/shared.lua` - Core projectile system
- `lua/entities/proj_drg_default/meta.lua` - Velocity metatable extensions

**Base Class:** `drgbase_entity`

---

## Creating a Projectile

### Basic Setup

```lua
-- lua/entities/proj_my_grenade/shared.lua
AddCSLuaFile()
ENT.Base = "proj_drg_default"
ENT.Type = "anim"
ENT.IsDrGProjectile = true

-- Misc
ENT.PrintName = "My Grenade"
ENT.Category = "My Projectiles"
ENT.Models = {"models/weapons/w_grenade.mdl"}
ENT.ModelScale = 1

-- Physics
ENT.Gravity = true
ENT.Physgun = false
ENT.Gravgun = false

-- Contact
ENT.OnContactDelay = 0.1
ENT.OnContactDelete = 0
ENT.OnContactDecals = {"Scorch"}

-- Sounds
ENT.LoopSounds = {}
ENT.OnContactSounds = {"weapons/explode3.wav"}
ENT.OnRemoveSounds = {}

-- Effects
ENT.AttachEffects = {}
ENT.OnContactEffects = {"explosion"}
ENT.OnRemoveEffects = {}

function ENT:OnContact(ent)
    if SERVER then
        self:Explosion(100, 300)
    end
end
```

---

## Core Properties

### Identification

#### ENT.IsDrGProjectile
```lua
ENT.IsDrGProjectile = true
```
**Type:** boolean
**Description:** Identifies this as a DrGBase projectile. Required for base functionality.

---

### Basic Properties

#### ENT.PrintName
```lua
ENT.PrintName = "My Projectile"
```
**Type:** string
**Description:** Display name.

---

#### ENT.Category
```lua
ENT.Category = "My Category"
```
**Type:** string
**Description:** Spawn menu category.

---

#### ENT.Models
```lua
ENT.Models = {"models/weapons/w_grenade.mdl"}
```
**Type:** table
**Description:** Array of possible models. One is chosen randomly on spawn.

**Example:**
```lua
ENT.Models = {
    "models/weapons/w_grenade.mdl",
    "models/weapons/w_eq_fraggrenade.mdl"
}
```

---

#### ENT.ModelScale
```lua
ENT.ModelScale = 1
```
**Type:** number
**Default:** 1
**Description:** Scale multiplier for the model.

---

### Physics Properties

#### ENT.Gravity
```lua
ENT.Gravity = true
```
**Type:** boolean
**Default:** true
**Description:** Whether gravity affects the projectile.

**Example:**
```lua
ENT.Gravity = false -- Rocket that flies straight
```

---

#### ENT.Physgun
```lua
ENT.Physgun = false
```
**Type:** boolean
**Default:** false
**Description:** Whether the physgun can pick up this projectile.

---

#### ENT.Gravgun
```lua
ENT.Gravgun = false
```
**Type:** boolean
**Default:** false
**Description:** Whether the gravity gun can pick up this projectile.

---

### Contact Properties

#### ENT.OnContactDelay
```lua
ENT.OnContactDelay = 0.1
```
**Type:** number
**Default:** 0.1
**Description:** Minimum delay between contact events (prevents spam).

---

#### ENT.OnContactDelete
```lua
ENT.OnContactDelete = 0
```
**Type:** number
**Default:** -1
**Values:**
- `-1` - Never auto-delete on contact
- `0` - Delete immediately on contact
- `> 0` - Delete after this many seconds

**Example:**
```lua
ENT.OnContactDelete = 0 -- Explode on impact
ENT.OnContactDelete = 3 -- Stick for 3 seconds then explode
ENT.OnContactDelete = -1 -- Never auto-delete
```

---

#### ENT.OnContactDecals
```lua
ENT.OnContactDecals = {"Scorch"}
```
**Type:** table
**Description:** Decals to apply on world contact (randomly selected).

**Common Decals:**
- `"Scorch"` - Burn mark
- `"BeerSplash"` - Liquid splash
- `"Blood"` - Blood splatter
- `"BulletProof"` - Bullet impact
- `"Cross"` - Cross mark

**Example:**
```lua
ENT.OnContactDecals = {"Scorch", "FadingScorch"}
```

---

### Sound Properties

#### ENT.LoopSounds
```lua
ENT.LoopSounds = {"weapons/physcannon/energy_sing_loop4.wav"}
```
**Type:** table
**Description:** Looping sounds (randomly selected, plays while projectile exists).

**Example:**
```lua
ENT.LoopSounds = {"weapons/physcannon/energy_sing_loop4.wav"}
```

---

#### ENT.OnContactSounds
```lua
ENT.OnContactSounds = {"weapons/explode3.wav"}
```
**Type:** table
**Description:** Sounds played on contact (randomly selected).

**Example:**
```lua
ENT.OnContactSounds = {
    "weapons/explode3.wav",
    "weapons/explode4.wav",
    "weapons/explode5.wav"
}
```

---

#### ENT.OnRemoveSounds
```lua
ENT.OnRemoveSounds = {}
```
**Type:** table
**Description:** Sounds played when projectile is removed (randomly selected).

---

### Effect Properties

#### ENT.AttachEffects
```lua
ENT.AttachEffects = {"RPGShotDown"}
```
**Type:** table
**Description:** Particle effects attached to projectile (randomly selected).

**Common Effects:**
- `"RPGShotDown"` - Rocket trail
- `"smoke_burning_engine_01"` - Smoke trail
- `"burning_engine_01"` - Fire trail

**Example:**
```lua
ENT.AttachEffects = {"RPGShotDown"}
```

---

#### ENT.OnContactEffects
```lua
ENT.OnContactEffects = {"Explosion"}
```
**Type:** table
**Description:** Particle effects spawned on contact (randomly selected).

**Common Effects:**
- `"Explosion"` - Standard explosion
- `"HelicopterMegaBomb"` - Large explosion
- `"WaterSplash"` - Water splash

**Example:**
```lua
ENT.OnContactEffects = {"Explosion", "HelicopterMegaBomb"}
```

---

#### ENT.OnRemoveEffects
```lua
ENT.OnRemoveEffects = {}
```
**Type:** table
**Description:** Particle effects spawned when projectile is removed (randomly selected).

---

## Configuration

### ConVars

#### `drgbase_projectile_tickrate`
```lua
CreateConVar("drgbase_projectile_tickrate", "-1", {FCVAR_ARCHIVE, FCVAR_NOTIFY, FCVAR_REPLICATED})
```
**Type:** number
**Default:** -1 (use engine tick interval)
**Description:** Think tickrate for projectiles. Set to specific value (e.g., 60) to limit think rate.

---

## Lifecycle Functions

### ENT:Initialize
Called when projectile is created. Handles model, physics, sounds, and effects.

**Realm:** SHARED
**Note:** Don't override. Use `CustomInitialize()` instead.

---

### ENT:CustomInitialize
```lua
ENT:CustomInitialize()
```
**Hook** for custom initialization logic.

**Realm:** SHARED
**Example:**
```lua
function ENT:CustomInitialize()
    if SERVER then
        self.ExplodeTime = CurTime() + 3 -- Timed grenade
    end
end
```

---

### ENT:Think
Called every tick. Handles base and custom think logic.

**Realm:** SHARED
**Note:** Don't override. Use `CustomThink()` instead.

---

### ENT:CustomThink
```lua
number ENT:CustomThink()
```
**Hook** for custom per-tick logic.

**Realm:** SHARED
**Returns:** (number) Delay until next call (0 = every tick)
**Example:**
```lua
function ENT:CustomThink()
    if SERVER and self.ExplodeTime and CurTime() >= self.ExplodeTime then
        self:Explosion(100, 300)
        self:Remove()
    end
    return 0.1 -- Think 10 times per second
end
```

---

### ENT:CustomDraw (CLIENT)
```lua
ENT:CustomDraw()
```
**Hook** for custom rendering.

**Realm:** CLIENT
**Example:**
```lua
function ENT:CustomDraw()
    local size = 20 * math.sin(CurTime() * 10)
    local lightColor = Color(255, 100, 100)

    render.SetMaterial(Material("sprites/light_glow02_add"))
    render.DrawSprite(self:GetPos(), size, size, lightColor)
end
```

---

## Contact/Collision

### ENT:PhysicsCollide (SERVER)
```lua
ENT:PhysicsCollide(table data)
```
Called when projectile collides with something. Automatically calls `Contact()`.

**Realm:** SERVER
**Parameters:**
- `data` (table) - Collision data (HitEntity, HitPos, HitNormal, etc.)

**Note:** Usually don't override. Use `OnContact()` hook instead.

---

### ENT:Touch (SERVER)
```lua
ENT:Touch(Entity ent)
```
Called when projectile touches a trigger or entity. Automatically calls `Contact()`.

**Realm:** SERVER
**Parameters:**
- `ent` (Entity) - Touched entity

**Note:** Usually don't override. Use `OnContact()` hook instead.

---

### ENT:Contact (SERVER)
```lua
ENT:Contact(Entity ent)
```
Internal handler that manages contact delays and triggers hooks/sounds/effects.

**Realm:** SERVER
**Note:** Don't override. Use `OnContact()` hook instead.

---

### ENT:OnContact
```lua
boolean ENT:OnContact(Entity ent)
```
**Hook** called when projectile contacts something (respects `OnContactDelay`).

**Realm:** SERVER
**Parameters:**
- `ent` (Entity) - Entity contacted (can be world)
**Returns:** (boolean) Return false to prevent default behavior (sounds/effects/deletion)
**Example:**
```lua
function ENT:OnContact(ent)
    if ent:IsWorld() then
        -- Stick to wall
        self:SetMoveType(MOVETYPE_NONE)

        timer.Simple(2, function()
            if IsValid(self) then
                self:Explosion(100, 200)
                self:Remove()
            end
        end)

        return false -- Prevent immediate deletion
    else
        -- Hit entity - explode immediately
        self:Explosion(100, 200)
        -- Return nothing (true) to allow default deletion
    end
end
```

**Special Cases:**
- `ent:IsWorld()` - Hit the world
- `ent:GetClass() == "trigger_soundscape"` - Automatically ignored

---

## Damage Functions (SERVER)

### ENT:DealDamage
```lua
ENT:DealDamage(Entity ent, number value, number type)
```
Deals damage to a single entity.

**Realm:** SERVER
**Parameters:**
- `ent` (Entity) - Target entity
- `value` (number) - Damage amount
- `type` (number) - Damage type (DMG_* constant, default: DMG_DIRECT)

**Example:**
```lua
function ENT:OnContact(ent)
    if not ent:IsWorld() then
        self:DealDamage(ent, 50, DMG_BLAST)
    end
end
```

**Damage Info:**
- Attacker: Projectile owner (or self if no owner)
- Inflictor: Projectile
- Force: Current velocity
- Position: Projectile position

---

### ENT:RadiusDamage
```lua
ENT:RadiusDamage(number damage, number type, number range, [function filter])
```
Deals damage to all entities in a radius with falloff.

**Realm:** SERVER
**Parameters:**
- `damage` (number) - Maximum damage (at center)
- `type` (number) - Damage type (DMG_* constant)
- `range` (number) - Damage radius
- `filter` (function) - Optional filter: `function(Entity ent)` return false to exclude

**Example:**
```lua
function ENT:OnContact(ent)
    -- Damage all enemies in 300 unit radius
    self:RadiusDamage(100, DMG_BLAST, 300)
end

-- Custom filter
function ENT:OnContact(ent)
    self:RadiusDamage(100, DMG_BLAST, 300, function(ent)
        -- Only damage players
        return ent:IsPlayer()
    end)
end
```

**Default Filter:**
Excludes:
- Projectile owner
- Allies of owner (if owner is DrGBase nextbot)

**Damage Falloff:**
Linear from full damage at center to 0 at edge:
```lua
damage = maxDamage * (1 - distance / range)
```

---

### ENT:Explosion
```lua
ENT:Explosion(number damage, number range, [function filter])
```
Creates an explosion effect and deals radius damage.

**Realm:** SERVER
**Parameters:**
- `damage` (number) - Maximum damage
- `range` (number) - Damage radius
- `filter` (function) - Optional damage filter

**Example:**
```lua
function ENT:OnContact(ent)
    self:Explosion(100, 300)
    self:Remove()
end
```

**Effects:**
- Creates `env_explosion` entity (visual/sound)
- Calls `RadiusDamage()` with DMG_BLAST

---

### ENT:OnDealtDamage
```lua
boolean ENT:OnDealtDamage(Entity ent, DamageInfo dmg)
```
**Hook** called when this projectile deals damage.

**Realm:** SERVER
**Parameters:**
- `ent` (Entity) - Entity being damaged
- `dmg` (DamageInfo) - Damage info object
**Returns:** (boolean) Return true to block damage
**Example:**
```lua
function ENT:OnDealtDamage(ent, dmg)
    if ent:IsPlayer() and ent:GetNWBool("HasShield") then
        -- Block damage to shielded players
        return true
    end

    -- Ignite targets
    if math.random() < 0.5 then
        ent:Ignite(5)
    end
end
```

**Default Behavior:**
Blocks physics damage (DMG_CRUSH) to prevent double-damage.

---

## Aiming Functions (SERVER)

### ENT:AimAt
```lua
Angle ENT:AimAt(Vector|Entity target, [number speed], [boolean feet])
```
Calculates aim angles to hit a target with current velocity.

**Realm:** SERVER
**Parameters:**
- `target` (Vector|Entity) - Target position or entity
- `speed` (number) - Projectile speed (default: current velocity magnitude)
- `feet` (boolean) - Aim at feet instead of center (default: false)
**Returns:** (Angle) Angles to hit target

**Example:**
```lua
-- When spawning
local grenade = ents.Create("proj_my_grenade")
grenade:SetPos(self:GetShootPos())
grenade:SetAngles(grenade:AimAt(enemy, 1000))
grenade:Spawn()

local phys = grenade:GetPhysicsObject()
if IsValid(phys) then
    phys:SetVelocity(grenade:GetForward() * 1000)
end
```

---

### ENT:ThrowAt
```lua
boolean ENT:ThrowAt(Vector|Entity target, [table options], [boolean feet])
```
Automatically sets velocity to hit a target.

**Realm:** SERVER
**Parameters:**
- `target` (Vector|Entity) - Target position or entity
- `options` (table) - Options:
  - `speed` (number) - Launch speed (default: 1000)
  - `arcmin` (number) - Minimum arc angle (default: 10)
  - `arcmax` (number) - Maximum arc angle (default: 70)
- `feet` (boolean) - Aim at feet instead of center
**Returns:** (boolean) True if successfully aimed

**Example:**
```lua
function ENT:CustomInitialize()
    if SERVER then
        local target = self:GetOwner():GetEnemy()
        if IsValid(target) then
            self:ThrowAt(target, {
                speed = 800,
                arcmin = 20,
                arcmax = 60
            })
        end
    end
end
```

---

## Velocity (SERVER)

### ENT:GetVelocity
```lua
Vector ENT:GetVelocity()
```
Gets projectile velocity from physics object.

**Realm:** SERVER
**Returns:** (Vector) Current velocity
**Note:** Overridden to use physics object velocity instead of entity velocity.

---

### ENT:SetVelocity
```lua
ENT:SetVelocity(Vector velocity)
```
Sets projectile velocity via physics object.

**Realm:** SERVER
**Parameters:**
- `velocity` (Vector) - New velocity
**Note:** Overridden to set physics object velocity instead of entity velocity.

---

## Complete Examples

### Simple Grenade
```lua
AddCSLuaFile()
ENT.Base = "proj_drg_default"
ENT.Type = "anim"
ENT.IsDrGProjectile = true

ENT.PrintName = "Simple Grenade"
ENT.Models = {"models/weapons/w_grenade.mdl"}
ENT.Gravity = true

ENT.OnContactDelay = 0.1
ENT.OnContactDelete = 0 -- Explode immediately
ENT.OnContactDecals = {"Scorch"}
ENT.OnContactSounds = {"weapons/explode3.wav"}
ENT.OnContactEffects = {"Explosion"}

function ENT:OnContact(ent)
    if SERVER then
        self:Explosion(100, 300)
    end
end
```

---

### Timed Grenade
```lua
AddCSLuaFile()
ENT.Base = "proj_drg_default"
ENT.Type = "anim"
ENT.IsDrGProjectile = true

ENT.PrintName = "Timed Grenade"
ENT.Models = {"models/weapons/w_grenade.mdl"}
ENT.Gravity = true

ENT.FuseTime = 3
ENT.OnContactDelete = -1 -- Don't auto-delete

ENT.LoopSounds = {"weapons/grenade/tick1.wav"}
ENT.OnContactSounds = {"physics/metal/metal_grenade_impact_hard1.wav"}
ENT.OnContactDecals = {}

function ENT:CustomInitialize()
    if SERVER then
        self.ExplodeTime = CurTime() + self.FuseTime

        -- Beep faster as timer runs down
        timer.Create("Beep_" .. self:EntIndex(), 0.5, 0, function()
            if not IsValid(self) then return end
            self:EmitSound("buttons/button17.wav")
        end)
    end
end

function ENT:CustomThink()
    if SERVER and CurTime() >= self.ExplodeTime then
        self:Explosion(150, 400)
        self:Remove()
    end
    return 0.1
end

function ENT:OnContact(ent)
    if SERVER and not ent:IsWorld() then
        -- Hit an entity - explode immediately
        self:Explosion(150, 400)
        self:Remove()
        return false
    end
    -- Hit world - continue counting
end
```

---

### Sticky Bomb
```lua
AddCSLuaFile()
ENT.Base = "proj_drg_default"
ENT.Type = "anim"
ENT.IsDrGProjectile = true

ENT.PrintName = "Sticky Bomb"
ENT.Models = {"models/weapons/w_grenade.mdl"}
ENT.Gravity = true

ENT.StickTime = 5
ENT.OnContactDelete = -1

function ENT:CustomInitialize()
    if SERVER then
        self.Stuck = false
    end
    if CLIENT then
        self.NextBeep = CurTime()
    end
end

function ENT:OnContact(ent)
    if SERVER and not self.Stuck then
        self.Stuck = true
        self.ExplodeTime = CurTime() + self.StickTime

        -- Freeze in place
        self:SetMoveType(MOVETYPE_NONE)
        self:SetCollisionGroup(COLLISION_GROUP_WEAPON)

        -- If stuck to entity, parent to it
        if not ent:IsWorld() and IsValid(ent) then
            self:SetParent(ent)
        end

        self:EmitSound("npc/barnacle/barnacle_gulp1.wav")
        return false -- Prevent auto-delete
    end
end

function ENT:CustomThink()
    if SERVER and self.Stuck and CurTime() >= self.ExplodeTime then
        self:Explosion(200, 350)
        self:Remove()
    end

    -- Beep faster as timer runs down
    if CLIENT and self.Stuck then
        if CurTime() >= self.NextBeep then
            local timeLeft = self.ExplodeTime - CurTime()
            local beepRate = math.Remap(timeLeft, self.StickTime, 0, 0.5, 0.05)
            self.NextBeep = CurTime() + beepRate

            local pitch = math.Remap(timeLeft, self.StickTime, 0, 100, 200)
            self:EmitSound("buttons/button17.wav", 75, pitch)
        end
    end

    return 0.05
end

function ENT:CustomDraw()
    if self.Stuck then
        local progress = (self.ExplodeTime - CurTime()) / self.StickTime
        local color = Color(255, progress * 255, progress * 255)

        render.SetMaterial(Material("sprites/light_glow02_add"))
        render.DrawSprite(self:GetPos(), 30, 30, color)
    end
end
```

---

### Homing Rocket
```lua
AddCSLuaFile()
ENT.Base = "proj_drg_default"
ENT.Type = "anim"
ENT.IsDrGProjectile = true

ENT.PrintName = "Homing Rocket"
ENT.Models = {"models/weapons/w_missile_closed.mdl"}
ENT.Gravity = false

ENT.Speed = 1500
ENT.TurnRate = 80
ENT.OnContactDelete = 0

ENT.AttachEffects = {"RPGShotDown"}
ENT.OnContactEffects = {"Explosion"}
ENT.OnContactSounds = {"weapons/explode3.wav"}
ENT.OnContactDecals = {"Scorch"}
ENT.LoopSounds = {"weapons/rpg/rocket1.wav"}

function ENT:CustomInitialize()
    if SERVER then
        -- Set initial velocity
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(self:GetForward() * self.Speed)
        end

        -- Find target
        self.Target = self:GetOwner().LastEnemy or NULL
    end
end

function ENT:CustomThink()
    if SERVER then
        local phys = self:GetPhysicsObject()
        if not IsValid(phys) then return end

        -- Track target
        if IsValid(self.Target) then
            local targetPos = self.Target:WorldSpaceCenter()
            local dir = (targetPos - self:GetPos()):GetNormalized()
            local currentDir = self:GetVelocity():GetNormalized()

            -- Lerp direction toward target
            local newDir = LerpVector(FrameTime() * self.TurnRate, currentDir, dir)
            phys:SetVelocity(newDir * self.Speed)

            -- Update angles to match direction
            self:SetAngles(newDir:Angle())
        end
    end

    return 0
end

function ENT:OnContact(ent)
    if SERVER then
        self:Explosion(150, 400)
    end
end
```

---

### Smoke Grenade
```lua
AddCSLuaFile()
ENT.Base = "proj_drg_default"
ENT.Type = "anim"
ENT.IsDrGProjectile = true

ENT.PrintName = "Smoke Grenade"
ENT.Models = {"models/weapons/w_eq_smokegrenade.mdl"}
ENT.Gravity = true

ENT.SmokeDuration = 15
ENT.OnContactDelete = -1

function ENT:CustomInitialize()
    if SERVER then
        self.Deployed = false
    end
end

function ENT:OnContact(ent)
    if SERVER and not self.Deployed then
        self.Deployed = true
        self:SetMoveType(MOVETYPE_NONE)

        -- Emit smoke
        ParticleEffect("smoke_burning_engine_01", self:GetPos(), Angle(0, 0, 0), self)

        -- Obscure vision
        self:Timer(self.SmokeDuration, self.Remove)

        self:EmitSound("weapons/smokegrenade/smoke_emit.wav")
        return false
    end
end
```

---

## Tips

### Spawning from Weapons
```lua
-- In weapon:FirePrimary()
if SERVER then
    local proj = ents.Create("proj_my_grenade")
    proj:SetPos(self.Owner:GetShootPos())
    proj:SetAngles(self.Owner:EyeAngles())
    proj:SetOwner(self.Owner)
    proj:Spawn()

    local phys = proj:GetPhysicsObject()
    if IsValid(phys) then
        phys:SetVelocity(self.Owner:GetAimVector() * 1000)
    end
end
```

### Spawning from NPCs
```lua
-- In nextbot attack behavior
local proj = ents.Create("proj_my_grenade")
proj:SetPos(self:GetShootPos())
proj:SetOwner(self)
proj:Spawn()

-- Aim at enemy
proj:ThrowAt(self:GetEnemy(), {speed = 800})
```

### Testing
```lua
-- Console command for testing
concommand.Add("test_projectile", function(ply)
    local proj = ents.Create("proj_my_grenade")
    proj:SetPos(ply:GetEyeTrace().HitPos)
    proj:SetAngles(AngleRand())
    proj:SetOwner(ply)
    proj:Spawn()
end)
```

---

## See Also

- [Weapon Base](weapon-base.md) - Creating weapons that fire projectiles
- [Combat System](../guides/combat.md) - Damage and combat mechanics
- [NPC Weapons](../guides/npc-weapons.md) - NPCs using projectile weapons
