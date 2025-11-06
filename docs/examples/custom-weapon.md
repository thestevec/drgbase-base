# Custom Weapon Example

A complete guide to creating custom weapons using DrGBase's weapon system.

## Table of Contents

- [Overview](#overview)
- [Complete Code](#complete-code)
- [Code Breakdown](#code-breakdown)
- [Testing the Weapon](#testing-the-weapon)
- [Customization Ideas](#customization-ideas)
- [Troubleshooting](#troubleshooting)
- [See Also](#see-also)

## Overview

This example shows how to create a fully functional custom weapon that:
- Fires automatic bullets with configurable damage and spread
- Uses Half-Life 2 weapon models and animations
- Has proper ammo management and reload mechanics
- Includes muzzle flash and sound effects
- Can be given to players and NPCs
- Supports both viewmodel (first-person) and worldmodel (third-person)

This example creates a custom assault rifle, but the same principles apply to any weapon type (pistols, shotguns, SMGs, etc.).

**Difficulty:** Beginner
**File Location:** `lua/weapons/weapon_drg_myar2/shared.lua`
**Base:** `drgbase_weapon`

## Complete Code

Create a file named `shared.lua` in `garrysmod/lua/weapons/weapon_drg_myar2/`:

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
SWEP.Base = "drgbase_weapon" -- DO NOT TOUCH (obviously)

-- Misc --
SWEP.PrintName = "My Custom AR2"
SWEP.Class = "weapon_drg_myar2"
SWEP.Category = "DrGBase"
SWEP.Author = "Your Name"
SWEP.Contact = ""
SWEP.Purpose = "A custom automatic rifle"
SWEP.Instructions = "Left click to fire, R to reload"
SWEP.Spawnable = true
SWEP.AdminOnly = false
SWEP.Slot = 2
SWEP.SlotPos = 0

-- Looks --
SWEP.HoldType = "ar2"
SWEP.ViewModelFOV = 54
SWEP.ViewModelFlip = false
SWEP.ViewModelOffset = Vector(0, 0, 0)
SWEP.ViewModelAngle = Angle(0, 0, 0)
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"
SWEP.CSMuzzleFlashes = false
SWEP.CSMuzzleX = false

-- Primary Attack --

-- Shooting
SWEP.Primary.Damage = 10
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.02
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.1
SWEP.Primary.Recoil = 0.5

-- Ammo
SWEP.Primary.Ammo = "AR2"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 30
SWEP.Primary.DefaultClip = 90

-- Effects
SWEP.Primary.Sound = "Weapon_AR2.Single"
SWEP.Primary.EmptySound = "Weapon_AR2.Empty"

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

## Code Breakdown

### File Structure

```lua
if not DrGBase then return end -- return if DrGBase isn't installed
SWEP.Base = "drgbase_weapon" -- DO NOT TOUCH (obviously)
```

**Purpose:** Safety check and base class definition.
- Prevents errors if DrGBase isn't installed
- Sets base class to inherit all DrGBase weapon functionality
- **Note:** Unlike NPCs (ENT.Base), weapons use SWEP.Base

### Spawn Menu Properties

```lua
SWEP.PrintName = "My Custom AR2"
SWEP.Class = "weapon_drg_myar2"
SWEP.Category = "DrGBase"
SWEP.Author = "Your Name"
SWEP.Contact = ""
SWEP.Purpose = "A custom automatic rifle"
SWEP.Instructions = "Left click to fire, R to reload"
SWEP.Spawnable = true
SWEP.AdminOnly = false
SWEP.Slot = 2
SWEP.SlotPos = 0
```

**Purpose:** Define how weapon appears in spawn menu and HUD.
- `PrintName`: Display name in spawn menu
- `Class`: Unique identifier (must match folder name minus "weapon_" prefix)
- `Category`: Spawn menu category
- `Author/Contact/Purpose/Instructions`: Info shown when hovering in spawn menu
- `Spawnable`: Whether it appears in spawn menu
- `AdminOnly`: Restrict to admins only
- `Slot`: Weapon slot (1-6, typically 1=melee, 2=pistols, 3=shotguns, 4=SMGs, 5=rifles, 6=explosives)
- `SlotPos`: Position within slot

### Visual Configuration

```lua
SWEP.HoldType = "ar2"
SWEP.ViewModelFOV = 54
SWEP.ViewModelFlip = false
SWEP.ViewModelOffset = Vector(0, 0, 0)
SWEP.ViewModelAngle = Angle(0, 0, 0)
SWEP.UseHands = true
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"
SWEP.CSMuzzleFlashes = false
SWEP.CSMuzzleX = false
```

**Purpose:** Control weapon appearance and animations.
- `HoldType`: Animation set for holding weapon (ar2, pistol, smg, shotgun, rpg, crossbow, melee, etc.)
- `ViewModelFOV`: Field of view for first-person view (54 is standard)
- `ViewModelFlip`: Mirror viewmodel (for left-handed models)
- `ViewModelOffset/Angle`: Adjust viewmodel position and rotation
- `UseHands`: Show player's hands (requires c_ models with hands)
- `ViewModel`: First-person model (c_ models)
- `WorldModel`: Third-person model (w_ models)
- `CSMuzzleFlashes/CSMuzzleX`: Counter-Strike style muzzle flashes (usually false)

### Damage and Firing

```lua
SWEP.Primary.Damage = 10
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.02
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.1
SWEP.Primary.Recoil = 0.5
```

**Purpose:** Configure weapon's combat properties.
- `Damage`: Damage per bullet (10 = moderate)
- `Bullets`: Number of bullets per shot (1 for rifles, 8+ for shotguns)
- `Spread`: Accuracy cone (0.02 = accurate, 0.1 = inaccurate)
- `Automatic`: Hold to fire (true) vs click to fire (false)
- `Delay`: Time between shots in seconds (0.1 = 10 shots per second)
- `Recoil`: Camera kick amount (0.5 = moderate kick)

### Ammo System

```lua
SWEP.Primary.Ammo = "AR2"
SWEP.Primary.Cost = 1
SWEP.Primary.ClipSize = 30
SWEP.Primary.DefaultClip = 90
```

**Purpose:** Configure ammunition.
- `Ammo`: Ammo type ("Pistol", "SMG1", "AR2", "Buckshot", "357", "XBowBolt", etc.)
- `Cost`: Ammo consumed per shot (1 for single bullets, can be more)
- `ClipSize`: Magazine capacity (30 = standard rifle magazine)
- `DefaultClip`: Total ammo when picked up (usually 3x clip size)

### Sound Effects

```lua
SWEP.Primary.Sound = "Weapon_AR2.Single"
SWEP.Primary.EmptySound = "Weapon_AR2.Empty"
```

**Purpose:** Audio feedback.
- `Sound`: Gunfire sound (can be string or table of sounds)
- `EmptySound`: Click sound when magazine is empty

Available sound types:
- `"Weapon_AR2.Single"` - Rifle shot
- `"Weapon_Pistol.Single"` - Pistol shot
- `"Weapon_SMG1.Single"` - SMG shot
- `"Weapon_Shotgun.Single"` - Shotgun blast
- `"Weapon_357.Single"` - Magnum shot
- Or custom: `"path/to/your/sound.wav"`

### File Footer

```lua
AddCSLuaFile()
DrGBase.AddWeapon(SWEP)
```

**Purpose:** Register weapon with system.
- `AddCSLuaFile()`: Send this file to clients
- `DrGBase.AddWeapon(SWEP)`: Register with DrGBase framework

## Testing the Weapon

### Spawning

1. Start Garry's Mod
2. Open spawn menu (Q)
3. Go to Weapons tab
4. Find "DrGBase" category
5. Click "My Custom AR2"

### Console Commands

```lua
-- Give yourself the weapon
give weapon_drg_myar2

-- Give to an NPC
-- 1. Spawn an NPC (like npc_citizen)
-- 2. Look at them and type:
impulse 101  -- Gives weapons to targeted NPC
```

### Behavior to Test

- **Firing**: Hold left mouse button - should fire automatically
- **Reloading**: Press R when magazine is low
- **Ammo**: Check HUD ammo counter (bottom right)
- **Accuracy**: Shoot at wall - should have tight spread
- **Sounds**: Should hear firing sound and empty click
- **Visuals**: Muzzle flash should appear
- **NPCs**: Give to NPC - they should use it against enemies

### Testing Tips

- Spawn `func_breakable` props to test damage
- Use `impulse 101` to test with NPCs
- Check console for any Lua errors
- Compare with default HL2 AR2 for reference

## Customization Ideas

### Burst Fire Weapon

```lua
-- Change to burst fire (3-round burst)
SWEP.Primary.Automatic = false

-- Add custom fire function (in lua/weapons/weapon_drg_myweapon/shared.lua)
if SERVER then
	function SWEP:CustomPrimaryAttack()
		-- Fire 3 bullets in quick succession
		for i = 1, 3 do
			self:Timer(i * 0.05, function()
				if IsValid(self) and IsValid(self:GetOwner()) then
					self:DefaultPrimaryAttack()
				end
			end)
		end
	end
end
```

### Shotgun

```lua
-- Shotgun configuration
SWEP.Primary.Damage = 8
SWEP.Primary.Bullets = 12  -- 12 pellets per shot
SWEP.Primary.Spread = 0.08  -- Wide spread
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 0.8  -- Slow fire rate
SWEP.Primary.Recoil = 2.5  -- High recoil

SWEP.Primary.Ammo = "Buckshot"
SWEP.Primary.ClipSize = 8
SWEP.Primary.DefaultClip = 32

SWEP.Primary.Sound = "Weapon_Shotgun.Single"

-- Change models and holdtype
SWEP.HoldType = "shotgun"
SWEP.ViewModel = "models/weapons/c_shotgun.mdl"
SWEP.WorldModel = "models/weapons/w_shotgun.mdl"
```

### Sniper Rifle

```lua
-- Sniper configuration
SWEP.Primary.Damage = 75  -- High damage
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.001  -- Very accurate
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 1.5  -- Very slow fire rate
SWEP.Primary.Recoil = 4  -- High recoil

SWEP.Primary.Ammo = "SniperRound"
SWEP.Primary.ClipSize = 5
SWEP.Primary.DefaultClip = 20

SWEP.Primary.Sound = "Weapon_AWP.Single"

-- Use crossbow models as placeholder
SWEP.HoldType = "ar2"
SWEP.ViewModel = "models/weapons/c_irifle.mdl"
SWEP.WorldModel = "models/weapons/w_irifle.mdl"
```

### Pistol

```lua
-- Pistol configuration
SWEP.PrintName = "My Pistol"
SWEP.Slot = 1  -- Pistol slot

SWEP.Primary.Damage = 12
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.03
SWEP.Primary.Automatic = false
SWEP.Primary.Delay = 0.15
SWEP.Primary.Recoil = 0.8

SWEP.Primary.Ammo = "Pistol"
SWEP.Primary.ClipSize = 18
SWEP.Primary.DefaultClip = 72

SWEP.Primary.Sound = "Weapon_Pistol.Single"

SWEP.HoldType = "pistol"
SWEP.ViewModel = "models/weapons/c_pistol.mdl"
SWEP.WorldModel = "models/weapons/w_pistol.mdl"
```

### Laser Weapon

```lua
-- Energy weapon configuration
SWEP.PrintName = "Laser Rifle"

SWEP.Primary.Damage = 15
SWEP.Primary.Bullets = 1
SWEP.Primary.Spread = 0.01
SWEP.Primary.Automatic = true
SWEP.Primary.Delay = 0.08
SWEP.Primary.Recoil = 0.2  -- Low recoil

SWEP.Primary.Ammo = "AR2"  -- Use AR2 ammo
SWEP.Primary.ClipSize = 40
SWEP.Primary.DefaultClip = 120

-- Custom laser sound
SWEP.Primary.Sound = "weapons/physcannon/physcannon_charge.wav"

-- Tracer effect
SWEP.Primary.Tracer = 1  -- Every shot shows tracer
SWEP.Primary.TracerName = "AirboatGunTracer"  -- Blue laser tracer
```

### Add Muzzle Flash Effect

```lua
-- Add in shared.lua after Primary properties
SWEP.MuzzleEffect = "MuzzleEffect"  -- Muzzle flash effect
SWEP.ShellEffect = "ShellEjectEffect"  -- Bullet shell ejection
```

### Custom Fire Behavior

Create a new function in `shared.lua`:

```lua
if SERVER then
	-- Override primary attack for custom behavior
	function SWEP:CustomPrimaryAttack()
		-- Play custom animation
		self:SendWeaponAnim(ACT_VM_PRIMARYATTACK)

		-- Create custom projectile instead of hitscan
		local owner = self:GetOwner()
		if IsValid(owner) then
			local proj = ents.Create("proj_drg_default")
			proj:SetPos(owner:GetShootPos())
			proj:SetAngles(owner:EyeAngles())
			proj:SetOwner(owner)
			proj:Spawn()

			local phys = proj:GetPhysicsObject()
			if IsValid(phys) then
				phys:SetVelocity(owner:GetAimVector() * 2000)
			end
		end

		-- Consume ammo manually
		self:TakePrimaryAmmo(1)
	end
end
```

### Explosive Bullets

```lua
if SERVER then
	function SWEP:CustomBulletCallback(attacker, tr, dmginfo)
		-- Called when bullet hits something
		if tr.Hit then
			-- Create explosion at impact point
			local explode = ents.Create("env_explosion")
			explode:SetPos(tr.HitPos)
			explode:SetOwner(attacker)
			explode:Spawn()
			explode:SetKeyValue("iMagnitude", "50")
			explode:Fire("Explode", 0, 0)
		end
	end
end
```

## Troubleshooting

### Weapon doesn't appear in spawn menu

**Problem:** File location or registration issue.

**Solutions:**
- Verify file is at `garrysmod/lua/weapons/weapon_drg_myar2/shared.lua`
- Make sure `SWEP.Spawnable = true`
- Check `SWEP.Class` matches weapon folder name (without "weapon_" prefix)
- Restart Garry's Mod after creating file
- Check console for Lua errors

### Weapon doesn't fire

**Problem:** Primary attack not configured or ammo issue.

**Solutions:**
- Verify `Primary.Damage` is greater than 0
- Check that weapon has ammo (reload or use `impulse 101`)
- Make sure `Primary.Delay` isn't too high
- Look for errors in console
- Try removing `CustomPrimaryAttack()` if you added one

### No sound when firing

**Problem:** Sound path incorrect or missing.

**Solutions:**
- Verify sound path is correct: `"Weapon_AR2.Single"`
- Try a different sound to test
- Make sure sound isn't muted in game settings
- Check if sound plays in console: `play Weapon_AR2.Single`

### Weapon does no damage

**Problem:** Damage value or bullet configuration.

**Solutions:**
- Check `Primary.Damage` is set and > 0
- Verify `Primary.Bullets` is at least 1
- Make sure target entity can take damage
- Test on different targets (props, NPCs, players)
- Add debug: `print("Damage:", self.Primary.Damage)`

### Weapon fires but bullets go nowhere

**Problem:** Spread value or owner issue.

**Solutions:**
- Check `Primary.Spread` isn't too high (> 1.0)
- Verify weapon has a valid owner
- Try shooting at close range to test
- Check `Primary.Bullets` is at least 1

### Ammo doesn't reload

**Problem:** Clip size or ammo type issue.

**Solutions:**
- Verify `Primary.ClipSize` is greater than 0
- Check `Primary.Ammo` is a valid ammo type
- Make sure you have reserve ammo (use `impulse 101` to get ammo)
- Try a different ammo type: `"Pistol"`, `"SMG1"`, `"AR2"`

### Viewmodel looks wrong

**Problem:** Model or offset configuration.

**Solutions:**
- Verify model path is correct
- Check model exists: `gmod_spawn model models/weapons/c_irifle.mdl`
- Adjust `ViewModelOffset` and `ViewModelAngle` to reposition
- Try `ViewModelFlip = true` if it's backwards
- Make sure `HoldType` matches the weapon style

### NPCs won't use weapon

**Problem:** Weapon class or configuration.

**Solutions:**
- Some weapon types require special NPC support
- Try giving NPCs the weapon via scripting:
```lua
local npc = ents.Create("npc_citizen")
npc:Spawn()
npc:Give("weapon_drg_myar2")
npc:SelectWeapon("weapon_drg_myar2")
```
- Check that `Primary.Delay` isn't too high for NPCs

### Weapon is too powerful/weak

**Problem:** Balance issues.

**Solutions:**
- Adjust `Primary.Damage` (typical: 5-15 for rifles, 50-100 for snipers)
- Modify `Primary.Spread` (lower = more accurate)
- Change `Primary.Delay` (lower = faster fire rate)
- Adjust `Primary.Bullets` for shotguns (8-12 pellets)
- Tweak `Primary.Recoil` for balance

## See Also

- [Custom Projectile Example](custom-projectile.md) - Create projectile weapons
- [Ranged NPC Example](ranged-npc.md) - Give weapons to NPCs
- [Creating NPCs Guide](../guides/creating-npcs.md) - NPC weapon integration
- [Weapon API](../api/weapon/README.md) - Complete weapon system reference
- [DrGBase Weapon Base](../../lua/weapons/drgbase_weapon/shared.lua) - Base weapon code
