# Code Organization Best Practices

## Overview

Well-organized code is easier to maintain, debug, and extend. This guide covers structuring your DrGBase NPCs for maximum clarity and maintainability.

## File Structure

### Basic NPC File Template

```lua
-- ================================================
-- NPC: My Zombie
-- Description: Basic melee zombie NPC
-- Author: YourName
-- Date: 2024-01-01
-- ================================================

if not DrGBase then return end

-- ================================================
-- BASE CONFIGURATION
-- ================================================

ENT.Base = "drgbase_nextbot"

-- ================================================
-- BASIC INFORMATION
-- ================================================

ENT.PrintName = "My Zombie"
ENT.Category = "My NPCs"
ENT.Author = "YourName"
ENT.Spawnable = true
ENT.AdminOnly = false

-- ================================================
-- MODELS & APPEARANCE
-- ================================================

ENT.Models = {"models/Zombie/Classic.mdl"}
ENT.ModelScale = 1.0
ENT.CollisionBounds = Vector(13, 13, 72)
ENT.BloodColor = BLOOD_COLOR_GREEN

-- ================================================
-- STATS
-- ================================================

ENT.SpawnHealth = 100
ENT.HealthRegen = 0
ENT.MinPhysDamage = 10

-- ================================================
-- AI CONFIGURATION
-- ================================================

ENT.RangeAttackRange = 0
ENT.MeleeAttackRange = 30
ENT.ReachEnemyRange = 30
ENT.AvoidEnemyRange = 0

-- ================================================
-- RELATIONSHIPS
-- ================================================

ENT.DefaultRelationship = D_HT
ENT.Factions = {FACTION_ZOMBIES}
ENT.Frightening = false

-- ================================================
-- DETECTION
-- ================================================

ENT.SightFOV = 120
ENT.SightRange = 2000
ENT.HearingCoefficient = 1.0
ENT.EyeBone = "ValveBiped.Bip01_Spine4"
ENT.EyeOffset = Vector(7.5, 0, 5)

-- ================================================
-- MOVEMENT
-- ================================================

ENT.WalkSpeed = 80
ENT.RunSpeed = 80
ENT.Acceleration = 900
ENT.JumpHeight = 70
ENT.StepHeight = 22
ENT.UseWalkframes = true

-- ================================================
-- ANIMATIONS
-- ================================================

ENT.WalkAnimation = ACT_WALK
ENT.RunAnimation = ACT_WALK
ENT.IdleAnimation = ACT_IDLE
ENT.JumpAnimation = ACT_JUMP

-- ================================================
-- SOUNDS
-- ================================================

ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- ================================================
-- POSSESSION
-- ================================================

ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
    {
        offset = Vector(0, 30, 20),
        distance = 100
    }
}

-- ================================================
-- SERVER-SIDE CODE
-- ================================================

if SERVER then

    -- =====================================
    -- INITIALIZATION
    -- =====================================

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    -- =====================================
    -- AI BEHAVIOR
    -- =====================================

    function ENT:OnMeleeAttack(enemy)
        self:EmitSound("Zombie.Attack")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(3, 7))
    end

    -- =====================================
    -- COMBAT
    -- =====================================

    function ENT:OnAnimEvent()
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
        end
    end

    -- =====================================
    -- DAMAGE & DEATH
    -- =====================================

    function ENT:OnDeath(dmg, hitgroup)
        -- Custom death behavior
    end

    -- =====================================
    -- SOUNDS & EFFECTS
    -- =====================================

    function ENT:OnNewEnemy()
        self:EmitSound("Zombie.Alert")
    end

end -- End SERVER

-- ================================================
-- CLIENT-SIDE CODE
-- ================================================

if CLIENT then

    function ENT:CustomDraw()
        -- Custom rendering
    end

end -- End CLIENT

-- ================================================
-- REQUIRED AT END
-- ================================================

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Organizing Complex NPCs

### Multi-File Structure

For complex NPCs, split code into modules:

```
lua/
  └── entities/
      └── npc_mypack_boss/
          ├── shared.lua         -- Main file, configuration
          ├── init.lua           -- Server init
          ├── cl_init.lua        -- Client init
          ├── sv_ai.lua          -- AI behavior
          ├── sv_combat.lua      -- Combat systems
          ├── sv_abilities.lua   -- Special abilities
          └── cl_effects.lua     -- Client effects
```

**shared.lua:**
```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"
ENT.Type = "nextbot"

-- Configuration here
ENT.PrintName = "Boss"
ENT.SpawnHealth = 1000

-- Load other files
if SERVER then
    include("sv_ai.lua")
    include("sv_combat.lua")
    include("sv_abilities.lua")
else
    include("cl_effects.lua")
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**sv_ai.lua:**
```lua
-- AI behavior only
function ENT:AIBehaviour()
    while true do
        self:BossAI()
        self:Wait(0.5)
    end
end

function ENT:BossAI()
    -- AI logic
end
```

## Code Organization Patterns

### Pattern 1: State Machine Organization

```lua
-- ================================================
-- STATE MACHINE
-- ================================================

function ENT:CustomInitialize()
    self.State = "idle"
end

function ENT:AIBehaviour()
    while true do
        self:ExecuteState()
        self:Wait(0.1)
    end
end

-- State execution
function ENT:ExecuteState()
    local stateFunc = self.States[self.State]
    if stateFunc then
        stateFunc(self)
    end
end

-- State definitions
ENT.States = {}

ENT.States.idle = function(self)
    -- Idle logic
end

ENT.States.patrol = function(self)
    -- Patrol logic
end

ENT.States.combat = function(self)
    -- Combat logic
end

ENT.States.flee = function(self)
    -- Flee logic
end

-- State transitions
function ENT:ChangeState(newState)
    self.State = newState
end
```

### Pattern 2: Ability System Organization

```lua
-- ================================================
-- ABILITIES
-- ================================================

function ENT:CustomInitialize()
    self:InitializeAbilities()
end

function ENT:InitializeAbilities()
    self.Abilities = {
        {
            name = "Fireball",
            cooldown = 5,
            lastUsed = 0,
            func = self.Ability_Fireball
        },
        {
            name = "Teleport",
            cooldown = 10,
            lastUsed = 0,
            func = self.Ability_Teleport
        }
    }
end

function ENT:UseAbility(abilityName)
    for _, ability in ipairs(self.Abilities) do
        if ability.name == abilityName then
            if CurTime() > ability.lastUsed + ability.cooldown then
                ability.func(self)
                ability.lastUsed = CurTime()
                return true
            end
        end
    end
    return false
end

-- Ability implementations
function ENT:Ability_Fireball()
    -- Fireball code
end

function ENT:Ability_Teleport()
    -- Teleport code
end
```

### Pattern 3: Behavior Tree Organization

```lua
-- ================================================
-- BEHAVIOR TREE
-- ================================================

function ENT:AIBehaviour()
    while true do
        self:BehaviorTree()
        self:Wait(0.1)
    end
end

function ENT:BehaviorTree()
    -- Priority order
    if self:Behavior_HandleCombat() then return end
    if self:Behavior_Investigate() then return end
    if self:Behavior_Patrol() then return end
    self:Behavior_Idle()
end

-- Behaviors return true if handled
function ENT:Behavior_HandleCombat()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return false end

    self:AttackEntity(enemy)
    return true
end

function ENT:Behavior_Investigate()
    if not self.InvestigatePos then return false end

    self:FollowPath(self.InvestigatePos)
    return true
end

function ENT:Behavior_Patrol()
    if not self:HasPatrolPos() then return false end

    -- Patrol code
    return true
end

function ENT:Behavior_Idle()
    self:AddPatrolPos(self:RandomPos(1500))
end
```

## Naming Conventions

### Variables

```lua
-- Configuration (ENT table)
ENT.SpawnHealth = 100          -- PascalCase for ENT properties
ENT.MeleeAttackRange = 50

-- Instance variables (self)
self.currentEnemy = nil        -- camelCase for instance vars
self.attackCooldown = 0
self.isCharging = false

-- Local variables
local targetPosition = ...      -- camelCase for locals
local enemyDistance = ...

-- Constants
local MAX_HEALTH = 100         -- UPPER_SNAKE_CASE for constants
local ATTACK_DAMAGE = 25
```

### Functions

```lua
-- Hook implementations (DrGBase hooks)
function ENT:CustomInitialize()      -- Matches DrGBase naming
function ENT:OnMeleeAttack(enemy)
function ENT:OnTakeDamage(dmg)

-- Custom functions (your code)
function ENT:CalculateDistance()     -- PascalCase for methods
function ENT:FindNearestEnemy()
function ENT:ApplyDamage(target, amount)

-- Private/internal functions (prefix with underscore)
function ENT:_InternalSetup()        -- Indicates internal use
function ENT:_ValidateState()

-- Ability functions (prefix with Ability_)
function ENT:Ability_Fireball()
function ENT:Ability_Teleport()

-- State functions (prefix with State_)
function ENT:State_Idle()
function ENT:State_Combat()
```

## Comments

### Good Commenting Practices

```lua
-- ================================================
-- SECTION: Major section divider
-- ================================================

-- -------------------------------------------------
-- Subsection: Minor section divider
-- -------------------------------------------------

-- Explanation comment: Explains why, not what
function ENT:ComplexFunction()
    -- Calculate damage multiplier based on distance
    -- Closer = more damage (inverse square law)
    local distance = self:GetPos():Distance(enemy:GetPos())
    local multiplier = 1000 / (distance * distance)

    -- TODO: Add maximum cap for very close range
    local damage = baseDamage * multiplier
end

-- Warning comment
function ENT:DangerousFunction()
    -- WARNING: This modifies global state!
    -- FIXME: Should be refactored to avoid globals
end

-- Parameter documentation
---@param enemy Entity The target entity to attack
---@param damage number Amount of damage to deal
---@return boolean success Whether attack landed
function ENT:AttackEnemy(enemy, damage)
end
```

### Avoid Over-Commenting

```lua
-- ❌ Bad: Obvious comments
self.health = 100  -- Set health to 100
local x = 5        -- Set x to 5

-- ✅ Good: Explain why, not what
self.health = 100  -- Start at full health for easy difficulty
local x = 5        -- Grid size for navigation
```

## Configuration Organization

### Group Related Settings

```lua
-- ================================================
-- COMBAT SETTINGS
-- ================================================

ENT.MeleeAttackRange = 50
ENT.MeleeDamage = 10
ENT.AttackCooldown = 1.5

ENT.RangeAttackRange = 500
ENT.RangeDamage = 15
ENT.RangeCooldown = 2.0

-- ================================================
-- MOVEMENT SETTINGS
-- ================================================

ENT.WalkSpeed = 100
ENT.RunSpeed = 200
ENT.SprintSpeed = 300

ENT.Acceleration = 900
ENT.Deceleration = 900

-- ================================================
-- DETECTION SETTINGS
-- ================================================

ENT.SightRange = 2000
ENT.SightFOV = 120
ENT.HearingRange = 1000
ENT.SmellRange = 500
```

## Function Organization

### Order Functions Logically

```lua
if SERVER then

    -- 1. Initialization (first)
    function ENT:CustomInitialize()
    function ENT:CustomThink()

    -- 2. AI & Behavior
    function ENT:AIBehaviour()
    function ENT:HandleEnemy(enemy)
    function ENT:OnIdle()

    -- 3. Combat
    function ENT:OnMeleeAttack(enemy)
    function ENT:OnRangeAttack(enemy)
    function ENT:OnAnimEvent()

    -- 4. Damage & Death
    function ENT:OnTakeDamage(dmg)
    function ENT:OnDeath(dmg, hitgroup)

    -- 5. Helper Functions (last)
    function ENT:FindNearestEnemy()
    function ENT:CalculateDamage(base, multiplier)

end
```

## Module Pattern for Large NPCs

```lua
-- ================================================
-- MODULE: Combat System
-- ================================================

local CombatModule = {}

function CombatModule:Initialize(npc)
    npc.CombatData = {
        attackQueue = {},
        currentCombo = 0,
        lastAttack = 0
    }
end

function CombatModule:ProcessAttack(npc, enemy)
    -- Combat logic
end

-- Register module
ENT.CombatModule = CombatModule

-- Use in NPC
function ENT:CustomInitialize()
    self.CombatModule:Initialize(self)
end

function ENT:OnMeleeAttack(enemy)
    self.CombatModule:ProcessAttack(self, enemy)
end
```

## Documentation Template

```lua
--[[
    NPC: Elite Guard
    Version: 1.2.0
    Author: YourName
    Description: Advanced guard NPC with squad AI and weapon usage

    Features:
    - Squad-based tactics
    - Multiple weapon types
    - Dynamic difficulty scaling
    - Custom voice lines

    Configuration:
    - SpawnHealth: 200
    - Weapons: AR2, SMG1, Shotgun
    - Faction: FACTION_GUARDS

    Dependencies:
    - DrGBase 1.0+
    - Custom sound pack (optional)

    Known Issues:
    - May get stuck on complex geometry
    - Voice lines don't play on dedicated servers

    TODO:
    - [ ] Add cover system
    - [ ] Implement grenade throwing
    - [ ] Add more voice lines
]]
```

## See Also

- [Performance Guide](performance.md)
- [Common Pitfalls](common-pitfalls.md)
- [Debugging Guide](debugging.md)
