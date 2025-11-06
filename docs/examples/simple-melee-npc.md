# Simple Melee NPC Example

This example demonstrates creating a basic melee combat NPC using DrGBase. We'll create a hostile creature that chases and attacks enemies with melee strikes.

## Overview

This example creates a "Crawler" NPC that:
- Uses simple AI to detect and chase enemies
- Performs melee attacks with claw swipes
- Has basic sounds and visual feedback
- Shows proper attack timing and damage dealing

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc --
ENT.PrintName = "Crawler"
ENT.Category = "Custom NPCs"
ENT.Models = {"models/headcrabclassic.mdl"}
ENT.CollisionBounds = Vector(12, 12, 24)
ENT.BloodColor = BLOOD_COLOR_YELLOW

-- Stats --
ENT.SpawnHealth = 50
ENT.HealthRegen = 0

-- AI --
ENT.MeleeAttackRange = 60
ENT.RangeAttackRange = 0
ENT.ReachEnemyRange = 50
ENT.AvoidEnemyRange = 0

-- Relationships --
ENT.Factions = {}  -- Hostile to everyone

-- Movements --
ENT.RunSpeed = 250
ENT.WalkSpeed = 100
ENT.Acceleration = 500
ENT.StepHeight = 24
ENT.JumpHeight = 64

-- Detection --
ENT.SightRange = 3000
ENT.SightFOV = 140

-- Animations --
ENT.WalkAnimation = ACT_RUN
ENT.RunAnimation = ACT_RUN
ENT.IdleAnimation = ACT_IDLE

-- Sounds --
ENT.OnIdleSounds = {"NPC_HeadCrab.Idle"}
ENT.OnDamageSounds = {"NPC_HeadCrab.Pain"}
ENT.OnDeathSounds = {"NPC_HeadCrab.Die"}

if SERVER then

    -- Init --

    function ENT:CustomInitialize()
        -- Set to hate everyone by default
        self:SetDefaultRelationship(D_HT)

        -- Optional: Set random size variation
        self:SetModelScale(math.Rand(0.9, 1.1))
    end

    -- AI --

    function ENT:OnMeleeAttack(enemy)
        -- Play attack animation
        self:EmitSound("NPC_HeadCrab.Attack")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnAnimEvent()
        -- Deal damage at the right moment in the attack animation
        if self:IsAttacking() and self:GetCycle() > 0.4 then
            self:Attack({
                damage = 15,
                type = DMG_SLASH,
                viewpunch = Angle(10, math.random(-5, 5), 0)
            }, function(self, hit)
                if #hit > 0 then
                    self:EmitSound("NPC_HeadCrab.Bite")
                else
                    self:EmitSound("NPC_HeadCrab.AttackMiss")
                end
            end)
        end
    end

    -- Patrol behavior --

    function ENT:OnIdle()
        -- Wander when no enemy
        self:AddPatrolPos(self:RandomPos(1500))
    end

    function ENT:OnReachedPatrol()
        -- Wait at patrol point
        self:Wait(math.random(2, 5))
    end

    -- Events --

    function ENT:OnNewEnemy(enemy)
        self:EmitSound("NPC_HeadCrab.Alert")
    end

end

-- Register --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Step-by-Step Explanation

### 1. Basic Setup

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Crawler"
ENT.Category = "Custom NPCs"
ENT.Models = {"models/headcrabclassic.mdl"}
```

- Check if DrGBase is installed
- Inherit from the nextbot base
- Set display name and spawn menu category
- Choose a model (you can add multiple models in the array)

### 2. Physical Properties

```lua
ENT.CollisionBounds = Vector(12, 12, 24)
ENT.BloodColor = BLOOD_COLOR_YELLOW
ENT.SpawnHealth = 50
```

- `CollisionBounds` - Defines the NPC's collision box (width x, width y, height)
- `BloodColor` - Blood particle color when damaged (BLOOD_COLOR_RED, BLOOD_COLOR_YELLOW, BLOOD_COLOR_GREEN, etc.)
- `SpawnHealth` - Starting health value

### 3. Combat Settings

```lua
ENT.MeleeAttackRange = 60
ENT.RangeAttackRange = 0
ENT.ReachEnemyRange = 50
```

- `MeleeAttackRange` - Distance at which the NPC will perform melee attacks
- `RangeAttackRange = 0` - Disables ranged attacks
- `ReachEnemyRange` - How close the NPC tries to get before attacking

### 4. Movement Configuration

```lua
ENT.RunSpeed = 250
ENT.WalkSpeed = 100
ENT.Acceleration = 500
ENT.StepHeight = 24
ENT.JumpHeight = 64
```

- `RunSpeed` - Speed when chasing enemies
- `WalkSpeed` - Speed when patrolling
- `Acceleration` - How quickly the NPC reaches full speed
- `StepHeight` - Maximum step height without jumping
- `JumpHeight` - How high the NPC can jump

### 5. Detection and AI

```lua
ENT.SightRange = 3000
ENT.SightFOV = 140
ENT.Factions = {}
```

- `SightRange` - Maximum vision distance
- `SightFOV` - Field of view in degrees (180 = can see to sides, 90 = narrow vision)
- `Factions = {}` - Empty factions means hostile to everyone

### 6. Attack Implementation

```lua
function ENT:OnMeleeAttack(enemy)
    self:EmitSound("NPC_HeadCrab.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({
            damage = 15,
            type = DMG_SLASH,
            viewpunch = Angle(10, math.random(-5, 5), 0)
        }, function(self, hit)
            if #hit > 0 then
                self:EmitSound("NPC_HeadCrab.Bite")
            end
        end)
    end
end
```

**How it works:**
1. `OnMeleeAttack` is called when the NPC decides to attack
2. It plays an attack animation using `PlayActivityAndMove`
3. `OnAnimEvent` is called during the animation
4. When the animation reaches 40% completion (`GetCycle() > 0.4`), damage is dealt
5. `self:Attack()` damages entities in front of the NPC
6. A callback confirms if anything was hit

### 7. Patrol Behavior

```lua
function ENT:OnIdle()
    self:AddPatrolPos(self:RandomPos(1500))
end

function ENT:OnReachedPatrol()
    self:Wait(math.random(2, 5))
end
```

- `OnIdle` - Called when the NPC has no enemy, adds a random patrol destination
- `OnReachedPatrol` - Called when reaching patrol point, waits 2-5 seconds

## Customization Examples

### Adjust Difficulty

```lua
-- Easy mode
ENT.SpawnHealth = 30
ENT.MeleeAttackRange = 50
self:Attack({damage = 10, type = DMG_SLASH})

-- Hard mode
ENT.SpawnHealth = 100
ENT.MeleeAttackRange = 80
ENT.RunSpeed = 350
self:Attack({damage = 25, type = DMG_SLASH})
```

### Make it Friendly

```lua
-- Join a faction to be friendly to that faction
ENT.Factions = {FACTION_REBELS}

function ENT:CustomInitialize()
    -- Friendly to players, hostile to others
    self:SetPlayersRelationship(D_LI)
    self:SetDefaultRelationship(D_HT)
end
```

### Add Special Effects

```lua
function ENT:OnMeleeAttack(enemy)
    -- Add particle effect
    local effectData = EffectData()
    effectData:SetOrigin(self:GetPos())
    util.Effect("ElectricSpark", effectData)

    self:EmitSound("NPC_HeadCrab.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### Multiple Attack Variations

```lua
function ENT:OnMeleeAttack(enemy)
    -- Randomly choose between two attack animations
    local attacks = {ACT_MELEE_ATTACK1, ACT_MELEE_ATTACK2}
    local attack = attacks[math.random(#attacks)]

    self:EmitSound("NPC_HeadCrab.Attack")
    self:PlayActivityAndMove(attack, 1, self.FaceEnemy)
end
```

### Damage Resistance

```lua
function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()

    -- Take less damage from bullets
    if dmg:IsBulletDamage() then
        return 0.5  -- Take 50% damage
    end

    -- Take extra damage from explosions
    if dmg:IsExplosionDamage() then
        return 2.0  -- Take 200% damage
    end

    return true  -- Normal damage
end
```

## Common Issues and Solutions

### Issue: NPC Doesn't Attack

**Problem:** NPC chases enemies but never attacks

**Solutions:**
1. Check that `MeleeAttackRange` is set (not 0)
2. Verify `OnMeleeAttack` function exists
3. Ensure enemy is within range - increase `MeleeAttackRange` for testing
4. Check that factions are set correctly (enemies should be different factions)

**Debug:**
```lua
function ENT:OnMeleeAttack(enemy)
    print("Attacking!", enemy)  -- This should print when attacking
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### Issue: Attacks Don't Deal Damage

**Problem:** Attack animation plays but enemy takes no damage

**Solutions:**
1. Verify `OnAnimEvent` function exists and contains `self:Attack()`
2. Check animation cycle timing - try different values: `GetCycle() > 0.3` or `0.5`
3. Make sure damage is positive: `damage = 15` not `damage = 0`
4. Verify the NPC is facing the enemy when attacking

**Debug:**
```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() then
        print("Attack cycle:", self:GetCycle())  -- See when damage should trigger

        if self:GetCycle() > 0.4 then
            self:Attack({damage = 15, type = DMG_SLASH}, function(self, hit)
                print("Hit", #hit, "entities")  -- See if anything was hit
            end)
        end
    end
end
```

### Issue: NPC Doesn't Move

**Problem:** NPC spawns but stands still

**Solutions:**
1. Check that `RunSpeed` and `WalkSpeed` are set to positive values
2. Verify no Lua errors in console (press `)
3. Make sure NavMesh exists (type `nav_generate` in console for single-player)
4. Check ConVar: `drgbase_ai_enabled` should be 1

**Test:**
```lua
function ENT:CustomInitialize()
    print("Speeds:", self.RunSpeed, self.WalkSpeed)  -- Should print positive numbers
    self:SetDefaultRelationship(D_HT)
end
```

### Issue: Model Not Showing

**Problem:** NPC is invisible or shows ERROR model

**Solutions:**
1. Verify model path is correct: `"models/headcrabclassic.mdl"` not `"models/headcrab.mdl"`
2. Check that model exists in Garry's Mod (try spawning it as a prop)
3. Make sure quotes are correct (not smart quotes)
4. Restart Garry's Mod to reload the entity

### Issue: Animation Doesn't Play

**Problem:** NPC slides around without animation or plays wrong animation

**Solutions:**
1. Check that the model has the animation - not all models have all activities
2. Try different activities: `ACT_WALK`, `ACT_RUN`, etc.
3. For models with limited animations, use: `ENT.UseWalkframes = true`
4. Set animation explicitly:
```lua
ENT.RunAnimation = ACT_WALK  -- If model doesn't have ACT_RUN
ENT.UseWalkframes = true     -- Use walk frames for running
```

## Performance Tips

1. **Limit sight range** - Don't use unnecessarily high values
```lua
ENT.SightRange = 3000  -- Good
ENT.SightRange = 50000  -- Wasteful, checks too many entities
```

2. **Optimize patrol behavior** - Don't add patrol points too frequently
```lua
function ENT:OnIdle()
    if CurTime() > (self.NextPatrol or 0) then
        self:AddPatrolPos(self:RandomPos(1500))
        self.NextPatrol = CurTime() + 5
    end
end
```

3. **Use sound tables efficiently** - Reuse sound arrays
```lua
ENT.OnIdleSounds = {"sound1.wav", "sound2.wav"}  -- Automatically randomized
```

## Next Steps

- Try the [Ranged NPC Example](./ranged-npc.md) for projectile-based combat
- Learn about [Advanced AI](./advanced-ai.md) for more complex behaviors
- Explore [Custom Weapons](./custom-weapon.md) for weapon-wielding NPCs
