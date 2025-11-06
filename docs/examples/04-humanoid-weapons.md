# Humanoid NPCs with Weapons

## Overview

DrGBase provides a specialized base class for humanoid NPCs that can use weapons. This guide covers weapon-wielding NPCs like soldiers, guards, and armed enemies.

## Basic Humanoid NPC

### Complete Example

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot_human" -- Use humanoid base class

-- Basic Info
ENT.PrintName = "Armed Guard"
ENT.Category = "My NPCs"
ENT.Models = {
    "models/player/combine_soldier.mdl",
    "models/player/combine_soldier_prisonguard.mdl"
}

-- Relationships
ENT.Factions = {FACTION_COMBINE}

-- Movement
ENT.UseWalkframes = true
ENT.WalkSpeed = 100
ENT.RunSpeed = 250

-- Weapons Configuration
ENT.Weapons = {
    "weapon_smg1",
    "weapon_ar2",
    "weapon_shotgun"
}
ENT.WeaponAccuracy = 0.75      -- 75% accuracy (0.0 - 1.0)
ENT.DropWeaponOnDeath = true   -- Drop weapon when killed

if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Key Differences from Standard Nextbot

### 1. Base Class

```lua
ENT.Base = "drgbase_nextbot_human" -- Not "drgbase_nextbot"
```

The human base class provides:
- Weapon handling
- Human animations (reload, aim, shoot)
- Cover system
- Squad behavior

### 2. Weapons Configuration

```lua
ENT.Weapons = {
    "weapon_smg1",     -- Will randomly pick one
    "weapon_ar2",      -- when spawned
    "weapon_shotgun"
}
```

**Weapon Options:**
- Garry's Mod weapons: `weapon_pistol`, `weapon_357`, `weapon_smg1`, `weapon_ar2`, `weapon_shotgun`, `weapon_crossbow`, `weapon_rpg`
- Custom weapons: Any SWEP class name
- DrGBase weapons: `weapon_drg_ar2`, etc.

### 3. Accuracy Setting

```lua
ENT.WeaponAccuracy = 0.75  -- 0.0 (terrible) to 1.0 (perfect)
```

Controls how precisely NPC aims:
- `1.0` = Perfect accuracy (sniper)
- `0.75` = Good accuracy (soldier)
- `0.5` = Average accuracy (civilian)
- `0.25` = Poor accuracy (drunk)
- `0.0` = Terrible accuracy (blind)

### 4. Drop Weapon on Death

```lua
ENT.DropWeaponOnDeath = true  -- Players can pick up weapon
```

## Advanced Weapon Configuration

### Multiple Weapon Loadouts

```lua
function ENT:CustomInitialize()
    local loadouts = {
        -- Sniper loadout
        {weapon = "weapon_crossbow", accuracy = 0.9, range = 2000},

        -- Assault loadout
        {weapon = "weapon_ar2", accuracy = 0.75, range = 1500},

        -- Close quarters
        {weapon = "weapon_shotgun", accuracy = 0.6, range = 500}
    }

    local loadout = loadouts[math.random(#loadouts)]

    self:GiveWeapon(loadout.weapon)
    self.WeaponAccuracy = loadout.accuracy
    self.RangeAttackRange = loadout.range
end
```

### Weapon Management

```lua
-- Give specific weapon
self:GiveWeapon("weapon_ar2")

-- Check if has weapon
if self:HasWeapon() then
    -- Do something
end

-- Get current weapon
local weapon = self:GetWeapon()
if IsValid(weapon) then
    print(weapon:GetClass())
end

-- Remove weapon
self:RemoveWeapon()

-- Fire weapon
self:PrimaryFire()
self:SecondaryFire()

-- Reload weapon
self:Reload()
```

### Custom Firing Behavior

```lua
function ENT:OnRangeAttack(enemy)
    -- Burst fire: 3 shots then pause
    for i = 1, 3 do
        if self:HasWeapon() then
            self:PrimaryFire()
            self:FaceTowards(enemy)
            self:Wait(0.1)
        end
    end

    self:Wait(1) -- Pause between bursts
end
```

### Weapon Switching

```lua
function ENT:CustomInitialize()
    self.PrimaryWeapon = "weapon_ar2"
    self.SecondaryWeapon = "weapon_pistol"

    self:GiveWeapon(self.PrimaryWeapon)
end

function ENT:OnRangeAttack(enemy)
    local distance = self:GetPos():Distance(enemy:GetPos())
    local weapon = self:GetWeapon()

    if distance < 200 and IsValid(weapon) and weapon:GetClass() == self.PrimaryWeapon then
        -- Switch to pistol for close range
        self:RemoveWeapon()
        self:GiveWeapon(self.SecondaryWeapon)
        self:Wait(1) -- Draw time
    elseif distance > 500 and IsValid(weapon) and weapon:GetClass() == self.SecondaryWeapon then
        -- Switch back to primary for long range
        self:RemoveWeapon()
        self:GiveWeapon(self.PrimaryWeapon)
        self:Wait(1)
    end

    self:PrimaryFire()
end
```

## Model-Based Factions

NPCs can form factions based on their model:

```lua
ENT.Models = {
    "models/player/kleiner.mdl",
    "models/player/magnusson.mdl"
}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)
    self:SetSelfModelRelationship(D_LI) -- Like same model

    -- Create unique factions per model
    if self:GetModel() == "models/player/kleiner.mdl" then
        self:JoinFaction("FACTION_KLEINER")
        self:SetFactionRelationship("FACTION_MAGNUSSON", D_HT)
    elseif self:GetModel() == "models/player/magnusson.mdl" then
        self:JoinFaction("FACTION_MAGNUSSON")
        self:SetFactionRelationship("FACTION_KLEINER", D_HT)
    end
end
```

This creates model-specific teams:
- Kleiners like other Kleiners
- Magnussons like other Magnussons
- Kleiners hate Magnussons and vice versa

## Combat Tactics

### Cover System (Basic)

```lua
function ENT:OnRangeAttack(enemy)
    local distance = self:GetPos():Distance(enemy:GetPos())

    -- If far away, find cover
    if distance > 1000 then
        local coverPos = self:FindCoverNearby(enemy)
        if coverPos then
            self:FollowPath(coverPos)
            self:Wait(2)
        end
    end

    -- Peek and shoot
    self:PrimaryFire()
    self:Wait(0.5)
end

function ENT:FindCoverNearby(threat)
    local myPos = self:GetPos()

    -- Search for solid objects between us and threat
    local searchRadius = 300
    for i = 1, 8 do
        local angle = (360 / 8) * i
        local testPos = myPos + Vector(math.cos(math.rad(angle)), math.sin(math.rad(angle)), 0) * searchRadius

        local tr = util.TraceLine({
            start = testPos + Vector(0, 0, 50),
            endpos = threat:GetPos(),
            filter = {self, threat}
        })

        if tr.Hit then
            return testPos -- Found cover position
        end
    end

    return nil
end
```

### Suppressive Fire

```lua
function ENT:OnRangeAttack(enemy)
    local distance = self:GetPos():Distance(enemy:GetPos())

    if distance > 800 then
        -- Suppressive fire: spray bullets
        self.WeaponAccuracy = 0.3 -- Low accuracy
        for i = 1, 10 do
            self:PrimaryFire()
            self:Wait(0.15)
        end
        self.WeaponAccuracy = 0.75 -- Restore
        self:Wait(2)
    else
        -- Aimed fire
        self:FaceTowards(enemy)
        self:Wait(0.2)
        self:PrimaryFire()
        self:Wait(0.5)
    end
end
```

### Grenade Throwing

```lua
ENT.HasGrenades = true
ENT.MaxGrenades = 3
ENT.GrenadesRemaining = 3

function ENT:OnRangeAttack(enemy)
    local distance = self:GetPos():Distance(enemy:GetPos())

    -- Throw grenade if enemy is clustered or in cover
    if distance > 400 and distance < 1000 and
       self.HasGrenades and self.GrenadesRemaining > 0 then

        if math.random(10) == 1 then -- 10% chance
            self:ThrowGrenade(enemy:GetPos())
            return
        end
    end

    -- Normal weapon fire
    self:PrimaryFire()
end

function ENT:ThrowGrenade(pos)
    self:EmitSound("WeaponFrag.Throw")
    self:PlayActivityAndMove(ACT_RANGE_ATTACK2, 1)

    self:Timer(0.5, function(self)
        local grenade = ents.Create("proj_drg_grenade")
        if not IsValid(grenade) then return end

        local attachment = self:GetAttachment(self:LookupAttachment("anim_attachment_RH"))
        grenade:SetPos(attachment.Pos)
        grenade:SetAngles(attachment.Ang)
        grenade:SetOwner(self)
        grenade:Spawn()

        local phys = grenade:GetPhysicsObject()
        if IsValid(phys) then
            local dir = (pos - grenade:GetPos()):GetNormalized()
            local distance = grenade:GetPos():Distance(pos)
            local force = math.Clamp(distance * 2, 500, 1500)
            phys:SetVelocity(dir * force + Vector(0, 0, 300))
        end

        self.GrenadesRemaining = self.GrenadesRemaining - 1
    end)

    self:Wait(2)
end
```

### Flanking Behavior

```lua
function ENT:HandleEnemy(enemy)
    if not IsValid(enemy) then return end

    local distance = self:GetPos():Distance(enemy:GetPos())

    if distance > 500 then
        -- Try to flank
        local flankPos = self:GetFlankPosition(enemy)
        if flankPos then
            self:FollowPath(flankPos)
        else
            self:ChaseEnemy(enemy) -- Default behavior
        end
    else
        self:ChaseEnemy(enemy)
    end
end

function ENT:GetFlankPosition(enemy)
    local enemyPos = enemy:GetPos()
    local enemyFwd = enemy:GetForward()
    local enemyRight = enemy:GetAngles():Right()

    -- Try positions to the left and right
    local candidates = {
        enemyPos + enemyRight * 400 - enemyFwd * 200,  -- Right flank
        enemyPos - enemyRight * 400 - enemyFwd * 200   -- Left flank
    }

    for _, pos in ipairs(candidates) do
        if util.IsInWorld(pos) then
            -- Check if position has line of sight to enemy
            local tr = util.TraceLine({
                start = pos + Vector(0, 0, 50),
                endpos = enemyPos + Vector(0, 0, 50),
                filter = {self, enemy}
            })

            if not tr.Hit or tr.Entity == enemy then
                return pos
            end
        end
    end

    return nil
end
```

## Animations

Humanoid NPCs automatically use appropriate animations:
- **ACT_WALK_AIM**: Walking with weapon
- **ACT_RUN_AIM**: Running with weapon
- **ACT_IDLE_ANGRY**: Standing alert with weapon
- **ACT_RANGE_ATTACK1**: Firing weapon
- **ACT_RELOAD**: Reloading weapon
- **ACT_COVER_LOW**: Crouching in cover

These are handled automatically by the human base class.

## Example: Elite Soldier

Complete example of an advanced soldier NPC:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot_human"

ENT.PrintName = "Elite Soldier"
ENT.Category = "DrGBase"
ENT.Models = {"models/player/combine_super_soldier.mdl"}

-- Stats
ENT.SpawnHealth = 200
ENT.HealthRegen = 2 -- Regenerate 2 HP/sec

-- AI
ENT.RangeAttackRange = 2000
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 800
ENT.AvoidEnemyRange = 600

-- Movement
ENT.WalkSpeed = 150
ENT.RunSpeed = 300
ENT.UseWalkframes = true

-- Weapons
ENT.Weapons = {"weapon_ar2", "weapon_smg1"}
ENT.WeaponAccuracy = 0.85
ENT.DropWeaponOnDeath = true

-- Faction
ENT.Factions = {FACTION_COMBINE}

if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
        self:SetPlayersRelationship(D_HT)

        -- Give grenades
        self.GrenadesRemaining = 2
    end

    function ENT:OnRangeAttack(enemy)
        local distance = self:GetPos():Distance(enemy:GetPos())

        -- Throw grenade occasionally
        if distance > 400 and distance < 1200 and
           self.GrenadesRemaining > 0 and
           math.random(20) == 1 then
            self:ThrowGrenade(enemy:GetPos())
            return
        end

        -- Burst fire
        for i = 1, 3 do
            if self:HasWeapon() then
                self:PrimaryFire()
                self:Wait(0.1)
            end
        end

        self:Wait(1)
    end

    function ENT:ThrowGrenade(pos)
        -- (Implementation from above)
    end

    function ENT:OnTakeDamage(dmg)
        -- 50% damage from bullets
        if dmg:IsBulletDamage() then
            return 0.5
        end
    end

    function ENT:OnNewEnemy(enemy)
        self:EmitSound("NPC_Combine.Die")
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Troubleshooting

### NPC doesn't fire weapon
- Ensure using `drgbase_nextbot_human` base class
- Check weapon class name is correct
- Verify weapon is valid with `self:HasWeapon()`
- Check `RangeAttackRange` is set

### NPC has wrong animations
- Ensure model is humanoid (playermodel)
- Check model has weapon holding bones
- Use `ent_info` to debug animation state

### Weapon fires continuously
- Add `self:Wait()` calls in `OnRangeAttack()`
- Use burst fire pattern (show above)

### Weapon doesn't drop on death
- Set `ENT.DropWeaponOnDeath = true`
- Ensure weapon exists when NPC dies

## Source Reference

Source: `lua/entities/npc_drg_testhuman.lua`

## Related Documentation

- [Weapon System](../reference/weapons.md)
- [Faction System](07-relationships.md)
- [Combat AI](06-custom-ai.md)
