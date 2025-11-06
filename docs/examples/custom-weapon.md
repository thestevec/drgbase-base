# Custom Weapon Example

This example demonstrates creating weapon-wielding NPCs using DrGBase's human nextbot system. NPCs can use standard GMod weapons with customizable accuracy and behavior.

## Overview

Weapon-wielding NPCs require:
- The human nextbot base (`drgbase_nextbot_human`)
- Weapon class names
- Accuracy and behavior settings
- Proper animations (handled automatically)

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot_human"  -- Use human base for weapons

-- Misc --
ENT.PrintName = "Rebel Soldier"
ENT.Category = "Custom NPCs"
ENT.Models = {
    "models/player/Group03/male_01.mdl",
    "models/player/Group03/male_02.mdl",
    "models/player/Group03/male_03.mdl"
}

-- Stats --
ENT.SpawnHealth = 100

-- Relationships --
ENT.Factions = {FACTION_REBELS}

-- Movements --
ENT.UseWalkframes = true
ENT.RunSpeed = 200
ENT.WalkSpeed = 100

-- Weapons --
ENT.Weapons = {
    "weapon_smg1",
    "weapon_ar2",
    "weapon_shotgun",
    "weapon_pistol"
}

ENT.WeaponAccuracy = 0.7  -- 0.0 = terrible, 1.0 = perfect
ENT.WeaponProficiency = WEAPON_PROFICIENCY_GOOD

if SERVER then

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
        self:SetPlayersRelationship(D_LI)  -- Friendly to players

        -- Random bodygroups for variety
        for i = 0, self:GetNumBodyGroups() - 1 do
            self:SetBodygroup(i, math.random(0, self:GetBodygroupCount(i) - 1))
        end
    end

    -- AI Behavior --

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(3, 7))
    end

    function ENT:OnNewEnemy(enemy)
        self:EmitSound("npc/metropolice/vo/contactconfirmprosecuting.wav")
    end

    -- Combat overrides (optional) --

    function ENT:OnWeaponFired(weapon)
        -- Called when weapon is fired
        -- Useful for custom behavior
    end

    function ENT:OnReload()
        -- Called when reloading
        -- NPC takes cover during reload automatically
    end

end

-- Register --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Key Properties

### Weapon Selection
```lua
ENT.Weapons = {
    "weapon_smg1",    -- NPC will randomly pick one
    "weapon_ar2",
    "weapon_shotgun"
}
```

NPCs choose one weapon randomly on spawn.

### Accuracy
```lua
ENT.WeaponAccuracy = 0.7  -- 70% accurate
```
- `0.0` = Can't hit anything
- `0.5` = Moderate accuracy
- `1.0` = Perfect aim

### Weapon Proficiency
```lua
ENT.WeaponProficiency = WEAPON_PROFICIENCY_GOOD
```

Options:
- `WEAPON_PROFICIENCY_POOR`
- `WEAPON_PROFICIENCY_AVERAGE`
- `WEAPON_PROFICIENCY_GOOD`
- `WEAPON_PROFICIENCY_VERY_GOOD`
- `WEAPON_PROFICIENCY_PERFECT`

## Customization Examples

### Sniper NPC
```lua
ENT.Weapons = {"weapon_crossbow"}
ENT.WeaponAccuracy = 0.95
ENT.SightRange = 5000
ENT.RangeAttackRange = 3000
ENT.ReachEnemyRange = 1500  -- Keep distance
ENT.AvoidEnemyRange = 500   -- Back away if too close
```

### Shotgunner
```lua
ENT.Weapons = {"weapon_shotgun"}
ENT.WeaponAccuracy = 0.6  -- Less accurate but devastating up close
ENT.RangeAttackRange = 500  -- Only shoot at close range
ENT.ReachEnemyRange = 200  -- Get close
ENT.Aggressive = true
```

### Heavy Gunner
```lua
ENT.Weapons = {"weapon_ar2"}
ENT.SpawnHealth = 150
ENT.WeaponAccuracy = 0.8
ENT.WeaponProficiency = WEAPON_PROFICIENCY_VERY_GOOD
ENT.RunSpeed = 150  -- Slower
```

### Suppressive Fire
```lua
function ENT:OnChaseEnemy(enemy)
    -- Fire continuously while chasing
    local wep = self:GetActiveWeapon()
    if IsValid(wep) and CurTime() > (self.NextShot or 0) then
        self:FaceTo(enemy:EyePos())
        wep:PrimaryAttack()
        self.NextShot = CurTime() + 0.3
    end
end
```

### Grenade Throwing
```lua
function ENT:CustomThink()
    local enemy = self:GetEnemy()
    if IsValid(enemy) and CurTime() > (self.NextGrenade or 0) then
        local dist = self:GetPos():Distance(enemy:GetPos())
        if dist > 300 and dist < 800 then
            self:ThrowGrenade(enemy:GetPos())
            self.NextGrenade = CurTime() + 15
        end
    end
end

function ENT:ThrowGrenade(targetPos)
    local gren = ents.Create("proj_drg_grenade")
    if IsValid(gren) then
        local startPos = self:EyePos()
        gren:SetPos(startPos)
        gren:Spawn()
        gren:SetOwner(self)

        local phys = gren:GetPhysicsObject()
        if IsValid(phys) then
            local dir = (targetPos - startPos):GetNormalized()
            local dist = startPos:Distance(targetPos)
            phys:SetVelocity(dir * (dist * 0.6) + Vector(0, 0, 300))
        end

        if gren.Unpin then gren:Unpin() end
    end

    self:EmitSound("weapons/slam/throw.wav")
end
```

### Switch Weapons Mid-Combat
```lua
function ENT:CustomInitialize()
    self.PrimaryWeapon = "weapon_ar2"
    self.SecondaryWeapon = "weapon_pistol"
    self:Give(self.PrimaryWeapon)
end

function ENT:CustomThink()
    local wep = self:GetActiveWeapon()
    if not IsValid(wep) then return end

    -- Switch to pistol when primary is out of ammo
    if wep:Clip1() == 0 and wep:GetClass() == self.PrimaryWeapon then
        self:Give(self.SecondaryWeapon)
        self:SelectWeapon(self.SecondaryWeapon)
    end
end
```

## Common Issues and Solutions

### Issue: NPC Won't Fire Weapon

**Solutions:**
1. Make sure using `drgbase_nextbot_human` base
2. Verify weapon class name is correct
3. Check that enemy is in `RangeAttackRange`
4. Weapon may be out of ammo - NPCs have infinite ammo by default but check weapon entity

### Issue: Shots Always Miss

**Solutions:**
1. Increase `WeaponAccuracy` value
2. Adjust `WeaponProficiency` setting
3. Make sure NPC is facing target
4. Target may be too far - adjust sight/attack ranges

### Issue: NPC Doesn't Use Weapon

**Solutions:**
```lua
-- Force weapon selection
function ENT:CustomInitialize()
    timer.Simple(0.1, function()
        if not IsValid(self) then return end
        self:SelectWeapon("weapon_smg1")
    end)
end
```

### Issue: Wrong Animations

**Solution:** Human base handles animations automatically, but ensure:
```lua
ENT.UseWalkframes = true  -- Important for humanoid models
```

## Performance Tips

1. **Limit weapon variety** - Each unique weapon must be cached
2. **Reasonable accuracy** - Perfect accuracy (1.0) is more expensive
3. **Don't spam fire** - Use built-in weapon fire rates

## Next Steps

- Try [Ranged NPC Example](./ranged-npc.md) for projectile-based combat
- Learn [Advanced AI](./advanced-ai.md) for tactical behavior
- Explore [Boss NPC](./boss-npc.md) for complex encounters
