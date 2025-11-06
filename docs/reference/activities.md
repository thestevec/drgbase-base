# Activity IDs

Animation activity reference for DrGBase NPCs.

---

## Overview

Activities (ACT_*) are animation identifiers used by Source Engine and Garry's Mod. DrGBase NPCs use these activities for animations. The model must support the activity for it to work.

**Usage:**
```lua
self:PlayActivity(ACT_RUN)
ENT.IdleAnimation = ACT_IDLE
ENT.RunAnimation = ACT_WALK
```

---

## Common Base Activities

### Movement

| Activity | Description | Common Use |
|----------|-------------|------------|
| `ACT_IDLE` | Standing idle | Default idle animation |
| `ACT_WALK` | Walking animation | Slow movement |
| `ACT_RUN` | Running animation | Fast movement |
| `ACT_JUMP` | Jump animation | Jumping/falling |
| `ACT_GLIDE` | Gliding animation | Airborne movement (Antlions) |
| `ACT_LAND` | Landing animation | Touch ground after jump |
| `ACT_CROUCHIDLE` | Crouched idle | Stationary while crouched |
| `ACT_CROUCH` | Crouch walking | Moving while crouched |

### Combat

| Activity | Description | Common Use |
|----------|-------------|------------|
| `ACT_MELEE_ATTACK1` | Primary melee attack | Basic melee attack |
| `ACT_MELEE_ATTACK2` | Secondary melee attack | Alternative melee |
| `ACT_RANGE_ATTACK1` | Primary ranged attack | Shooting/throwing |
| `ACT_RANGE_ATTACK2` | Secondary ranged attack | Alternative ranged |
| `ACT_RELOAD` | Reload weapon | Weapon reload |
| `ACT_COVER_LOW` | Low cover position | Taking cover (low) |
| `ACT_COVER_MED` | Medium cover position | Taking cover (medium) |

### Reactions

| Activity | Description | Common Use |
|----------|-------------|------------|
| `ACT_FLINCH_PHYSICS` | React to physics | Being hit by objects |
| `ACT_FLINCH_CHEST` | Chest flinch | Taking damage to chest |
| `ACT_FLINCH_HEAD` | Head flinch | Taking damage to head |
| `ACT_FLINCH_LEFTARM` | Left arm flinch | Taking damage to left arm |
| `ACT_FLINCH_RIGHTARM` | Right arm flinch | Taking damage to right arm |

### Signals

| Activity | Description | Common Use |
|----------|-------------|------------|
| `ACT_SIGNAL_GROUP` | Signal to group | Alert allies |
| `ACT_SIGNAL_HALT` | Stop signal | Halt command |
| `ACT_SIGNAL_ADVANCE` | Advance signal | Move forward command |

---

## HL2 Deathmatch Activities (Humanoids)

These activities are specific to Half-Life 2 Deathmatch models and the `drgbase_nextbot_human` base.

### Hold Types

Activities vary by weapon hold type. Common hold types:
- `normal` - Default/unarmed
- `pistol` - Pistols
- `smg` - SMGs
- `ar2` - Assault rifles
- `shotgun` - Shotguns
- `rpg` - RPGs and heavy weapons
- `crossbow` - Crossbows
- `grenade` - Grenades
- `melee` - Melee weapons
- `fist` - Fists/unarmed combat
- `physgun` - Physics gun

### Running Activities

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_RUN` | normal | Running (default) |
| `ACT_HL2MP_RUN_PISTOL` | pistol | Running with pistol |
| `ACT_HL2MP_RUN_SMG1` | smg | Running with SMG |
| `ACT_HL2MP_RUN_AR2` | ar2 | Running with AR2 |
| `ACT_HL2MP_RUN_SHOTGUN` | shotgun | Running with shotgun |
| `ACT_HL2MP_RUN_RPG` | rpg | Running with RPG |
| `ACT_HL2MP_RUN_CROSSBOW` | crossbow | Running with crossbow |
| `ACT_HL2MP_RUN_GRENADE` | grenade | Running with grenade |
| `ACT_HL2MP_RUN_MELEE` | melee | Running with melee weapon |
| `ACT_HL2MP_RUN_FIST` | fist | Running unarmed |
| `ACT_HL2MP_RUN_PHYSGUN` | physgun | Running with physics gun |

### Walking Activities

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_WALK` | normal | Walking (default) |
| `ACT_HL2MP_WALK_PISTOL` | pistol | Walking with pistol |
| `ACT_HL2MP_WALK_SMG1` | smg | Walking with SMG |
| `ACT_HL2MP_WALK_AR2` | ar2 | Walking with AR2 |
| `ACT_HL2MP_WALK_SHOTGUN` | shotgun | Walking with shotgun |
| `ACT_HL2MP_WALK_RPG` | rpg | Walking with RPG |
| `ACT_HL2MP_WALK_CROSSBOW` | crossbow | Walking with crossbow |
| `ACT_HL2MP_WALK_GRENADE` | grenade | Walking with grenade |
| `ACT_HL2MP_WALK_MELEE` | melee | Walking with melee weapon |
| `ACT_HL2MP_WALK_FIST` | fist | Walking unarmed |

### Idle Activities

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_IDLE` | normal | Idle (default) |
| `ACT_HL2MP_IDLE_PISTOL` | pistol | Idle with pistol |
| `ACT_HL2MP_IDLE_SMG1` | smg | Idle with SMG |
| `ACT_HL2MP_IDLE_AR2` | ar2 | Idle with AR2 |
| `ACT_HL2MP_IDLE_SHOTGUN` | shotgun | Idle with shotgun |
| `ACT_HL2MP_IDLE_RPG` | rpg | Idle with RPG |
| `ACT_HL2MP_IDLE_CROSSBOW` | crossbow | Idle with crossbow |
| `ACT_HL2MP_IDLE_GRENADE` | grenade | Idle with grenade |
| `ACT_HL2MP_IDLE_MELEE` | melee | Idle with melee weapon |
| `ACT_HL2MP_IDLE_FIST` | fist | Idle unarmed |

### Crouching Activities

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_WALK_CROUCH` | normal | Crouch walk (default) |
| `ACT_HL2MP_WALK_CROUCH_PISTOL` | pistol | Crouch walk with pistol |
| `ACT_HL2MP_WALK_CROUCH_AR2` | ar2 | Crouch walk with AR2 |
| `ACT_HL2MP_IDLE_CROUCH` | normal | Crouch idle (default) |
| `ACT_HL2MP_IDLE_CROUCH_PISTOL` | pistol | Crouch idle with pistol |
| `ACT_HL2MP_IDLE_CROUCH_AR2` | ar2 | Crouch idle with AR2 |

### Jump Activities

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_JUMP_KNIFE` | normal | Jumping (default) |
| `ACT_HL2MP_JUMP_PISTOL` | pistol | Jumping with pistol |
| `ACT_HL2MP_JUMP_AR2` | ar2 | Jumping with AR2 |
| `ACT_HL2MP_JUMP_SMG1` | smg | Jumping with SMG |
| `ACT_HL2MP_JUMP_FIST` | fist | Jumping unarmed |

### Attack Gestures

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_GESTURE_RANGE_ATTACK` | normal | Attack gesture (default) |
| `ACT_HL2MP_GESTURE_RANGE_ATTACK_PISTOL` | pistol | Pistol fire gesture |
| `ACT_HL2MP_GESTURE_RANGE_ATTACK_AR2` | ar2 | AR2 fire gesture |
| `ACT_HL2MP_GESTURE_RANGE_ATTACK_SMG1` | smg | SMG fire gesture |
| `ACT_HL2MP_GESTURE_RANGE_ATTACK_SHOTGUN` | shotgun | Shotgun fire gesture |

### Reload Gestures

| Activity | Hold Type | Description |
|----------|-----------|-------------|
| `ACT_HL2MP_GESTURE_RELOAD` | normal | Reload gesture (default) |
| `ACT_HL2MP_GESTURE_RELOAD_PISTOL` | pistol | Pistol reload gesture |
| `ACT_HL2MP_GESTURE_RELOAD_AR2` | ar2 | AR2 reload gesture |
| `ACT_HL2MP_GESTURE_RELOAD_SMG1` | smg | SMG reload gesture |
| `ACT_HL2MP_GESTURE_RELOAD_SHOTGUN` | shotgun | Shotgun reload gesture |

---

## Weapon-Specific Activities

### View Model Activities

| Activity | Description |
|----------|-------------|
| `ACT_VM_IDLE` | Weapon idle in view |
| `ACT_VM_DRAW` | Drawing weapon |
| `ACT_VM_HOLSTER` | Holstering weapon |
| `ACT_VM_PRIMARYATTACK` | Primary attack animation |
| `ACT_VM_SECONDARYATTACK` | Secondary attack animation |
| `ACT_VM_RELOAD` | Reload animation |

---

## Usage Examples

### Basic NPC

```lua
ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.JumpAnimation = ACT_JUMP

function ENT:OnMeleeAttack(enemy)
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### Humanoid NPC

```lua
-- Base automatically handles weapon hold types
ENT.Base = "drgbase_nextbot_human"
ENT.Weapons = {"weapon_ar2", "weapon_pistol"}

-- Animations are automatically set based on weapon
-- ACT_HL2MP_RUN_AR2 when holding AR2
-- ACT_HL2MP_IDLE_PISTOL when holding pistol
```

### Custom Activity Usage

```lua
function ENT:CustomInitialize()
    -- Override default animations
    self.IdleAnimation = ACT_CROUCH
    self.RunAnimation = ACT_WALK
end

function ENT:OnNewEnemy(enemy)
    -- Play signal activity
    self:PlayActivity(ACT_SIGNAL_GROUP)
    self:EmitSound("NPC.Alert")
end
```

### Playing Activities

```lua
-- Play activity and wait for completion
self:PlayActivityAndWait(ACT_RELOAD)

-- Play activity with movement
self:PlayActivityAndMove(ACT_RUN, 1.5) -- 1.5x speed

-- Play activity as gesture (doesn't interrupt movement)
self:PlayActivity(ACT_SIGNAL_GROUP)
```

---

## Activity Types

### Regular Activities
Play as the main animation, interrupting current movement animation:
```lua
self:StartActivity(ACT_IDLE)
```

### Gesture Activities
Play as overlay, doesn't interrupt base animation:
```lua
self:PlayActivity(ACT_SIGNAL_GROUP) -- Gesture
```

### Sequences vs Activities
- **Activities** (ACT_*) - Generic animation categories
- **Sequences** - Specific animations in the model

```lua
-- Using activity (model-independent)
self:PlayActivity(ACT_RUN)

-- Using sequence (model-specific)
self:PlaySequence("attack1")
```

---

## Model Support

Not all models support all activities. Check model compatibility:

1. **HL2 NPC Models** - Support base activities (ACT_IDLE, ACT_WALK, etc.)
2. **Player Models** - Support HL2MP activities (ACT_HL2MP_*)
3. **Custom Models** - Varies by model, check in Model Viewer

**Testing Activities:**
```lua
-- Check if model has activity
local seq = self:SelectRandomSequence(ACT_RUN)
if seq == -1 then
    print("Model doesn't support ACT_RUN")
end
```

---

## See Also

- [Animation System](../api/nextbot/animations.md)
- [Animation Events Reference](./anim-events.md)
- [Creating NPCs Guide](../guides/creating-npcs.md)
- [Example NPCs](../examples/README.md)
