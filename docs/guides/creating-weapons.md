# Creating Custom Weapons

A comprehensive guide to creating weapons for DrGBase. This guide covers everything from basic weapon setup to advanced projectile systems and custom behaviors.

## Table of Contents

1. [Introduction to the Weapon System](#introduction-to-the-weapon-system)
2. [Creating a Basic Weapon](#creating-a-basic-weapon)
3. [Primary and Secondary Fire](#primary-and-secondary-fire)
4. [Weapon Properties](#weapon-properties)
5. [Animations and Sounds](#animations-and-sounds)
6. [Projectile Weapons](#projectile-weapons)
7. [Melee Weapons](#melee-weapons)
8. [Custom Weapon Behaviors](#custom-weapon-behaviors)
9. [Complete Examples](#complete-examples)
10. [Best Practices](#best-practices)

---

## Introduction to the Weapon System

DrGBase provides a powerful weapon framework built on top of Garry's Mod's SWEP (Scripted Weapon) system. The framework includes:

- **Base Weapon Class**: `drgbase_weapon` - The foundation for all DrGBase weapons
- **Automatic Registration**: Weapons are automatically added to the spawn menu
- **Dual Fire Modes**: Support for primary and secondary attacks
- **Built-in Ammo System**: Integrated clip and reserve ammunition
- **Recoil System**: Automatic view punch and aim displacement
- **Hook System**: Extensible pre/post attack functions
- **Projectile Support**: Helper functions for creating projectiles

### Key Features

- Simplified weapon creation with sensible defaults
- Automatic ammo management
- Built-in spread and recoil handling
- Support for both hitscan and projectile weapons
- Customizable attack behaviors via hooks
- Compatible with DrGBase NPCs

---

## Creating a Basic Weapon

### File Structure

Weapons in DrGBase follow Garry's Mod's standard weapon structure:

```
lua/weapons/weapon_drg_myweapon/
    shared.lua
```

For simple weapons, a single `shared.lua` file is all you need.

### Minimal Weapon Example

Here's the simplest possible weapon:

```lua
if not DrGBase then return end -- Check if DrGBase is installed
SWEP.Base = "drgbase_weapon" -- Inherit from DrGBase weapon base

-- Identification
SWEP.PrintName = "My Weapon"
SWEP.Category = "DrGBase - Custom"
SWEP.Author = "YourName"
SWEP.Spawnable = true

-- Appearance
SWEP.ViewModel = "models/weapons/c_pistol.mdl"
SWEP.WorldModel = "models/weapons/w_pistol.mdl"

-- Primary Fire
SWEP.Primary.Damage = 15
SWEP.Primary.Delay = 0.2
SWEP.Primary.Sound = "Weapon_Pistol.Single"

-- Registration (always at the end)
AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

Save this to `lua/weapons/weapon_drg_myweapon/shared.lua` and your weapon is ready to use!

### Understanding Base Properties

The `drgbase_weapon` base provides these default properties:

```lua
-- Identification
SWEP.PrintName = ""           -- Display name
SWEP.Category = ""            -- Spawn menu category
SWEP.Author = ""              -- Creator name
SWEP.Purpose = ""             -- Weapon description
SWEP.Instructions = ""        -- Usage instructions
SWEP.Spawnable = false        -- Can spawn in menu

-- Appearance
SWEP.HoldType = "ar2"         -- Animation hold type
SWEP.ViewModelFOV = 62        -- FOV for viewmodel
SWEP.ViewModelFlip = true     -- Mirror viewmodel
SWEP.UseHands = false         -- Show player hands
SWEP.ViewModel = ""           -- First-person model
SWEP.WorldModel = ""          -- Third-person model
```

---

## Primary and Secondary Fire

### Primary Fire

Primary fire is the main attack mode (left mouse button). It's configured through the `SWEP.Primary` table:

```lua
-- Shooting Behavior
SWEP.Primary.Damage = 10      -- Damage per bullet
SWEP.Primary.Bullets = 1      -- Bullets per shot
SWEP.Primary.Spread = 0.02    -- Bullet spread (cone)
SWEP.Primary.Automatic = true -- Hold to fire continuously
SWEP.Primary.Delay = 0.1      -- Time between shots
SWEP.Primary.Recoil = 0.5     -- Camera kick amount

-- Ammunition
SWEP.Primary.Ammo = "AR2"     -- Ammo type
SWEP.Primary.Cost = 1         -- Ammo per shot
SWEP.Primary.ClipSize = 30    -- Magazine size
SWEP.Primary.DefaultClip = 90 -- Starting reserve ammo

-- Audio
SWEP.Primary.Sound = "Weapon_AR2.Single"
SWEP.Primary.EmptySound = "Weapon_AR2.Empty"
```

### Secondary Fire

Secondary fire (right mouse button) is disabled by default. Enable it like this:

```lua
SWEP.Secondary.Enabled = true

-- Configure like primary
SWEP.Secondary.Damage = 50
SWEP.Secondary.Bullets = 1
SWEP.Secondary.Spread = 0.01
SWEP.Secondary.Automatic = false
SWEP.Secondary.Delay = 1.0
SWEP.Secondary.Recoil = 2.0

SWEP.Secondary.Ammo = "AR2AltFire"
SWEP.Secondary.Cost = 1
SWEP.Secondary.ClipSize = 0
SWEP.Secondary.DefaultClip = 5

SWEP.Secondary.Sound = "Weapon_AR2.Double"
SWEP.Secondary.EmptySound = "Weapon_AR2.Empty"
```

### Fire Mode Differences

**Primary Fire Default:**
- Automatic by default
- Uses clips (magazines)
- Faster fire rate
- Lower damage per shot

**Secondary Fire Default:**
- Semi-automatic by default
- Often uses reserve ammo directly (ClipSize = 0)
- Slower fire rate
- Higher damage per shot

---

## Weapon Properties

### Hold Types

The `HoldType` affects animations and how the weapon is held:

```lua
SWEP.HoldType = "ar2"        -- Rifle (two-handed)
SWEP.HoldType = "pistol"     -- Pistol (one-handed)
SWEP.HoldType = "shotgun"    -- Shotgun
SWEP.HoldType = "smg"        -- SMG
SWEP.HoldType = "rpg"        -- Rocket launcher
SWEP.HoldType = "crossbow"   -- Crossbow
SWEP.HoldType = "grenade"    -- Grenade throwing
SWEP.HoldType = "melee"      -- Melee weapon
SWEP.HoldType = "melee2"     -- Two-handed melee
SWEP.HoldType = "knife"      -- Knife
SWEP.HoldType = "fist"       -- Fists
SWEP.HoldType = "slam"       -- SLAM mine
SWEP.HoldType = "normal"     -- Default stance
SWEP.HoldType = "passive"    -- Relaxed stance
```

### Damage and Ballistics

```lua
-- Single powerful shot
SWEP.Primary.Damage = 50
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.01

-- Shotgun spread
SWEP.Primary.Damage = 8
SWEP.Primary.Bullets = 8
SWEP.Primary.Spread = 0.15

-- Accurate sniper
SWEP.Primary.Damage = 100
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0      -- Perfect accuracy
```

### Recoil and Spread

```lua
-- Low recoil, accurate weapon
SWEP.Primary.Recoil = 0.3
SWEP.Primary.Spread = 0.01

-- High recoil, inaccurate weapon
SWEP.Primary.Recoil = 2.0
SWEP.Primary.Spread = 0.08
```

**Recoil** affects:
- Vertical camera kick (upward)
- View punch effect (screen shake)

**Spread** is the bullet cone in degrees. Values:
- `0` = Perfect accuracy
- `0.01` = Very accurate (sniper)
- `0.02` = Accurate (assault rifle)
- `0.05` = Moderate (SMG)
- `0.15` = Wide (shotgun)

### Ammo Types

Common Garry's Mod ammo types:

```lua
SWEP.Primary.Ammo = "Pistol"      -- Pistol rounds
SWEP.Primary.Ammo = "SMG1"        -- SMG rounds
SWEP.Primary.Ammo = "AR2"         -- Rifle rounds
SWEP.Primary.Ammo = "Buckshot"    -- Shotgun shells
SWEP.Primary.Ammo = "357"         -- Magnum rounds
SWEP.Primary.Ammo = "XBowBolt"    -- Crossbow bolts
SWEP.Primary.Ammo = "AR2AltFire"  -- Energy balls
SWEP.Primary.Ammo = "RPG_Round"   -- Rockets
SWEP.Primary.Ammo = "Grenade"     -- Grenades
SWEP.Primary.Ammo = "SniperRound" -- Sniper rounds
```

You can also define custom ammo types in your addon.

---

## Animations and Sounds

### View Model Configuration

Adjust the first-person view:

```lua
SWEP.ViewModelFOV = 54         -- Field of view (default: 62)
SWEP.ViewModelFlip = false     -- Don't flip model
SWEP.ViewModelOffset = Vector(0, 0, 0)  -- Position offset
SWEP.ViewModelAngle = Angle(0, 0, 0)    -- Rotation offset
SWEP.UseHands = true           -- Show player arms
```

### Sound Configuration

```lua
-- Single sound
SWEP.Primary.Sound = "Weapon_AR2.Single"

-- Random sound selection
function SWEP:FirePrimary()
    local sounds = {
        "weapons/ar2/fire1.wav",
        "weapons/ar2/fire2.wav",
        "weapons/ar2/fire3.wav"
    }
    self:EmitSound(sounds[math.random(#sounds)])
    self:CallBaseClass("FirePrimary")
end
```

### Custom Animations

```lua
function SWEP:CustomInitialize()
    -- Play custom animation on spawn
    self:SendWeaponAnim(ACT_VM_DRAW)
end

function SWEP:Reload()
    -- Custom reload with longer animation
    if self:Clip1() < self.Primary.ClipSize and self.Owner:GetAmmoCount(self.Primary.Ammo) > 0 then
        self:DefaultReload(ACT_VM_RELOAD)
        self:SetNextPrimaryFire(CurTime() + 2.5)
    end
end
```

---

## Projectile Weapons

### Creating Projectile Weapons

Instead of hitscan bullets, fire physical projectiles:

```lua
function SWEP:FirePrimary()
    if not SERVER then return end

    local owner = self.Owner
    local projectile = DrGBase.CreateProjectile("models/weapons/w_grenade.mdl", {
        Init = function(proj)
            -- Setup projectile
            proj:SetOwner(owner)
            proj:SetPos(owner:GetShootPos())

            -- Set velocity
            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(owner:GetAimVector() * 1500)
                phys:AddAngleVelocity(Vector(0, 600, 0))
            end
        end,

        Contact = function(proj, ent)
            -- On hit
            if IsValid(ent) and ent != owner then
                proj:DealDamage(ent, self.Primary.Damage)
            end
            return true -- Allow default contact behavior
        end
    })
end
```

### Projectile Configuration

The `DrGBase.CreateProjectile` function accepts these binds:

```lua
DrGBase.CreateProjectile("model/path.mdl", {
    Init = function(proj)
        -- Called when projectile spawns
        -- Setup initial state here
    end,

    Think = function(proj)
        -- Called every tick
        -- Return number to set think delay
    end,

    Contact = function(proj, ent)
        -- Called on collision
        -- Return false to prevent default behavior
    end,

    DealtDamage = function(proj, ent, dmg)
        -- Called when projectile damages something
    end,

    TakeDamage = function(proj, dmg)
        -- Called when projectile takes damage
    end,

    Remove = function(proj)
        -- Called when projectile is removed
    end
})
```

### Explosive Projectiles

```lua
function SWEP:FirePrimary()
    if not SERVER then return end

    local owner = self.Owner
    local projectile = DrGBase.CreateProjectile("models/weapons/w_grenade.mdl", {
        Init = function(proj)
            proj:SetOwner(owner)
            proj:SetPos(owner:GetShootPos())

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(owner:GetAimVector() * 1200)
            end

            -- Explode after 3 seconds
            proj:Timer(3, function()
                if IsValid(proj) then
                    proj:Explosion(100, 300) -- 100 damage, 300 units radius
                    proj:Remove()
                end
            end)
        end,

        Contact = function(proj, ent)
            -- Explode on impact with world
            if ent:IsWorld() then
                proj:Explosion(100, 300)
                proj:Remove()
            end
            return false -- Prevent default contact behavior
        end
    })

    self:ShootEffects()
end
```

### Guided Projectiles

```lua
function SWEP:FirePrimary()
    if not SERVER then return end

    local owner = self.Owner
    local target = owner:GetEyeTrace().Entity

    local projectile = DrGBase.CreateProjectile("models/weapons/w_missile.mdl", {
        Init = function(proj)
            proj:SetOwner(owner)
            proj:SetPos(owner:GetShootPos())
            proj.Target = target

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(owner:GetAimVector() * 800)
                phys:EnableGravity(false)
            end
        end,

        Think = function(proj)
            -- Home in on target
            if IsValid(proj.Target) then
                local phys = proj:GetPhysicsObject()
                if IsValid(phys) then
                    local vel = proj:AimAt(proj.Target, 1000)
                    phys:SetVelocity(vel)
                end
            end
            return 0.1 -- Think every 0.1 seconds
        end,

        Contact = function(proj, ent)
            proj:Explosion(150, 250)
            proj:Remove()
        end
    })
end
```

---

## Melee Weapons

### Basic Melee Weapon

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

-- Identification
SWEP.PrintName = "Crowbar"
SWEP.Category = "DrGBase - Melee"
SWEP.Spawnable = true

-- Appearance
SWEP.HoldType = "melee"
SWEP.ViewModel = "models/weapons/c_crowbar.mdl"
SWEP.WorldModel = "models/weapons/w_crowbar.mdl"

-- Primary Attack
SWEP.Primary.Damage = 25
SWEP.Primary.Delay = 0.5
SWEP.Primary.Automatic = true
SWEP.Primary.Sound = "Weapon_Crowbar.Single"
SWEP.Primary.ClipSize = -1  -- No ammo required
SWEP.Primary.DefaultClip = -1

-- Mark as melee weapon for DrGBase NPCs
SWEP.DrGBase_Melee = true

-- Custom melee attack
function SWEP:FirePrimary()
    local owner = self.Owner
    if not IsValid(owner) then return end

    -- Play animation
    owner:SetAnimation(PLAYER_ATTACK1)

    -- Trace ahead
    local tr = util.TraceLine({
        start = owner:GetShootPos(),
        endpos = owner:GetShootPos() + owner:GetAimVector() * 75,
        filter = owner
    })

    if SERVER then
        if tr.Hit and IsValid(tr.Entity) then
            -- Deal damage
            local dmg = DamageInfo()
            dmg:SetDamage(self.Primary.Damage)
            dmg:SetAttacker(owner)
            dmg:SetInflictor(self)
            dmg:SetDamageType(DMG_CLUB)
            dmg:SetDamagePosition(tr.HitPos)
            dmg:SetDamageForce(owner:GetAimVector() * 5000)
            tr.Entity:TakeDamageInfo(dmg)

            -- Hit effect
            local effect = EffectData()
            effect:SetOrigin(tr.HitPos)
            effect:SetNormal(tr.HitNormal)
            util.Effect("Impact", effect)
        else
            -- Miss effect
            self:EmitSound("Weapon_Crowbar.Melee_Miss")
        end
    end
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

### Advanced Melee with Swing Arc

```lua
function SWEP:FirePrimary()
    local owner = self.Owner
    if not IsValid(owner) then return end

    owner:SetAnimation(PLAYER_ATTACK1)

    if SERVER then
        -- Check arc in front of player
        local hitAnything = false
        local startPos = owner:GetShootPos()
        local forward = owner:GetAimVector()

        -- Sweep arc
        for angle = -30, 30, 15 do
            local dir = forward
            dir:Rotate(Angle(0, angle, 0))

            local tr = util.TraceLine({
                start = startPos,
                endpos = startPos + dir * 80,
                filter = owner
            })

            if tr.Hit and IsValid(tr.Entity) and not hitAnything then
                hitAnything = true

                local dmg = DamageInfo()
                dmg:SetDamage(self.Primary.Damage)
                dmg:SetAttacker(owner)
                dmg:SetInflictor(self)
                dmg:SetDamageType(DMG_SLASH)
                dmg:SetDamagePosition(tr.HitPos)
                dmg:SetDamageForce(forward * 8000)
                tr.Entity:TakeDamageInfo(dmg)

                -- Blood effect
                if tr.Entity:IsNPC() or tr.Entity:IsPlayer() then
                    local effect = EffectData()
                    effect:SetOrigin(tr.HitPos)
                    util.Effect("BloodImpact", effect)
                end
            end
        end

        if not hitAnything then
            self:EmitSound("Weapon_Crowbar.Melee_Miss")
        end
    end
end
```

---

## Custom Weapon Behaviors

### Attack Hooks

The weapon system provides several hooks to customize behavior:

```lua
-- Called before primary attack
-- Return false to cancel attack
function SWEP:PrePrimaryAttack()
    if self.Owner:WaterLevel() >= 3 then
        -- Can't fire underwater
        return false
    end
end

-- Override default firing behavior
function SWEP:FirePrimary()
    -- Your custom firing code here
    -- Default: self:ShootBullet(damage, bullets, spread)
end

-- Called after primary attack
function SWEP:PostPrimaryAttack(nextFireTime)
    -- Custom logic after firing
    -- nextFireTime is when weapon can fire again
end

-- Called when trying to fire with no ammo
function SWEP:TriedToPrimaryAttack()
    self:EmitSound(self.Primary.EmptySound)
    -- Add custom behavior here
end
```

Secondary attack has identical hooks:
- `PreSecondaryAttack()`
- `FireSecondary()`
- `PostSecondaryAttack(nextFireTime)`
- `TriedToSecondaryAttack()`

### Custom Think Function

```lua
function SWEP:CustomThink()
    -- Called every frame
    -- Add custom logic here

    -- Example: Regenerate ammo
    if SERVER and CurTime() > (self.NextAmmoRegen or 0) then
        local clip = self:Clip1()
        local maxClip = self.Primary.ClipSize

        if clip < maxClip then
            self:SetClip1(math.min(clip + 1, maxClip))
            self.NextAmmoRegen = CurTime() + 2
        end
    end
end
```

### Custom Initialize

```lua
function SWEP:CustomInitialize()
    -- Called when weapon is created

    -- Example: Set random skin
    self:SetSkin(math.random(0, 3))

    -- Example: Start with full clip
    if SERVER then
        self:SetClip1(self.Primary.ClipSize)
    end
end
```

### Burst Fire Weapon

```lua
SWEP.Primary.Automatic = false
SWEP.BurstCount = 3
SWEP.BurstDelay = 0.1

function SWEP:PrePrimaryAttack()
    if not self.BurstActive then
        self.BurstActive = true
        self.BurstsFired = 0
        return true
    end
    return false
end

function SWEP:PostPrimaryAttack(nextFireTime)
    self.BurstsFired = (self.BurstsFired or 0) + 1

    if self.BurstsFired >= self.BurstCount then
        -- Burst complete
        self.BurstActive = false
        self:SetNextPrimaryFire(CurTime() + self.Primary.Delay)
    else
        -- Continue burst
        timer.Simple(self.BurstDelay, function()
            if IsValid(self) and IsValid(self.Owner) then
                self:PrimaryAttack()
            end
        end)
    end
end
```

### Charge-up Weapon

```lua
SWEP.ChargeTime = 2.0  -- Seconds to fully charge
SWEP.MinCharge = 0.2   -- Minimum charge (20%)

function SWEP:CustomThink()
    if self.Owner:KeyDown(IN_ATTACK) and self:CanPrimaryAttack() then
        if not self.ChargeStart then
            self.ChargeStart = CurTime()
        end
    elseif self.ChargeStart then
        -- Release charged shot
        local chargePercent = math.Clamp((CurTime() - self.ChargeStart) / self.ChargeTime, self.MinCharge, 1)
        self:FireChargedShot(chargePercent)
        self.ChargeStart = nil
    end
end

function SWEP:FireChargedShot(chargePercent)
    if not IsFirstTimePredicted() then return end

    -- Scale damage by charge
    local damage = self.Primary.Damage * chargePercent
    self:ShootBullet(damage, 1, self.Primary.Spread / chargePercent)

    self:EmitSound(self.Primary.Sound, 100, 100 + chargePercent * 20)
    self:TakePrimaryAmmo(self.Primary.Cost)

    -- Add recoil
    if SERVER and self.Owner:IsPlayer() then
        local recoil = self.Primary.Recoil * chargePercent
        local eyeangles = self.Owner:EyeAngles()
        eyeangles.p = eyeangles.p - recoil
        self.Owner:SetEyeAngles(eyeangles)
        self.Owner:ViewPunch(Angle(-recoil/3, 0, 0))
    end

    self:SetNextPrimaryFire(CurTime() + self.Primary.Delay)
end

-- Prevent normal primary attack
function SWEP:PrimaryAttack()
    -- Handled in Think
end
```

---

## Complete Examples

### Example 1: Assault Rifle

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

-- Identification
SWEP.PrintName = "AR2 Rifle"
SWEP.Class = "weapon_drg_ar2"
SWEP.Category = "DrGBase - Half Life 2"
SWEP.Author = "Dragoteryx"
SWEP.Spawnable = true
SWEP.Slot = 2
SWEP.SlotPos = 0

-- Appearance
SWEP.HoldType = "ar2"
SWEP.ViewModelFOV = 54
SWEP.ViewModelFlip = false
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"

-- Primary Fire (Automatic Rifle)
SWEP.Primary.Damage = 10
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.02
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.1
SWEP.Primary.Recoil = 0.5

SWEP.Primary.Ammo = "AR2"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 30
SWEP.Primary.DefaultClip = 90

SWEP.Primary.Sound = "Weapon_AR2.Single"
SWEP.Primary.EmptySound = "Weapon_AR2.Empty"

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

### Example 2: Shotgun

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Combat Shotgun"
SWEP.Category = "DrGBase - Custom"
SWEP.Author = "YourName"
SWEP.Spawnable = true

-- Appearance
SWEP.HoldType = "shotgun"
SWEP.ViewModelFOV = 54
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_shotgun.mdl"
SWEP.WorldModel = "models/weapons/w_shotgun.mdl"

-- Primary Fire (Pump Shotgun)
SWEP.Primary.Damage = 8        -- Per pellet
SWEP.Primary.Bullets = 8       -- 8 pellets
SWEP.Primary.Spread = 0.15     -- Wide spread
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 1.0       -- Slow fire rate
SWEP.Primary.Recoil = 3.0      -- High recoil

SWEP.Primary.Ammo = "Buckshot"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 6
SWEP.Primary.DefaultClip = 24

SWEP.Primary.Sound = "Weapon_Shotgun.Single"
SWEP.Primary.EmptySound = "Weapon_Shotgun.Empty"

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

### Example 3: Grenade Launcher

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Grenade Launcher"
SWEP.Category = "DrGBase - Explosives"
SWEP.Author = "YourName"
SWEP.Spawnable = true

SWEP.HoldType = "rpg"
SWEP.ViewModelFOV = 54
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_rpg.mdl"
SWEP.WorldModel = "models/weapons/w_rocket_launcher.mdl"

-- Primary Fire
SWEP.Primary.Damage = 100
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 1.5
SWEP.Primary.Recoil = 2.0

SWEP.Primary.Ammo = "RPG_Round"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 1
SWEP.Primary.DefaultClip = 6

SWEP.Primary.Sound = "Weapon_RPG.Single"
SWEP.Primary.EmptySound = "Weapon_RPG.Empty"

-- Fire grenade projectile
function SWEP:FirePrimary()
    if not SERVER then return end

    local owner = self.Owner
    local shootPos = owner:GetShootPos()
    local shootDir = owner:GetAimVector()

    local grenade = DrGBase.CreateProjectile("models/weapons/w_grenade.mdl", {
        Init = function(proj)
            proj:SetOwner(owner)
            proj:SetPos(shootPos + shootDir * 32)

            -- Set velocity
            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(shootDir * 1500)
                phys:AddAngleVelocity(Vector(math.random(-300, 300), math.random(-300, 300), 0))
            end

            -- Explode after 2 seconds
            proj:Timer(2, function()
                if IsValid(proj) then
                    proj:Explosion(self.Primary.Damage, 300)
                    proj:Remove()
                end
            end)
        end,

        Contact = function(proj, ent)
            -- Bounce off surfaces
            if ent:IsWorld() then
                proj:EmitSound("weapons/hegrenade/he_bounce-1.wav")
            end
        end
    })

    self:ShootEffects()
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

### Example 4: Dual-Mode Weapon (Rifle + Grenade)

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Advanced Rifle"
SWEP.Category = "DrGBase - Custom"
SWEP.Author = "YourName"
SWEP.Spawnable = true

SWEP.HoldType = "ar2"
SWEP.ViewModelFOV = 54
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"

-- Primary Fire (Rapid Fire)
SWEP.Primary.Damage = 12
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.02
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.08
SWEP.Primary.Recoil = 0.4

SWEP.Primary.Ammo = "AR2"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 40
SWEP.Primary.DefaultClip = 120

SWEP.Primary.Sound = "Weapon_AR2.Single"
SWEP.Primary.EmptySound = "Weapon_AR2.Empty"

-- Secondary Fire (Grenade)
SWEP.Secondary.Enabled = true
SWEP.Secondary.Damage = 75
SWEP.Secondary.Automatic = false
SWEP.Secondary.Delay = 2.0
SWEP.Secondary.Recoil = 1.0

SWEP.Secondary.Ammo = "AR2AltFire"
SWEP.Secondary.Cost = 1
SWEP.Secondary.ClipSize = 0
SWEP.Secondary.DefaultClip = 3

SWEP.Secondary.Sound = "Weapon_AR2.Double"
SWEP.Secondary.EmptySound = "Weapon_AR2.Empty"

-- Override secondary fire for grenade
function SWEP:FireSecondary()
    if not SERVER then return end

    local owner = self.Owner
    local shootPos = owner:GetShootPos()
    local shootDir = owner:GetAimVector()

    local grenade = DrGBase.CreateProjectile("models/weapons/w_ngrenade.mdl", {
        Init = function(proj)
            proj:SetOwner(owner)
            proj:SetPos(shootPos + shootDir * 32)

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(shootDir * 1200)
                phys:EnableGravity(true)
            end

            proj:Timer(1.5, function()
                if IsValid(proj) then
                    proj:Explosion(self.Secondary.Damage, 250)
                    proj:Remove()
                end
            end)
        end
    })

    self:ShootEffects()
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

### Example 5: Plasma Rifle

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Plasma Rifle"
SWEP.Category = "DrGBase - Energy"
SWEP.Author = "YourName"
SWEP.Spawnable = true

SWEP.HoldType = "ar2"
SWEP.ViewModelFOV = 54
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"

-- Primary Fire
SWEP.Primary.Damage = 25
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.3
SWEP.Primary.Recoil = 1.0

SWEP.Primary.Ammo = "AR2AltFire"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 20
SWEP.Primary.DefaultClip = 60

SWEP.Primary.Sound = "weapons/physcannon/energy_sing_explosion2.wav"
SWEP.Primary.EmptySound = "weapons/pistol/pistol_empty.wav"

function SWEP:FirePrimary()
    if not SERVER then return end

    local owner = self.Owner
    local shootPos = owner:GetShootPos()
    local shootDir = owner:GetAimVector()

    local plasma = DrGBase.CreateProjectile("models/effects/combineball.mdl", {
        Init = function(proj)
            proj:SetOwner(owner)
            proj:SetPos(shootPos + shootDir * 32)
            proj:SetColor(Color(0, 255, 255))

            -- Glowing effect
            proj:SetRenderMode(RENDERMODE_TRANSADD)
            proj:SetMaterial("sprites/light_glow02_add")

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(shootDir * 2000)
                phys:EnableGravity(false)
                phys:EnableDrag(false)
            end

            -- Trail effect
            util.SpriteTrail(proj, 0, Color(0, 255, 255), false, 15, 0, 0.5, 1/15, "trails/plasma.vmt")

            -- Auto-remove after 5 seconds
            proj:Timer(5, function()
                if IsValid(proj) then
                    proj:Remove()
                end
            end)
        end,

        Think = function(proj)
            -- Emit light
            local dlight = DynamicLight(proj:EntIndex())
            if dlight then
                dlight.pos = proj:GetPos()
                dlight.r = 0
                dlight.g = 255
                dlight.b = 255
                dlight.brightness = 3
                dlight.size = 128
                dlight.decay = 512
                dlight.dietime = CurTime() + 0.1
            end
            return 0.05
        end,

        Contact = function(proj, ent)
            if IsValid(ent) and ent != owner then
                -- Deal damage
                proj:DealDamage(ent, self.Primary.Damage, DMG_ENERGYBEAM)

                -- Small explosion effect
                local effect = EffectData()
                effect:SetOrigin(proj:GetPos())
                util.Effect("cball_explode", effect)
            end

            proj:Remove()
            return false
        end
    })

    self:ShootEffects()
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

---

## Best Practices

### 1. Always Check for DrGBase

```lua
if not DrGBase then return end
```

This prevents errors if DrGBase isn't installed.

### 2. Use Appropriate Base

Always use `drgbase_weapon` as your base:

```lua
SWEP.Base = "drgbase_weapon"
```

### 3. Register Weapon Properly

Always include these at the end:

```lua
AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

### 4. Use IsFirstTimePredicted()

For client-side effects that shouldn't run twice:

```lua
if IsFirstTimePredicted() then
    self:EmitSound(self.Primary.Sound)
end
```

### 5. Check SERVER/CLIENT Realms

Projectile creation and damage should be server-side:

```lua
function SWEP:FirePrimary()
    if not SERVER then return end
    -- Server-only code
end
```

### 6. Validate Entities

Always check if entities are valid:

```lua
if IsValid(owner) and IsValid(projectile) then
    -- Safe to use
end
```

### 7. Set Appropriate Properties

Match properties to weapon type:

**Fast weapon:**
```lua
SWEP.Primary.Delay = 0.08
SWEP.Primary.Damage = 8
SWEP.Primary.Recoil = 0.3
```

**Slow weapon:**
```lua
SWEP.Primary.Delay = 1.5
SWEP.Primary.Damage = 50
SWEP.Primary.Recoil = 2.5
```

### 8. Use Descriptive Names

Use clear, descriptive names for custom weapons:

```lua
SWEP.PrintName = "Advanced Combat Rifle"
SWEP.Category = "DrGBase - Modern Warfare"
```

### 9. Balance Considerations

Consider these factors when balancing:
- **DPS** (Damage Per Second) = Damage × (1 / Delay)
- High damage = high recoil + slow fire rate
- High fire rate = low damage + high spread
- Effective range based on spread and damage falloff

### 10. Cleanup Timers

Use entity timers instead of global timers:

```lua
-- Good
proj:Timer(2, function() proj:Remove() end)

-- Bad
timer.Simple(2, function() proj:Remove() end)
```

Entity timers are automatically cleaned up when the entity is removed.

### 11. Test with DrGBase NPCs

DrGBase NPCs can use your weapons. Test that:
- Melee detection works (set `DrGBase_Melee = true`)
- Projectiles don't hit the shooter
- Ammo consumption is reasonable
- Fire rate isn't too fast for NPCs

### 12. Performance Optimization

- Avoid creating entities in Think loops
- Use timer delays for repeated tasks
- Clean up effects and sounds properly
- Don't spawn too many projectiles at once

### 13. Error Handling

```lua
function SWEP:FirePrimary()
    if not SERVER then return end

    local owner = self.Owner
    if not IsValid(owner) then return end

    -- Safe to proceed
end
```

### 14. Documentation

Comment complex functions:

```lua
-- Fires a charged plasma shot
-- @param chargePercent: 0-1, how charged the shot is
-- @return bool: whether shot was fired
function SWEP:FireChargedShot(chargePercent)
    -- Implementation
end
```

---

## Troubleshooting

### Weapon Doesn't Appear

- Check `SWEP.Spawnable = true`
- Verify `DrGBase.AddWeapon(SWEP)` is called
- Ensure `SWEP.PrintName` and `SWEP.Category` are set
- Check for Lua errors in console

### Weapon Won't Fire

- Check ammo configuration
- Verify `SWEP.Primary.Delay` isn't too high
- Check `CanPrimaryAttack()` isn't returning false
- Look for errors in `FirePrimary()`

### Projectiles Don't Work

- Ensure code is in `if SERVER then` block
- Check projectile model exists
- Verify physics object is valid
- Make sure velocity is being set

### Recoil Too Strong/Weak

Adjust these values:

```lua
SWEP.Primary.Recoil = 0.5  -- Lower = less recoil
```

### Spread Too Wide/Narrow

```lua
SWEP.Primary.Spread = 0.02  -- Lower = more accurate
```

### NPCs Won't Use Weapon

- Set `SWEP.Spawnable = true`
- For melee, add `SWEP.DrGBase_Melee = true`
- Check hold type is appropriate

---

## Additional Resources

- **DrGBase Documentation**: `/docs/README.md`
- **GMod Wiki**: https://wiki.facepunch.com/gmod/
- **Weapon Base**: `/lua/weapons/drgbase_weapon/`
- **Example Weapons**: `/lua/weapons/weapon_drg_*/`
- **Projectile System**: `/lua/entities/proj_drg_default/`

---

## Conclusion

You now have a comprehensive understanding of creating weapons for DrGBase. Start with simple weapons and gradually add complexity. Experiment with different configurations, and don't hesitate to look at the base weapon code for reference.

Key takeaways:
- DrGBase weapons are based on `drgbase_weapon`
- Configure properties through `SWEP.Primary` and `SWEP.Secondary` tables
- Override `FirePrimary()` and `FireSecondary()` for custom behavior
- Use `DrGBase.CreateProjectile()` for projectile weapons
- Always validate entities and use appropriate realms (SERVER/CLIENT)
- Test thoroughly with both players and DrGBase NPCs

Happy weapon creating!
