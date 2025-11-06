# Nextbot Base Configuration

All configurable properties for `drgbase_nextbot`.

**Primary Source:** `lua/entities/drgbase_nextbot/shared.lua` (621 lines)

---

## Basic Properties

### ENT.Base
```lua
ENT.Base = "base_nextbot"
```
**Type:** string
**Default:** `"base_nextbot"`
**Description:** Base class to inherit from
**Options:** `"drgbase_nextbot"`, `"drgbase_nextbot_human"`, `"drgbase_nextbot_sprite"`
**Source:** shared.lua

### ENT.Type
```lua
ENT.Type = "nextbot"
```
**Type:** string
**Default:** `"nextbot"`
**Description:** Entity type identifier
**Source:** shared.lua

### ENT.IsDrGEntity
```lua
ENT.IsDrGEntity = true
```
**Type:** boolean
**Default:** `true`
**Description:** Marks entity as a DrGBase entity
**Source:** shared.lua

### ENT.IsDrGNextbot
```lua
ENT.IsDrGNextbot = true
```
**Type:** boolean
**Default:** `true`
**Description:** Marks entity as a DrGBase Nextbot
**Source:** shared.lua

### ENT.PrintName
```lua
ENT.PrintName = "Template"
```
**Type:** string
**Default:** `"Template"`
**Description:** Display name in spawn menu
**Source:** shared.lua

### ENT.Category
```lua
ENT.Category = "Other"
```
**Type:** string
**Default:** `"Other"`
**Description:** Spawn menu category
**Source:** shared.lua

### ENT.Spawnable
```lua
ENT.Spawnable = true
```
**Type:** boolean
**Description:** Can be spawned from menu
**Source:** (standard SENT property)

### ENT.AdminOnly
```lua
ENT.AdminOnly = false
```
**Type:** boolean
**Description:** Admin-only spawn
**Source:** (standard SENT property)

---

## Model & Appearance

### ENT.Models
```lua
ENT.Models = {"models/player/kleiner.mdl"}
```
**Type:** table (array of strings)
**Default:** `{"models/player/kleiner.mdl"}`
**Description:** List of models (random selection on spawn)
**Source:** shared.lua

### ENT.Skins
```lua
ENT.Skins = {0}
```
**Type:** table (array of numbers) or number
**Default:** `{0}`
**Description:** Skin indices to randomly choose from on spawn
**Source:** shared.lua

### ENT.ModelScale
```lua
ENT.ModelScale = 1
```
**Type:** number or table
**Default:** `1`
**Description:** Model scale multiplier. Can be a number or a table `{min, max}` for random scaling
**Source:** shared.lua

### ENT.CollisionBounds
```lua
ENT.CollisionBounds = Vector(10, 10, 72)
```
**Type:** Vector
**Default:** `Vector(10, 10, 72)`
**Description:** Collision box size (half-extents from center)
**Source:** shared.lua

### ENT.BloodColor
```lua
ENT.BloodColor = BLOOD_COLOR_RED
```
**Type:** number
**Default:** `BLOOD_COLOR_RED`
**Description:** Blood particle color constant
**Options:** `BLOOD_COLOR_RED`, `BLOOD_COLOR_YELLOW`, `BLOOD_COLOR_GREEN`, `BLOOD_COLOR_MECH`
**Source:** shared.lua

---

## Health & Status

### ENT.SpawnHealth
```lua
ENT.SpawnHealth = 100
```
**Type:** number
**Default:** `100`
**Description:** Starting health
**Source:** shared.lua, status.lua

### ENT.HealthRegen
```lua
ENT.HealthRegen = 0
```
**Type:** number
**Default:** `0`
**Description:** Health regenerated per second
**Source:** shared.lua, status.lua

### ENT.MinPhysDamage
```lua
ENT.MinPhysDamage = 10
```
**Type:** number
**Default:** `10`
**Description:** Minimum physics damage to register
**Source:** shared.lua

### ENT.MinFallDamage
```lua
ENT.MinFallDamage = 10
```
**Type:** number
**Default:** `10`
**Description:** Minimum fall damage to register
**Source:** shared.lua

### ENT.RagdollOnDeath
```lua
ENT.RagdollOnDeath = true
```
**Type:** boolean
**Default:** `true`
**Description:** Create ragdoll corpse on death
**Source:** shared.lua

---

## Movement & Locomotion

### ENT.WalkSpeed
```lua
ENT.WalkSpeed = 100
```
**Type:** number
**Default:** `100`
**Description:** Walking speed (units/second)
**Source:** shared.lua, movements.lua

### ENT.RunSpeed
```lua
ENT.RunSpeed = 200
```
**Type:** number
**Default:** `200`
**Description:** Running speed (units/second)
**Source:** shared.lua, movements.lua

### ENT.UseWalkframes
```lua
ENT.UseWalkframes = false
```
**Type:** boolean
**Default:** `false`
**Description:** Use animation walkframes for movement speed calculation
**Source:** shared.lua, movements.lua

### ENT.Acceleration
```lua
ENT.Acceleration = 1000
```
**Type:** number
**Default:** `1000`
**Description:** Movement acceleration rate
**Source:** shared.lua, locomotion.lua

### ENT.Deceleration
```lua
ENT.Deceleration = 1000
```
**Type:** number
**Default:** `1000`
**Description:** Movement deceleration rate
**Source:** shared.lua, locomotion.lua

### ENT.JumpHeight
```lua
ENT.JumpHeight = 50
```
**Type:** number
**Default:** `50`
**Description:** Jump height (units)
**Source:** shared.lua, locomotion.lua

### ENT.StepHeight
```lua
ENT.StepHeight = 20
```
**Type:** number
**Default:** `20`
**Description:** Maximum step height (units)
**Source:** shared.lua, locomotion.lua

### ENT.MaxYawRate
```lua
ENT.MaxYawRate = 250
```
**Type:** number
**Default:** `250`
**Description:** Maximum turning rate (degrees/second)
**Source:** shared.lua, locomotion.lua

### ENT.DeathDropHeight
```lua
ENT.DeathDropHeight = 200
```
**Type:** number
**Default:** `200`
**Description:** Maximum fall height before death (units)
**Source:** shared.lua, locomotion.lua

---

## Climbing

### ENT.ClimbLedges
```lua
ENT.ClimbLedges = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can climb ledges
**Source:** shared.lua, movements.lua

### ENT.ClimbLedgesMaxHeight
```lua
ENT.ClimbLedgesMaxHeight = math.huge
```
**Type:** number
**Default:** `math.huge`
**Description:** Maximum ledge height to climb (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbLedgesMinHeight
```lua
ENT.ClimbLedgesMinHeight = 0
```
**Type:** number
**Default:** `0`
**Description:** Minimum ledge height to climb (units)
**Source:** shared.lua, movements.lua

### ENT.LedgeDetectionDistance
```lua
ENT.LedgeDetectionDistance = 20
```
**Type:** number
**Default:** `20`
**Description:** Distance to detect ledges ahead (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbProps
```lua
ENT.ClimbProps = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can climb props/physics objects
**Source:** shared.lua, movements.lua

### ENT.ClimbLadders
```lua
ENT.ClimbLadders = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can climb ladders (general setting)
**Source:** shared.lua, movements.lua

### ENT.ClimbLaddersUp
```lua
ENT.ClimbLaddersUp = true
```
**Type:** boolean
**Default:** `true`
**Description:** Can climb ladders upward
**Source:** shared.lua, movements.lua

### ENT.LaddersUpDistance
```lua
ENT.LaddersUpDistance = 20
```
**Type:** number
**Default:** `20`
**Description:** Distance to detect ladders for climbing up (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbLaddersUpMaxHeight
```lua
ENT.ClimbLaddersUpMaxHeight = math.huge
```
**Type:** number
**Default:** `math.huge`
**Description:** Maximum ladder height to climb up (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbLaddersUpMinHeight
```lua
ENT.ClimbLaddersUpMinHeight = 0
```
**Type:** number
**Default:** `0`
**Description:** Minimum ladder height to climb up (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbLaddersDown
```lua
ENT.ClimbLaddersDown = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can climb ladders downward
**Source:** shared.lua, movements.lua

### ENT.LaddersDownDistance
```lua
ENT.LaddersDownDistance = 20
```
**Type:** number
**Default:** `20`
**Description:** Distance to detect ladders for climbing down (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbLaddersDownMaxHeight
```lua
ENT.ClimbLaddersDownMaxHeight = math.huge
```
**Type:** number
**Default:** `math.huge`
**Description:** Maximum ladder height to climb down (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbLaddersDownMinHeight
```lua
ENT.ClimbLaddersDownMinHeight = 0
```
**Type:** number
**Default:** `0`
**Description:** Minimum ladder height to climb down (units)
**Source:** shared.lua, movements.lua

### ENT.ClimbSpeed
```lua
ENT.ClimbSpeed = 60
```
**Type:** number
**Default:** `60`
**Description:** Climbing speed (units/second)
**Source:** shared.lua, movements.lua

### ENT.ClimbOffset
```lua
ENT.ClimbOffset = Vector(0, 0, 0)
```
**Type:** Vector
**Default:** `Vector(0, 0, 0)`
**Description:** Position offset while climbing
**Source:** shared.lua, movements.lua

### ENT.ClimbUpAnimation
```lua
ENT.ClimbUpAnimation = ACT_CLIMB_UP
```
**Type:** number or string
**Default:** `ACT_CLIMB_UP`
**Description:** Animation for climbing upward
**Source:** shared.lua, movements.lua

### ENT.ClimbDownAnimation
```lua
ENT.ClimbDownAnimation = ACT_CLIMB_DOWN
```
**Type:** number or string
**Default:** `ACT_CLIMB_DOWN`
**Description:** Animation for climbing downward
**Source:** shared.lua, movements.lua

### ENT.ClimbAnimRate
```lua
ENT.ClimbAnimRate = 1
```
**Type:** number
**Default:** `1`
**Description:** Playback rate for climb animations
**Source:** shared.lua, movements.lua

---

## AI & Detection

### ENT.BehaviourType
```lua
ENT.BehaviourType = AI_BEHAV_BASE
```
**Type:** number
**Default:** `AI_BEHAV_BASE`
**Description:** AI behavior type
**Options:** `AI_BEHAV_BASE`, `AI_BEHAV_HUMAN`, `AI_BEHAV_CUSTOM`
**Source:** shared.lua, ai.lua

### ENT.Omniscient
```lua
ENT.Omniscient = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can see all entities without line of sight
**Source:** shared.lua, awareness.lua

### ENT.SpotDuration
```lua
ENT.SpotDuration = 30
```
**Type:** number
**Default:** `30`
**Description:** Duration (seconds) to remember spotted entities
**Source:** shared.lua, awareness.lua

### ENT.RangeAttackRange
```lua
ENT.RangeAttackRange = 0
```
**Type:** number
**Default:** `0`
**Description:** Maximum range for ranged attacks (units)
**Source:** shared.lua, ai.lua

### ENT.MeleeAttackRange
```lua
ENT.MeleeAttackRange = 50
```
**Type:** number
**Default:** `50`
**Description:** Maximum range for melee attacks (units)
**Source:** shared.lua, ai.lua

### ENT.ReachEnemyRange
```lua
ENT.ReachEnemyRange = 50
```
**Type:** number
**Default:** `50`
**Description:** Distance considered "reached" for enemies (units)
**Source:** shared.lua, ai.lua

### ENT.AvoidEnemyRange
```lua
ENT.AvoidEnemyRange = 0
```
**Type:** number
**Default:** `0`
**Description:** Distance to maintain from enemies (units)
**Source:** shared.lua, ai.lua

### ENT.AvoidAfraidOfRange
```lua
ENT.AvoidAfraidOfRange = 500
```
**Type:** number
**Default:** `500`
**Description:** Distance to flee from feared entities (units)
**Source:** shared.lua, ai.lua

### ENT.WatchAfraidOfRange
```lua
ENT.WatchAfraidOfRange = 750
```
**Type:** number
**Default:** `750`
**Description:** Distance to keep feared entities in sight (units)
**Source:** shared.lua, ai.lua

---

## Vision & Detection

### ENT.EyeBone
```lua
ENT.EyeBone = ""
```
**Type:** string
**Default:** `""`
**Description:** Bone name to use for eye position
**Source:** shared.lua, detection.lua

### ENT.EyeOffset
```lua
ENT.EyeOffset = Vector(0, 0, 0)
```
**Type:** Vector
**Default:** `Vector(0, 0, 0)`
**Description:** Eye position offset from bone/origin
**Source:** shared.lua, detection.lua

### ENT.EyeAngle
```lua
ENT.EyeAngle = Angle(0, 0, 0)
```
**Type:** Angle
**Default:** `Angle(0, 0, 0)`
**Description:** Eye angle offset
**Source:** shared.lua, detection.lua

### ENT.SightFOV
```lua
ENT.SightFOV = 150
```
**Type:** number
**Default:** `150`
**Description:** Field of view angle (degrees)
**Source:** shared.lua, detection.lua

### ENT.SightRange
```lua
ENT.SightRange = 15000
```
**Type:** number
**Default:** `15000`
**Description:** Maximum vision distance (units)
**Source:** shared.lua, detection.lua

### ENT.MinLuminosity
```lua
ENT.MinLuminosity = 0
```
**Type:** number
**Default:** `0`
**Description:** Minimum brightness to see players (0-1)
**Source:** shared.lua, detection.lua

### ENT.MaxLuminosity
```lua
ENT.MaxLuminosity = 1
```
**Type:** number
**Default:** `1`
**Description:** Maximum brightness to see players (0-1)
**Source:** shared.lua, detection.lua

### ENT.HearingCoefficient
```lua
ENT.HearingCoefficient = 1
```
**Type:** number
**Default:** `1`
**Description:** Hearing sensitivity multiplier
**Source:** shared.lua, detection.lua

---

## Relationships & Factions

### ENT.DefaultRelationship
```lua
ENT.DefaultRelationship = D_NU
```
**Type:** number
**Default:** `D_NU`
**Description:** Default relationship to unknown entities
**Options:** `D_LI` (like), `D_HT` (hate), `D_FR` (fear), `D_NU` (neutral)
**Source:** shared.lua, relationships.lua

### ENT.Factions
```lua
ENT.Factions = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Faction names this NPC belongs to
**Source:** shared.lua, relationships.lua

### ENT.Frightening
```lua
ENT.Frightening = false
```
**Type:** boolean
**Default:** `false`
**Description:** NPCs fear this entity instead of hating it
**Source:** shared.lua, relationships.lua

### ENT.AllyDamageTolerance
```lua
ENT.AllyDamageTolerance = 0.33
```
**Type:** number
**Default:** `0.33`
**Description:** Damage tolerance before turning on allies (0-1)
**Source:** shared.lua, relationships.lua

### ENT.AfraidDamageTolerance
```lua
ENT.AfraidDamageTolerance = 0.33
```
**Type:** number
**Default:** `0.33`
**Description:** Damage tolerance for feared entities (0-1)
**Source:** shared.lua, relationships.lua

### ENT.NeutralDamageTolerance
```lua
ENT.NeutralDamageTolerance = 0.33
```
**Type:** number
**Default:** `0.33`
**Description:** Damage tolerance for neutral entities (0-1)
**Source:** shared.lua, relationships.lua

---

## Animations

### ENT.IdleAnimation
```lua
ENT.IdleAnimation = ACT_IDLE
```
**Type:** number or string
**Default:** `ACT_IDLE`
**Description:** Idle animation activity
**Source:** shared.lua, animations.lua

### ENT.IdleAnimRate
```lua
ENT.IdleAnimRate = 1
```
**Type:** number
**Default:** `1`
**Description:** Playback rate for idle animation
**Source:** shared.lua, animations.lua

### ENT.WalkAnimation
```lua
ENT.WalkAnimation = ACT_WALK
```
**Type:** number or string
**Default:** `ACT_WALK`
**Description:** Walking animation
**Source:** shared.lua, animations.lua

### ENT.WalkAnimRate
```lua
ENT.WalkAnimRate = 1
```
**Type:** number
**Default:** `1`
**Description:** Playback rate for walk animation
**Source:** shared.lua, animations.lua

### ENT.RunAnimation
```lua
ENT.RunAnimation = ACT_RUN
```
**Type:** number or string
**Default:** `ACT_RUN`
**Description:** Running animation
**Source:** shared.lua, animations.lua

### ENT.RunAnimRate
```lua
ENT.RunAnimRate = 1
```
**Type:** number
**Default:** `1`
**Description:** Playback rate for run animation
**Source:** shared.lua, animations.lua

### ENT.JumpAnimation
```lua
ENT.JumpAnimation = ACT_JUMP
```
**Type:** number or string
**Default:** `ACT_JUMP`
**Description:** Jump animation
**Source:** shared.lua, animations.lua

### ENT.JumpAnimRate
```lua
ENT.JumpAnimRate = 1
```
**Type:** number
**Default:** `1`
**Description:** Playback rate for jump animation
**Source:** shared.lua, animations.lua

---

## Sounds

### ENT.OnSpawnSounds
```lua
ENT.OnSpawnSounds = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Sound files to play on spawn
**Source:** shared.lua

### ENT.OnIdleSounds
```lua
ENT.OnIdleSounds = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Idle sound files (plays randomly)
**Source:** shared.lua

### ENT.IdleSoundDelay
```lua
ENT.IdleSoundDelay = 2
```
**Type:** number
**Default:** `2`
**Description:** Delay between idle sounds (seconds)
**Source:** shared.lua

### ENT.ClientIdleSounds
```lua
ENT.ClientIdleSounds = false
```
**Type:** boolean
**Default:** `false`
**Description:** Play idle sounds clientside
**Source:** shared.lua

### ENT.OnDamageSounds
```lua
ENT.OnDamageSounds = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Pain/damage sound files
**Source:** shared.lua

### ENT.DamageSoundDelay
```lua
ENT.DamageSoundDelay = 0.25
```
**Type:** number
**Default:** `0.25`
**Description:** Minimum delay between damage sounds (seconds)
**Source:** shared.lua

### ENT.OnDeathSounds
```lua
ENT.OnDeathSounds = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Death sound files
**Source:** shared.lua

### ENT.OnDownedSounds
```lua
ENT.OnDownedSounds = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Downed state sound files
**Source:** shared.lua

### ENT.Footsteps
```lua
ENT.Footsteps = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Footstep sound files
**Source:** shared.lua

---

## Weapons

### ENT.UseWeapons
```lua
ENT.UseWeapons = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can use weapons
**Source:** shared.lua, weapons.lua

### ENT.Weapons
```lua
ENT.Weapons = {}
```
**Type:** table (array of strings)
**Default:** `{}`
**Description:** Weapon class names to spawn with
**Source:** shared.lua, weapons.lua

### ENT.DropWeaponOnDeath
```lua
ENT.DropWeaponOnDeath = false
```
**Type:** boolean
**Default:** `false`
**Description:** Drop weapon on death
**Source:** shared.lua, weapons.lua

### ENT.AcceptPlayerWeapons
```lua
ENT.AcceptPlayerWeapons = true
```
**Type:** boolean
**Default:** `true`
**Description:** Can pick up player weapons
**Source:** shared.lua, weapons.lua

---

## Possession

### ENT.PossessionEnabled
```lua
ENT.PossessionEnabled = false
```
**Type:** boolean
**Default:** `false`
**Description:** Can be possessed by players
**Source:** shared.lua, possession.lua

### ENT.PossessionPrompt
```lua
ENT.PossessionPrompt = true
```
**Type:** boolean
**Default:** `true`
**Description:** Show possession prompt
**Source:** shared.lua, possession.lua

### ENT.PossessionCrosshair
```lua
ENT.PossessionCrosshair = false
```
**Type:** boolean
**Default:** `false`
**Description:** Show crosshair when possessed
**Source:** shared.lua, possession.lua

### ENT.PossessionMovement
```lua
ENT.PossessionMovement = POSSESSION_MOVE_1DIR
```
**Type:** number
**Default:** `POSSESSION_MOVE_1DIR`
**Description:** Movement mode when possessed
**Options:** `POSSESSION_MOVE_8DIR`, `POSSESSION_MOVE_1DIR`, `POSSESSION_MOVE_4DIR`, `POSSESSION_MOVE_CUSTOM`
**Source:** shared.lua, possession.lua

### ENT.PossessionViews
```lua
ENT.PossessionViews = {}
```
**Type:** table (array of view definitions)
**Default:** `{}`
**Description:** Camera view presets for possession
**Source:** shared.lua, possession.lua

### ENT.PossessionBinds
```lua
ENT.PossessionBinds = {}
```
**Type:** table
**Default:** `{}`
**Description:** Key bindings for possession controls
**Source:** shared.lua, possession.lua

---

## Complete Property Reference Table

| Property | Type | Default | Description | Source File |
|----------|------|---------|-------------|-------------|
| **Basic** |
| `Base` | string | `"base_nextbot"` | Base class | shared.lua |
| `Type` | string | `"nextbot"` | Entity type | shared.lua |
| `IsDrGEntity` | boolean | `true` | DrGBase entity marker | shared.lua |
| `IsDrGNextbot` | boolean | `true` | DrGBase nextbot marker | shared.lua |
| `PrintName` | string | `"Template"` | Spawn menu name | shared.lua |
| `Category` | string | `"Other"` | Spawn menu category | shared.lua |
| **Model & Appearance** |
| `Models` | table | `{"models/player/kleiner.mdl"}` | Model paths | shared.lua |
| `Skins` | table/number | `{0}` | Skin indices | shared.lua |
| `ModelScale` | number/table | `1` | Model scale | shared.lua |
| `CollisionBounds` | Vector | `Vector(10,10,72)` | Collision box | shared.lua |
| `BloodColor` | number | `BLOOD_COLOR_RED` | Blood color | shared.lua |
| **Health & Status** |
| `SpawnHealth` | number | `100` | Starting health | shared.lua |
| `HealthRegen` | number | `0` | HP regen per second | shared.lua |
| `MinPhysDamage` | number | `10` | Min physics damage | shared.lua |
| `MinFallDamage` | number | `10` | Min fall damage | shared.lua |
| `RagdollOnDeath` | boolean | `true` | Create ragdoll | shared.lua |
| **Movement** |
| `WalkSpeed` | number | `100` | Walk speed | shared.lua |
| `RunSpeed` | number | `200` | Run speed | shared.lua |
| `UseWalkframes` | boolean | `false` | Use animation speed | shared.lua |
| **Locomotion** |
| `Acceleration` | number | `1000` | Acceleration rate | shared.lua |
| `Deceleration` | number | `1000` | Deceleration rate | shared.lua |
| `JumpHeight` | number | `50` | Jump height | shared.lua |
| `StepHeight` | number | `20` | Max step height | shared.lua |
| `MaxYawRate` | number | `250` | Turn rate (deg/s) | shared.lua |
| `DeathDropHeight` | number | `200` | Fatal fall height | shared.lua |
| **Climbing - Ledges** |
| `ClimbLedges` | boolean | `false` | Can climb ledges | shared.lua |
| `ClimbLedgesMaxHeight` | number | `math.huge` | Max ledge height | shared.lua |
| `ClimbLedgesMinHeight` | number | `0` | Min ledge height | shared.lua |
| `LedgeDetectionDistance` | number | `20` | Ledge detect range | shared.lua |
| `ClimbProps` | boolean | `false` | Can climb props | shared.lua |
| **Climbing - Ladders** |
| `ClimbLadders` | boolean | `false` | Can climb ladders | shared.lua |
| `ClimbLaddersUp` | boolean | `true` | Can climb up | shared.lua |
| `LaddersUpDistance` | number | `20` | Up detect range | shared.lua |
| `ClimbLaddersUpMaxHeight` | number | `math.huge` | Max up height | shared.lua |
| `ClimbLaddersUpMinHeight` | number | `0` | Min up height | shared.lua |
| `ClimbLaddersDown` | boolean | `false` | Can climb down | shared.lua |
| `LaddersDownDistance` | number | `20` | Down detect range | shared.lua |
| `ClimbLaddersDownMaxHeight` | number | `math.huge` | Max down height | shared.lua |
| `ClimbLaddersDownMinHeight` | number | `0` | Min down height | shared.lua |
| `ClimbSpeed` | number | `60` | Climb speed | shared.lua |
| `ClimbOffset` | Vector | `Vector(0,0,0)` | Climb position offset | shared.lua |
| `ClimbUpAnimation` | ACT | `ACT_CLIMB_UP` | Climb up anim | shared.lua |
| `ClimbDownAnimation` | ACT | `ACT_CLIMB_DOWN` | Climb down anim | shared.lua |
| `ClimbAnimRate` | number | `1` | Climb anim rate | shared.lua |
| **AI Behavior** |
| `BehaviourType` | number | `AI_BEHAV_BASE` | AI behavior type | shared.lua |
| `Omniscient` | boolean | `false` | See everything | shared.lua |
| `SpotDuration` | number | `30` | Remember time (s) | shared.lua |
| `RangeAttackRange` | number | `0` | Range attack distance | shared.lua |
| `MeleeAttackRange` | number | `50` | Melee attack distance | shared.lua |
| `ReachEnemyRange` | number | `50` | Reach distance | shared.lua |
| `AvoidEnemyRange` | number | `0` | Avoid distance | shared.lua |
| `AvoidAfraidOfRange` | number | `500` | Flee distance | shared.lua |
| `WatchAfraidOfRange` | number | `750` | Watch fear distance | shared.lua |
| **Vision** |
| `EyeBone` | string | `""` | Eye bone name | shared.lua |
| `EyeOffset` | Vector | `Vector(0,0,0)` | Eye position offset | shared.lua |
| `EyeAngle` | Angle | `Angle(0,0,0)` | Eye angle offset | shared.lua |
| `SightFOV` | number | `150` | Field of view (deg) | shared.lua |
| `SightRange` | number | `15000` | Vision distance | shared.lua |
| `MinLuminosity` | number | `0` | Min light to see (0-1) | shared.lua |
| `MaxLuminosity` | number | `1` | Max light to see (0-1) | shared.lua |
| `HearingCoefficient` | number | `1` | Hearing multiplier | shared.lua |
| **Relationships** |
| `DefaultRelationship` | D_* | `D_NU` | Default disposition | shared.lua |
| `Factions` | table | `{}` | Faction list | shared.lua |
| `Frightening` | boolean | `false` | Causes fear | shared.lua |
| `AllyDamageTolerance` | number | `0.33` | Ally damage tolerance | shared.lua |
| `AfraidDamageTolerance` | number | `0.33` | Fear damage tolerance | shared.lua |
| `NeutralDamageTolerance` | number | `0.33` | Neutral damage tolerance | shared.lua |
| **Animations** |
| `IdleAnimation` | ACT/string | `ACT_IDLE` | Idle animation | shared.lua |
| `IdleAnimRate` | number | `1` | Idle anim rate | shared.lua |
| `WalkAnimation` | ACT/string | `ACT_WALK` | Walk animation | shared.lua |
| `WalkAnimRate` | number | `1` | Walk anim rate | shared.lua |
| `RunAnimation` | ACT/string | `ACT_RUN` | Run animation | shared.lua |
| `RunAnimRate` | number | `1` | Run anim rate | shared.lua |
| `JumpAnimation` | ACT/string | `ACT_JUMP` | Jump animation | shared.lua |
| `JumpAnimRate` | number | `1` | Jump anim rate | shared.lua |
| **Sounds** |
| `OnSpawnSounds` | table | `{}` | Spawn sounds | shared.lua |
| `OnIdleSounds` | table | `{}` | Idle sounds | shared.lua |
| `IdleSoundDelay` | number | `2` | Idle sound delay (s) | shared.lua |
| `ClientIdleSounds` | boolean | `false` | Clientside idle sounds | shared.lua |
| `OnDamageSounds` | table | `{}` | Damage sounds | shared.lua |
| `DamageSoundDelay` | number | `0.25` | Damage sound delay (s) | shared.lua |
| `OnDeathSounds` | table | `{}` | Death sounds | shared.lua |
| `OnDownedSounds` | table | `{}` | Downed sounds | shared.lua |
| `Footsteps` | table | `{}` | Footstep sounds | shared.lua |
| **Weapons** |
| `UseWeapons` | boolean | `false` | Can use weapons | shared.lua |
| `Weapons` | table | `{}` | Starting weapons | shared.lua |
| `DropWeaponOnDeath` | boolean | `false` | Drop weapon on death | shared.lua |
| `AcceptPlayerWeapons` | boolean | `true` | Can pick up weapons | shared.lua |
| **Possession** |
| `PossessionEnabled` | boolean | `false` | Can be possessed | shared.lua |
| `PossessionPrompt` | boolean | `true` | Show prompt | shared.lua |
| `PossessionCrosshair` | boolean | `false` | Show crosshair | shared.lua |
| `PossessionMovement` | number | `POSSESSION_MOVE_1DIR` | Movement mode | shared.lua |
| `PossessionViews` | table | `{}` | Camera views | shared.lua |
| `PossessionBinds` | table | `{}` | Key bindings | shared.lua |

---

## Property Categories Summary

- **Basic Properties:** 6 properties
- **Model & Appearance:** 5 properties
- **Health & Status:** 5 properties
- **Movement:** 3 properties
- **Locomotion:** 6 properties
- **Climbing (Ledges):** 5 properties
- **Climbing (Ladders):** 14 properties
- **AI Behavior:** 8 properties
- **Vision:** 8 properties
- **Relationships:** 6 properties
- **Animations:** 8 properties
- **Sounds:** 9 properties
- **Weapons:** 4 properties
- **Possession:** 6 properties

**Total Configurable Properties: 93**

---

## See Also

- [AI Functions](./ai.md)
- [Movement Functions](./movement.md)
- [Hooks](./hooks.md)
- [Getting Started](../../guides/first-npc.md)
