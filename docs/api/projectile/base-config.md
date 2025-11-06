# Projectile Base Configuration

All configurable properties for `proj_drg_default`.

**File:** `entities/proj_drg_default/shared.lua`

---

## Basic Properties

### ENT.IsDrGProjectile
**Type:** `boolean`
**Default:** `true`

Marks this entity as a DrGBase projectile. Used internally to identify DrGBase projectiles.

```lua
ENT.IsDrGProjectile = true -- Always true for DrGBase projectiles
```

### ENT.PrintName
**Type:** `string`
**Default:** `"Projectile"`

The display name shown in spawn menus.

```lua
ENT.PrintName = "Plasma Ball"
```

### ENT.Category
**Type:** `string`
**Default:** `"DrGBase"`

The category this projectile appears under in spawn menus.

```lua
ENT.Category = "DrGBase"
```

### ENT.Spawnable
**Type:** `boolean`
**Default:** `false`

Whether the projectile can be spawned from the spawn menu.

```lua
ENT.Spawnable = true
```

### ENT.AdminOnly
**Type:** `boolean`
**Default:** `false`

Whether only admins can spawn this projectile.

```lua
ENT.AdminOnly = false
```

---

## Appearance

### ENT.Models
**Type:** `table`
**Default:** `{}`

Array of model paths. A random model is selected when the projectile spawns. If empty, uses a default invisible model.

```lua
ENT.Models = {"models/weapons/w_eq_fraggrenade.mdl"}

-- Multiple models (random selection)
ENT.Models = {
    "models/weapons/w_eq_flashbang.mdl",
    "models/weapons/w_eq_fraggrenade.mdl",
    "models/weapons/w_eq_smokegrenade.mdl"
}

-- No model (invisible)
ENT.Models = {}
```

### ENT.ModelScale
**Type:** `number`
**Default:** `1`

Scale multiplier for the model. `1` = normal size, `2` = double size, `0.5` = half size.

```lua
ENT.ModelScale = 1.5 -- 150% size
```

---

## Physics

### ENT.Gravity
**Type:** `boolean`
**Default:** `true`

Whether the projectile is affected by gravity.

```lua
ENT.Gravity = true -- Falls to ground
ENT.Gravity = false -- Flies straight
```

### ENT.Physgun
**Type:** `boolean`
**Default:** `false`

Whether the projectile can be picked up with the physics gun.

```lua
ENT.Physgun = true -- Can be physgunned
ENT.Physgun = false -- Cannot be physgunned
```

### ENT.Gravgun
**Type:** `boolean`
**Default:** `false`

Whether the projectile can be picked up with the gravity gun.

```lua
ENT.Gravgun = true -- Can be gravgunned
ENT.Gravgun = false -- Cannot be gravgunned
```

---

## Contact Behavior

### ENT.OnContactDelay
**Type:** `number`
**Default:** `0.1`

Minimum time (in seconds) between contact events. Prevents spam when bouncing or sliding.

```lua
ENT.OnContactDelay = 0.1 -- Can trigger contact every 0.1 seconds
ENT.OnContactDelay = 0 -- No delay (triggers every touch)
```

### ENT.OnContactDelete
**Type:** `number`
**Default:** `-1`

Controls when the projectile is removed after contact:
- `-1` = Never auto-remove
- `0` = Remove immediately on contact
- `> 0` = Remove after this many seconds

```lua
ENT.OnContactDelete = -1 -- Never auto-remove
ENT.OnContactDelete = 0 -- Remove immediately on contact
ENT.OnContactDelete = 2 -- Remove 2 seconds after contact
```

### ENT.OnContactDecals
**Type:** `table`
**Default:** `{}`

Array of decal names to place on surfaces when hitting the world. A random decal is selected.

```lua
ENT.OnContactDecals = {"Scorch"}

-- Multiple decals (random selection)
ENT.OnContactDecals = {"Scorch", "FadingScorch", "BeerSplash"}
```

**Common Decal Names:**
- `"Scorch"` - Burn mark
- `"FadingScorch"` - Fading burn mark
- `"BeerSplash"` - Liquid splash
- `"Blood"` - Blood splatter
- `"BulletProof"` - Bullet hole
- `"Concrete.BulletProof"` - Concrete bullet hole
- `"Metal.BulletProof"` - Metal bullet hole

---

## Sounds

### ENT.LoopSounds
**Type:** `table`
**Default:** `{}`

Array of sounds to loop while the projectile exists. A random sound is selected and plays continuously.

```lua
ENT.LoopSounds = {"ambient/fire/fire_small_loop1.wav"}

-- Multiple sounds (random selection)
ENT.LoopSounds = {
    "ambient/fire/fire_small_loop1.wav",
    "ambient/fire/fire_med_loop1.wav"
}
```

### ENT.OnContactSounds
**Type:** `table`
**Default:** `{}`

Array of sounds to play when the projectile contacts something. A random sound is selected.

```lua
ENT.OnContactSounds = {"weapons/stunstick/stunstick_fleshhit1.wav"}

-- Multiple sounds (random selection)
ENT.OnContactSounds = {
    "weapons/hegrenade/he_bounce-1.wav",
    "weapons/grenade/grenade_hit1.wav"
}
```

### ENT.OnRemoveSounds
**Type:** `table`
**Default:** `{}`

Array of sounds to play when the projectile is removed. A random sound is selected.

```lua
ENT.OnRemoveSounds = {"weapons/explode3.wav"}

-- Multiple sounds (random selection)
ENT.OnRemoveSounds = {
    "weapons/explode3.wav",
    "weapons/explode4.wav",
    "weapons/explode5.wav"
}
```

---

## Effects

### ENT.AttachEffects
**Type:** `table`
**Default:** `{}`

Array of particle effects to attach to the projectile. A random effect is selected and follows the projectile.

```lua
ENT.AttachEffects = {"drg_plasma_ball"}

-- Multiple effects (random selection)
ENT.AttachEffects = {
    "fire_small_01",
    "fire_medium_01"
}
```

**Common Particle Effects:**
- `"fire_small_01"` - Small fire
- `"fire_medium_01"` - Medium fire
- `"fire_large_01"` - Large fire
- `"smoke_burning_engine_01"` - Black smoke
- `"drg_plasma_ball"` - Plasma effect (DrGBase custom)

### ENT.OnContactEffects
**Type:** `table`
**Default:** `{}`

Array of particle effects to play at the contact point. A random effect is selected.

```lua
ENT.OnContactEffects = {"explosion"}

-- Multiple effects (random selection)
ENT.OnContactEffects = {
    "explosion",
    "ManhackSparks"
}
```

### ENT.OnRemoveEffects
**Type:** `table`
**Default:** `{}`

Array of particle effects to play when the projectile is removed. A random effect is selected.

```lua
ENT.OnRemoveEffects = {"explosion"}

-- Multiple effects (random selection)
ENT.OnRemoveEffects = {
    "explosion",
    "Explosion_Turret_Break"
}
```

---

## Complete Property Table

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| **Basic** |
| IsDrGProjectile | boolean | `true` | Identifies as DrGBase projectile |
| PrintName | string | `"Projectile"` | Display name |
| Category | string | `"DrGBase"` | Spawn menu category |
| Spawnable | boolean | `false` | Can be spawned |
| AdminOnly | boolean | `false` | Admin-only spawning |
| **Appearance** |
| Models | table | `{}` | Model paths (random) |
| ModelScale | number | `1` | Model scale multiplier |
| **Physics** |
| Gravity | boolean | `true` | Affected by gravity |
| Physgun | boolean | `false` | Can be physgunned |
| Gravgun | boolean | `false` | Can be gravgunned |
| **Contact** |
| OnContactDelay | number | `0.1` | Min time between contacts |
| OnContactDelete | number | `-1` | Auto-remove timing |
| OnContactDecals | table | `{}` | Decals on world contact |
| **Sounds** |
| LoopSounds | table | `{}` | Looping sounds |
| OnContactSounds | table | `{}` | Contact sounds |
| OnRemoveSounds | table | `{}` | Removal sounds |
| **Effects** |
| AttachEffects | table | `{}` | Attached particle effects |
| OnContactEffects | table | `{}` | Contact particle effects |
| OnRemoveEffects | table | `{}` | Removal particle effects |

---

## Example Configurations

### Example 1: Grenade

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "HE Grenade"
ENT.Category = "DrGBase"
ENT.Spawnable = true

-- Appearance
ENT.Models = {"models/weapons/w_eq_fraggrenade.mdl"}

-- Physics
ENT.Physgun = true
ENT.Gravgun = true

-- Bounce sounds
ENT.OnContactSounds = {"weapons/hegrenade/he_bounce-1.wav"}

-- Does not auto-remove (explodes manually)
ENT.OnContactDelete = -1

AddCSLuaFile()
DrGBase.AddEntity(ENT)
```

### Example 2: Plasma Ball

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Plasma Ball"
ENT.Category = "DrGBase"
ENT.Spawnable = true

-- No visible model
ENT.Models = {}

-- Physics
ENT.Gravity = false -- Flies straight
ENT.Gravgun = true

-- Effects
ENT.AttachEffects = {"drg_plasma_ball"}
ENT.OnContactDecals = {"Scorch"}
ENT.OnContactSounds = {"weapons/stunstick/stunstick_fleshhit1.wav"}

-- Remove immediately on contact
ENT.OnContactDelete = 0

AddCSLuaFile()
DrGBase.AddEntity(ENT)
```

### Example 3: Molotov Cocktail

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Molotov Cocktail"
ENT.Category = "DrGBase"
ENT.Spawnable = true

-- Appearance
ENT.Models = {"models/props_junk/garbage_glassbottle003a.mdl"}

-- Physics
ENT.Physgun = true
ENT.Gravgun = true

-- Fire effects
ENT.AttachEffects = {"fire_small_01"}
ENT.OnRemoveEffects = {"explosion"}
ENT.OnContactDecals = {"Scorch", "FadingScorch"}

-- Sounds
ENT.LoopSounds = {"ambient/fire/fire_small_loop1.wav"}
ENT.OnContactSounds = {"physics/glass/glass_bottle_break1.wav"}
ENT.OnRemoveSounds = {"ambient/fire/ignite.wav"}

-- Remove 5 seconds after contact
ENT.OnContactDelete = 5

AddCSLuaFile()
DrGBase.AddEntity(ENT)
```

### Example 4: Sticky Bomb

```lua
if not DrGBase then return end
ENT.Base = "proj_drg_default"

ENT.PrintName = "Sticky Bomb"
ENT.Category = "DrGBase"
ENT.Spawnable = true

-- Appearance
ENT.Models = {"models/weapons/w_slam.mdl"}
ENT.ModelScale = 0.5

-- Physics
ENT.Physgun = true
ENT.Gravgun = true

-- Effects
ENT.LoopSounds = {"weapons/c4/c4_beep1.wav"}

-- Never auto-remove (explodes manually)
ENT.OnContactDelete = -1

-- Only trigger contact once (sticks on first hit)
ENT.OnContactDelay = 999999

AddCSLuaFile()
DrGBase.AddEntity(ENT)
```

---

## See Also

- [Projectile Functions & Hooks](functions.md) - Projectile behavior and logic
- [Creating Projectiles Guide](../../guides/creating-projectiles.md) - Step-by-step projectile creation tutorial
