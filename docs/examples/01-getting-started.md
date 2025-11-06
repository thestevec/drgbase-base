# Getting Started with DrGBase

## Introduction

DrGBase is a powerful Garry's Mod framework for creating custom Nextbot NPCs with advanced AI, animations, weapons, and behaviors. This guide will walk you through creating your first NPC from scratch.

## Prerequisites

- Garry's Mod installed
- DrGBase addon installed from the Workshop
- Basic knowledge of Lua
- Text editor for writing code

## Your First NPC: A Simple Zombie

Let's create a basic melee NPC that walks around and attacks players.

### Step 1: Create the Entity File

Create a new file in `garrysmod/addons/your_addon/lua/entities/npc_mypack_zombie.lua`

```lua
if not DrGBase then return end -- Check if DrGBase is installed

ENT.Base = "drgbase_nextbot" -- Use DrGBase nextbot as the base

-- Basic Information
ENT.PrintName = "My Zombie"
ENT.Category = "My NPCs"
ENT.Models = {"models/Zombie/Classic.mdl"}
ENT.BloodColor = BLOOD_COLOR_GREEN

-- Health
ENT.SpawnHealth = 100

-- AI Configuration
ENT.RangeAttackRange = 0      -- No ranged attacks
ENT.MeleeAttackRange = 30     -- Attack range in units
ENT.ReachEnemyRange = 30      -- How close to get to enemy
ENT.AvoidEnemyRange = 0       -- Don't avoid enemies

-- Faction
ENT.Factions = {FACTION_ZOMBIES}

-- Movement
ENT.WalkSpeed = 80
ENT.RunSpeed = 80
ENT.UseWalkframes = true

-- Detection
ENT.SightFOV = 120             -- Field of view (degrees)
ENT.SightRange = 10000         -- How far can see (units)

-- Sounds
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- Server-side logic
if SERVER then
    -- Called when NPC spawns
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT) -- Hate everything by default
    end

    -- Called when NPC performs melee attack
    function ENT:OnMeleeAttack(enemy)
        self:EmitSound("Zombie.Attack")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    -- Called on animation events (e.g., footsteps, attack hits)
    function ENT:OnAnimEvent()
        if self:IsAttacking() and self:GetCycle() > 0.3 then
            -- Deal damage at 30% through attack animation
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

    -- Idle behavior
    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500)) -- Wander randomly
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(3, 7)) -- Wait 3-7 seconds at patrol point
    end

    -- Sound on detecting enemy
    function ENT:OnNewEnemy()
        self:EmitSound("Zombie.Alert")
    end
end

-- Required at end of file
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### Step 2: Test Your NPC

1. Save the file
2. Launch Garry's Mod
3. Start a singleplayer game
4. Open the spawn menu (Q key)
5. Go to the "NPCs" tab
6. Find "My NPCs" category
7. Click on "My Zombie" to spawn it

### Step 3: Understanding the Code

#### **Entity Configuration (ENT table)**

- `ENT.Base`: The base class to inherit from (always `drgbase_nextbot`)
- `ENT.PrintName`: Name shown in spawn menu
- `ENT.Category`: Category in spawn menu
- `ENT.Models`: Array of models the NPC can use
- `ENT.SpawnHealth`: Starting health

#### **AI Ranges**

- `MeleeAttackRange`: How close NPC needs to be for melee attacks
- `ReachEnemyRange`: How close NPC tries to get to enemies
- `RangeAttackRange`: How far NPC can use ranged attacks (0 = disabled)

#### **Key Functions**

- `CustomInitialize()`: Called when NPC spawns, set up initial state
- `OnMeleeAttack(enemy)`: Called when NPC performs melee attack
- `OnAnimEvent()`: Called on animation events (footsteps, attack frames)
- `OnIdle()`: Called when NPC has nothing to do
- `OnNewEnemy()`: Called when NPC acquires a new enemy

#### **Useful Methods**

- `self:Attack({...})`: Perform area attack with damage/type
- `self:EmitSound(sound)`: Play a sound
- `self:PlayActivityAndMove(anim, rate, callback)`: Play animation while moving
- `self:AddPatrolPos(pos)`: Add patrol point
- `self:RandomPos(distance)`: Get random position nearby
- `self:Wait(seconds)`: Wait for X seconds (in coroutine)

### Step 4: Common Customizations

#### Change Movement Speed
```lua
ENT.WalkSpeed = 50   -- Slower walk
ENT.RunSpeed = 200   -- Faster run
```

#### Make NPC Friendly
```lua
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_LI) -- Like everything
    self:SetPlayersRelationship(D_LI) -- Like players
end
```

#### Add More Sounds
```lua
ENT.OnIdleSounds = {"Zombie.Idle"}
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}
ENT.OnFootstepSounds = {"Zombie.FootstepLeft", "Zombie.FootstepRight"}
```

#### Custom Death Behavior
```lua
function ENT:OnDeath(dmg, hitgroup)
    -- Explode on death
    local effectdata = EffectData()
    effectdata:SetOrigin(self:GetPos())
    util.Effect("Explosion", effectdata)

    -- Spawn ragdoll
    self:BecomeRagdoll(dmg)
end
```

## Next Steps

Now that you have a basic NPC, you can explore:

- **Ranged Attacks**: See `02-ranged-npc.md`
- **Weapon Usage**: See `04-humanoid-weapons.md`
- **Advanced AI**: See `06-custom-ai.md`
- **Relationships**: See `07-relationships.md`
- **Player Possession**: See `08-possession.md`

## Troubleshooting

### NPC doesn't appear in spawn menu
- Check console for Lua errors
- Ensure `DrGBase.AddNextbot(ENT)` is at the end of the file
- Ensure `AddCSLuaFile()` is at the end of the file

### NPC spawns but doesn't move
- Check navmesh exists (`nav_generate` in console)
- Check for Lua errors in server console
- Ensure `ENT.Base = "drgbase_nextbot"` is set

### NPC doesn't attack
- Check `MeleeAttackRange` is greater than 0
- Ensure relationship is set to hostile (`D_HT`)
- Check `OnMeleeAttack()` function exists

### Animation issues
- Verify model exists and has animations
- Use `ent_info` while looking at NPC to see animation info
- Check console for animation errors

## File Location Reference

Source: `lua/entities/npc_drg_zombie.lua`

## Related Documentation

- [API Reference](../reference/hooks.md)
- [Configuration Properties](../reference/properties.md)
- [Best Practices](../guides/best-practices.md)
