# Secondary Attack Functions

Secondary attack system for DrGBase weapons.

**File:** `weapons/drgbase_weapon/secondary.lua`

---

## Overview

The secondary attack system is nearly identical to the primary attack system, but must be explicitly enabled via `SWEP.Secondary.Enabled = true`. Secondary attacks are commonly used for:
- Alternate fire modes (zoom, burst fire, etc.)
- Melee attacks on ranged weapons
- Special abilities (grenades, flashlights, etc.)
- Alternative ammo types

---

## Attack Flow

When the player presses secondary fire, the weapon system follows this flow:

1. `SWEP:SecondaryAttack()` is called automatically by Garry's Mod
2. Checks if `SWEP.Secondary.Enabled` is `true` - if false, returns immediately
3. Checks `SWEP:CanSecondaryAttack()` - if false, calls `SWEP:TriedToSecondaryAttack()` and returns
4. Calls `SWEP:PreSecondaryAttack()` hook - return `false` to cancel attack
5. Calls `SWEP:FireSecondary()` to execute the actual attack
6. Plays the fire sound
7. Consumes ammo
8. Applies recoil (for players)
9. Calls `SWEP:PostSecondaryAttack(nextFireTime)` hook
10. Sets next fire time based on `SWEP.Secondary.Delay`

---

## Attack Hooks

### SWEP:PreSecondaryAttack()
**Realm:** Shared (with `IsFirstTimePredicted()` check)
**Returns:** `boolean` - Return `false` to cancel the attack

Called before the weapon fires secondary. Use this to add custom conditions or behavior before attacking.

```lua
function SWEP:PreSecondaryAttack()
    -- Check if weapon is zoomed in
    if not self.IsZoomed then
        return false -- Must zoom first
    end
end
```

**Example - Energy Requirement:**
```lua
function SWEP:PreSecondaryAttack()
    if self.Energy < 25 then
        if CLIENT then
            notification.AddLegacy("Insufficient energy!", NOTIFY_ERROR, 2)
        end
        return false
    end
end
```

### SWEP:FireSecondary()
**Realm:** Shared (with `IsFirstTimePredicted()` check)
**Default Behavior:** Calls `self:ShootBullet()`

The actual fire logic for secondary attack. Override this to implement custom firing behavior.

```lua
-- Default implementation
function SWEP:FireSecondary()
    self:ShootBullet(self.Secondary.Damage, self.Secondary.Bullets, self.Secondary.Spread)
end
```

**Example - Grenade Launcher:**
```lua
function SWEP:FireSecondary()
    if SERVER then
        local grenade = ents.Create("proj_drg_grenade")
        grenade:SetPos(self.Owner:GetShootPos() + self.Owner:GetAimVector() * 20)
        grenade:SetAngles(self.Owner:EyeAngles())
        grenade:SetOwner(self.Owner)
        grenade:Spawn()
        grenade:Activate()
        grenade:Unpin()

        local phys = grenade:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(self.Owner:GetAimVector() * 1200 + Vector(0, 0, 100))
        end
    end
end
```

**Example - Zoom/Aim:**
```lua
function SWEP:FireSecondary()
    -- Toggle zoom (no ammo consumed)
    self.IsZoomed = not self.IsZoomed

    if CLIENT then
        if self.IsZoomed then
            self.Owner:SetFOV(20, 0.3)
        else
            self.Owner:SetFOV(0, 0.2)
        end
    end
end
```

### SWEP:PostSecondaryAttack(nextFireTime)
**Realm:** Shared (with `IsFirstTimePredicted()` check)
**Parameters:**
- `nextFireTime` (number) - When the weapon can fire again (`CurTime() + Delay`)

Called after the weapon fires secondary. Use this for effects, animations, or follow-up actions.

```lua
function SWEP:PostSecondaryAttack(nextFireTime)
    -- Play special animation
    self:SendWeaponAnim(ACT_VM_SECONDARYATTACK)

    -- Consume energy instead of ammo
    self.Energy = self.Energy - 25
end
```

---

## Attack Checks

### SWEP:CanSecondaryAttack()
**Realm:** Shared
**Returns:** `boolean` - `true` if weapon can fire

Checks if the weapon can fire secondary. Automatically called before firing.

**Default Behavior:**
- If `Secondary.Enabled` is `false`, returns `false`
- If weapon has no ammo type (`GetSecondaryAmmoType() < 0`), returns `true`
- If `ClipSize > 0`, checks if clip has enough ammo (`Clip2() >= Cost`)
- Otherwise, checks if player has enough reserve ammo

```lua
-- Default implementation
function SWEP:CanSecondaryAttack()
    if not self.Secondary.Enabled then return false end
    if self:GetSecondaryAmmoType() < 0 then return true end
    if self.Secondary.ClipSize > 0 then
        return self.Weapon:Clip2() >= self.Secondary.Cost
    else
        return self.Owner:GetAmmoCount(self.Secondary.Ammo) >= self.Secondary.Cost
    end
end
```

**Example - Custom Condition:**
```lua
function SWEP:CanSecondaryAttack()
    -- Check base requirements
    if not BaseClass.CanSecondaryAttack(self) then
        return false
    end

    -- Also check cooldown
    if CurTime() < self.NextSpecialAttack then
        return false
    end

    return true
end
```

### SWEP:TriedToSecondaryAttack()
**Realm:** Shared (with `IsFirstTimePredicted()` check)

Called when the player tries to fire secondary but `CanSecondaryAttack()` returns `false`. Default behavior plays the empty sound and sets the next fire delay.

```lua
-- Default implementation
function SWEP:TriedToSecondaryAttack()
    self:EmitSound(self.Secondary.EmptySound)
    self:SetNextSecondaryFire(CurTime() + self.Secondary.Delay)
end
```

**Example - Custom Feedback:**
```lua
function SWEP:TriedToSecondaryAttack()
    self:EmitSound(self.Secondary.EmptySound)
    self:SetNextSecondaryFire(CurTime() + self.Secondary.Delay)

    if CLIENT and self.Owner == LocalPlayer() then
        notification.AddLegacy("Special attack on cooldown!", NOTIFY_HINT, 2)
    end
end
```

---

## Core Functions

### SWEP:SecondaryAttack()
**Realm:** Shared
**Returns:** `boolean` - `true` if attack succeeded, `false` otherwise

The main secondary attack function. Called automatically by Garry's Mod when the player uses secondary fire. You typically don't override this - instead use the hooks above.

**Flow:**
1. Check if `Secondary.Enabled` - if false, return immediately
2. Check `CanSecondaryAttack()` - if false, trigger `TriedToSecondaryAttack()` and return
3. Call `PreSecondaryAttack()` - if returns `false`, cancel attack
4. Call `FireSecondary()` to do the actual attack
5. Play fire sound
6. Consume ammo via `TakeSecondaryAmmo()`
7. Apply recoil (players only)
8. Call `PostSecondaryAttack()`
9. Set next fire time

---

## Common Use Cases

### Zoom/Aim Down Sights

```lua
SWEP.Secondary.Enabled = true
SWEP.Secondary.Delay = 0.1
SWEP.Secondary.Ammo = "" -- No ammo required

function SWEP:FireSecondary()
    -- Toggle zoom (no ammo consumed since Ammo is "")
    self.IsZoomed = not self.IsZoomed

    if CLIENT and self.Owner == LocalPlayer() then
        if self.IsZoomed then
            self.Owner:SetFOV(40, 0.2)
        else
            self.Owner:SetFOV(0, 0.2)
        end
    end
end

function SWEP:CustomThink()
    -- Unzoom when weapon is put away
    if CLIENT and self.Owner == LocalPlayer() and not self.Owner:GetActiveWeapon() == self then
        if self.IsZoomed then
            self.Owner:SetFOV(0, 0.1)
            self.IsZoomed = false
        end
    end
end
```

### Melee Attack on Ranged Weapon

```lua
SWEP.Secondary.Enabled = true
SWEP.Secondary.Damage = 50
SWEP.Secondary.Delay = 0.8
SWEP.Secondary.Ammo = "" -- No ammo required
SWEP.Secondary.Sound = "weapons/crowbar/crowbar_impact1.wav"

function SWEP:FireSecondary()
    if SERVER then
        -- Melee trace
        local tr = util.TraceLine({
            start = self.Owner:GetShootPos(),
            endpos = self.Owner:GetShootPos() + self.Owner:GetAimVector() * 60,
            filter = self.Owner
        })

        if IsValid(tr.Entity) then
            local dmg = DamageInfo()
            dmg:SetDamage(self.Secondary.Damage)
            dmg:SetAttacker(self.Owner)
            dmg:SetInflictor(self)
            dmg:SetDamageType(DMG_CLUB)
            dmg:SetDamageForce(self.Owner:GetAimVector() * 5000)
            tr.Entity:TakeDamageInfo(dmg)
        end
    end

    -- Swing animation
    self:SendWeaponAnim(ACT_VM_HITCENTER)
    self.Owner:SetAnimation(PLAYER_ATTACK1)
end
```

### Grenade Launcher (Underbarrel)

```lua
SWEP.Secondary.Enabled = true
SWEP.Secondary.ClipSize = 1
SWEP.Secondary.DefaultClip = 3
SWEP.Secondary.Ammo = "SMG1_Grenade"
SWEP.Secondary.Delay = 1.5
SWEP.Secondary.Sound = "weapons/ar2/ar2_altfire.wav"

function SWEP:FireSecondary()
    if SERVER then
        local grenade = ents.Create("proj_drg_grenade")
        grenade:SetPos(self.Owner:GetShootPos() + self.Owner:GetAimVector() * 30)
        grenade:SetAngles(self.Owner:EyeAngles())
        grenade:SetOwner(self.Owner)
        grenade:SetDamage(150)
        grenade:SetRange(300)
        grenade:Spawn()
        grenade:Activate()
        grenade:Unpin()

        local phys = grenade:GetPhysicsObject()
        if IsValid(phys) then
            local vel = self.Owner:GetAimVector() * 1500 + Vector(0, 0, 50)
            phys:SetVelocity(vel)
            phys:AddAngleVelocity(VectorRand() * 300)
        end
    end

    -- Kick back
    if SERVER and self.Owner:IsPlayer() then
        self.Owner:ViewPunch(Angle(-5, 0, 0))
    end
end
```

### Burst Fire Mode

```lua
SWEP.Secondary.Enabled = true
SWEP.Secondary.Damage = 15
SWEP.Secondary.Bullets = 1
SWEP.Secondary.Spread = 0.03
SWEP.Secondary.Delay = 0.5
SWEP.Secondary.Ammo = "AR2"
SWEP.Secondary.Cost = 3 -- Uses 3 bullets
SWEP.Secondary.Sound = "Weapon_AR2.Single"

function SWEP:FireSecondary()
    -- Fire 3-round burst
    for i = 1, 3 do
        timer.Simple(i * 0.08, function()
            if IsValid(self) and IsValid(self.Owner) then
                self:ShootBullet(self.Secondary.Damage, 1, self.Secondary.Spread)
                self:EmitSound(self.Secondary.Sound)
            end
        end)
    end
end
```

---

## Complete Example

Example weapon with both primary and secondary attacks:

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Tactical Rifle"
SWEP.Category = "My Weapons"

-- Primary: Full auto rifle
SWEP.Primary.Damage = 12
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.03
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.08
SWEP.Primary.Recoil = 0.3
SWEP.Primary.ClipSize = 30
SWEP.Primary.DefaultClip = 90
SWEP.Primary.Ammo = "AR2"
SWEP.Primary.Sound = "Weapon_AR2.Single"

-- Secondary: Zoom
SWEP.Secondary.Enabled = true
SWEP.Secondary.Delay = 0.1
SWEP.Secondary.Ammo = "" -- No ammo required

function SWEP:CustomInitialize()
    self.IsZoomed = false
end

-- Toggle zoom on secondary fire
function SWEP:FireSecondary()
    self.IsZoomed = not self.IsZoomed

    if CLIENT and self.Owner == LocalPlayer() then
        if self.IsZoomed then
            self.Owner:SetFOV(30, 0.2)
        else
            self.Owner:SetFOV(0, 0.2)
        end
    end
end

-- Reduce spread when zoomed
function SWEP:PrePrimaryAttack()
    if self.IsZoomed then
        self.Primary.Spread = 0.01 -- More accurate when zoomed
    else
        self.Primary.Spread = 0.03 -- Normal spread
    end
end

function SWEP:OnRemove()
    -- Unzoom when weapon is removed
    if CLIENT and IsValid(self.Owner) and self.Owner == LocalPlayer() then
        self.Owner:SetFOV(0, 0.1)
    end
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

---

## See Also

- [Weapon Base Configuration](base-config.md) - Secondary attack properties
- [Primary Attack Functions](primary.md) - Primary attack system
- [Weapon Functions & Hooks](functions.md) - General weapon functions
- [Creating Weapons Guide](../../guides/creating-weapons.md) - Step-by-step weapon creation tutorial
