# DrGBase Quick Reference

## NPC Configuration Properties

### Basic Information
```lua
ENT.PrintName = "NPC Name"          -- Name in spawn menu
ENT.Category = "Category"           -- Spawn menu category
ENT.Models = {"model.mdl"}          -- Array of models
ENT.ModelScale = 1.0                -- Size multiplier
ENT.BloodColor = BLOOD_COLOR_RED    -- Blood color
ENT.CollisionBounds = Vector(x,y,z) -- Collision box
```

### Health & Stats
```lua
ENT.SpawnHealth = 100               -- Starting health
ENT.HealthRegen = 0                 -- HP per second
ENT.MinPhysDamage = 10              -- Min physics damage
```

### AI Configuration
```lua
ENT.MeleeAttackRange = 50           -- Melee range
ENT.RangeAttackRange = 500          -- Ranged range
ENT.ReachEnemyRange = 100           -- Chase distance
ENT.AvoidEnemyRange = 0             -- Flee distance
```

### Detection
```lua
ENT.SightFOV = 120                  -- Field of view (degrees)
ENT.SightRange = 2000               -- Sight distance
ENT.HearingCoefficient = 1.0        -- Hearing multiplier
ENT.EyeBone = "Bone Name"           -- Eye bone for vision
ENT.EyeOffset = Vector(x,y,z)       -- Eye position offset
```

### Movement
```lua
ENT.WalkSpeed = 100                 -- Walk speed
ENT.RunSpeed = 200                  -- Run speed
ENT.Acceleration = 900              -- Acceleration rate
ENT.JumpHeight = 70                 -- Jump height
ENT.StepHeight = 22                 -- Max step height
ENT.UseWalkframes = true            -- Use walk animations
```

### Animations
```lua
ENT.WalkAnimation = ACT_WALK        -- Walk animation
ENT.RunAnimation = ACT_RUN          -- Run animation
ENT.IdleAnimation = ACT_IDLE        -- Idle animation
ENT.JumpAnimation = ACT_JUMP        -- Jump animation
```

### Relationships
```lua
ENT.DefaultRelationship = D_HT      -- Default disposition
ENT.Factions = {FACTION_ZOMBIES}    -- Faction list
ENT.Frightening = false             -- Causes fear
```

### Sounds
```lua
ENT.OnSpawnSounds = {"sound"}       -- Spawn sounds
ENT.OnIdleSounds = {"sound"}        -- Idle sounds
ENT.OnDamageSounds = {"sound"}      -- Damage sounds
ENT.OnDeathSounds = {"sound"}       -- Death sounds
ENT.OnFootstepSounds = {"sound"}    -- Footstep sounds
```

### Weapons (Human Base Only)
```lua
ENT.Base = "drgbase_nextbot_human"  -- Required for weapons
ENT.Weapons = {"weapon_ar2"}        -- Weapon list
ENT.WeaponAccuracy = 0.75           -- Accuracy (0-1)
ENT.DropWeaponOnDeath = true        -- Drop on death
```

### Possession
```lua
ENT.PossessionEnabled = true                    -- Allow possession
ENT.PossessionMovement = POSSESSION_MOVE_8DIR   -- Movement type
ENT.PossessionCrosshair = true                  -- Show crosshair
ENT.PossessionViews = {{...}}                   -- Camera views
ENT.PossessionBinds = {[IN_ATTACK] = {{...}}}   -- Key binds
```

---

## Essential Hooks

### Lifecycle Hooks
```lua
function ENT:CustomInitialize()
    -- Called when NPC spawns
end

function ENT:CustomThink()
    -- Called every frame
end

function ENT:AIBehaviour()
    -- Custom AI (coroutine)
    while true do
        -- AI logic
        self:Wait(0.5)
    end
end
```

### Combat Hooks
```lua
function ENT:OnMeleeAttack(enemy)
    -- Called for melee attack
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnRangeAttack(enemy)
    -- Called for ranged attack
end

function ENT:OnAnimEvent()
    -- Called on animation events
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({damage = 10})
    end
end
```

### Damage Hooks
```lua
function ENT:OnTakeDamage(dmg)
    -- Called when damaged
    -- Return multiplier (0 = no damage, 1 = normal, 2 = double)
    return 1.0
end

function ENT:OnDeath(dmg, hitgroup)
    -- Called on death
end
```

### Enemy Hooks
```lua
function ENT:OnNewEnemy(enemy)
    -- Called when acquiring new enemy
end

function ENT:HandleEnemy(enemy)
    -- Override default enemy handling
    self:ChaseEnemy(enemy) -- Or custom logic
end
```

### Movement Hooks
```lua
function ENT:OnIdle()
    -- Called when idle
    self:AddPatrolPos(self:RandomPos(1500))
end

function ENT:OnReachedPatrol(pos)
    -- Called when reaching patrol point
    self:Wait(5)
end

function ENT:OnLeaveGround()
    -- Called when jumping/falling
end

function ENT:OnLandOnGround()
    -- Called when landing
end
```

---

## Essential Methods

### Animation
```lua
self:PlayActivity(ACT_WALK)
self:PlaySequence("sequence_name")
self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, callback)
self:SetPlaybackRate(1.5)
self:GetCycle() -- Returns 0.0-1.0
```

### Movement
```lua
self:FollowPath(pos)
self:FaceTowards(entity)
self:FaceTo(pos)
self:Jump()
self:Leap(target_pos, speed)
```

### Combat
```lua
self:Attack({
    damage = 10,
    type = DMG_SLASH,
    range = 50
}, callback)

self:AttackEntity(entity)
self:IsAttacking()
self:SetCooldown("name", duration)
self:GetCooldown("name")
```

### Detection
```lua
self:Visible(entity)
self:IsInRange(pos, range)
self:SetSightRange(distance)
self:SetSightFOV(angle)
```

### Relationships
```lua
self:GetRelationship(entity)
self:SetRelationship(entity, disp)
self:SetDefaultRelationship(D_HT)
self:SetPlayersRelationship(D_HT)
self:JoinFaction("FACTION_NAME")
```

### Health
```lua
self:Health()
self:SetHealth(amount)
self:GetMaxHealth()
self:SetMaxHealth(amount)
self:SetGodMode(bool)
self:GetGodMode()
```

### Sounds
```lua
self:EmitSound("sound")
self:EmitSlotSound("slot", duration, "sound")
self:EmitFootstep()
```

### Utility
```lua
self:Timer(delay, callback)
self:LoopTimer(interval, callback)
self:Wait(seconds) -- In coroutines only
self:RandomPos(distance)
```

### Weapons (Human Base)
```lua
self:GiveWeapon("weapon_class")
self:RemoveWeapon()
self:GetWeapon()
self:HasWeapon()
self:PrimaryFire()
self:SecondaryFire()
self:Reload()
```

### Possession
```lua
self:Possess(player)
self:Dispossess()
self:IsPossessed()
self:GetPossessor()
self:PossessorTrace()
```

---

## Constants

### Dispositions
```lua
D_HT  -- Hate (attack)
D_LI  -- Like (friendly)
D_FR  -- Fear (flee)
D_NU  -- Neutral (ignore)
```

### Built-in Factions
```lua
FACTION_PLAYERS
FACTION_ZOMBIES
FACTION_ANTLIONS
FACTION_COMBINE
FACTION_REBELS
FACTION_PASSIVE
```

### Possession Movement Types
```lua
POSSESSION_MOVE_CUSTOM   -- Manual control
POSSESSION_MOVE_8DIR     -- 8-directional
POSSESSION_MOVE_4DIR     -- 4-directional
POSSESSION_MOVE_ANALOG   -- Analog movement
POSSESSION_MOVE_CTRL     -- Full control
```

### Input Keys
```lua
IN_ATTACK       -- Left mouse
IN_ATTACK2      -- Right mouse
IN_JUMP         -- Space
IN_DUCK         -- Ctrl
IN_FORWARD      -- W
IN_BACK         -- S
IN_MOVELEFT     -- A
IN_MOVERIGHT    -- D
IN_USE          -- E
IN_RELOAD       -- R
IN_SPEED        -- Shift
```

### Activities
```lua
ACT_IDLE
ACT_WALK
ACT_RUN
ACT_JUMP
ACT_MELEE_ATTACK1
ACT_MELEE_ATTACK2
ACT_RANGE_ATTACK1
ACT_RANGE_ATTACK2
ACT_CROUCH
ACT_FLINCH_PHYSICS
```

### Damage Types
```lua
DMG_GENERIC
DMG_CRUSH
DMG_BULLET
DMG_SLASH
DMG_BURN
DMG_VEHICLE
DMG_FALL
DMG_BLAST
DMG_CLUB
DMG_SHOCK
DMG_SONIC
DMG_ENERGYBEAM
DMG_POISON
```

---

## File Template

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Configuration
ENT.PrintName = "My NPC"
ENT.Models = {"models/model.mdl"}
ENT.SpawnHealth = 100
ENT.MeleeAttackRange = 50
ENT.Factions = {FACTION_ZOMBIES}

if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    function ENT:OnMeleeAttack(enemy)
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnAnimEvent()
        if self:IsAttacking() and self:GetCycle() > 0.3 then
            self:Attack({damage = 10})
        end
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

---

## Console Commands

```
drgbase_debug 1              -- Enable debug mode
drgbase_debug_animations 1   -- Show animation info
drgbase_draw_paths 1         -- Draw paths
drgbase_draw_ranges 1        -- Draw detection ranges
drgbase_ai_radius <num>      -- Set AI detection range
drgbase_ai_fov <num>         -- Set AI FOV

nav_generate                 -- Generate navmesh
```

---

## Lua Commands

```lua
-- Get entity
ent = Entity(1)

-- Health
ent:SetHealth(100)
ent:SetGodMode(true)

-- AI
ent:DisableAI(true)
ent:SetOmniscient(true)

-- Relationships
ent:SetEntityRelationship(Player(1), D_HT)

-- Factions
ent:JoinFaction("FACTION_CUSTOM")
PrintTable(ent:GetFactions())

-- Weapons
ent:GiveWeapon("weapon_ar2")

-- Animations
ent:PlayActivity(ACT_MELEE_ATTACK1)

-- Position
ent:SetPos(Vector(0, 0, 100))
ent:SetModelScale(2.0)
```

---

## Common Patterns

### Basic Melee NPC
```lua
ENT.MeleeAttackRange = 50
ENT.RangeAttackRange = 0

function ENT:OnMeleeAttack(enemy)
    self:EmitSound("Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({damage = 10, type = DMG_SLASH})
    end
end
```

### Basic Ranged NPC
```lua
ENT.RangeAttackRange = 500
ENT.MeleeAttackRange = 0

function ENT:OnRangeAttack(enemy)
    -- Fire projectile or hitscan
    self:PlayActivityAndMove(ACT_RANGE_ATTACK1, 1, self.FaceEnemy)
end
```

### Humanoid with Weapons
```lua
ENT.Base = "drgbase_nextbot_human"
ENT.Weapons = {"weapon_ar2"}
ENT.WeaponAccuracy = 0.75

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)
end
```

### Custom AI
```lua
function ENT:AIBehaviour()
    while true do
        if self:HasEnemy() then
            self:AttackEntity(self:GetEnemy())
        else
            self:AddPatrolPos(self:RandomPos(1500))
        end
        self:Wait(0.5)
    end
end
```

---

## See Full Documentation

- [Code Examples](../examples/) - Complete tutorials
- [Developer Tools](../tools/) - Testing tools
- [Best Practices](../guides/) - Optimization & patterns
- [Source Files](../../lua/entities/drgbase_nextbot/) - Framework code
