# Code Organization Best Practices

Proper code organization makes your DrGBase addon maintainable, debuggable, and professional.

## File Structure

### Recommended Addon Structure
```
your_addon/
├── lua/
│   ├── autorun/
│   │   ├── your_addon_init.lua (shared initialization)
│   │   └── server/
│   │       └── sv_your_addon.lua (server initialization)
│   ├── entities/
│   │   ├── npc_your_zombie.lua
│   │   ├── npc_your_soldier.lua
│   │   └── proj_your_fireball.lua
│   ├── weapons/
│   │   └── weapon_your_gun.lua
│   └── your_addon/
│       ├── config.lua (shared configuration)
│       ├── factions.lua (faction definitions)
│       ├── server/
│       │   ├── sv_spawning.lua
│       │   └── sv_database.lua
│       └── client/
│           └── cl_hud.lua
├── materials/
│   └── your_addon/
│       └── sprites/
├── models/
│   └── your_addon/
└── sound/
    └── your_addon/
```

## Entity Organization

### One File Per Entity
```lua
-- Good: lua/entities/npc_your_zombie.lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"
-- ... entity code ...
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### Group Related Functionality
```lua
-- Group properties by purpose
-- Misc
ENT.PrintName = "Zombie"
ENT.Category = "My NPCs"

-- Stats
ENT.SpawnHealth = 100
ENT.HealthRegen = 0

-- AI
ENT.MeleeAttackRange = 50
ENT.RangeAttackRange = 0

-- Relationships
ENT.Factions = {FACTION_ZOMBIES}
```

## Code Style

### Naming Conventions
```lua
-- Good naming
ENT.MeleeAttackRange = 50    -- Clear, descriptive
self.NextAttackTime = 0      -- Clear purpose
local enemyPos = enemy:GetPos()  -- Clear variable name

-- Bad naming
ENT.MAR = 50                 -- Unclear abbreviation
self.t = 0                   -- Meaningless
local p = enemy:GetPos()     -- Too short
```

### Comments
```lua
-- Good comments
function ENT:OnMeleeAttack(enemy)
    -- Play attack animation and sound
    self:EmitSound("Zombie.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

-- Bad comments
function ENT:OnMeleeAttack(enemy)
    -- Attack
    self:EmitSound("Zombie.Attack")  -- Sound
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)  -- Animation
end
```

### Consistent Indentation
```lua
-- Good: Consistent 4-space or tab indentation
if SERVER then

    function ENT:CustomInitialize()
        if self:GetModel() == "models/zombie.mdl" then
            self:SetBodygroup(1, 1)
        end
    end

end
```

## Modularity

### Extract Common Logic
```lua
-- Good: Reusable helper
function ENT:DealMeleeDamage(damage, damageType)
    self:Attack({
        damage = damage,
        type = damageType or DMG_SLASH,
        viewpunch = Angle(10, math.random(-5, 5), 0)
    })
end

function ENT:OnMeleeAttack(enemy)
    self:DealMeleeDamage(15)
end

-- Bad: Repeated code
function ENT:OnMeleeAttack(enemy)
    self:Attack({
        damage = 15,
        type = DMG_SLASH,
        viewpunch = Angle(10, math.random(-5, 5), 0)
    })
end

function ENT:OnSpecialAttack(enemy)
    self:Attack({
        damage = 25,
        type = DMG_SLASH,
        viewpunch = Angle(10, math.random(-5, 5), 0)
    })
end
```

### Shared Code in Separate Files
```lua
-- lua/your_addon/shared_functions.lua
function YourAddon.PlayHitSound(ent, soundType)
    if soundType == "flesh" then
        ent:EmitSound("physics/flesh/flesh_impact_hard" .. math.random(1, 5) .. ".wav")
    elseif soundType == "metal" then
        ent:EmitSound("physics/metal/metal_solid_impact_hard" .. math.random(1, 5) .. ".wav")
    end
end

-- Use in multiple entities
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({damage = 15}, function(self, hit)
            if #hit > 0 then
                YourAddon.PlayHitSound(self, "flesh")
            end
        end)
    end
end
```

## Configuration

### Centralized Config
```lua
-- lua/your_addon/config.lua
YourAddon = YourAddon or {}
YourAddon.Config = {
    -- Difficulty
    DifficultyMultiplier = 1.0,
    EnableHealthScaling = true,

    -- Gameplay
    EnableFriendlyFire = false,
    RespawnEnabled = true,
    RespawnTime = 30,

    -- Performance
    MaxNPCsPerPlayer = 10,
    AIUpdateRate = 30,
}

-- Use in entities
function ENT:CustomInitialize()
    local mult = YourAddon.Config.DifficultyMultiplier
    self:SetHealth(self.SpawnHealth * mult)
end
```

## Error Handling

### Validate Inputs
```lua
-- Good: Validate before use
function ENT:SetCustomTarget(target)
    if not IsValid(target) then
        ErrorNoHalt("[DrGBase] Invalid target provided to SetCustomTarget\n")
        return false
    end

    self.CustomTarget = target
    return true
end

-- Bad: No validation
function ENT:SetCustomTarget(target)
    self.CustomTarget = target
end
```

### Safe Function Calls
```lua
-- Good: Check existence
if self.OnCustomEvent then
    self:OnCustomEvent(data)
end

-- Better: pcall for complex operations
local success, err = pcall(function()
    self:OnCustomEvent(data)
end)

if not success then
    ErrorNoHalt("[DrGBase] Error in OnCustomEvent: " .. tostring(err) .. "\n")
end
```

## Tips

1. **Keep entities under 500 lines** - Split large entities into multiple files
2. **Use meaningful names** - Code should be self-documenting
3. **Comment complex logic** - Explain why, not what
4. **Group related code** - Keep initialization together, AI together, etc.
5. **Avoid global pollution** - Use local variables when possible
6. **Follow Lua conventions** - camelCase for variables, PascalCase for tables

## Anti-Patterns to Avoid

### Global Variables
```lua
-- Bad: Pollutes global namespace
myNPCData = {}

-- Good: Scoped to your addon
YourAddon.NPCData = {}
```

### Magic Numbers
```lua
-- Bad: Magic numbers
if self:Health() < 50 then
    self:Retreat()
end

-- Good: Named constants
ENT.RetreatHealthThreshold = 50

if self:Health() < self.RetreatHealthThreshold then
    self:Retreat()
end
```

### Deep Nesting
```lua
-- Bad: Too deeply nested
if IsValid(enemy) then
    if enemy:Health() > 0 then
        if self:GetPos():Distance(enemy:GetPos()) < 100 then
            if CurTime() > self.NextAttack then
                self:Attack()
            end
        end
    end
end

-- Good: Early returns
if not IsValid(enemy) then return end
if enemy:Health() <= 0 then return end
if self:GetPos():Distance(enemy:GetPos()) >= 100 then return end
if CurTime() <= self.NextAttack then return end

self:Attack()
```
