# Weapon Base API Reference

DrGBase provides a weapon base (`drgbase_weapon`) that simplifies weapon creation with built-in support for primary/secondary fire, ammo management, and customization hooks.

**Source Files:**
- `lua/weapons/drgbase_weapon/shared.lua` - Core weapon setup
- `lua/weapons/drgbase_weapon/primary.lua` - Primary fire system
- `lua/weapons/drgbase_weapon/secondary.lua` - Secondary fire system
- `lua/weapons/drgbase_weapon/misc.lua` - Utility functions

**Base Class:** `weapon_base`

---

## Creating a Weapon

### Basic Setup

```lua
-- lua/weapons/weapon_mygun/shared.lua
SWEP.Base = "drgbase_weapon"
SWEP.IsDrGWeapon = true

-- Weapon Info
SWEP.PrintName = "My Custom Gun"
SWEP.Category = "My Weapons"
SWEP.Author = "Your Name"
SWEP.Contact = ""
SWEP.Purpose = "Shoot things"
SWEP.Instructions = "Left click to fire, Right click for alt fire"
SWEP.Spawnable = true
SWEP.AdminOnly = false

-- Visual
SWEP.ViewModel = "models/weapons/c_smg1.mdl"
SWEP.WorldModel = "models/weapons/w_smg1.mdl"
SWEP.HoldType = "ar2"
SWEP.ViewModelFOV = 62
SWEP.UseHands = true

-- Primary Fire
SWEP.Primary.Damage = 10
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.02
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.1
SWEP.Primary.Recoil = 1

-- Primary Ammo
SWEP.Primary.Ammo = "SMG1"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 30
SWEP.Primary.DefaultClip = 90

-- Primary Effects
SWEP.Primary.Sound = "Weapon_SMG1.Single"
SWEP.Primary.EmptySound = "Weapon_Pistol.Empty"
```

---

## Core Properties

### Identification

#### SWEP.IsDrGWeapon
```lua
SWEP.IsDrGWeapon = true
```
**Type:** boolean
**Description:** Identifies this as a DrGBase weapon. Required for base functionality.

---

### Weapon Information

#### SWEP.PrintName
```lua
SWEP.PrintName = "My Weapon"
```
**Type:** string
**Description:** Display name in spawn menu.

---

#### SWEP.Category
```lua
SWEP.Category = "My Category"
```
**Type:** string
**Description:** Spawn menu category.

---

#### SWEP.Author / Contact / Purpose / Instructions
```lua
SWEP.Author = "Author Name"
SWEP.Contact = "email@example.com"
SWEP.Purpose = "What the weapon is for"
SWEP.Instructions = "How to use the weapon"
```
**Type:** string
**Description:** Weapon metadata displayed in spawn menu.

---

#### SWEP.Spawnable
```lua
SWEP.Spawnable = true
```
**Type:** boolean
**Default:** false
**Description:** Whether the weapon appears in spawn menu.

---

#### SWEP.AdminOnly
```lua
SWEP.AdminOnly = false
```
**Type:** boolean
**Default:** false
**Description:** Restrict to admins only.

---

### Visual Properties

#### SWEP.ViewModel
```lua
SWEP.ViewModel = "models/weapons/c_smg1.mdl"
```
**Type:** string
**Description:** First-person view model path.

---

#### SWEP.WorldModel
```lua
SWEP.WorldModel = "models/weapons/w_smg1.mdl"
```
**Type:** string
**Description:** Third-person world model path.

---

#### SWEP.HoldType
```lua
SWEP.HoldType = "ar2"
```
**Type:** string
**Default:** "ar2"
**Description:** Animation hold type. Common values: `"pistol"`, `"smg"`, `"ar2"`, `"shotgun"`, `"rpg"`, `"melee"`, `"grenade"`, `"fist"`, `"slam"`, `"normal"`.

---

#### SWEP.ViewModelFOV
```lua
SWEP.ViewModelFOV = 62
```
**Type:** number
**Default:** 62
**Description:** Field of view for the viewmodel.

---

#### SWEP.ViewModelFlip
```lua
SWEP.ViewModelFlip = true
```
**Type:** boolean
**Default:** true
**Description:** Whether to flip the viewmodel horizontally.

---

#### SWEP.ViewModelOffset
```lua
SWEP.ViewModelOffset = Vector(0, 0, 0)
```
**Type:** Vector
**Default:** Vector(0, 0, 0)
**Description:** Positional offset for the viewmodel (X = forward, Y = right, Z = up).

**Example:**
```lua
SWEP.ViewModelOffset = Vector(2, 1, -1) -- Forward 2, right 1, down 1
```

---

#### SWEP.ViewModelAngle
```lua
SWEP.ViewModelAngle = Angle(0, 0, 0)
```
**Type:** Angle
**Default:** Angle(0, 0, 0)
**Description:** Angular offset for the viewmodel.

---

#### SWEP.UseHands
```lua
SWEP.UseHands = true
```
**Type:** boolean
**Default:** false
**Description:** Use player hands with the viewmodel (c_ model must support hands).

---

#### SWEP.CSMuzzleFlashes
```lua
SWEP.CSMuzzleFlashes = false
```
**Type:** boolean
**Default:** false
**Description:** Use Counter-Strike style muzzle flashes.

---

## Primary Fire

### Shooting Properties

#### SWEP.Primary.Damage
```lua
SWEP.Primary.Damage = 10
```
**Type:** number
**Default:** 1
**Description:** Damage per bullet.

---

#### SWEP.Primary.Bullets
```lua
SWEP.Primary.Bullets = 1
```
**Type:** number
**Default:** 1
**Description:** Number of bullets per shot (for shotguns).

---

#### SWEP.Primary.Spread
```lua
SWEP.Primary.Spread = 0.05
```
**Type:** number
**Default:** 0
**Description:** Bullet spread/accuracy (0 = perfect accuracy, higher = less accurate).

---

#### SWEP.Primary.Automatic
```lua
SWEP.Primary.Automatic = true
```
**Type:** boolean
**Default:** true
**Description:** Whether the weapon fires automatically when holding the attack button.

---

#### SWEP.Primary.Delay
```lua
SWEP.Primary.Delay = 0.1
```
**Type:** number
**Default:** 0
**Description:** Delay between shots in seconds.

---

#### SWEP.Primary.Recoil
```lua
SWEP.Primary.Recoil = 1
```
**Type:** number
**Default:** 0
**Description:** Recoil amount (pitch adjustment in degrees).

---

### Ammo Properties

#### SWEP.Primary.Ammo
```lua
SWEP.Primary.Ammo = "AR2"
```
**Type:** string
**Default:** "AR2"
**Description:** Ammo type. Common types: `"Pistol"`, `"SMG1"`, `"AR2"`, `"357"`, `"Buckshot"`, `"XBowBolt"`, `"Grenade"`, `"RPG_Round"`.

---

#### SWEP.Primary.Cost
```lua
SWEP.Primary.Cost = 1
```
**Type:** number
**Default:** 1
**Description:** Ammo consumed per shot.

---

#### SWEP.Primary.ClipSize
```lua
SWEP.Primary.ClipSize = 30
```
**Type:** number
**Default:** 30
**Description:** Magazine size. Set to `-1` for infinite clip (uses reserve ammo directly).

---

#### SWEP.Primary.DefaultClip
```lua
SWEP.Primary.DefaultClip = 90
```
**Type:** number
**Default:** 90
**Description:** Starting ammo when picked up.

---

### Effects

#### SWEP.Primary.Sound
```lua
SWEP.Primary.Sound = "Weapon_SMG1.Single"
```
**Type:** string
**Description:** Sound played on fire.

---

#### SWEP.Primary.EmptySound
```lua
SWEP.Primary.EmptySound = "Weapon_Pistol.Empty"
```
**Type:** string
**Description:** Sound played when trying to fire with no ammo.

---

### Primary Fire Functions

#### SWEP:CanPrimaryAttack
```lua
boolean SWEP:CanPrimaryAttack()
```
Checks if primary fire is possible (has ammo).

**Realm:** SHARED
**Returns:** (boolean) True if can fire
**Example:**
```lua
function SWEP:Think()
    if self:CanPrimaryAttack() then
        -- Ready to fire
    end
end
```

---

#### SWEP:PrimaryAttack
```lua
boolean SWEP:PrimaryAttack()
```
Main primary fire function. Called when attack button is pressed.

**Realm:** SHARED
**Returns:** (boolean) True if attack succeeded
**Note:** Automatically handles ammo, delay, recoil, and hooks. Usually don't override this.

---

#### SWEP:PrePrimaryAttack
```lua
boolean SWEP:PrePrimaryAttack()
```
**Hook** called before firing. Return false to prevent firing.

**Realm:** SHARED
**Returns:** (boolean) Return false to cancel attack
**Example:**
```lua
function SWEP:PrePrimaryAttack()
    if self:WaterLevel() >= 3 then
        -- Can't fire underwater
        return false
    end
end
```

---

#### SWEP:FirePrimary
```lua
SWEP:FirePrimary()
```
**Hook** for the actual firing logic. Override to customize bullet behavior.

**Realm:** SHARED
**Default:** Calls `ShootBullet()`
**Example:**
```lua
function SWEP:FirePrimary()
    -- Custom bullet behavior
    self:ShootBullet(self.Primary.Damage * 2, 1, 0)

    -- Or fire a projectile
    if SERVER then
        local proj = ents.Create("proj_drg_grenade")
        proj:SetPos(self.Owner:GetShootPos())
        proj:SetAngles(self.Owner:EyeAngles())
        proj:SetOwner(self.Owner)
        proj:Spawn()
    end
end
```

---

#### SWEP:PostPrimaryAttack
```lua
SWEP:PostPrimaryAttack(number nextFireTime)
```
**Hook** called after firing.

**Realm:** SHARED
**Parameters:**
- `nextFireTime` (number) - CurTime() when next fire is allowed
**Example:**
```lua
function SWEP:PostPrimaryAttack(nextFireTime)
    self:SendWeaponAnim(ACT_VM_PRIMARYATTACK)

    timer.Simple(0.5, function()
        if IsValid(self) then
            self:SendWeaponAnim(ACT_VM_IDLE)
        end
    end)
end
```

---

#### SWEP:TriedToPrimaryAttack
```lua
SWEP:TriedToPrimaryAttack()
```
**Hook** called when trying to fire with no ammo.

**Realm:** SHARED
**Default:** Plays empty sound
**Example:**
```lua
function SWEP:TriedToPrimaryAttack()
    self:EmitSound("weapons/clipempty_rifle.wav")
    self:SendWeaponAnim(ACT_VM_DRYFIRE)
end
```

---

## Secondary Fire

Secondary fire is identical to primary fire but disabled by default.

### Enable Secondary Fire

```lua
SWEP.Secondary.Enabled = true
```

### Properties

All properties mirror primary fire:

```lua
SWEP.Secondary.Damage = 5
SWEP.Secondary.Bullets = 1
SWEP.Secondary.Spread = 0.01
SWEP.Secondary.Automatic = false
SWEP.Secondary.Delay = 0.5
SWEP.Secondary.Recoil = 0.5

SWEP.Secondary.Ammo = "Pistol"
SWEP.Secondary.Cost = 1
SWEP.Secondary.ClipSize = 10
SWEP.Secondary.DefaultClip = 30

SWEP.Secondary.Sound = "Weapon_Pistol.Single"
SWEP.Secondary.EmptySound = "Weapon_Pistol.Empty"
```

### Functions

- `SWEP:CanSecondaryAttack()`
- `SWEP:SecondaryAttack()`
- `SWEP:PreSecondaryAttack()` - Return false to cancel
- `SWEP:FireSecondary()` - Override for custom behavior
- `SWEP:PostSecondaryAttack(nextFireTime)`
- `SWEP:TriedToSecondaryAttack()`

**Example:**
```lua
SWEP.Secondary.Enabled = true

function SWEP:FireSecondary()
    -- Zoom toggle
    if not self.Zoomed then
        self.Zoomed = true
        self.Owner:SetFOV(30, 0.2)
    else
        self.Zoomed = false
        self.Owner:SetFOV(0, 0.2)
    end
end
```

---

## Utility Functions

### SWEP:ShootBullet
```lua
SWEP:ShootBullet(number damage, number num_bullets, number aimcone)
```
Fires hitscan bullets.

**Realm:** SHARED
**Parameters:**
- `damage` (number) - Damage per bullet
- `num_bullets` (number) - Number of bullets
- `aimcone` (number) - Spread cone
**Example:**
```lua
function SWEP:FirePrimary()
    -- Shotgun blast
    self:ShootBullet(8, 8, 0.1)
end
```

**Bullet Properties:**
- Force: `damage/10`
- Tracer: Every bullet
- AmmoType: `"Pistol"`
- Attacker: Weapon owner
- Inflictor: Weapon

---

## Lifecycle Hooks

### SWEP:Initialize
```lua
SWEP:Initialize()
```
Called when weapon is created.

**Realm:** SHARED
**Default:** Calls `_BaseInitialize()` then `CustomInitialize()`
**Note:** Don't override `Initialize()`. Use `CustomInitialize()` instead.

---

### SWEP:CustomInitialize
```lua
SWEP:CustomInitialize()
```
**Hook** for custom initialization logic.

**Realm:** SHARED
**Example:**
```lua
function SWEP:CustomInitialize()
    self.Zoomed = false
    self.BurstCount = 0
end
```

---

### SWEP:Think
```lua
SWEP:Think()
```
Called every frame.

**Realm:** SHARED
**Default:** Calls `_BaseThink()` then `CustomThink()`
**Note:** Don't override `Think()`. Use `CustomThink()` instead.

---

### SWEP:CustomThink
```lua
SWEP:CustomThink()
```
**Hook** for custom per-frame logic.

**Realm:** SHARED
**Example:**
```lua
function SWEP:CustomThink()
    if self.Overheated then
        self.HeatLevel = math.Approach(self.HeatLevel or 0, 0, FrameTime() * 10)
        if self.HeatLevel <= 0 then
            self.Overheated = false
        end
    end
end
```

---

### SWEP:Reload
```lua
SWEP:Reload()
```
Called when reload button is pressed.

**Realm:** SHARED
**Default:** Calls `DefaultReload(ACT_VM_RELOAD)`
**Example:**
```lua
function SWEP:Reload()
    if self:Clip1() < self.Primary.ClipSize and
       self.Owner:GetAmmoCount(self.Primary.Ammo) > 0 then
        self:DefaultReload(ACT_VM_RELOAD)
        self.Owner:SetFOV(0, 0.2) -- Un-zoom on reload
        self.Zoomed = false
    end
end
```

---

## Complete Examples

### Basic Assault Rifle
```lua
SWEP.Base = "drgbase_weapon"
SWEP.IsDrGWeapon = true

SWEP.PrintName = "Assault Rifle"
SWEP.Category = "My Weapons"
SWEP.Spawnable = true

SWEP.ViewModel = "models/weapons/c_smg1.mdl"
SWEP.WorldModel = "models/weapons/w_smg1.mdl"
SWEP.HoldType = "ar2"
SWEP.UseHands = true

-- Full-auto rifle
SWEP.Primary.Damage = 15
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.02
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.08
SWEP.Primary.Recoil = 0.5

SWEP.Primary.Ammo = "AR2"
SWEP.Primary.ClipSize = 30
SWEP.Primary.DefaultClip = 90

SWEP.Primary.Sound = "Weapon_AR2.Single"
SWEP.Primary.EmptySound = "Weapon_Pistol.Empty"
```

---

### Shotgun
```lua
SWEP.Base = "drgbase_weapon"
SWEP.IsDrGWeapon = true

SWEP.PrintName = "Combat Shotgun"
SWEP.Category = "My Weapons"
SWEP.Spawnable = true

SWEP.ViewModel = "models/weapons/c_shotgun.mdl"
SWEP.WorldModel = "models/weapons/w_shotgun.mdl"
SWEP.HoldType = "shotgun"
SWEP.UseHands = true

-- Pump-action shotgun
SWEP.Primary.Damage = 8
SWEP.Primary.Bullets = 8 -- 8 pellets
SWEP.Primary.Spread = 0.1
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 0.8
SWEP.Primary.Recoil = 5

SWEP.Primary.Ammo = "Buckshot"
SWEP.Primary.ClipSize = 6
SWEP.Primary.DefaultClip = 24

SWEP.Primary.Sound = "Weapon_Shotgun.Single"
SWEP.Primary.EmptySound = "Weapon_Shotgun.Empty"
```

---

### Sniper Rifle with Zoom
```lua
SWEP.Base = "drgbase_weapon"
SWEP.IsDrGWeapon = true

SWEP.PrintName = "Sniper Rifle"
SWEP.Category = "My Weapons"
SWEP.Spawnable = true

SWEP.ViewModel = "models/weapons/c_357.mdl"
SWEP.WorldModel = "models/weapons/w_357.mdl"
SWEP.HoldType = "ar2"
SWEP.UseHands = true

-- High-damage, slow-firing
SWEP.Primary.Damage = 100
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 1.5
SWEP.Primary.Recoil = 10

SWEP.Primary.Ammo = "357"
SWEP.Primary.ClipSize = 5
SWEP.Primary.DefaultClip = 20

SWEP.Primary.Sound = "Weapon_357.Single"
SWEP.Primary.EmptySound = "Weapon_Pistol.Empty"

-- Zoom on secondary fire
SWEP.Secondary.Enabled = true
SWEP.Secondary.Automatic = false
SWEP.Secondary.Delay = 0.3

function SWEP:CustomInitialize()
    self.Zoomed = false
end

function SWEP:FireSecondary()
    if not self.Zoomed then
        self.Owner:SetFOV(15, 0.2)
        self.Zoomed = true
    else
        self.Owner:SetFOV(0, 0.2)
        self.Zoomed = false
    end
end

function SWEP:PrePrimaryAttack()
    if self.Zoomed then
        -- Perfect accuracy when zoomed
        self.Primary.Spread = 0
    else
        -- Inaccurate when not zoomed
        self.Primary.Spread = 0.05
    end
end

function SWEP:Reload()
    self.Owner:SetFOV(0, 0.2)
    self.Zoomed = false
    return self.BaseClass.Reload(self)
end

function SWEP:Holster()
    self.Owner:SetFOV(0, 0)
    self.Zoomed = false
    return true
end
```

---

### Burst-Fire Weapon
```lua
SWEP.Base = "drgbase_weapon"
SWEP.IsDrGWeapon = true

SWEP.PrintName = "Burst Rifle"
SWEP.Spawnable = true

-- ... (visual setup)

SWEP.Primary.Damage = 12
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 0.4

SWEP.BurstCount = 3
SWEP.BurstDelay = 0.08

function SWEP:CustomInitialize()
    self.BurstShots = 0
end

function SWEP:FirePrimary()
    self:ShootBullet(self.Primary.Damage, 1, 0.01)
    self.BurstShots = (self.BurstShots or 0) + 1

    if self.BurstShots < self.BurstCount then
        timer.Simple(self.BurstDelay, function()
            if IsValid(self) and IsValid(self.Owner) then
                self:PrimaryAttack()
            end
        end)
    else
        self.BurstShots = 0
    end
end
```

---

### Grenade Launcher
```lua
SWEP.Base = "drgbase_weapon"
SWEP.IsDrGWeapon = true

SWEP.PrintName = "Grenade Launcher"
SWEP.Spawnable = true

-- ... (visual setup)

SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 1.0
SWEP.Primary.Ammo = "Grenade"
SWEP.Primary.ClipSize = 6
SWEP.Primary.DefaultClip = 18

function SWEP:FirePrimary()
    if SERVER then
        local grenade = ents.Create("proj_drg_grenade")
        grenade:SetPos(self.Owner:GetShootPos() + self.Owner:GetAimVector() * 50)
        grenade:SetAngles(self.Owner:EyeAngles())
        grenade:SetOwner(self.Owner)
        grenade:Spawn()

        local phys = grenade:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(self.Owner:GetAimVector() * 1500)
        end
    end

    self:SendWeaponAnim(ACT_VM_PRIMARYATTACK)
end
```

---

## See Also

- [Projectile Base](projectile-base.md) - Creating custom projectiles
- [NPC Weapons](../guides/npc-weapons.md) - Giving weapons to NPCs
- [Combat System](../guides/combat.md) - Combat mechanics
