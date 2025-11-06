# Creating Custom NPCs

A comprehensive guide to creating custom NPCs with DrGBase, from basic setup to advanced features.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Basic NPC Setup](#basic-npc-setup)
- [Essential Properties](#essential-properties)
- [AI & Detection](#ai--detection)
- [Movement & Pathfinding](#movement--pathfinding)
- [Combat System](#combat-system)
- [Animations](#animations)
- [Sounds](#sounds)
- [Complete Examples](#complete-examples)
  - [Melee NPC: Zombie](#example-1-melee-npc-zombie)
  - [Ranged/Leaping NPC: Headcrab](#example-2-rangedleaping-npc-headcrab)
  - [Flying NPC: Scanner](#example-3-flying-npc-scanner)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Introduction

DrGBase makes creating custom NPCs straightforward by providing a powerful nextbot base entity with built-in AI, pathfinding, combat, and more. Instead of writing complex AI from scratch, you configure properties and override hooks to customize behavior.

**What you can create:**
- Melee NPCs (zombies, monsters, aggressive creatures)
- Ranged NPCs (soldiers, turrets, spitters)
- Flying NPCs (scanners, birds, drones)
- Passive NPCs (animals, citizens, followers)
- Boss NPCs with unique mechanics

**Key features:**
- Advanced AI with sight, hearing, and smell detection
- Automatic pathfinding with obstacle avoidance
- Faction system for complex relationships
- Built-in weapon support
- Possession system (control NPCs like a player)
- Coroutine-based behavior for smooth animations
- Extensive hook system for customization

## Prerequisites

Before creating NPCs, ensure you have:

1. **Garry's Mod installed** - Obviously!
2. **DrGBase installed** - Subscribe to DrGBase on Steam Workshop
3. **Basic Lua knowledge** - Understanding variables, functions, and tables
4. **Text editor** - Notepad++, VS Code, or any code editor
5. **Generated NavMesh** (optional but recommended) - For complex pathfinding, run `nav_generate` in console (single-player only)

**Recommended reading:**
- [Your First NPC](../getting-started/04-first-npc.md) - Quick start tutorial
- [Quick Start Guide](../getting-started/03-quick-start.md) - Installation basics

## Basic NPC Setup

### File Structure

Every DrGBase NPC is a Lua file in the entities folder:

```
garrysmod/addons/your_addon/
└── lua/
    └── entities/
        └── npc_your_custom_npc.lua
```

**Naming conventions:**
- Filename must start with `npc_` for proper entity recognition
- Use lowercase and underscores: `npc_custom_zombie.lua`
- The filename becomes the spawn command: `npc_custom_zombie`

### Minimal Template

Every DrGBase NPC starts with this basic structure:

```lua
-- Check if DrGBase is installed
if not DrGBase then return end

-- Entity base
ENT.Base = "drgbase_nextbot"  -- DO NOT CHANGE

-- Basic info
ENT.PrintName = "My Custom NPC"
ENT.Category = "My NPCs"

-- Model
ENT.Models = {"models/player/kleiner.mdl"}

-- Stats
ENT.SpawnHealth = 100

-- AI ranges
ENT.MeleeAttackRange = 50
ENT.RangeAttackRange = 0

-- Faction
ENT.Factions = {}

-- Server-side behavior
if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    function ENT:OnMeleeAttack(enemy)
        self:EmitSound("NPC.Attack")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnAnimEvent()
        if self:IsAttacking() and self:GetCycle() > 0.3 then
            self:Attack({
                damage = 10,
                type = DMG_SLASH
            })
        end
    end
end

-- Register (must be last)
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Critical elements:**
- `if not DrGBase then return end` - Prevents errors if DrGBase is missing
- `ENT.Base = "drgbase_nextbot"` - **Never change this!** Inherits all DrGBase features
- `AddCSLuaFile()` - Tells server to send file to clients (must be second-to-last line)
- `DrGBase.AddNextbot(ENT)` - Registers the NPC (must be last line)

## Essential Properties

### Appearance

```lua
-- Model
ENT.Models = {"models/zombie/classic.mdl"}        -- Single model
ENT.Models = {"models/zombie/classic.mdl",
              "models/zombie/classic_torso.mdl"}  -- Random selection

-- Skins
ENT.Skins = {0}           -- Single skin
ENT.Skins = {0, 1, 2}     -- Random skin selection

-- Scale
ENT.ModelScale = 1        -- Normal size
ENT.ModelScale = 1.5      -- 50% larger
ENT.ModelScale = {0.8, 1.2}  -- Random between 0.8 and 1.2

-- Collision
ENT.CollisionBounds = Vector(10, 10, 72)  -- Width X, Width Y, Height

-- Blood
ENT.BloodColor = BLOOD_COLOR_RED      -- Red blood (humans)
ENT.BloodColor = BLOOD_COLOR_GREEN    -- Green blood (aliens/zombies)
ENT.BloodColor = BLOOD_COLOR_YELLOW   -- Yellow blood
```

**Tips:**
- Use multiple models for variety in spawns
- Adjust `CollisionBounds` to match your model's size (too large = gets stuck, too small = clips through walls)
- Use `developer 1` console command to visualize collision bounds

### Health & Damage

```lua
-- Health
ENT.SpawnHealth = 100         -- Starting health
ENT.HealthRegen = 0           -- HP regenerated per second (0 = no regen)

-- Damage thresholds
ENT.MinPhysDamage = 10        -- Min physics damage to hurt NPC
ENT.MinFallDamage = 10        -- Min fall damage to hurt NPC

-- Death
ENT.RagdollOnDeath = true     -- Spawn ragdoll when killed (default: true)
```

**Examples:**

```lua
-- Weak enemy
ENT.SpawnHealth = 50
ENT.MinFallDamage = 5

-- Tank enemy
ENT.SpawnHealth = 500
ENT.HealthRegen = 1           -- Regenerates 1 HP/second
ENT.MinPhysDamage = 50        -- Resists physics damage

-- Glass cannon (high damage, low health)
ENT.SpawnHealth = 30
ENT.MinPhysDamage = 5
```

### Faction System

Factions determine friend/foe relationships:

```lua
-- Single faction
ENT.Factions = {FACTION_ZOMBIES}

-- Multiple factions
ENT.Factions = {FACTION_REBELS, FACTION_PLAYERS}

-- No faction (hostile to everyone by default)
ENT.Factions = {}
```

**Built-in factions:**
- `FACTION_PLAYERS` - Player faction
- `FACTION_REBELS` - Rebel/Resistance (Alyx, Barney)
- `FACTION_COMBINE` - Combine soldiers
- `FACTION_ZOMBIES` - Zombies and headcrabs
- `FACTION_ANTLIONS` - Antlions
- `FACTION_NEUTRAL` - Neutral faction

**Custom factions:**

```lua
-- In shared.lua or autorun file
FACTION_CUSTOM_GUARDS = DrGBase.AddFaction("Custom Guards")

-- Then in your NPC
ENT.Factions = {FACTION_CUSTOM_GUARDS}
```

**Relationship settings:**

```lua
function ENT:CustomInitialize()
    -- Hate everyone not in faction
    self:SetDefaultRelationship(D_HT)

    -- Fear everyone not in faction
    self:SetDefaultRelationship(D_FR)

    -- Neutral to everyone
    self:SetDefaultRelationship(D_NU)

    -- Like everyone
    self:SetDefaultRelationship(D_LI)

    -- Custom relationships
    self:SetRelationship(D_HT, FACTION_COMBINE)  -- Hate Combine
    self:SetRelationship(D_LI, FACTION_PLAYERS)  -- Like players
end
```

**Relationship types:**
- `D_HT` - Hate (will attack)
- `D_FR` - Fear (will flee)
- `D_LI` - Like (friendly, will help)
- `D_NU` - Neutral (ignore unless provoked)

## AI & Detection

### Sight Configuration

```lua
-- Sight
ENT.SightRange = 15000        -- Max sight distance (units)
ENT.SightFOV = 150            -- Field of view (degrees)

-- Lighting
ENT.MinLuminosity = 0         -- Min light level to see (0-1)
ENT.MaxLuminosity = 1         -- Max light level to see (0-1)

-- Eye position (for accurate sight)
ENT.EyeBone = "ValveBiped.Bip01_Head1"   -- Bone name
ENT.EyeOffset = Vector(0, 0, 0)           -- Offset from bone
ENT.EyeAngle = Angle(0, 0, 0)             -- Eye angle offset
```

**Finding eye bones:**
1. Spawn your NPC
2. Run `developer 1` in console
3. Look at NPC and note bone names
4. Common eye bones:
   - `"ValveBiped.Bip01_Head1"` - Humanoid head
   - `"ValveBiped.Bip01_Spine4"` - Upper spine (zombies)
   - `"HeadcrabClassic.SpineControl"` - Headcrab

**Sight examples:**

```lua
-- Eagle eyes (far sight, narrow FOV)
ENT.SightRange = 25000
ENT.SightFOV = 90

-- Paranoid (360° vision)
ENT.SightRange = 10000
ENT.SightFOV = 360

-- Blind (relies on hearing)
ENT.SightRange = 100
ENT.SightFOV = 10

-- Night vision only
ENT.MinLuminosity = 0
ENT.MaxLuminosity = 0.3

-- Prefers darkness
ENT.MinLuminosity = 0.7
ENT.MaxLuminosity = 1
```

### Hearing & Other Senses

```lua
-- Hearing
ENT.HearingCoefficient = 1    -- Hearing sensitivity (0 = deaf, 2 = super hearing)

-- AI behavior
ENT.BehaviourType = AI_BEHAV_BASE     -- Standard AI
ENT.Omniscient = false                 -- If true, always knows enemy location
ENT.SpotDuration = 30                  -- Seconds to remember enemy after losing sight
```

**Hearing examples:**

```lua
-- Deaf
ENT.HearingCoefficient = 0

-- Super hearing
ENT.HearingCoefficient = 2

-- Omniscient (always knows enemy location, wallhacks)
ENT.Omniscient = true
ENT.SpotDuration = math.huge
```

### AI Ranges

These ranges control AI behavior:

```lua
-- Attack ranges
ENT.MeleeAttackRange = 50       -- Range to trigger melee attack
ENT.RangeAttackRange = 500      -- Range to trigger ranged attack

-- Behavior ranges
ENT.ReachEnemyRange = 50        -- How close to get before stopping chase
ENT.AvoidEnemyRange = 0         -- Keep this distance from enemy (ranged NPCs)

-- Fear ranges (when relationship is D_FR)
ENT.AvoidAfraidOfRange = 500    -- Flee if entity is closer than this
ENT.WatchAfraidOfRange = 750    -- Watch entity from this distance
```

**Examples:**

```lua
-- Melee brawler
ENT.MeleeAttackRange = 80
ENT.RangeAttackRange = 0
ENT.ReachEnemyRange = 60
ENT.AvoidEnemyRange = 0

-- Sniper (ranged, keeps distance)
ENT.MeleeAttackRange = 0
ENT.RangeAttackRange = 3000
ENT.ReachEnemyRange = 1500
ENT.AvoidEnemyRange = 300

-- Skirmisher (ranged but can melee)
ENT.MeleeAttackRange = 60
ENT.RangeAttackRange = 800
ENT.ReachEnemyRange = 400
ENT.AvoidEnemyRange = 200

-- Coward
ENT.MeleeAttackRange = 0
ENT.RangeAttackRange = 0
ENT.AvoidAfraidOfRange = 1000
```

## Movement & Pathfinding

### Basic Movement

```lua
-- Speed
ENT.WalkSpeed = 100           -- Walking speed (patrolling)
ENT.RunSpeed = 200            -- Running speed (chasing enemies)

-- Acceleration
ENT.Acceleration = 1000       -- How fast to reach target speed
ENT.Deceleration = 1000       -- How fast to stop

-- Rotation
ENT.MaxYawRate = 250          -- Max rotation speed (degrees/second)

-- Jump/Step
ENT.JumpHeight = 50           -- Jump height (units)
ENT.StepHeight = 20           -- Max step height without jumping
ENT.DeathDropHeight = 200     -- Height that is considered fatal
```

**Movement examples:**

```lua
-- Slow tank
ENT.WalkSpeed = 50
ENT.RunSpeed = 100
ENT.Acceleration = 200
ENT.MaxYawRate = 90

-- Fast skirmisher
ENT.WalkSpeed = 150
ENT.RunSpeed = 350
ENT.Acceleration = 2000
ENT.MaxYawRate = 360

-- Sluggish zombie
ENT.WalkSpeed = 30
ENT.RunSpeed = 30
ENT.UseWalkframes = true      -- Always use walk animation
ENT.Acceleration = 400
ENT.MaxYawRate = 180
```

### Climbing System

DrGBase NPCs can climb ledges, props, and ladders:

```lua
-- Ledge climbing
ENT.ClimbLedges = false               -- Enable ledge climbing
ENT.ClimbLedgesMaxHeight = math.huge  -- Max ledge height to climb
ENT.ClimbLedgesMinHeight = 0          -- Min ledge height to climb
ENT.LedgeDetectionDistance = 20       -- How far to detect ledges

-- Prop climbing
ENT.ClimbProps = false                -- Climb over props

-- Ladder climbing
ENT.ClimbLadders = false              -- Enable ladder climbing
ENT.ClimbLaddersUp = true             -- Can climb up
ENT.ClimbLaddersUpMaxHeight = math.huge
ENT.ClimbLaddersUpMinHeight = 0
ENT.LaddersUpDistance = 20
ENT.ClimbLaddersDown = false          -- Can climb down
ENT.LaddersDownDistance = 20

-- Climb settings
ENT.ClimbSpeed = 60                   -- Climbing speed
ENT.ClimbUpAnimation = ACT_CLIMB_UP
ENT.ClimbDownAnimation = ACT_CLIMB_DOWN
ENT.ClimbAnimRate = 1
ENT.ClimbOffset = Vector(0, 0, 0)     -- Position offset while climbing
```

**Climbing examples:**

```lua
-- Agile climber
ENT.ClimbLedges = true
ENT.ClimbProps = true
ENT.ClimbLadders = true
ENT.ClimbSpeed = 100
ENT.ClimbLedgesMaxHeight = 128

-- Ground-only NPC
ENT.ClimbLedges = false
ENT.ClimbProps = false
ENT.ClimbLadders = false

-- Limited climber (small ledges only)
ENT.ClimbLedges = true
ENT.ClimbLedgesMaxHeight = 32
ENT.ClimbProps = false
```

### Pathfinding Functions

Use these functions to control NPC movement:

```lua
function ENT:CustomThink()
    -- Follow a target entity
    self:FollowPath(enemy)

    -- Go to a position
    self:FollowPath(Vector(0, 0, 0))

    -- Move away from position
    self:FollowPath(self:GetPos():DrG_Away(enemy:GetPos()))

    -- Stop moving
    self:StopMoving()

    -- Face target
    self:FaceTowards(enemy)

    -- Check if moving
    if self:IsMoving() then
        -- Do something
    end

    -- Check if path is complete
    local result = self:FollowPath(target)
    if result == "reached" then
        -- Reached destination
    elseif result == "unreachable" then
        -- Can't reach destination
    end
end
```

**Advanced pathfinding:**

```lua
function ENT:OnChaseEnemy(enemy)
    -- Override chase behavior
    local result = self:FollowPath(enemy)

    if result == "unreachable" then
        -- Try alternative path
        local pos = enemy:GetPos() + enemy:GetForward() * 100
        self:FollowPath(pos)
    end

    return true  -- Don't run default behavior
end
```

## Combat System

### Melee Combat

```lua
ENT.MeleeAttackRange = 50     -- Trigger melee when enemy is this close

function ENT:OnMeleeAttack(enemy)
    -- Play attack sound
    self:EmitSound("NPC.Attack")

    -- Play attack animation
    -- Parameters: animation, speed multiplier, callback during animation
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    -- Deal damage at right point in animation
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({
            damage = 15,
            type = DMG_SLASH,
            viewpunch = Angle(20, math.random(-10, 10), 0)  -- Camera shake
        }, function(self, hit)
            -- Callback after attack
            if #hit > 0 then
                self:EmitSound("NPC.AttackHit")
            else
                self:EmitSound("NPC.AttackMiss")
            end
        end)
    end
end
```

**Damage types:**
- `DMG_SLASH` - Slash/cut damage
- `DMG_CLUB` - Blunt damage
- `DMG_CRUSH` - Crushing damage
- `DMG_BULLET` - Bullet damage
- `DMG_BLAST` - Explosive damage
- `DMG_BURN` - Fire damage
- `DMG_POISON` - Poison damage
- `DMG_ACID` - Acid damage

**Advanced melee:**

```lua
function ENT:OnMeleeAttack(enemy)
    -- Random attack animations
    local attacks = {ACT_MELEE_ATTACK1, ACT_MELEE_ATTACK2}
    local anim = attacks[math.random(#attacks)]

    self:EmitSound("NPC.Attack")
    self:PlayActivityAndMove(anim, 1.5, self.FaceEnemy)  -- 1.5x speed
end

function ENT:OnAnimEvent()
    if self:IsAttacking() then
        local cycle = self:GetCycle()

        -- Two-hit combo (at 30% and 70% of animation)
        if cycle > 0.3 and not self.FirstHit then
            self.FirstHit = true
            self:Attack({damage = 10, type = DMG_SLASH})
        elseif cycle > 0.7 and not self.SecondHit then
            self.SecondHit = true
            self:Attack({damage = 15, type = DMG_SLASH})
        end

        -- Reset flags when animation ends
        if cycle > 0.95 then
            self.FirstHit = false
            self.SecondHit = false
        end
    end
end
```

### Ranged Combat

```lua
ENT.RangeAttackRange = 1000   -- Trigger ranged attack when enemy is this close

function ENT:OnRangeAttack(enemy)
    -- Face target
    self:FaceTo(enemy:EyePos())

    -- Play attack animation
    self:PlayActivityAndMove(ACT_RANGE_ATTACK1, 1, self.FaceEnemy)

    -- Fire projectile
    local bullet = ents.Create("grenade_spit")
    if IsValid(bullet) then
        bullet:SetPos(self:EyePos())
        bullet:SetAngles(self:GetAngles())
        bullet:Spawn()
        bullet:SetVelocity((enemy:EyePos() - self:EyePos()):GetNormalized() * 1000)
        bullet:SetOwner(self)
    end

    -- Wait before next attack
    self:Wait(2)
end
```

**Hitscan ranged attack:**

```lua
function ENT:OnRangeAttack(enemy)
    self:FaceTowards(enemy)
    self:PlayActivityAndMove(ACT_RANGE_ATTACK1, 1, self.FaceEnemy)

    -- Wait for animation to reach firing point
    self:PauseCoroutine(0.3)

    -- Fire hitscan bullet
    local bullet = {}
    bullet.Num = 1
    bullet.Src = self:EyePos()
    bullet.Dir = (enemy:EyePos() - self:EyePos()):GetNormalized()
    bullet.Spread = Vector(0.02, 0.02, 0)
    bullet.Tracer = 1
    bullet.TracerName = "Tracer"
    bullet.Force = 5
    bullet.Damage = 10
    bullet.AmmoType = "Pistol"
    bullet.Attacker = self

    self:FireBullets(bullet)
    self:EmitSound("Weapon_Pistol.Single")

    self:Wait(1.5)  -- Cooldown
end
```

**Projectile helper:**

```lua
function ENT:FireProjectile(target, speed, damage)
    local projectile = ents.Create("prop_combine_ball")
    if not IsValid(projectile) then return end

    projectile:SetPos(self:EyePos())
    projectile:Spawn()

    local phys = projectile:GetPhysicsObject()
    if IsValid(phys) then
        local dir = (target - self:EyePos()):GetNormalized()
        phys:SetVelocity(dir * speed)
    end

    projectile:SetOwner(self)
    return projectile
end
```

### Weapon Support

DrGBase NPCs can use weapons:

```lua
ENT.UseWeapons = true
ENT.Weapons = {"weapon_pistol", "weapon_smg1", "weapon_ar2"}
ENT.DropWeaponOnDeath = true
ENT.AcceptPlayerWeapons = true

function ENT:CustomInitialize()
    -- Give specific weapon
    self:GiveWeapon("weapon_shotgun")

    -- Or random from list
    local weapon = self.Weapons[math.random(#self.Weapons)]
    self:GiveWeapon(weapon)
end

-- Weapon functions
function ENT:OnPickupWeapon(weapon, class)
    -- Called when picking up weapon
end

function ENT:OnDropWeapon(weapon, class)
    -- Called when dropping weapon
end

-- Combat with weapons is handled automatically!
```

## Animations

### Animation Properties

```lua
-- Basic animations
ENT.IdleAnimation = ACT_IDLE
ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_RUN
ENT.JumpAnimation = ACT_JUMP

-- Animation rates (speed multipliers)
ENT.IdleAnimRate = 1
ENT.WalkAnimRate = 1
ENT.RunAnimRate = 1
ENT.JumpAnimRate = 1

-- Special settings
ENT.UseWalkframes = false     -- Use walk animation for running (zombies)
```

**Common animation activities:**
- `ACT_IDLE` - Standing idle
- `ACT_WALK` - Walking
- `ACT_RUN` - Running
- `ACT_JUMP` - Jumping
- `ACT_MELEE_ATTACK1` - Melee attack
- `ACT_RANGE_ATTACK1` - Ranged attack
- `ACT_CLIMB_UP` - Climbing up
- `ACT_CLIMB_DOWN` - Climbing down
- `ACT_FLINCH_PHYSICS` - Hit by physics
- `ACT_DIERAGDOLL` - Death animation

### Playing Animations

```lua
-- Play animation and continue
self:PlayAnimation(ACT_GESTURE_FLINCH_HEAD)

-- Play animation and wait
self:PlaySequence("taunt_laugh")

-- Play animation while moving
self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)

-- Play animation with custom speed
self:PlayActivityAndMove(ACT_RUN, 1.5)  -- 50% faster

-- Check if animation is playing
if self:GetActivity() == ACT_MELEE_ATTACK1 then
    -- Attack animation playing
end

-- Get animation progress (0-1)
local progress = self:GetCycle()
```

**Custom animation sequences:**

```lua
function ENT:CustomInitialize()
    -- Find animation sequence by name
    local sequence = self:LookupSequence("scene_walk_01")
    if sequence > 0 then
        self.CustomWalkSequence = sequence
    end
end

function ENT:OnIdle()
    -- Play custom sequence
    if self.CustomWalkSequence then
        self:PlaySequence(self.CustomWalkSequence)
    end
end
```

## Sounds

### Sound Properties

```lua
-- Spawn sounds
ENT.OnSpawnSounds = {"NPC.Spawn"}

-- Idle sounds (looping)
ENT.OnIdleSounds = {"NPC.Idle1", "NPC.Idle2", "NPC.Idle3"}
ENT.IdleSoundDelay = 2        -- Delay between idle sounds (seconds)
ENT.ClientIdleSounds = false  -- Play on client instead of server

-- Damage sounds
ENT.OnDamageSounds = {"NPC.Pain1", "NPC.Pain2"}
ENT.DamageSoundDelay = 0.25   -- Minimum time between damage sounds

-- Death sounds
ENT.OnDeathSounds = {"NPC.Die"}

-- Downed sounds (when disabled but alive)
ENT.OnDownedSounds = {"NPC.Downed"}

-- Footsteps
ENT.Footsteps = {"NPC.FootstepLeft", "NPC.FootstepRight"}
```

**Sound examples:**

```lua
-- Zombie
ENT.OnSpawnSounds = {}
ENT.OnIdleSounds = {"npc_zombie.idle"}
ENT.IdleSoundDelay = 3
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- Headcrab
ENT.OnIdleSounds = {"NPC_HeadCrab.Idle"}
ENT.OnDamageSounds = {"NPC_HeadCrab.Pain"}
ENT.OnDeathSounds = {"NPC_HeadCrab.Die"}

-- Combine soldier
ENT.OnDamageSounds = {
    "npc/combine_soldier/pain1.wav",
    "npc/combine_soldier/pain2.wav",
    "npc/combine_soldier/pain3.wav"
}
```

### Playing Sounds

```lua
-- Play sound directly
self:EmitSound("NPC.Alert")

-- Play with parameters
self:EmitSound("NPC.Roar", 100, 100)  -- Volume, pitch

-- Play sound at specific position
sound.Play("NPC.Footstep", self:GetPos(), 75, 100)

-- Stop sound
self:StopSound("NPC.Idle")

-- Play sound with variations
local sounds = {"NPC.Pain1", "NPC.Pain2", "NPC.Pain3"}
self:EmitSound(sounds[math.random(#sounds)])
```

### Sound Hooks

```lua
function ENT:OnNewEnemy(enemy)
    -- Alerted to new enemy
    self:EmitSound("NPC.Alert")
end

function ENT:OnTakeDamage(dmg)
    -- Damage sounds handled automatically
    -- But you can add custom logic
    if self:Health() < self:GetMaxHealth() * 0.3 then
        self:EmitSound("NPC.LowHealth")
    end
end

function ENT:OnDeath(dmg, hitgroup)
    -- Death sounds handled automatically
    -- Add death roar
    self:EmitSound("NPC.DeathRoar")
end
```

## Complete Examples

### Example 1: Melee NPC (Zombie)

A complete zombie NPC with melee attacks, headcrab spawning, and idle behavior:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc
ENT.PrintName = "Zombie"
ENT.Category = "DrGBase"
ENT.Models = {"models/Zombie/Classic.mdl"}
ENT.BloodColor = BLOOD_COLOR_GREEN

-- Sounds
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- Stats
ENT.SpawnHealth = 100

-- AI
ENT.RangeAttackRange = 0
ENT.MeleeAttackRange = 30
ENT.ReachEnemyRange = 30
ENT.AvoidEnemyRange = 0

-- Relationships
ENT.Factions = {FACTION_ZOMBIES}

-- Movements/animations
ENT.UseWalkframes = true
ENT.RunAnimation = ACT_WALK

-- Detection
ENT.EyeBone = "ValveBiped.Bip01_Spine4"
ENT.EyeOffset = Vector(7.5, 0, 5)

-- Possession
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
    {
        offset = Vector(0, 30, 20),
        distance = 100
    },
    {
        offset = Vector(7.5, 0, 0),
        distance = 0,
        eyepos = true
    }
}
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:EmitSound("Zombie.Attack")
            self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.PossessionFaceForward)
        end
    }}
}

if SERVER then

    -- Init/Think

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
        self:SetBodygroup(1, 1)  -- Hide headcrab initially
    end

    -- AI

    function ENT:OnMeleeAttack(enemy)
        self:EmitSound("Zombie.Attack")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(3, 7))
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end

    -- Damage

    function ENT:OnDeath(dmg, delay, hitgroup)
        -- Spawn headcrab unless killed by headshot
        if hitgroup ~= HITGROUP_HEAD then
            self:SetBodygroup(1, 0)  -- Show headcrab hole
            local headcrab = ents.Create("npc_drg_headcrab")
            if not IsValid(headcrab) then return end
            headcrab:SetPos(self:EyePos())
            headcrab:SetAngles(self:GetAngles())
            headcrab:Spawn()
            if IsValid(self:GetCreator()) then
                self:GetCreator():DrG_AddUndo(headcrab, "NPC", "Undone Headcrab")
            end
        end
    end

    -- Animations/Sounds

    function ENT:OnNewEnemy()
        self:EmitSound("Zombie.Alert")
    end

    function ENT:OnAnimEvent()
        -- Deal damage during attack animation
        if self:IsAttacking() and self:GetCycle() > 0.3 then
            self:Attack({
                damage = 10,
                type = DMG_SLASH,
                viewpunch = Angle(20, math.random(-10, 10), 0)
            }, function(self, hit)
                if #hit > 0 then
                    self:EmitSound("Zombie.AttackHit")
                else
                    self:EmitSound("Zombie.AttackMiss")
                end
            end)
        -- Play footstep sounds
        elseif math.random(2) == 1 then
            self:EmitSound("Zombie.FootstepLeft")
        else
            self:EmitSound("Zombie.FootstepRight")
        end
    end

end

-- DO NOT TOUCH
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Key features:**
- Slow shambling movement (`UseWalkframes = true`)
- Spawns headcrab on death (unless headshot)
- Random patrol behavior when idle
- Possession support for player control
- Footstep sounds in `OnAnimEvent`

### Example 2: Ranged/Leaping NPC (Headcrab)

A headcrab that leaps at enemies:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc
ENT.PrintName = "Headcrab"
ENT.Category = "DrGBase"
ENT.Models = {"models/headcrabclassic.mdl"}
ENT.CollisionBounds = Vector(12, 12, 24)
ENT.BloodColor = BLOOD_COLOR_GREEN
ENT.VJ_ID_Headcrab = true

-- Stats
ENT.SpawnHealth = 40

-- Sounds
ENT.OnIdleSounds = {"NPC_HeadCrab.Idle"}
ENT.OnDamageSounds = {"NPC_HeadCrab.Pain"}
ENT.OnDeathSounds = {"NPC_HeadCrab.Die"}

-- AI
ENT.RangeAttackRange = 150
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 125
ENT.AvoidEnemyRange = 100

-- Relationships
ENT.Factions = {FACTION_ZOMBIES}

-- Animations
ENT.WalkAnimation = ACT_RUN
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE
ENT.JumpAnimation = ACT_IDLE

-- Movements
ENT.UseWalkframes = true

-- Detection
ENT.EyeBone = "HeadcrabClassic.SpineControl"
ENT.EyeOffset = Vector(4, 0, 0)

-- Possession
ENT.PossessionEnabled = true
ENT.PossessionCrosshair = true
ENT.PossessionViews = {
    {
        offset = Vector(0, 10, 10),
        distance = 50
    },
    {
        offset = Vector(7.5, 0, 0),
        distance = 0,
        eyepos = true
    }
}
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:HeadcrabLeap(self:PossessorTrace().HitPos)
        end
    }}
}

if SERVER then

    -- Headcrab

    function ENT:HeadcrabLeap(pos)
        self:FaceTo(pos)
        self.OnIdleSounds = {}
        self:PlaySequence("jumpattack_broadcast")
        self:PauseCoroutine(0.5)
        self.CanBite = true
        self:EmitSound("NPC_Headcrab.Attack")
        self:Leap(pos, 400)
        self.CanBite = false
        self.OnIdleSounds = {"NPC_HeadCrab.Idle"}
    end

    -- Init/Think

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    function ENT:CustomThink() end

    -- AI

    function ENT:OnRangeAttack(enemy)
        self:HeadcrabLeap(enemy:EyePos()-Vector(0, 0, 10))
    end

    function ENT:OnContact(ent)
        if not IsValid(ent) then return end
        if self.CanBite and
        (self:IsPossessed() or ent == self:GetEnemy()) then
            self:EmitSound("NPC_HeadCrab.Bite")
            self.CanBite = false
            local dmg = DamageInfo()
            dmg:SetDamage(20)
            dmg:SetAttacker(self)
            dmg:SetInflictor(self)
            dmg:SetDamageType(DMG_SLASH)
            ent:TakeDamageInfo(dmg)
        end
    end

    function ENT:OnReachedPatrol(pos)
        self:Wait(math.random(3, 7))
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end

    -- Damage

    function ENT:OnTakeDamage(dmg)
        -- Crowbar deals double damage
        local attacker = dmg:GetAttacker()
        if IsValid(attacker) and attacker:IsPlayer() then
            local weapon = attacker:GetActiveWeapon()
            if IsValid(weapon) and weapon:GetClass() == "weapon_crowbar" then
                return 2  -- 2x damage multiplier
            end
        end
    end

    -- Sounds

    function ENT:OnNewEnemy()
        self:EmitSound("NPC_HeadCrab.Alert")
    end

end

-- DO NOT TOUCH
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Key features:**
- Uses "range attack" for leap (not true ranged attack)
- `HeadcrabLeap()` custom function for leap behavior
- Deals damage on contact during leap
- Keeps distance from enemies (`AvoidEnemyRange = 100`)
- Crowbar weakness (2x damage)
- Small collision bounds for tiny creature

### Example 3: Flying NPC (Scanner)

A flying scanner that hovers and takes photos:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc
ENT.PrintName = "Scanner"
ENT.Category = "DrGBase Examples"
ENT.Models = {"models/shield_scanner.mdl"}
ENT.CollisionBounds = Vector(16, 16, 16)
ENT.BloodColor = BLOOD_COLOR_MECH

-- Stats
ENT.SpawnHealth = 60

-- Sounds
ENT.OnIdleSounds = {"npc/scanner/scanner_talk1.wav", "npc/scanner/scanner_talk2.wav"}
ENT.IdleSoundDelay = 5
ENT.OnDamageSounds = {"npc/scanner/scanner_pain1.wav", "npc/scanner/scanner_pain2.wav"}
ENT.OnDeathSounds = {"npc/scanner/scanner_explode_crash2.wav"}

-- AI
ENT.RangeAttackRange = 600
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 400
ENT.AvoidEnemyRange = 200

-- Relationships
ENT.Factions = {FACTION_COMBINE}

-- Movements (flying)
ENT.WalkSpeed = 100
ENT.RunSpeed = 250
ENT.Acceleration = 600
ENT.JumpHeight = 0
ENT.StepHeight = 0

-- Detection
ENT.EyeBone = "Scanner.Body"
ENT.SightRange = 5000
ENT.SightFOV = 180

-- Animations (minimal for flying NPCs)
ENT.IdleAnimation = -1
ENT.WalkAnimation = -1
ENT.RunAnimation = -1

if SERVER then

    -- Init

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_NU)  -- Neutral (non-aggressive)

        -- Flying setup
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            phys:EnableGravity(false)
            phys:SetMass(20)
        end

        -- Start hovering
        self.HoverHeight = 100
        self.HoverTimer = 0
    end

    -- Flying behavior

    function ENT:CustomThink()
        -- Hover up and down
        local pos = self:GetPos()
        local groundPos = util.TraceLine({
            start = pos,
            endpos = pos - Vector(0, 0, 10000),
            filter = self
        }).HitPos

        local targetHeight = groundPos.z + self.HoverHeight
        local currentHeight = pos.z

        -- Apply hover force
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            if currentHeight < targetHeight - 5 then
                phys:AddVelocity(Vector(0, 0, 5))
            elseif currentHeight > targetHeight + 5 then
                phys:AddVelocity(Vector(0, 0, -5))
            end
        end

        -- Gentle bobbing
        self.HoverTimer = self.HoverTimer + 0.1
        local bob = math.sin(self.HoverTimer) * 2
        self:SetPos(pos + Vector(0, 0, bob * FrameTime()))

        return 0.05  -- Think every 0.05 seconds
    end

    -- Movement overrides for 3D movement

    function ENT:OnChaseEnemy(enemy)
        -- Fly towards enemy
        local targetPos = enemy:EyePos() + Vector(0, 0, 50)

        -- Move in 3D space
        local dir = (targetPos - self:GetPos()):GetNormalized()
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(dir * self.RunSpeed)
        end

        self:FaceTowards(enemy)
        return true  -- Override default chase
    end

    function ENT:OnAvoidEnemy(enemy)
        -- Fly away from enemy
        local awayPos = self:GetPos() + (self:GetPos() - enemy:GetPos()):GetNormalized() * 200
        awayPos.z = awayPos.z + 50  -- Fly up

        local dir = (awayPos - self:GetPos()):GetNormalized()
        local phys = self:GetPhysicsObject()
        if IsValid(phys) then
            phys:SetVelocity(dir * self.RunSpeed)
        end

        return true
    end

    -- AI

    function ENT:OnRangeAttack(enemy)
        -- Take a photo
        self:FaceTowards(enemy)
        self:EmitSound("npc/scanner/scanner_photo1.wav")

        -- Flash effect
        local flash = EffectData()
        flash:SetOrigin(self:GetPos())
        flash:SetScale(1)
        util.Effect("ManhackSparks", flash)

        -- Wait between photos
        self:Wait(3)
    end

    function ENT:OnIdle()
        -- Fly to random positions
        local randomPos = self:RandomPos(1000)
        randomPos.z = randomPos.z + math.random(50, 150)  -- Fly at height
        self:AddPatrolPos(randomPos)
    end

    function ENT:OnReachedPatrol()
        -- Hover at patrol point
        self:Wait(math.random(2, 5))
    end

    -- Death

    function ENT:OnDeath(dmg, hitgroup)
        -- Explosion effect
        local explosion = EffectData()
        explosion:SetOrigin(self:GetPos())
        explosion:SetScale(1)
        util.Effect("Explosion", explosion)

        -- Sparks
        local sparks = EffectData()
        sparks:SetOrigin(self:GetPos())
        sparks:SetMagnitude(2)
        sparks:SetScale(2)
        util.Effect("ElectricSpark", sparks)
    end

    function ENT:OnNewEnemy(enemy)
        self:EmitSound("npc/scanner/scanner_alert1.wav")
    end

end

-- DO NOT TOUCH
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Key features:**
- Physics-based flying with hover system
- 3D pathfinding (overrides default ground pathfinding)
- Gentle bobbing motion for realism
- Non-aggressive (takes photos instead of attacking)
- Custom death effects
- No animations (uses physics orientation)

**Flying NPC tips:**
- Disable gravity: `phys:EnableGravity(false)`
- Override chase/avoid functions for 3D movement
- Use `phys:SetVelocity()` instead of pathfinding
- Add patrol points at elevated heights
- Consider using `ENT.ClimbLedges = false` since they fly

## Advanced Features

### Coroutines & Timing

DrGBase uses coroutines for smooth, non-blocking behavior:

```lua
function ENT:OnMeleeAttack(enemy)
    -- Play animation
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)

    -- Wait for animation to finish (non-blocking)
    self:PauseCoroutine(1.0)

    -- Do something after animation
    self:EmitSound("NPC.Growl")
end

-- Advanced timing
function ENT:CustomBehavior()
    -- Wait 2 seconds
    self:Wait(2)

    -- Pause coroutine for exact time
    self:PauseCoroutine(0.5)

    -- Pause until resumed
    self:PauseCoroutine()

    -- Resume from elsewhere
    self:ResumeCoroutine()
end

-- Call functions in coroutine
function ENT:OnNewEnemy(enemy)
    self:CallInCoroutine(function(self, elapsed)
        -- This runs in coroutine context
        print("Enemy detected " .. elapsed .. " seconds ago")
    end)
end
```

### Patrol System

Create complex patrol routes:

```lua
function ENT:OnIdle()
    -- Simple: random nearby position
    self:AddPatrolPos(self:RandomPos(1000))

    -- Add multiple patrol points
    self:AddPatrolPos(Vector(0, 0, 0))
    self:AddPatrolPos(Vector(500, 0, 0))
    self:AddPatrolPos(Vector(500, 500, 0))

    -- Check if has patrol
    if self:HasPatrol() then
        local patrol = self:GetPatrol()
        -- Do something
    end
end

function ENT:OnReachedPatrol(pos, patrol)
    -- Called when reaching patrol point
    self:Wait(math.random(2, 5))  -- Wait before next point
end

function ENT:OnPatrolUnreachable(pos, patrol)
    -- Called if can't reach patrol point
    -- Try alternative
    self:AddPatrolPos(self:RandomPos(500))
end
```

### Custom AI Behavior

Override the entire AI system:

```lua
ENT.BehaviourType = AI_BEHAV_CUSTOM

function ENT:AIBehaviour()
    while true do
        -- Your custom AI loop

        if self:HasEnemy() then
            local enemy = self:GetEnemy()

            -- Custom combat behavior
            if self:IsInRange(enemy, 200) then
                -- Close range: melee
                self:OnMeleeAttack(enemy)
                self:Wait(2)
            elseif self:IsInRange(enemy, 1000) then
                -- Mid range: shoot
                self:OnRangeAttack(enemy)
                self:Wait(1)
            else
                -- Far range: chase
                self:FollowPath(enemy)
            end
        else
            -- No enemy: patrol
            if not self:HasPatrol() then
                self:AddPatrolPos(self:RandomPos(1500))
            end
            self:Patrol()
        end

        -- Yield to let other code run
        self:YieldCoroutine(true)
    end
end
```

### Custom Hooks

Override many hooks for custom behavior:

```lua
-- Lifecycle hooks
function ENT:CustomInitialize()
    -- Called when spawned
end

function ENT:CustomThink()
    -- Called every think interval
    return 0.1  -- Return delay for next think
end

function ENT:OnSpawn()
    -- Called once when first spawned
end

function ENT:OnDeath(dmg, hitgroup)
    -- Called when killed
end

function ENT:OnRemove()
    -- Called when removed (not just killed)
end

-- AI hooks
function ENT:OnNewEnemy(enemy)
    -- New enemy detected
end

function ENT:OnLostEnemy(enemy)
    -- Lost sight of enemy
end

function ENT:OnChaseEnemy(enemy)
    -- Chasing enemy
    -- Return true to override default
end

function ENT:OnAvoidEnemy(enemy)
    -- Keeping distance from enemy
    -- Return true to override default
end

function ENT:OnIdleEnemy(enemy)
    -- Within reach of enemy, not moving
    -- Return true to override default
end

function ENT:OnEnemyUnreachable(enemy)
    -- Can't path to enemy
end

-- Relationship hooks (when enemy has D_FR relationship to us)
function ENT:OnAllyEnemy(enemy)
    -- Ally detected our enemy
end

function ENT:OnNeutralEnemy(enemy)
    -- Neutral entity detected our enemy
end

function ENT:OnAvoidAfraidOf(enemy)
    -- Running from entity we fear
end

function ENT:OnIdleAfraidOf(enemy)
    -- At safe distance from feared entity
end

-- Combat hooks
function ENT:OnMeleeAttack(enemy, weapon)
    -- Trigger melee attack
end

function ENT:OnRangeAttack(enemy, weapon)
    -- Trigger ranged attack
end

function ENT:OnTakeDamage(dmg)
    -- Taking damage
    -- Return false to block damage
    -- Return number to multiply damage
    -- Return true or nothing for normal damage
end

function ENT:OnKilledEnemy(enemy, dmg)
    -- We killed an enemy
end

-- Movement hooks
function ENT:OnContact(ent)
    -- Collided with entity
end

function ENT:OnLanded()
    -- Landed after jump/fall
end

function ENT:OnLeap(pos)
    -- Started a leap
end

function ENT:OnClimbStart(pos)
    -- Started climbing
end

function ENT:OnClimbEnd()
    -- Finished climbing
end

-- Animation hooks
function ENT:OnAnimEvent()
    -- Called every animation frame
end

-- Misc hooks
function ENT:OnHealthChange(old, new)
    -- Health changed
end

function ENT:OnExtinguish()
    -- Fire was extinguished
end

function ENT:OnWaterLevelChange(old, new)
    -- Water level changed
end
```

### Downed State

NPCs can be disabled but not killed:

```lua
function ENT:OnTakeDamage(dmg)
    if self:Health() - dmg:GetDamage() <= 0 then
        -- Don't die, go to downed state
        self:SetHealth(1)
        self:DisableAI()
        self:PlayAnimation(ACT_DIERAGDOLL)
        self:EmitSound("NPC.Downed")

        -- Revive after 10 seconds
        self:Timer(10, function()
            self:SetHealth(self:GetMaxHealth())
            self:EnableAI()
            self:PlayAnimation(ACT_IDLE)
        end)

        return false  -- Block damage that would kill
    end
end
```

### Scaling

Scale NPC size and stats:

```lua
function ENT:CustomInitialize()
    -- Random size
    local scale = math.random(80, 120) / 100
    self:SetModelScale(scale)

    -- Scale health with size
    local health = self.SpawnHealth * scale
    self:SetHealth(health)
    self:SetMaxHealth(health)

    -- Scale speed
    self:SetRunSpeed(self.RunSpeed * scale)
    self:SetWalkSpeed(self.WalkSpeed * scale)
end

-- Or use built-in property
ENT.ModelScale = {0.8, 1.2}  -- Random scale on spawn
```

### Bodygroups & Skins

Customize NPC appearance:

```lua
function ENT:CustomInitialize()
    -- Set bodygroup
    self:SetBodygroup(0, 1)

    -- Random bodygroup
    local variants = self:GetBodygroupCount(1)
    self:SetBodygroup(1, math.random(0, variants - 1))

    -- Random skin
    self:SetSkin(math.random(0, self:SkinCount() - 1))
end

-- Or use property for automatic random
ENT.Skins = {0, 1, 2, 3}
```

### Timers

Create timers for delayed actions:

```lua
function ENT:CustomInitialize()
    -- Simple timer
    self:Timer(5, function()
        self:EmitSound("NPC.Taunt")
    end)

    -- Repeating timer
    self:Timer("roar_loop", 10, 0, function()
        self:EmitSound("NPC.Roar")
    end)

    -- Remove timer
    self:RemoveTimer("roar_loop")
end
```

### Possession System

Let players control your NPC:

```lua
-- Enable possession
ENT.PossessionEnabled = true
ENT.PossessionPrompt = true           -- Show "Press E to possess" prompt
ENT.PossessionCrosshair = true        -- Show crosshair when possessed
ENT.PossessionMovement = POSSESSION_MOVE_8DIR  -- 8-directional movement

-- Camera views
ENT.PossessionViews = {
    {
        offset = Vector(0, 30, 20),   -- Third person
        distance = 100
    },
    {
        offset = Vector(0, 0, 0),     -- First person
        distance = 0,
        eyepos = true
    }
}

-- Control bindings
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            -- Left click action
            self:OnMeleeAttack(self:GetEnemy())
        end
    }},
    [IN_ATTACK2] = {{
        coroutine = true,
        onkeydown = function(self)
            -- Right click action
            self:OnRangeAttack(self:GetEnemy())
        end
    }},
    [IN_RELOAD] = {{
        onkeydown = function(self)
            -- R key action
            self:EmitSound("NPC.Taunt")
        end
    }}
}

-- Possession hooks
function ENT:OnPossess(ply)
    -- Player started possessing
end

function ENT:OnDispossess(ply)
    -- Player stopped possessing
end

function ENT:PossessionThink(possessor)
    -- Called while possessed
    return 0.1  -- Think delay
end
```

**Movement modes:**
- `POSSESSION_MOVE_1DIR` - Forward only (like a car)
- `POSSESSION_MOVE_8DIR` - 8-directional (WASD)
- `POSSESSION_MOVE_FREE` - Free movement (strafe, backpedal)

### Weapon Drops

```lua
ENT.UseWeapons = true
ENT.DropWeaponOnDeath = true

function ENT:OnDeath(dmg, hitgroup)
    -- Drop custom item
    local item = ents.Create("item_healthkit")
    item:SetPos(self:GetPos() + Vector(0, 0, 20))
    item:Spawn()

    -- Or spawn weapon
    local weapon = ents.Create("weapon_smg1")
    weapon:SetPos(self:GetPos() + Vector(0, 0, 10))
    weapon:Spawn()
end
```

## Best Practices

### Performance

```lua
-- Good: Use appropriate sight range
ENT.SightRange = 5000

-- Bad: Excessive sight range causes lag
ENT.SightRange = 50000

-- Good: Reasonable think intervals
function ENT:CustomThink()
    -- Expensive operations
    return 0.5  -- Only think every 0.5 seconds
end

-- Bad: Think every frame (default)
function ENT:CustomThink()
    -- Expensive operations
    -- No return = thinks every frame
end

-- Good: Cache expensive lookups
function ENT:CustomInitialize()
    self.CachedEyeBone = self:LookupBone(self.EyeBone)
end

-- Good: Use timers for periodic actions
self:Timer(1, function()
    -- Check something every second
end)
```

### Code Organization

```lua
-- Good: Organize by functionality
if SERVER then
    -- ========== Initialization ==========

    function ENT:CustomInitialize()
        -- Setup
    end

    -- ========== AI Behavior ==========

    function ENT:OnMeleeAttack(enemy)
        -- Attack logic
    end

    function ENT:OnRangeAttack(enemy)
        -- Ranged logic
    end

    -- ========== Damage & Death ==========

    function ENT:OnTakeDamage(dmg)
        -- Damage logic
    end

    function ENT:OnDeath(dmg, hitgroup)
        -- Death logic
    end

    -- ========== Custom Functions ==========

    function ENT:CustomFunction()
        -- Your functions
    end
end
```

### Debugging

```lua
-- Enable developer mode
developer 1

-- Display debug info
drgbase_display_collisions 1  -- Show collision bounds
drgbase_display_sight 1        -- Show sight lines

-- Print debug info
function ENT:CustomThink()
    if GetConVar("developer"):GetBool() then
        print("NPC State:", self:GetActivity(), "Enemy:", self:GetEnemy())
    end
    return 0.5
end

-- Error handling
function ENT:OnError(err)
    print("Error in NPC:", self, err)
    return true  -- Return true to restart behavior thread
end
```

### Testing Checklist

When creating NPCs, test these aspects:

**Basic Functionality:**
- [ ] Spawns without errors
- [ ] Correct model and appearance
- [ ] Collision bounds appropriate
- [ ] Animations play correctly

**AI & Detection:**
- [ ] Detects enemies in sight range
- [ ] Respects field of view
- [ ] Hears sounds appropriately
- [ ] Remembers enemies correctly

**Movement:**
- [ ] Walks/runs at correct speed
- [ ] Follows paths properly
- [ ] Climbs obstacles if enabled
- [ ] Doesn't get stuck

**Combat:**
- [ ] Attacks at correct range
- [ ] Deals appropriate damage
- [ ] Attack animations play
- [ ] Attack sounds play

**Relationships:**
- [ ] Correct faction behavior
- [ ] Attacks enemies
- [ ] Friendly to allies
- [ ] Neutral behavior works

**Health & Death:**
- [ ] Takes damage correctly
- [ ] Dies at 0 health
- [ ] Death sounds play
- [ ] Ragdoll spawns

**Performance:**
- [ ] No lag with multiple NPCs
- [ ] No console errors
- [ ] Think functions optimized

### Common Mistakes

```lua
-- Mistake: Wrong base
ENT.Base = "base_nextbot"  -- WRONG
ENT.Base = "drgbase_nextbot"  -- CORRECT

-- Mistake: Forgetting SERVER check
function ENT:OnMeleeAttack(enemy)  -- WRONG: runs on client too
end

if SERVER then  -- CORRECT
    function ENT:OnMeleeAttack(enemy)
    end
end

-- Mistake: Not calling attack in OnAnimEvent
function ENT:OnMeleeAttack(enemy)
    self:Attack({damage = 10})  -- WRONG: instant attack
end

function ENT:OnMeleeAttack(enemy)
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({damage = 10})  -- CORRECT: timed with animation
    end
end

-- Mistake: Blocking all damage
function ENT:OnTakeDamage(dmg)
    return false  -- WRONG: invincible
end

function ENT:OnTakeDamage(dmg)
    return true  -- CORRECT: takes damage
end

-- Mistake: Not registering entity
-- (missing AddCSLuaFile and DrGBase.AddNextbot)

-- Mistake: Wrong property names
ENT.MoveSpeed = 200  -- WRONG: doesn't exist
ENT.RunSpeed = 200   -- CORRECT

ENT.VisionRange = 5000  -- WRONG: doesn't exist
ENT.SightRange = 5000   -- CORRECT
```

### Optimization Tips

```lua
-- Cache entity lookups
function ENT:CustomInitialize()
    self.CachedTarget = self:FindBestTarget()
end

-- Use appropriate think delays
function ENT:CustomThink()
    -- Expensive operation
    return 1  -- Only run once per second
end

-- Limit AI range for NPCs that don't need long range
ENT.SightRange = 2000  -- Instead of 15000

-- Use `OnContact` instead of checking distance every frame
function ENT:OnContact(ent)
    if ent:IsPlayer() then
        -- Do something
    end
end

-- Disable unused features
ENT.ClimbLedges = false
ENT.ClimbProps = false
ENT.ClimbLadders = false

-- Use simpler animations
ENT.UseWalkframes = true  -- Use walk for everything
```

## Troubleshooting

### NPC Doesn't Appear in Spawn Menu

**Solutions:**
1. Check DrGBase is installed
2. Verify file is in `lua/entities/` folder
3. Check filename starts with `npc_`
4. Ensure `AddCSLuaFile()` and `DrGBase.AddNextbot(ENT)` at end
5. Check console for Lua errors

### NPC Spawns But Doesn't Move

**Solutions:**
1. Check `RunSpeed` and `WalkSpeed` are set
2. Generate NavMesh: `nav_generate` in console
3. Ensure `StepHeight` and `JumpHeight` are reasonable
4. Check collision bounds aren't too large
5. Enable patrol: `drgbase_ai_patrol 1`

### NPC Doesn't Attack

**Solutions:**
1. Check `MeleeAttackRange` or `RangeAttackRange` is set
2. Verify `OnMeleeAttack` or `OnRangeAttack` is defined
3. Check factions - ensure NPC and target are enemies
4. Call `self:SetDefaultRelationship(D_HT)` in `CustomInitialize`
5. Ensure enemy is within sight range and FOV

### Animations Don't Play

**Solutions:**
1. Check model has the activity (try `developer 1`)
2. Use activities (`ACT_WALK`) not sequences (`"walk_all"`)
3. For models with limited anims, use `UseWalkframes = true`
4. Check animation rates aren't 0

### Console Errors

**Common errors:**

```lua
-- "attempt to call field 'AddNextbot' (a nil value)"
-- Fix: Install DrGBase

-- "'}' expected near..."
-- Fix: Check all {}, if/end, function/end pairs match

-- "attempt to index field 'Models' (a nil value)"
-- Fix: Define ENT.Models before SERVER block

-- "bad argument #1 to 'EmitSound'"
-- Fix: Use valid sound path or remove sound

-- "BaseClass is nil"
-- Fix: Set ENT.Base = "drgbase_nextbot"
```

### Performance Issues

**Solutions:**
1. Reduce `SightRange` to reasonable values (< 5000)
2. Limit number of NPCs spawned
3. Generate NavMesh: `nav_generate`
4. Add think delays to `CustomThink`
5. Disable unused features (climbing, etc.)
6. Use appropriate `CollisionBounds`

### NPC Gets Stuck

**Solutions:**
1. Adjust `CollisionBounds` to match model
2. Increase `StepHeight` for climbing obstacles
3. Generate/regenerate NavMesh
4. Enable climbing features if needed
5. Check for map geometry issues

---

## Additional Resources

- [Your First NPC](../getting-started/04-first-npc.md) - Quick start guide
- [API Reference](../api/README.md) - Complete function reference
- [DrGBase Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=131759821) - Steam Workshop page

## Credits

**DrGBase** by Dr. Magnusson
**Documentation** by the DrGBase community

---

**Happy NPC creating!**
