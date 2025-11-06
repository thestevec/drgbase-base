# Primary Attack Functions

Primary attack system for DrGBase weapons.

**File:** `weapons/drgbase_weapon/primary.lua`

---

## Attack Flow

When the player presses primary fire, the weapon system follows this flow:

1. `SWEP:PrimaryAttack()` is called automatically by Garry's Mod
2. Checks `SWEP:CanPrimaryAttack()` - if false, calls `SWEP:TriedToPrimaryAttack()` and returns
3. Calls `SWEP:PrePrimaryAttack()` hook - return `false` to cancel attack
4. Calls `SWEP:FirePrimary()` to execute the actual attack
5. Plays the fire sound
6. Consumes ammo
7. Applies recoil (for players)
8. Calls `SWEP:PostPrimaryAttack(nextFireTime)` hook
9. Sets next fire time based on `SWEP.Primary.Delay`

---

## Attack Hooks

### SWEP:PrePrimaryAttack()
**Realm:** Shared (with `IsFirstTimePredicted()` check)
**Returns:** `boolean` - Return `false` to cancel the attack

Called before the weapon fires. Use this to add custom conditions or behavior before attacking.

```lua
function SWEP:PrePrimaryAttack()
    -- Check if player is underwater
    if self.Owner:WaterLevel() >= 3 then
        self:EmitSound("weapons/pistol/pistol_empty.wav")
        return false -- Cancel attack
    end

    -- Allow attack to proceed
end
```

**Example - Require Charge:**
```lua
function SWEP:PrePrimaryAttack()
    if not self.ChargedUp then
        return false -- Cancel if not charged
    end
end
```

### SWEP:FirePrimary()
**Realm:** Shared (with `IsFirstTimePredicted()` check)
**Default Behavior:** Calls `self:ShootBullet()`

The actual fire logic. Override this to implement custom firing behavior.

```lua
-- Default implementation
function SWEP:FirePrimary()
    self:ShootBullet(self.Primary.Damage, self.Primary.Bullets, self.Primary.Spread)
end
```

**Example - Fire Projectile:**
```lua
function SWEP:FirePrimary()
    if SERVER then
        local proj = ents.Create("proj_drg_grenade")
        proj:SetPos(self.Owner:GetShootPos())
        proj:SetAngles(self.Owner:EyeAngles())
        proj:SetOwner(self.Owner)
        proj:Spawn()
        proj:Activate()

        local phys = proj:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(self.Owner:GetAimVector() * 1000)
        end
    end
end
```

**Example - Melee Attack:**
```lua
function SWEP:FirePrimary()
    if SERVER then
        local tr = util.TraceLine({
            start = self.Owner:GetShootPos(),
            endpos = self.Owner:GetShootPos() + self.Owner:GetAimVector() * 75,
            filter = self.Owner
        })

        if IsValid(tr.Entity) then
            local dmg = DamageInfo()
            dmg:SetDamage(self.Primary.Damage)
            dmg:SetAttacker(self.Owner)
            dmg:SetInflictor(self)
            dmg:SetDamageType(DMG_CLUB)
            tr.Entity:TakeDamageInfo(dmg)
        end
    end
end
```

### SWEP:PostPrimaryAttack(nextFireTime)
**Realm:** Shared (with `IsFirstTimePredicted()` check)
**Parameters:**
- `nextFireTime` (number) - When the weapon can fire again (`CurTime() + Delay`)

Called after the weapon fires. Use this for effects, animations, or follow-up actions.

```lua
function SWEP:PostPrimaryAttack(nextFireTime)
    -- Play muzzle flash effect
    self:SendWeaponAnim(ACT_VM_PRIMARYATTACK)

    -- Decrease weapon durability
    self.Durability = self.Durability - 1
    if self.Durability <= 0 then
        self:Remove()
    end
end
```

---

## Attack Checks

### SWEP:CanPrimaryAttack()
**Realm:** Shared
**Returns:** `boolean` - `true` if weapon can fire

Checks if the weapon has enough ammo to fire. Automatically called before firing.

**Default Behavior:**
- If weapon has no ammo type (`GetPrimaryAmmoType() < 0`), always returns `true`
- If `ClipSize > 0`, checks if clip has enough ammo (`Clip1() >= Cost`)
- Otherwise, checks if player has enough reserve ammo

```lua
-- Default implementation
function SWEP:CanPrimaryAttack()
    if self:GetPrimaryAmmoType() < 0 then return true end
    if self.Primary.ClipSize > 0 then
        return self.Weapon:Clip1() >= self.Primary.Cost
    else
        return self.Owner:GetAmmoCount(self.Primary.Ammo) >= self.Primary.Cost
    end
end
```

**Example - Custom Condition:**
```lua
function SWEP:CanPrimaryAttack()
    -- Still check ammo
    if not BaseClass.CanPrimaryAttack(self) then
        return false
    end

    -- Also check if weapon is charged
    return self.ChargedUp
end
```

### SWEP:TriedToPrimaryAttack()
**Realm:** Shared (with `IsFirstTimePredicted()` check)

Called when the player tries to fire but `CanPrimaryAttack()` returns `false`. Default behavior plays the empty sound and sets the next fire delay.

```lua
-- Default implementation
function SWEP:TriedToPrimaryAttack()
    self:EmitSound(self.Primary.EmptySound)
    self:SetNextPrimaryFire(CurTime() + self.Primary.Delay)
end
```

**Example - Show Warning:**
```lua
function SWEP:TriedToPrimaryAttack()
    self:EmitSound(self.Primary.EmptySound)
    self:SetNextPrimaryFire(CurTime() + self.Primary.Delay)

    if CLIENT and self.Owner == LocalPlayer() then
        notification.AddLegacy("Out of ammo!", NOTIFY_ERROR, 2)
    end
end
```

---

## Core Functions

### SWEP:PrimaryAttack()
**Realm:** Shared
**Returns:** `boolean` - `true` if attack succeeded, `false` otherwise

The main primary attack function. Called automatically by Garry's Mod when the player fires. You typically don't override this - instead use the hooks above.

**Flow:**
1. Check `CanPrimaryAttack()` - if false, trigger `TriedToPrimaryAttack()` and return
2. Call `PrePrimaryAttack()` - if returns `false`, cancel attack
3. Call `FirePrimary()` to do the actual attack
4. Play fire sound
5. Consume ammo via `TakePrimaryAmmo()`
6. Apply recoil (players only)
7. Call `PostPrimaryAttack()`
8. Set next fire time

### SWEP:ShootBullet(damage, num_bullets, aimcone)
**Realm:** Shared
**Parameters:**
- `damage` (number) - Damage per bullet
- `num_bullets` (number) - Number of bullets to fire
- `aimcone` (number) - Spread/accuracy (0 = perfect accuracy)

Helper function to fire hitscan bullets. Called by the default `FirePrimary()`.

**Default Implementation:**
```lua
function SWEP:ShootBullet(damage, num_bullets, aimcone)
    local bullet = {}
    bullet.Num = num_bullets
    bullet.Src = self.Owner:GetShootPos()
    bullet.Dir = self.Owner:GetAimVector()
    bullet.Spread = Vector(aimcone, aimcone, 0)
    bullet.Tracer = 1
    bullet.Force = damage/10
    bullet.Damage = damage
    bullet.AmmoType = "Pistol"
    bullet.Callback = function(ent, tr, dmg)
        dmg:SetAttacker(self.Owner)
        dmg:SetInflictor(self)
    end
    self.Owner:FireBullets(bullet)
    self:ShootEffects()
end
```

**Example - Custom Tracer:**
```lua
function SWEP:ShootBullet(damage, num_bullets, aimcone)
    local bullet = {}
    bullet.Num = num_bullets
    bullet.Src = self.Owner:GetShootPos()
    bullet.Dir = self.Owner:GetAimVector()
    bullet.Spread = Vector(aimcone, aimcone, 0)
    bullet.Tracer = 1
    bullet.TracerName = "ToolTracer" -- Custom tracer effect
    bullet.Force = damage/10
    bullet.Damage = damage
    bullet.AmmoType = "Pistol"
    bullet.Callback = function(ent, tr, dmg)
        dmg:SetAttacker(self.Owner)
        dmg:SetInflictor(self)

        -- Extra effect on hit
        if IsValid(tr.Entity) then
            local effectdata = EffectData()
            effectdata:SetOrigin(tr.HitPos)
            effectdata:SetNormal(tr.HitNormal)
            util.Effect("ManhackSparks", effectdata)
        end
    end
    self.Owner:FireBullets(bullet)
    self:ShootEffects()
end
```

---

## Reloading

### SWEP:Reload()
**Realm:** Shared
**Returns:** `boolean` - `true` if reload started

Handles weapon reloading. The default implementation uses Garry's Mod's standard reload system.

```lua
-- Default implementation
function SWEP:Reload()
    self:DefaultReload(ACT_VM_RELOAD)
end
```

**Example - Custom Reload:**
```lua
function SWEP:Reload()
    if self.ReloadingTime and CurTime() < self.ReloadingTime then
        return false
    end

    -- Check if we need to reload
    if self.Weapon:Clip1() >= self.Primary.ClipSize then
        return false
    end
    if self.Owner:GetAmmoCount(self.Primary.Ammo) <= 0 then
        return false
    end

    -- Start reload
    self.Weapon:SetNextPrimaryFire(CurTime() + 2)
    self.Weapon:SetNextSecondaryFire(CurTime() + 2)
    self.ReloadingTime = CurTime() + 2

    self:SendWeaponAnim(ACT_VM_RELOAD)
    self.Owner:SetAnimation(PLAYER_RELOAD)

    timer.Simple(2, function()
        if IsValid(self) and IsValid(self.Owner) then
            self:DefaultReload(ACT_VM_IDLE)
        end
    end)

    return true
end
```

---

## Complete Example

Example of a weapon with custom primary attack behavior:

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Plasma Rifle"
SWEP.Category = "My Weapons"

-- Primary configuration
SWEP.Primary.Damage = 25
SWEP.Primary.Delay = 0.2
SWEP.Primary.ClipSize = 20
SWEP.Primary.DefaultClip = 60
SWEP.Primary.Ammo = "AR2"
SWEP.Primary.Sound = "weapons/ar2/fire1.wav"

-- Charge system
SWEP.ChargeTime = 0.5
SWEP.MaxCharge = 100

function SWEP:CustomInitialize()
    self.CurrentCharge = 0
end

function SWEP:CustomThink()
    -- Charge while holding fire
    if self.Owner:KeyDown(IN_ATTACK) and self.CurrentCharge < self.MaxCharge then
        self.CurrentCharge = math.min(self.CurrentCharge + 2, self.MaxCharge)
    end
end

function SWEP:PrePrimaryAttack()
    -- Require minimum charge
    if self.CurrentCharge < 20 then
        return false
    end
end

function SWEP:FirePrimary()
    -- Damage scales with charge
    local damage = self.Primary.Damage * (self.CurrentCharge / 100)
    self:ShootBullet(damage, 1, 0.01)
end

function SWEP:PostPrimaryAttack()
    -- Reset charge after firing
    self.CurrentCharge = 0
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

---

## See Also

- [Weapon Base Configuration](base-config.md) - Primary attack properties
- [Secondary Attack Functions](secondary.md) - Secondary attack system
- [Weapon Functions & Hooks](functions.md) - General weapon functions
- [Creating Weapons Guide](../../guides/creating-weapons.md) - Step-by-step weapon creation tutorial
