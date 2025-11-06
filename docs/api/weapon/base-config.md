# Weapon Base Configuration

All configurable properties for `drgbase_weapon`.

**File:** `weapons/drgbase_weapon/shared.lua`

## Basic Properties

### SWEP.IsDrGWeapon
**Type:** `boolean`
**Default:** `true`

Marks this weapon as a DrGBase weapon. Used internally to identify DrGBase weapons from other weapon bases.

```lua
SWEP.IsDrGWeapon = true -- Always true for DrGBase weapons
```

### SWEP.PrintName
**Type:** `string`
**Default:** `""`

The display name shown in spawn menus and when picked up.

```lua
SWEP.PrintName = "AR2"
```

### SWEP.Category
**Type:** `string`
**Default:** `""`

The category this weapon appears under in spawn menus.

```lua
SWEP.Category = "DrG - Half Life 2"
```

### SWEP.Author
**Type:** `string`
**Default:** `""`

The name of the weapon creator.

```lua
SWEP.Author = "Dragoteryx"
```

### SWEP.Contact
**Type:** `string`
**Default:** `""`

Contact information for the author.

```lua
SWEP.Contact = ""
```

### SWEP.Purpose
**Type:** `string`
**Default:** `""`

Brief description of the weapon's purpose.

```lua
SWEP.Purpose = ""
```

### SWEP.Instructions
**Type:** `string`
**Default:** `""`

Instructions on how to use the weapon.

```lua
SWEP.Instructions = ""
```

### SWEP.Spawnable
**Type:** `boolean`
**Default:** `false`

Whether the weapon can be spawned from the spawn menu.

```lua
SWEP.Spawnable = true
```

### SWEP.AdminOnly
**Type:** `boolean`
**Default:** `false`

Whether only admins can spawn this weapon.

```lua
SWEP.AdminOnly = false
```

### SWEP.Slot
**Type:** `number`
**Default:** `0`

Weapon slot (0-5) for inventory organization.

```lua
SWEP.Slot = 2 -- Slot 2 is typically for rifles
```

### SWEP.SlotPos
**Type:** `number`
**Default:** `0`

Position within the slot.

```lua
SWEP.SlotPos = 0
```

---

## Appearance

### SWEP.ViewModel
**Type:** `string`
**Default:** `""`

Path to the viewmodel (first-person) model.

```lua
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
```

### SWEP.WorldModel
**Type:** `string`
**Default:** `""`

Path to the worldmodel (third-person) model.

```lua
SWEP.WorldModel = "models/weapons/w_irifle.mdl"
```

### SWEP.HoldType
**Type:** `string`
**Default:** `"ar2"`

Animation hold type for the weapon. Common values:
- `"ar2"` - Assault rifle
- `"pistol"` - Pistol
- `"shotgun"` - Shotgun
- `"smg"` - SMG
- `"crossbow"` - Crossbow
- `"melee"` - Melee weapon
- `"slam"` - C4/deployable
- `"rpg"` - RPG/launcher

```lua
SWEP.HoldType = "ar2"
```

### SWEP.ViewModelFOV
**Type:** `number`
**Default:** `62`

Field of view for the viewmodel. Lower values zoom in, higher values zoom out.

```lua
SWEP.ViewModelFOV = 54
```

### SWEP.ViewModelFlip
**Type:** `boolean`
**Default:** `true`

Whether to flip the viewmodel horizontally.

```lua
SWEP.ViewModelFlip = false
```

### SWEP.ViewModelOffset
**Type:** `Vector`
**Default:** `Vector(0, 0, 0)`

Offset for viewmodel positioning (forward/right, right/left, up/down).

```lua
SWEP.ViewModelOffset = Vector(5, 0, -2) -- Forward 5, down 2
```

### SWEP.ViewModelAngle
**Type:** `Angle`
**Default:** `Angle(0, 0, 0)`

Angle offset for viewmodel rotation.

```lua
SWEP.ViewModelAngle = Angle(0, 5, 0) -- Rotate 5 degrees on yaw
```

### SWEP.UseHands
**Type:** `boolean`
**Default:** `false`

Whether to use player hands on the viewmodel.

```lua
SWEP.UseHands = true
```

### SWEP.CSMuzzleFlashes
**Type:** `boolean`
**Default:** `false`

Whether to use Counter-Strike style muzzle flashes.

```lua
SWEP.CSMuzzleFlashes = false
```

### SWEP.CSMuzzleX
**Type:** `boolean`
**Default:** `false`

Whether to use CS:S extended muzzle flash effects.

```lua
SWEP.CSMuzzleX = false
```

---

## Primary Attack

See [Primary Attack Functions](primary.md) for detailed primary attack behavior.

### SWEP.Primary.Damage
**Type:** `number`
**Default:** `1`

Damage dealt per bullet for primary fire.

```lua
SWEP.Primary.Damage = 10
```

### SWEP.Primary.Bullets
**Type:** `number`
**Default:** `1`

Number of bullets fired per shot. Useful for shotguns.

```lua
SWEP.Primary.Bullets = 8 -- Fires 8 pellets like a shotgun
```

### SWEP.Primary.Spread
**Type:** `number`
**Default:** `0`

Bullet spread/accuracy. Lower = more accurate. `0` = perfect accuracy.

```lua
SWEP.Primary.Spread = 0.02 -- Slight spread
```

### SWEP.Primary.Automatic
**Type:** `boolean`
**Default:** `true`

Whether holding the fire button continues firing.

```lua
SWEP.Primary.Automatic = true -- Full auto
SWEP.Primary.Automatic = false -- Semi-auto
```

### SWEP.Primary.Delay
**Type:** `number`
**Default:** `0`

Delay in seconds between shots.

```lua
SWEP.Primary.Delay = 0.1 -- 10 shots per second
```

### SWEP.Primary.Recoil
**Type:** `number`
**Default:** `0`

Camera recoil when firing. Higher values = more recoil.

```lua
SWEP.Primary.Recoil = 0.5
```

### SWEP.Primary.ClipSize
**Type:** `number`
**Default:** `30`

Magazine/clip size. Set to `-1` for infinite ammo in clip.

```lua
SWEP.Primary.ClipSize = 30 -- 30 round magazine
```

### SWEP.Primary.DefaultClip
**Type:** `number`
**Default:** `90`

Starting reserve ammo when weapon is spawned/given.

```lua
SWEP.Primary.DefaultClip = 90 -- Start with 90 reserve rounds
```

### SWEP.Primary.Ammo
**Type:** `string`
**Default:** `"AR2"`

Ammo type. Common types:
- `"AR2"` - Rifle ammo
- `"Pistol"` - Pistol ammo
- `"SMG1"` - SMG ammo
- `"Buckshot"` - Shotgun shells
- `"357"` - Magnum rounds
- `"XBowBolt"` - Crossbow bolts
- `"RPG_Round"` - RPG rockets

```lua
SWEP.Primary.Ammo = "AR2"
```

### SWEP.Primary.Cost
**Type:** `number`
**Default:** `1`

Ammo consumed per shot.

```lua
SWEP.Primary.Cost = 1 -- Uses 1 round per shot
SWEP.Primary.Cost = 2 -- Uses 2 rounds per shot
```

### SWEP.Primary.Sound
**Type:** `string`
**Default:** `""`

Sound played when firing.

```lua
SWEP.Primary.Sound = "Weapon_AR2.Single"
```

### SWEP.Primary.EmptySound
**Type:** `string`
**Default:** `""`

Sound played when trying to fire with no ammo.

```lua
SWEP.Primary.EmptySound = "Weapon_AR2.Empty"
```

---

## Secondary Attack

See [Secondary Attack Functions](secondary.md) for detailed secondary attack behavior.

### SWEP.Secondary.Enabled
**Type:** `boolean`
**Default:** `false`

Whether the secondary attack is enabled. Must be `true` for secondary fire to work.

```lua
SWEP.Secondary.Enabled = true
```

### SWEP.Secondary.Damage
**Type:** `number`
**Default:** `1`

Damage dealt per bullet for secondary fire.

```lua
SWEP.Secondary.Damage = 15
```

### SWEP.Secondary.Bullets
**Type:** `number`
**Default:** `1`

Number of bullets fired per secondary shot.

```lua
SWEP.Secondary.Bullets = 1
```

### SWEP.Secondary.Spread
**Type:** `number`
**Default:** `0`

Bullet spread for secondary fire. Lower = more accurate.

```lua
SWEP.Secondary.Spread = 0.01
```

### SWEP.Secondary.Automatic
**Type:** `boolean`
**Default:** `false`

Whether holding secondary fire continues firing.

```lua
SWEP.Secondary.Automatic = false -- Semi-auto secondary
```

### SWEP.Secondary.Delay
**Type:** `number`
**Default:** `0`

Delay in seconds between secondary shots.

```lua
SWEP.Secondary.Delay = 0.5 -- 2 shots per second
```

### SWEP.Secondary.Recoil
**Type:** `number`
**Default:** `0`

Camera recoil for secondary fire.

```lua
SWEP.Secondary.Recoil = 1
```

### SWEP.Secondary.ClipSize
**Type:** `number`
**Default:** `0`

Secondary magazine size. Usually `0` if using separate ammo pool.

```lua
SWEP.Secondary.ClipSize = 0
```

### SWEP.Secondary.DefaultClip
**Type:** `number`
**Default:** `0`

Starting reserve ammo for secondary fire.

```lua
SWEP.Secondary.DefaultClip = 0
```

### SWEP.Secondary.Ammo
**Type:** `string`
**Default:** `""`

Ammo type for secondary fire. Empty string means no ammo requirement.

```lua
SWEP.Secondary.Ammo = ""
```

### SWEP.Secondary.Cost
**Type:** `number`
**Default:** `1`

Ammo consumed per secondary shot.

```lua
SWEP.Secondary.Cost = 1
```

### SWEP.Secondary.Sound
**Type:** `string`
**Default:** `""`

Sound played when firing secondary.

```lua
SWEP.Secondary.Sound = "Weapon_SMG1.Single"
```

### SWEP.Secondary.EmptySound
**Type:** `string`
**Default:** `""`

Sound played when trying to fire secondary with no ammo.

```lua
SWEP.Secondary.EmptySound = "Weapon_SMG1.Empty"
```

---

## Complete Property Table

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| **Basic** |
| IsDrGWeapon | boolean | `true` | Identifies as DrGBase weapon |
| PrintName | string | `""` | Display name |
| Category | string | `""` | Spawn menu category |
| Author | string | `""` | Creator name |
| Contact | string | `""` | Contact info |
| Purpose | string | `""` | Weapon purpose |
| Instructions | string | `""` | Usage instructions |
| Spawnable | boolean | `false` | Can be spawned |
| AdminOnly | boolean | `false` | Admin-only spawning |
| Slot | number | `0` | Weapon slot (0-5) |
| SlotPos | number | `0` | Position in slot |
| **Appearance** |
| ViewModel | string | `""` | First-person model |
| WorldModel | string | `""` | Third-person model |
| HoldType | string | `"ar2"` | Animation hold type |
| ViewModelFOV | number | `62` | Viewmodel FOV |
| ViewModelFlip | boolean | `true` | Flip viewmodel |
| ViewModelOffset | Vector | `Vector(0,0,0)` | Viewmodel position offset |
| ViewModelAngle | Angle | `Angle(0,0,0)` | Viewmodel angle offset |
| UseHands | boolean | `false` | Use player hands |
| CSMuzzleFlashes | boolean | `false` | CS-style muzzle flash |
| CSMuzzleX | boolean | `false` | CS:S extended flash |
| **Primary Attack** |
| Primary.Damage | number | `1` | Damage per bullet |
| Primary.Bullets | number | `1` | Bullets per shot |
| Primary.Spread | number | `0` | Bullet spread |
| Primary.Automatic | boolean | `true` | Full auto |
| Primary.Delay | number | `0` | Fire delay |
| Primary.Recoil | number | `0` | Camera recoil |
| Primary.ClipSize | number | `30` | Magazine size |
| Primary.DefaultClip | number | `90` | Starting reserve |
| Primary.Ammo | string | `"AR2"` | Ammo type |
| Primary.Cost | number | `1` | Ammo per shot |
| Primary.Sound | string | `""` | Fire sound |
| Primary.EmptySound | string | `""` | Empty click sound |
| **Secondary Attack** |
| Secondary.Enabled | boolean | `false` | Enable secondary |
| Secondary.Damage | number | `1` | Damage per bullet |
| Secondary.Bullets | number | `1` | Bullets per shot |
| Secondary.Spread | number | `0` | Bullet spread |
| Secondary.Automatic | boolean | `false` | Full auto |
| Secondary.Delay | number | `0` | Fire delay |
| Secondary.Recoil | number | `0` | Camera recoil |
| Secondary.ClipSize | number | `0` | Magazine size |
| Secondary.DefaultClip | number | `0` | Starting reserve |
| Secondary.Ammo | string | `""` | Ammo type |
| Secondary.Cost | number | `1` | Ammo per shot |
| Secondary.Sound | string | `""` | Fire sound |
| Secondary.EmptySound | string | `""` | Empty click sound |

---

## Example Configuration

Complete example weapon configuration:

```lua
if not DrGBase then return end
SWEP.Base = "drgbase_weapon"

-- Basic Information
SWEP.PrintName = "AR2"
SWEP.Category = "DrG - Half Life 2"
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

-- Primary Fire
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

-- Finalize
AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

---

## See Also

- [Primary Attack Functions](primary.md) - Primary attack behavior and hooks
- [Secondary Attack Functions](secondary.md) - Secondary attack behavior and hooks
- [Weapon Functions & Hooks](functions.md) - General weapon functions
- [Creating Weapons Guide](../../guides/creating-weapons.md) - Step-by-step weapon creation tutorial
