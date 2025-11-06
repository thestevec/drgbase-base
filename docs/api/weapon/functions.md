# Weapon Functions & Hooks

Common weapon functions and hooks for DrGBase weapons.

**Files:** `weapons/drgbase_weapon/*.lua`

---

## Initialization

### SWEP:Initialize()
**Realm:** Shared

The main initialization function called when the weapon is created. Sets up the weapon and calls initialization hooks.

**Default Behavior:**
```lua
function SWEP:Initialize()
    if SERVER then
        self:SetHoldType(self.HoldType)
    end
    self:_BaseInitialize()
    self:CustomInitialize()
end
```

Do not override this function. Use `CustomInitialize()` instead.

### SWEP:_BaseInitialize()
**Realm:** Shared

Internal hook for base weapon initialization. Reserved for future use by DrGBase.

### SWEP:CustomInitialize()
**Realm:** Shared

Custom initialization hook for your weapon. Called after the base weapon is initialized.

```lua
function SWEP:CustomInitialize()
    -- Initialize weapon variables
    self.Charge = 0
    self.Overheat = 0
    self.IsZoomed = false

    -- Set up initial state
    if SERVER then
        self:SetNW2Int("Charge", 0)
    end
end
```

**Example - Weapon Durability:**
```lua
function SWEP:CustomInitialize()
    self.Durability = 100
    self.MaxDurability = 100

    if SERVER then
        self:SetNW2Int("Durability", self.Durability)
    end
end
```

---

## Think

### SWEP:Think()
**Realm:** Shared

The main think function called every frame. Calls think hooks.

**Default Behavior:**
```lua
function SWEP:Think()
    self:_BaseThink()
    self:CustomThink()
end
```

Do not override this function. Use `CustomThink()` instead.

### SWEP:_BaseThink()
**Realm:** Shared

Internal hook for base weapon thinking. Reserved for future use by DrGBase.

### SWEP:CustomThink()
**Realm:** Shared

Custom think hook for your weapon. Called every frame while the weapon exists.

```lua
function SWEP:CustomThink()
    -- Regenerate charge over time
    if self.Charge < 100 then
        self.Charge = math.min(self.Charge + 0.5, 100)
    end

    -- Check for overheat
    if self.Overheat > 0 then
        self.Overheat = math.max(self.Overheat - 0.3, 0)

        -- Prevent firing while overheated
        if self.Overheat > 0 then
            self:SetNextPrimaryFire(CurTime() + 0.1)
        end
    end
end
```

**Example - Laser Sight:**
```lua
function SWEP:CustomThink()
    if CLIENT then return end

    -- Draw laser sight
    if IsValid(self.Owner) and self.Owner:IsPlayer() then
        local tr = util.TraceLine({
            start = self.Owner:GetShootPos(),
            endpos = self.Owner:GetShootPos() + self.Owner:GetAimVector() * 5000,
            filter = self.Owner
        })

        if tr.Hit then
            local effect = EffectData()
            effect:SetOrigin(tr.HitPos)
            effect:SetNormal(tr.HitNormal)
            effect:SetScale(0.5)
            util.Effect("RedGlowStick", effect)
        end
    end
end
```

**Example - Energy System:**
```lua
function SWEP:CustomThink()
    -- Recharge energy when not firing
    if not self.Owner:KeyDown(IN_ATTACK) then
        self.Energy = math.min(self.Energy + 0.5, self.MaxEnergy)
    end

    -- Network energy to client
    if SERVER then
        self:SetNW2Float("Energy", self.Energy)
    end

    -- HUD display
    if CLIENT and self.Owner == LocalPlayer() then
        local x = ScrW() / 2
        local y = ScrH() - 100
        draw.SimpleText("Energy: " .. math.Round(self.Energy), "DermaDefault", x, y, Color(255, 255, 255), TEXT_ALIGN_CENTER)
    end
end
```

---

## Reload

### SWEP:Reload()
**Realm:** Shared
**Returns:** `boolean` - `true` if reload started

Handles weapon reloading. See [Primary Attack Functions](primary.md#swe preload) for details.

```lua
-- Default implementation
function SWEP:Reload()
    self:DefaultReload(ACT_VM_RELOAD)
end
```

---

## Drawing & Rendering

### SWEP:CalcViewModelView(vm, oldpos, oldang, pos, ang)
**Realm:** Client
**Parameters:**
- `vm` (Entity) - The viewmodel entity
- `oldpos` (Vector) - Original viewmodel position
- `oldang` (Angle) - Original viewmodel angles
- `pos` (Vector) - Current eye position
- `ang` (Angle) - Current eye angles
**Returns:** `Vector, Angle` - New viewmodel position and angles

Calculates the viewmodel position and angle based on `ViewModelOffset` and `ViewModelAngle` properties.

**Default Behavior:**
```lua
function SWEP:CalcViewModelView(vm, oldpos, oldang, pos, ang)
    local aimpos = pos +
        self.Owner:GetForward()*self.ViewModelOffset.x +
        self.Owner:GetUp()*self.ViewModelOffset.z

    if self.ViewModelFlip then
        aimpos = aimpos - self.Owner:GetRight()*self.ViewModelOffset.y
    else
        aimpos = aimpos + self.Owner:GetRight()*self.ViewModelOffset.y
    end

    local aimang = ang + self.ViewModelAngle
    return aimpos, aimang
end
```

You can override this for custom viewmodel positioning:

```lua
function SWEP:CalcViewModelView(vm, oldpos, oldang, pos, ang)
    -- Bob the weapon when walking
    local bob = 0
    if self.Owner:GetVelocity():Length() > 0 then
        bob = math.sin(CurTime() * 10) * 2
    end

    local offset = self.ViewModelOffset + Vector(0, 0, bob)

    local aimpos = pos +
        self.Owner:GetForward()*offset.x +
        self.Owner:GetRight()*offset.y +
        self.Owner:GetUp()*offset.z

    return aimpos, ang + self.ViewModelAngle
end
```

---

## Network/Prediction

### SWEP:OnRemove()
**Realm:** Shared

Called when the weapon is removed. Use for cleanup.

```lua
function SWEP:OnRemove()
    -- Clean up client effects
    if CLIENT and IsValid(self.Owner) and self.Owner == LocalPlayer() then
        -- Reset FOV if zoomed
        if self.IsZoomed then
            self.Owner:SetFOV(0, 0.1)
        end
    end

    -- Clean up server timers
    if SERVER then
        timer.Remove("weapon_" .. self:EntIndex() .. "_reload")
    end
end
```

### SWEP:OnDrop()
**Realm:** Server

Called when the weapon is dropped by a player.

```lua
function SWEP:OnDrop()
    -- Reset weapon state when dropped
    self.Charge = 0
    self.Overheat = 0

    -- Remove any attachments
    if IsValid(self.LaserSight) then
        self.LaserSight:Remove()
    end
end
```

### SWEP:OwnerChanged()
**Realm:** Shared

Called when the weapon's owner changes.

```lua
function SWEP:OwnerChanged()
    -- Reset zoom when weapon changes hands
    self.IsZoomed = false

    -- Update HUD for new owner
    if CLIENT and IsValid(self.Owner) and self.Owner == LocalPlayer() then
        self:SetupHUD()
    end
end
```

---

## Utility Functions

### SWEP:ShootBullet(damage, num_bullets, aimcone)
**Realm:** Shared
**Parameters:**
- `damage` (number) - Damage per bullet
- `num_bullets` (number) - Number of bullets to fire
- `aimcone` (number) - Bullet spread (0 = perfect accuracy)

Fires hitscan bullets. See [Primary Attack Functions](primary.md#swepshootbulletdamage-num_bullets-aimcone) for details.

### SWEP:ShootEffects()
**Realm:** Shared

Plays muzzle flash and shell eject effects. Called automatically by `ShootBullet()`.

---

## Complete Example

Example weapon with custom functions:

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

SWEP.PrintName = "Energy Rifle"
SWEP.Category = "My Weapons"

-- Appearance
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"
SWEP.HoldType = "ar2"

-- Primary fire
SWEP.Primary.Damage = 20
SWEP.Primary.Delay = 0.1
SWEP.Primary.ClipSize = -1  -- Infinite clip
SWEP.Primary.Ammo = ""  -- No ammo
SWEP.Primary.Sound = "weapons/ar2/fire1.wav"

-- Energy system
SWEP.MaxEnergy = 100
SWEP.EnergyPerShot = 5
SWEP.RechargeRate = 2

function SWEP:CustomInitialize()
    self.Energy = self.MaxEnergy
    self.Overheated = false
end

function SWEP:CustomThink()
    -- Recharge energy
    if not self.Owner:KeyDown(IN_ATTACK) then
        self.Energy = math.min(self.Energy + self.RechargeRate * FrameTime(), self.MaxEnergy)

        -- Cool down from overheat
        if self.Overheated and self.Energy >= self.MaxEnergy then
            self.Overheated = false
        end
    end

    -- Network energy to client
    if SERVER then
        self:SetNW2Float("Energy", self.Energy)
        self:SetNW2Bool("Overheated", self.Overheated)
    end

    -- Draw HUD
    if CLIENT and self.Owner == LocalPlayer() then
        local x, y = ScrW() / 2, ScrH() - 80
        local barWidth = 200
        local barHeight = 20

        -- Background
        draw.RoundedBox(4, x - barWidth/2, y, barWidth, barHeight, Color(0, 0, 0, 200))

        -- Energy bar
        local fillWidth = (self:GetNW2Float("Energy", 100) / self.MaxEnergy) * barWidth
        local color = self:GetNW2Bool("Overheated", false) and Color(255, 0, 0) or Color(0, 150, 255)
        draw.RoundedBox(4, x - barWidth/2, y, fillWidth, barHeight, color)

        -- Text
        local text = self:GetNW2Bool("Overheated", false) and "OVERHEATED!" or math.Round(self:GetNW2Float("Energy", 100))
        draw.SimpleText(text, "DermaDefault", x, y + barHeight/2, Color(255, 255, 255), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
    end
end

function SWEP:CanPrimaryAttack()
    -- Check energy instead of ammo
    if self.Overheated then
        return false
    end

    return self.Energy >= self.EnergyPerShot
end

function SWEP:TriedToPrimaryAttack()
    -- Custom empty sound
    self:EmitSound("buttons/button10.wav")
    self:SetNextPrimaryFire(CurTime() + 0.2)
end

function SWEP:PostPrimaryAttack()
    -- Consume energy
    self.Energy = math.max(self.Energy - self.EnergyPerShot, 0)

    -- Check for overheat
    if self.Energy <= 0 then
        self.Overheated = true
        self:EmitSound("ambient/alarms/warningbell1.wav")
        self:SetNextPrimaryFire(CurTime() + 2)
    end
end

function SWEP:OnRemove()
    -- Clean up HUD
    if CLIENT and IsValid(self.Owner) and self.Owner == LocalPlayer() then
        -- Any cleanup needed
    end
end

AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

---

## Hook Summary

| Hook | Realm | Purpose |
|------|-------|---------|
| `CustomInitialize()` | Shared | Initialize weapon variables |
| `CustomThink()` | Shared | Per-frame logic |
| `PrePrimaryAttack()` | Shared | Before primary fire |
| `FirePrimary()` | Shared | Primary fire logic |
| `PostPrimaryAttack()` | Shared | After primary fire |
| `PreSecondaryAttack()` | Shared | Before secondary fire |
| `FireSecondary()` | Shared | Secondary fire logic |
| `PostSecondaryAttack()` | Shared | After secondary fire |
| `CanPrimaryAttack()` | Shared | Check if can fire primary |
| `CanSecondaryAttack()` | Shared | Check if can fire secondary |
| `TriedToPrimaryAttack()` | Shared | Failed primary fire |
| `TriedToSecondaryAttack()` | Shared | Failed secondary fire |
| `Reload()` | Shared | Reload weapon |
| `CalcViewModelView()` | Client | Position viewmodel |
| `OnRemove()` | Shared | Cleanup |
| `OnDrop()` | Server | When dropped |
| `OwnerChanged()` | Shared | Owner changed |

---

## See Also

- [Weapon Base Configuration](base-config.md) - All weapon properties
- [Primary Attack Functions](primary.md) - Primary attack system
- [Secondary Attack Functions](secondary.md) - Secondary attack system
- [Creating Weapons Guide](../../guides/creating-weapons.md) - Step-by-step weapon creation tutorial
