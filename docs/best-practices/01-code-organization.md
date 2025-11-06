# Code Organization

Proper code organization is essential for maintaining DrGBase addons, especially as they grow in complexity. Well-organized code is easier to debug, extend, and collaborate on. This guide covers best practices for structuring your DrGBase NPCs, weapons, and other entities.

## Best Practice 1: Follow the Standard Entity Structure

Keep your entity code organized by following DrGBase's standard structure and initialization order.

### ✅ DO

```lua
if not DrGBase then return end -- Guard clause
ENT.Base = "drgbase_nextbot"

-- Group properties by category with comments
-- Misc --
ENT.PrintName = "My NPC"
ENT.Category = "My Addon"
ENT.Models = {"models/player.mdl"}

-- Stats --
ENT.SpawnHealth = 100
ENT.HealthRegen = 0

-- AI --
ENT.RangeAttackRange = 500
ENT.MeleeAttackRange = 50

-- Relationships --
ENT.Factions = {FACTION_PLAYERS}

-- Sounds --
ENT.OnDamageSounds = {"npc/sound.wav"}

if SERVER then
    function ENT:CustomInitialize()
        -- Server-only initialization
    end

    function ENT:OnMeleeAttack(enemy)
        -- Combat logic
    end
end

-- DO NOT TOUCH --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### ❌ DON'T

```lua
-- No guard clause
ENT.Base = "drgbase_nextbot"

-- Properties scattered randomly
ENT.PrintName = "My NPC"
function ENT:OnMeleeAttack(enemy)
    -- Function before properties
end
ENT.SpawnHealth = 100
ENT.OnDamageSounds = {"npc/sound.wav"}
ENT.RangeAttackRange = 500

-- Mixed server/client code without guards
function ENT:CustomInitialize()
    -- No SERVER check
end

-- Missing AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### Why This Matters

Following a consistent structure makes your code predictable and easier to navigate. Other developers (or you, six months later) can quickly find what they need. The property grouping matches DrGBase's internal organization, making it easier to understand which systems are being configured.

## Best Practice 2: Use Proper File Naming Conventions

DrGBase uses file prefixes to control file loading across realms. Understanding and using these correctly prevents bugs and improves performance.

### ✅ DO

```lua
-- File structure:
-- lua/entities/npc_mymod_soldier.lua (shared)
-- lua/entities/npc_mymod_soldier/sv_combat.lua (server-only)
-- lua/entities/npc_mymod_soldier/cl_display.lua (client-only)
-- lua/entities/npc_mymod_soldier/sh_sounds.lua (shared)

-- In main entity file:
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Properties here

if SERVER then
    -- Include server files
    include("npc_mymod_soldier/sv_combat.lua")
end

if CLIENT then
    -- Include client files
    include("npc_mymod_soldier/cl_display.lua")
end

-- Shared file (both realms)
include("npc_mymod_soldier/sh_sounds.lua")
AddCSLuaFile("npc_mymod_soldier/sh_sounds.lua")

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### ❌ DON'T

```lua
-- Bad: No prefixes, unclear what loads where
-- lua/entities/npc_mymod_soldier/combat.lua
-- lua/entities/npc_mymod_soldier/display.lua

-- Bad: Everything in one massive file
ENT.Base = "drgbase_nextbot"

-- 500 lines of mixed server/client code...

function ENT:SomeServerFunction()
    -- No SERVER guard
end

function ENT:SomeClientFunction()
    -- No CLIENT guard
end
```

### Why This Matters

Proper file naming and realm separation prevents crashes, reduces network overhead, and makes debugging easier. Server-only code should never run on clients, and client-only code often won't work on dedicated servers.

## Best Practice 3: Initialize Properties, Don't Modify Them in Initialize

ENT properties should be set as defaults at the class level, not modified in CustomInitialize unless absolutely necessary.

### ✅ DO

```lua
-- Set defaults at class level
ENT.SpawnHealth = 100
ENT.WalkSpeed = 100
ENT.RunSpeed = 200
ENT.Models = {"models/player.mdl"}

if SERVER then
    function ENT:CustomInitialize()
        -- Use setters for runtime changes
        self:SetDefaultRelationship(D_HT)

        -- Random selection from predefined options
        if math.random(2) == 1 then
            self:SetModelScale(1.5)
        end
    end
end
```

### ❌ DON'T

```lua
-- Don't initialize as nil/empty then set in Initialize
ENT.SpawnHealth = nil
ENT.WalkSpeed = nil

if SERVER then
    function ENT:CustomInitialize()
        -- Bad: Setting core properties at runtime
        self.SpawnHealth = 100
        self.WalkSpeed = 100
        self.RunSpeed = 200

        -- Bad: These won't work correctly
        self.Models = {"models/player.mdl"}
    end
end
```

### Why This Matters

DrGBase reads many properties during the base Initialize() function, which runs before CustomInitialize(). Setting properties too late means they won't be used. Class-level properties also make it easier to see an NPC's configuration at a glance.

## Best Practice 4: Group Related Functionality into Modules

For complex NPCs, split functionality into logical modules rather than one massive file.

### ✅ DO

```lua
-- lua/entities/npc_mymod_boss.lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Boss NPC"
ENT.SpawnHealth = 1000

if SERVER then
    -- Load modular components
    include("npc_mymod_boss/sv_phases.lua")
    include("npc_mymod_boss/sv_abilities.lua")
    include("npc_mymod_boss/sv_minions.lua")
    include("npc_mymod_boss/sv_loot.lua")
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)

-- lua/entities/npc_mymod_boss/sv_phases.lua
-- Phase management
function ENT:GetCurrentPhase()
    return self._CurrentPhase or 1
end

function ENT:EnterPhase(phase)
    self._CurrentPhase = phase
    self:OnPhaseChange(phase)
end

function ENT:OnPhaseChange(phase)
    -- Override in main file
end

-- lua/entities/npc_mymod_boss/sv_abilities.lua
-- Special abilities
function ENT:UseAbility(name)
    local ability = self.Abilities[name]
    if not ability then return false end
    return ability(self)
end
```

### ❌ DON'T

```lua
-- Everything in one 2000-line file
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Boss NPC"

-- Phase code
function ENT:GetCurrentPhase()
    -- ...
end

-- 50 more phase functions...

-- Ability code
function ENT:UseAbility()
    -- ...
end

-- 50 more ability functions...

-- Minion code
-- 50 more minion functions...

-- Loot code
-- 50 more loot functions...

-- Impossible to navigate or maintain
```

### Why This Matters

Large monolithic files become difficult to navigate and maintain. Breaking code into logical modules improves readability, makes collaboration easier, and allows you to reuse components across different NPCs.

## Best Practice 5: Use Consistent Naming Conventions

Follow consistent naming patterns for functions, variables, and properties throughout your addon.

### ✅ DO

```lua
-- Properties: PascalCase
ENT.SpawnHealth = 100
ENT.MeleeAttackRange = 50
ENT.CustomAbilityName = "fireball"

-- Public functions: PascalCase
function ENT:CustomInitialize()
end

function ENT:GetCurrentTarget()
    return self._currentTarget
end

function ENT:SetCustomMode(mode)
    self._customMode = mode
end

-- Private/internal: _PascalCase or _camelCase
function ENT:_InternalHelper()
    -- Internal use only
end

-- Local variables: camelCase
if SERVER then
    function ENT:OnMeleeAttack(enemy)
        local damageAmount = 10
        local attackType = DMG_SLASH
        local hitPosition = enemy:GetPos()
    end
end
```

### ❌ DON'T

```lua
-- Inconsistent naming
ENT.spawnhealth = 100  -- lowercase
ENT.MeleeAttackRange = 50  -- PascalCase
ENT.custom_ability_name = "fireball"  -- snake_case

-- Mixed function naming
function ENT:custominitialize()  -- lowercase
end

function ENT:get_current_target()  -- snake_case
    return self.currentTarget  -- camelCase
end

function ENT:SetCustomMode(Mode)  -- parameter PascalCase
    self.CUSTOM_MODE = Mode  -- SCREAMING_CASE
end
```

### Why This Matters

Consistent naming makes code more readable and professional. It also helps distinguish between DrGBase framework functions (which use PascalCase) and your custom additions.

## Best Practice 6: Comment Complex Logic and Non-Obvious Decisions

Add comments to explain why code exists, not just what it does. Document non-obvious decisions and workarounds.

### ✅ DO

```lua
function ENT:OnMeleeAttack(enemy)
    -- Play attack animation first, damage happens in OnAnimEvent
    -- This ensures the damage timing matches the visual
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        -- Damage at 30% through animation (when claw extends)
        self:Attack({damage = 10, type = DMG_SLASH})
    end
end

function ENT:CustomThink()
    -- Check for nearby allies every 2 seconds to reduce CPU usage
    -- More frequent checks caused noticeable lag with 20+ NPCs
    if CurTime() > self._nextAllyCheck then
        self._nextAllyCheck = CurTime() + 2
        self:UpdateNearbyAllies()
    end
end

function ENT:OnTakeDamage(dmg)
    -- Workaround: Source engine bug causes physics explosions to deal double damage
    -- Halve damage from blast sources
    if dmg:IsDamageType(DMG_BLAST) then
        dmg:ScaleDamage(0.5)
    end
end
```

### ❌ DON'T

```lua
function ENT:OnMeleeAttack(enemy)
    -- Play attack animation
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    -- Attack
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({damage = 10, type = DMG_SLASH})
    end
end

function ENT:CustomThink()
    -- Check allies
    if CurTime() > self._nextAllyCheck then
        self._nextAllyCheck = CurTime() + 2  -- Why 2 seconds?
        self:UpdateNearbyAllies()
    end
end

function ENT:OnTakeDamage(dmg)
    if dmg:IsDamageType(DMG_BLAST) then
        dmg:ScaleDamage(0.5)  -- Why half? This is mysterious
    end
end
```

### Why This Matters

Comments should explain the reasoning behind code, especially for timing values, workarounds, and optimization decisions. Future maintainers (including you) will understand why code was written a certain way.

## Best Practice 7: Keep Configuration Separate from Logic

Separate data and configuration from implementation logic to improve maintainability.

### ✅ DO

```lua
-- Configuration at the top
ENT.Base = "drgbase_nextbot"

-- Combat configuration
ENT.Abilities = {
    fireball = {
        damage = 50,
        radius = 200,
        cooldown = 5,
        range = 1000
    },
    lightning = {
        damage = 30,
        chain = 3,
        cooldown = 8,
        range = 800
    }
}

ENT.PhaseThresholds = {
    [1] = 0.75,  -- Phase 2 at 75% health
    [2] = 0.50,  -- Phase 3 at 50% health
    [3] = 0.25   -- Phase 4 at 25% health
}

-- Implementation uses configuration
if SERVER then
    function ENT:UseAbility(abilityName)
        local config = self.Abilities[abilityName]
        if not config then return false end

        if not self:CanUseAbility(abilityName) then
            return false
        end

        -- Implementation reads from config
        local target = self:GetEnemy()
        if self:GetRangeTo(target) > config.range then
            return false
        end

        self:DealDamage(target, config.damage, config.radius)
        self:SetAbilityCooldown(abilityName, config.cooldown)

        return true
    end

    function ENT:OnHealthChange(old, new)
        local healthPercent = new / self:GetMaxHealth()

        for phase, threshold in ipairs(self.PhaseThresholds) do
            if healthPercent <= threshold and self:GetPhase() < phase + 1 then
                self:EnterPhase(phase + 1)
            end
        end
    end
end
```

### ❌ DON'T

```lua
-- Configuration mixed with logic
if SERVER then
    function ENT:UseFireball()
        local target = self:GetEnemy()

        -- Magic numbers scattered throughout
        if self:GetRangeTo(target) > 1000 then
            return false
        end

        if CurTime() < self._fireballCooldown then
            return false
        end

        self:DealDamage(target, 50, 200)
        self._fireballCooldown = CurTime() + 5

        return true
    end

    function ENT:UseLightning()
        -- Duplicate structure with different hardcoded values
        local target = self:GetEnemy()

        if self:GetRangeTo(target) > 800 then
            return false
        end

        if CurTime() < self._lightningCooldown then
            return false
        end

        self:ChainLightning(target, 30, 3)
        self._lightningCooldown = CurTime() + 8

        return true
    end

    function ENT:OnHealthChange(old, new)
        local hp = new / self:GetMaxHealth()

        -- Phase thresholds hardcoded in logic
        if hp <= 0.75 and self:GetPhase() < 2 then
            self:EnterPhase(2)
        elseif hp <= 0.50 and self:GetPhase() < 3 then
            self:EnterPhase(3)
        elseif hp <= 0.25 and self:GetPhase() < 4 then
            self:EnterPhase(4)
        end
    end
end
```

### Why This Matters

Separating configuration from logic makes it easier to balance your NPCs, reuse code, and create variants. When all the values are at the top, they're easy to find and modify. When they're scattered through implementation, you'll miss some when trying to adjust behavior.

## Best Practice 8: Use DrGBase's Include System

Take advantage of DrGBase's built-in file inclusion system for cleaner code organization.

### ✅ DO

```lua
-- In your addon initialization (lua/autorun/mymod_init.lua)
if SERVER then
    AddCSLuaFile()
end

-- Use DrGBase's include system
DrGBase.IncludeFolder("mymod/shared")
DrGBase.IncludeFolder("mymod/entities")

if SERVER then
    DrGBase.IncludeFolder("mymod/sv_systems")
end

if CLIENT then
    DrGBase.IncludeFolder("mymod/cl_ui")
end

-- Or for specific files with automatic realm handling
DrGBase.IncludeFile("mymod/sv_database.lua")  -- Server-only
DrGBase.IncludeFile("mymod/cl_hud.lua")       -- Client-only
DrGBase.IncludeFile("mymod/sh_config.lua")    -- Shared
```

### ❌ DON'T

```lua
-- Manual inclusion without using DrGBase helpers
if SERVER then
    include("mymod/sv_database.lua")
    include("mymod/shared/sh_config.lua")
    AddCSLuaFile("mymod/shared/sh_config.lua")
elseif CLIENT then
    include("mymod/shared/sh_config.lua")
    include("mymod/cl_ui.lua")
end

-- Or even worse: no autorun, relying on specific load order
```

### Why This Matters

DrGBase's include system handles AddCSLuaFile automatically for shared files and provides consistent file loading. It also provides helpful console output showing which files are being loaded, making debugging easier.

## Tips and Recommendations

1. **Start Simple**: Begin with a basic NPC structure and add features incrementally. Don't try to build everything at once.

2. **Use Templates**: Create template NPCs that you can copy and modify for new variants. This ensures consistency across your addon.

3. **Version Control Friendly**: Organize code so files can be easily tracked in git. Avoid having all code in one file.

4. **Documentation**: Create a README.md for your addon explaining its structure and how to extend it.

5. **Consistent Prefixes**: Use a consistent prefix for all your entities (e.g., `npc_mymod_`, `weapon_mymod_`) to avoid conflicts with other addons.

6. **Test Organization**: Keep your test/example NPCs in a separate folder or mark them clearly so they can be easily removed for release.

## Real-World Example: Well-Organized Addon Structure

```
lua/
  autorun/
    mymod_init.lua                    # Main initialization

  entities/
    npc_mymod_soldier.lua             # Basic soldier NPC
    npc_mymod_heavy.lua               # Heavy variant

    npc_mymod_boss/
      shared.lua                      # Main boss file
      sv_phases.lua                   # Phase management
      sv_abilities.lua                # Special abilities
      sv_minions.lua                  # Minion spawning
      cl_effects.lua                  # Client effects

  mymod/
    shared/
      sh_config.lua                   # Shared configuration
      sh_factions.lua                 # Faction definitions
      sh_enums.lua                    # Custom enumerations

    sv_systems/
      sv_spawning.lua                 # Spawn system
      sv_director.lua                 # AI director

    cl_ui/
      cl_hud.lua                      # HUD elements
      cl_menus.lua                    # Configuration menus

materials/
  mymod/
    icons/                            # Entity icons

models/
  mymod/
    soldier.mdl                       # Custom models

sound/
  mymod/
    soldier/                          # Sound effects
```

This structure is clear, scalable, and follows all the best practices outlined in this guide.
