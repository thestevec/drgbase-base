# Combat & Weapons System

Melee, ranged attacks, weapons, and projectiles.

**Files:** `weapons.lua` (nextbot), `drgbase_weapon/`, `proj_drg_default/`

## Overview

The combat system allows NPCs to equip, manage, and fire weapons. It includes support for Half-Life 2 weapons and custom DrGBase weapons. The system handles:

- **Weapon Management** - Picking up, dropping, and switching between weapons
- **Melee Combat** - Close-range attacks
- **Ranged Combat** - Firearms and projectile weapons
- **Ammo System** - Ammunition tracking and reloading
- **Auto-Configuration** - Combat ranges automatically adjust based on weapon type

## Components

### Melee Combat

Melee attacks are simple close-range interactions. Configure via properties:

```lua
ENT.MeleeAttackRange = 30  -- Range to perform melee attack
ENT.ReachEnemyRange = 25  -- Distance considered "in range"
```

The NPC will automatically use melee attacks when:
- Enemy is within `MeleeAttackRange`
- `OnMeleeAttack()` hook is called by behavior tree

**Melee Hook:**
```lua
function ENT:OnMeleeAttack()
    -- Perform melee attack behavior
    self:Attack(self.MeleeDamage)
end
```

### Ranged Combat

Ranged combat uses weapons. NPCs can fire primary and secondary attacks, reload, and manage ammo.

**Supported Half-Life 2 Weapons:**
- `weapon_pistol` - Pistol (damage: 5, delay: 0.75s)
- `weapon_357` - Magnum Revolver (damage: 40, delay: 1.25s)
- `weapon_smg1` - SMG (damage: 4, delay: 0.065s)
- `weapon_ar2` - Pulse Rifle (damage: 8, delay: 0.1s, tracer: AR2Tracer)
- `weapon_shotgun` - Shotgun (damage: 8×7 pellets, delay: 1.25s)
- `weapon_crossbow` - Crossbow (spawns bolt entity, delay: 2.5s)
- `gmod_tool` - Tool Gun (random effects on entities)
- `gmod_camera` - Camera (2.5s delay)

**Combat Properties:**
```lua
ENT.RangeAttackRange = 1500  -- Range to fire weapon
ENT.AvoidEnemyRange = 750  -- Range to back away from enemy
```

**Weapon Type Auto-Configuration:**

When a weapon is equipped, combat ranges auto-adjust based on hold type:
- **Long Range** (crossbow): 3000 range, 750 avoid, 2000 reach
- **Medium Range** (pistol, revolver, grenade): 750 range, 350 avoid, 500 reach
- **Close Range** (shotgun, camera): 325 range, 175 avoid, 250 reach
- **Melee** (crowbar, stunstick): 30 melee range, 25 reach, 0 ranged

### Weapon Management

NPCs can carry multiple weapons and switch between them.

**Initial Weapon Setup:**
```lua
ENT.UseWeapons = true  -- Enable weapon system
ENT.Weapons = {"weapon_pistol", "weapon_shotgun"}  -- Starting weapons
```

**Weapon Functions:**
- `GiveWeapon(class)` - Spawn and give weapon to NPC
- `PickupWeapon(weapon)` - Pick up existing weapon entity
- `DropWeapon(weapon)` - Drop weapon back into world
- `RemoveWeapon(weapon)` - Drop and remove weapon
- `SelectWeapon(class)` - Switch to specific weapon
- `SwitchWeapon()` - Intelligently switch weapon (calls `OnSwitchWeapon` hook)
- `RandomWeapon()` - Switch to random owned weapon

**Weapon Getters:**
- `GetActiveWeapon()` - Get currently equipped weapon
- `GetWeapon(class)` - Get weapon by class name
- `GetWeapons()` - Get table of all owned weapons
- `GetWeaponCount()` - Count owned weapons
- `HasWeapon(class)` - Check if NPC owns weapon

**Weapon Hooks:**
- `OnWeaponChange(old, new)` - Called when active weapon changes
- `OnPickupWeapon(weapon, class)` - Called when weapon picked up
- `OnDropWeapon(weapon, class)` - Called when weapon dropped
- `OnSwitchWeapon(weap1, weap2)` - Compare weapons to decide which to switch to

### Projectile System

The crossbow and custom weapons can spawn projectile entities.

**Crossbow Projectiles:**
- Automatically fires `crossbow_bolt` entities
- Automatically predicts target movement
- Customizable via `OnAimAtEntity(target)` hook

**Custom DrGBase Weapons:**

DrGBase includes a weapon base (`drgbase_weapon`) and projectile base (`proj_drg_default`). See [Weapon Base API](../../api/weapon/README.md) and [Projectile API](../../api/projectile/README.md) for details.

## Configuration

**Combat Properties:**
```lua
ENT.UseWeapons = false  -- Enable/disable weapon system
ENT.Weapons = {}  -- Table of weapon classes to spawn with
ENT.MeleeAttackRange = 50  -- Melee attack distance
ENT.RangeAttackRange = 1500  -- Ranged attack distance
ENT.AvoidEnemyRange = 0  -- Distance to keep from enemy
ENT.ReachEnemyRange = 1000  -- Distance considered "reached" enemy
```

**Firing and Reloading:**

NPCs fire weapons via:
- `WeaponPrimaryFire(animation)` - Fire primary attack, optionally play animation
- `WeaponSecondaryFire(animation)` - Fire secondary attack, optionally play animation
- `WeaponReload(animation)` - Reload weapon, play animation

**Ammo Management:**
- `GetWeaponPrimaryAmmo(class)` - Get primary ammo count
- `GetWeaponSecondaryAmmo(class)` - Get secondary ammo count
- `IsWeaponPrimaryEmpty(class)` - Check if primary ammo empty
- `IsWeaponSecondaryEmpty(class)` - Check if secondary ammo empty
- `IsWeaponPrimaryFull(class)` - Check if primary ammo full
- `IsWeaponSecondaryFull(class)` - Check if secondary ammo full
- `IsReloadingWeapon()` - Check if currently reloading

## Usage Examples

**Basic Weapon Setup:**
```lua
function ENT:SetupDataTables()
    -- Give NPC a shotgun
    self.UseWeapons = true
    self.Weapons = {"weapon_shotgun"}
end

function ENT:OnRangeAttack()
    -- Fire weapon when in range attack behavior
    if self:WeaponPrimaryFire(ACT_GESTURE_RANGE_ATTACK1) then
        -- Shot fired successfully
    end
end
```

**Multiple Weapons with Switching:**
```lua
ENT.UseWeapons = true
ENT.Weapons = {"weapon_shotgun", "weapon_pistol"}

function ENT:Initialize()
    -- Start with shotgun
    self:SelectWeapon("weapon_shotgun")
end

function ENT:OnSwitchWeapon(weap1, weap2)
    -- Prefer shotgun at close range
    if self:HasEnemy() and self:IsInRange(self:GetEnemy(), 500) then
        return weap1:GetClass() == "weapon_shotgun"
    end
    -- Otherwise use default (weapon weight)
end
```

**Ammo Management:**
```lua
function ENT:OnRangeAttack()
    local weapon = self:GetActiveWeapon()

    -- Check if need to reload
    if self:IsWeaponPrimaryEmpty() then
        self:WeaponReload(ACT_GESTURE_RELOAD)
        return
    end

    -- Fire weapon
    self:WeaponPrimaryFire(ACT_GESTURE_RANGE_ATTACK1)
end
```

**Custom Aiming:**
```lua
function ENT:OnAimAtEntity(target)
    -- Aim at head instead of center
    if target:IsPlayer() or target:IsNPC() then
        local boneID = target:LookupBone("ValveBiped.Bip01_Head1")
        if boneID then
            local headPos = target:GetBonePosition(boneID)
            return headPos
        end
    end
    -- Default: aim at center
end
```

**Tool Gun Behavior:**
```lua
function ENT:OnUseToolgun(ent, trace)
    -- Custom tool gun effect: always ignite target
    if IsValid(ent) then
        ent:Ignite(10)
        return true  -- Return true to apply effect
    end
    return false  -- Return false to do nothing
end
```

**Weapon Drops on Death:**
```lua
function ENT:OnDeath(dmg)
    -- Drop all weapons
    for class, weapon in pairs(self:GetWeapons()) do
        self:DropWeapon(weapon)
    end
end
```

**Dynamic Weapon Selection:**
```lua
function ENT:Think()
    if not self:HasEnemy() then return end

    local dist = self:GetRangeTo(self:GetEnemy())

    -- Switch to shotgun at close range
    if dist < 500 and not self:GetActiveWeapon():GetClass() == "weapon_shotgun" then
        self:SelectWeapon("weapon_shotgun")

    -- Switch to rifle at long range
    elseif dist > 1000 and not self:GetActiveWeapon():GetClass() == "weapon_ar2" then
        self:SelectWeapon("weapon_ar2")
    end
end
```

**Melee Attacks:**
```lua
function ENT:SetupDataTables()
    self.MeleeAttackRange = 50
    self.MeleeDamage = 25
end

function ENT:OnMeleeAttack()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    -- Deal damage to enemy
    local dmg = DamageInfo()
    dmg:SetDamage(self.MeleeDamage)
    dmg:SetAttacker(self)
    dmg:SetInflictor(self)
    dmg:SetDamageType(DMG_SLASH)
    enemy:TakeDamageInfo(dmg)

    -- Play attack animation
    self:PlayAnimation(ACT_MELEE_ATTACK1)
    self:EmitSound("Zombie.AttackMiss")
end
```

**Secondary Fire:**
```lua
function ENT:OnRangeAttack()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    local dist = self:GetRangeTo(enemy)

    -- Use secondary fire at close range (shotgun double-shot)
    if dist < 250 and not self:IsWeaponSecondaryEmpty() then
        self:WeaponSecondaryFire(ACT_GESTURE_RANGE_ATTACK2)
    else
        self:WeaponPrimaryFire(ACT_GESTURE_RANGE_ATTACK1)
    end
end
```

## API Reference

- [Weapon Functions](../../api/nextbot/weapons.md)
- [Weapon Base API](../../api/weapon/README.md)
- [Projectile API](../../api/projectile/README.md)

## Best Practices

**Weapon Selection:**
- Set `UseWeapons = true` to enable the system
- Provide initial `Weapons` table for spawning with weapons
- Use `OnSwitchWeapon()` to control automatic weapon switching
- Weapons are automatically hidden/shown when switching

**Combat Ranges:**
- Let the system auto-configure ranges based on weapon type
- Override `RangeAttackRange` / `AvoidEnemyRange` for custom behavior
- Use `ReachEnemyRange` to determine when enemy is "in range"

**Ammo Management:**
- Always check `IsWeaponPrimaryEmpty()` before firing
- Call `WeaponReload()` when ammo is depleted
- Reload animation duration controls reload time
- Primary ammo refills to `GetMaxClip1()` automatically

**Performance:**
- Weapons use `EF_BONEMERGE` for attachment (no parenting overhead)
- Inactive weapons are hidden via `SetNoDraw(true)`
- Weapons are `MOVETYPE_NONE` and not solid when equipped

**Aiming:**
- `GetShootPos()` returns weapon bone position if available
- `GetAimVector()` automatically aims at enemy or possessed player's crosshair
- Override `OnAimAtEntity(target)` to customize aim point
- Crossbow automatically predicts target movement
