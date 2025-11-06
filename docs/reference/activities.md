# Activities Reference

Complete reference of activity constants used for DrGBase NPC animations.

## What Are Activities?

Activities (ACT_*) are animation constants that map to model-specific animations. Using activities instead of sequence names makes NPCs work with different models automatically.

```lua
-- Good: Uses activity (works with any model)
ENT.RunAnimation = ACT_RUN

-- Bad: Uses sequence name (model-specific)
ENT.RunAnimation = "run_all"  // Only works with specific models
```

## Common Activities

### Movement Activities

#### ACT_IDLE
Default idle animation when NPC is stationary.
```lua
ENT.IdleAnimation = ACT_IDLE
```

#### ACT_WALK
Walking animation for slow movement.
```lua
ENT.WalkAnimation = ACT_WALK
```

#### ACT_RUN
Running animation for fast movement.
```lua
ENT.RunAnimation = ACT_RUN
```

#### ACT_JUMP
Jump animation when NPC leaves ground.
```lua
ENT.JumpAnimation = ACT_JUMP
```

#### ACT_LAND
Landing animation when NPC hits ground.
```lua
ENT.LandAnimation = ACT_LAND
```

#### ACT_CLIMB_UP
Climbing upward animation.
```lua
function ENT:OnLadder()
    self:PlayActivity(ACT_CLIMB_UP)
end
```

#### ACT_CLIMB_DOWN
Climbing downward animation.
```lua
function ENT:OnLadderDown()
    self:PlayActivity(ACT_CLIMB_DOWN)
end
```

### Combat Activities

#### ACT_MELEE_ATTACK1
Primary melee attack animation.
```lua
function ENT:OnMeleeAttack(enemy)
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

#### ACT_MELEE_ATTACK2
Secondary melee attack animation.
```lua
function ENT:OnMeleeAttack(enemy)
    if math.random(1, 2) == 1 then
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    else
        self:PlayActivityAndMove(ACT_MELEE_ATTACK2, 1, self.FaceEnemy)
    end
end
```

#### ACT_RANGE_ATTACK1
Primary ranged attack animation.
```lua
function ENT:OnRangeAttack(enemy)
    self:PlayActivity(ACT_RANGE_ATTACK1)
end
```

#### ACT_RANGE_ATTACK2
Secondary ranged attack animation.
```lua
function ENT:OnRangeAttack(enemy)
    self:PlayActivity(ACT_RANGE_ATTACK2)
end
```

### Reaction Activities

#### ACT_FLINCH_PHYSICS
Flinch when hit by physics object.
```lua
function ENT:OnTakeDamage(dmg)
    if dmg:GetDamageType() == DMG_CRUSH then
        self:PlayActivity(ACT_FLINCH_PHYSICS)
    end
    return true
end
```

#### ACT_FLINCH_HEAD
Flinch when hit in head.
```lua
function ENT:OnTakeDamage(dmg, hitgroup)
    if hitgroup == HITGROUP_HEAD then
        self:PlayActivity(ACT_FLINCH_HEAD)
    end
    return true
end
```

#### ACT_SMALL_FLINCH
Small flinch for minor damage.
```lua
function ENT:OnTakeDamage(dmg)
    if dmg:GetDamage() < 10 then
        self:PlayActivity(ACT_SMALL_FLINCH)
    end
    return true
end
```

### Death Activities

#### ACT_DIESIMPLE
Simple death animation.
```lua
function ENT:OnDeath()
    self:PlayActivity(ACT_DIESIMPLE)
end
```

#### ACT_DIEVIOLENT
Violent death animation.
```lua
function ENT:OnDeath(dmg)
    if dmg:GetDamage() > 50 then
        self:PlayActivity(ACT_DIEVIOLENT)
    else
        self:PlayActivity(ACT_DIESIMPLE)
    end
end
```

## Human NPC Activities

Human NPCs (drgbase_nextbot_human) use HL2 multiplayer activities with weapon modifiers.

### Base Movement

#### ACT_HL2MP_IDLE
Idle animation for human NPCs.
```lua
ENT.IdleAnimation = ACT_HL2MP_IDLE
```

#### ACT_HL2MP_WALK
Walking animation for human NPCs.
```lua
ENT.WalkAnimation = ACT_HL2MP_WALK
```

#### ACT_HL2MP_RUN
Running animation for human NPCs.
```lua
ENT.RunAnimation = ACT_HL2MP_RUN
```

### Weapon-Specific Activities

#### ACT_HL2MP_RUN_AR2
Running with assault rifle.
```lua
-- Automatically selected based on weapon
-- ENT:SetWeapon("weapon_ar2") uses this
```

#### ACT_HL2MP_RUN_PISTOL
Running with pistol.
```lua
-- Used when holding pistol weapons
```

#### ACT_HL2MP_RUN_SHOTGUN
Running with shotgun.
```lua
-- Used when holding shotgun weapons
```

#### ACT_HL2MP_RUN_SMG1
Running with SMG.
```lua
-- Used when holding SMG weapons
```

#### ACT_HL2MP_RUN_CROSSBOW
Running with crossbow.
```lua
-- Used when holding crossbow
```

#### ACT_HL2MP_RUN_RPG
Running with RPG.
```lua
-- Used when holding RPG
```

#### ACT_HL2MP_RUN_MELEE
Running with melee weapon.
```lua
-- Used for crowbar and other melee weapons
```

#### ACT_HL2MP_RUN_FIST
Running with fists.
```lua
-- Used when unarmed
```

### Weapon Animations

#### ACT_HL2MP_GESTURE_RELOAD
Reload gesture animation.
```lua
function ENT:OnReload()
    self:PlayGesture(ACT_HL2MP_GESTURE_RELOAD)
end
```

#### ACT_HL2MP_GESTURE_RANGE_ATTACK
Firing weapon gesture.
```lua
function ENT:OnWeaponFired(weapon)
    self:PlayGesture(ACT_HL2MP_GESTURE_RANGE_ATTACK)
end
```

## Sprite NPC Activities

Sprite NPCs (drgbase_nextbot_sprite) can use regular activities if sprite frames are named accordingly.

```lua
-- Map activities to sprite frames
ENT.RunAnimation = "run"      // Uses run_*.png frames
ENT.WalkAnimation = "walk"    // Uses walk_*.png frames
ENT.IdleAnimation = "idle"    // Uses idle_*.png frames
ENT.JumpAnimation = "jump"    // Uses jump_*.png frames
```

## Custom Activities

You can also play custom sequences directly:

```lua
function ENT:PlayCustomAnimation()
    -- Play by sequence name (model-specific)
    self:PlaySequenceAndMove("special_taunt")
    
    -- Or convert sequence to activity
    local seq = self:LookupSequence("special_taunt")
    local act = self:GetSequenceActivity(seq)
    self:PlayActivity(act)
end
```

## Activity Helpers

### PlayActivity()
Play activity once.
```lua
self:PlayActivity(ACT_MELEE_ATTACK1)
```

### PlayActivityAndMove()
Play activity and continue moving.
```lua
self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
```

### SetActivity()
Set looping activity.
```lua
self:SetActivity(ACT_IDLE)
```

### GetActivity()
Get current activity.
```lua
if self:GetActivity() == ACT_RUN then
    // NPC is running
end
```

## Checking Activity Support

Not all models support all activities. Check if a model has an activity:

```lua
function ENT:CustomInitialize()
    -- Check if model has activity
    local seq = self:SelectWeightedSequence(ACT_RUN)
    if seq == -1 then
        -- Model doesn't have ACT_RUN, use alternative
        ENT.RunAnimation = ACT_WALK
        ENT.UseWalkframes = true
    end
end
```

## Common Issues

### Activity Not Playing
```lua
// Problem: Activity doesn't exist for model
ENT.RunAnimation = ACT_RUN

// Solution: Check model has activity
function ENT:CustomInitialize()
    if self:SelectWeightedSequence(ACT_RUN) == -1 then
        self.RunAnimation = ACT_WALK
    end
end
```

### Wrong Animation Speed
```lua
// Use playback rate to adjust speed
self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1.5, self.FaceEnemy)  // 1.5x speed
```

For animation best practices, see [Examples: Simple Melee NPC](../examples/simple-melee-npc.md).
