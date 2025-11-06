# Status & Health System

Health management and regeneration.

**File:** `status.lua` (205 lines)

## Overview

The status system manages NPC health, regeneration, scaling, down/death states, and god mode. It provides enhanced health management beyond Source Engine's basic system with automatic health regeneration, model scaling, and fine-grained health control.

**Key Features:**
- **Health Management** - Get/set health with clamping and validation
- **Health Regeneration** - Automatic health regeneration over time
- **Down/Death States** - Distinct states for downed, dying, and dead
- **Model Scaling** - Scale NPC size with automatic stat adjustments
- **God Mode** - Invulnerability toggle
- **Networked Health** - Client-side health display synchronization

## Health Management

**Basic Health Functions:**
```lua
-- Get health (networked to clients)
local hp = self:Health()

-- Set health (clamped to 0-MaxHealth by default)
self:SetHealth(100)

-- Set without clamping
self:SetHealth(150, false)

-- Add/remove health
self:AddHealth(50)
self:RemoveHealth(25)

-- Get/set max health
local maxHP = self:GetMaxHealth()
self:SetMaxHealth(200)

-- Scale both health and max health
self:ScaleHealth(1.5)  -- 1.5x health and max health
```

## Health Regeneration

NPCs can automatically regenerate (or degenerate) health over time.

**Regeneration:**
```lua
-- Set regeneration rate (HP per second)
self:SetHealthRegen(5)  -- Regenerate 5 HP/s

-- Negative values cause degeneration
self:SetHealthRegen(-2)  -- Lose 2 HP/s (damage over time)

-- Get current regen rate
local regen = self:GetHealthRegen()
```

**Regenerate to Target (Blocking):**
```lua
-- Regenerate to 200 HP over 5 seconds
self:RegenHealth(200, 5)

-- With callback
self:RegenHealth(200, 5, function(self, currentHP)
    print("HP: " .. currentHP)

    if self:HasEnemy() then
        return true  -- Stop regenerating
    end
end)

-- Instant heal (0 duration)
self:RegenHealth(200, 0)
```

## Down/Death States

NPCs can be in several health states:

**States:**
- **Alive** - Normal state (`IsAlive()`)
- **Down** - Incapacitated but can be revived (`IsDown()` / `IsDowned()`)
- **Dying** - Currently dying animation/process (`IsDying()`)
- **Dead** - Fully dead (`IsDead()`)

**State Queries:**
```lua
if self:IsAlive() then end
if self:IsDown() then end   -- Incapacitated
if self:IsDying() then end  -- Death in progress
if self:IsDead() then end   -- Fully dead

-- Get down count (how many times downed)
local downs = self:GetDowned()
```

**Setting States:**

These states are typically set by death/damage hooks, but can be manipulated:
```lua
self:SetNW2Bool("DrGBaseDown", true)    -- Mark as down
self:SetNW2Bool("DrGBaseDying", true)   -- Mark as dying
self:SetNW2Bool("DrGBaseDead", true)    -- Mark as dead
```

## Model Scaling

Scale the NPC's model size with automatic stat adjustments.

**Scaling:**
```lua
-- Set absolute scale
self:SetScale(2)  -- 2x size

-- Set with animation duration
self:SetScale(0.5, 1)  -- 0.5x size over 1 second

-- Relative scaling (multiply current scale)
self:Scale(1.5)  -- 1.5x current size

-- Get current scale
local scale = self:GetScale()
```

**Scale Effects:**
- Movement speed scales with size
- Collision hull scales
- Model visually scales
- Physics shadow updates

## God Mode

Make NPCs invulnerable to damage.

**God Mode Control:**
```lua
-- Enable god mode
self:SetGodMode(true)
self:EnableGodMode()

-- Disable god mode
self:SetGodMode(false)
self:DisableGodMode()

-- Check god mode
if self:GetGodMode() then end
```

**Behavior:**
- God mode NPCs take no damage
- Still trigger `OnTakeDamage()` hook
- Health cannot decrease
- Visual/audio effects still play

## Configuration

**Health Properties:**
```lua
ENT.MaxHealth = 100  -- Starting max health (set in SetupDataTables)
```

**Setting Initial Health:**
```lua
function ENT:SetupDataTables()
    self:SetMaxHealth(200)
    self:SetHealth(200)
end
```

## Usage Examples

**Health Regeneration:**
```lua
function ENT:Initialize()
    -- Regenerate 2 HP per second
    self:SetHealthRegen(2)
end
```

**Heal Over Time:**
```lua
function ENT:OnReachPatrol()
    -- Heal to full over 10 seconds
    self:RegenHealth(self:GetMaxHealth(), 10)
end
```

**Poison/Damage Over Time:**
```lua
function ENT:OnTakeDamage(dmg)
    if dmg:GetDamageType() == DMG_POISON then
        -- Poison effect: lose 5 HP/s for 10 seconds
        self:SetHealthRegen(-5)
        timer.Simple(10, function()
            if IsValid(self) then
                self:SetHealthRegen(0)
            end
        end)
    end
end
```

**Boss Scaling:**
```lua
function ENT:Initialize()
    -- Boss is 2x size
    self:SetScale(2)

    -- Scale health accordingly
    self:ScaleHealth(2)  -- 2x health
end
```

**Dynamic Scaling:**
```lua
function ENT:OnTakeDamage(dmg)
    -- Shrink when damaged
    if self:Health() < self:GetMaxHealth() * 0.5 then
        self:SetScale(0.75, 0.5)  -- Shrink to 75% over 0.5s
    end
end
```

**Down State (Incapacitation):**
```lua
function ENT:OnTakeDamage(dmg)
    if self:Health() <= 0 and not self:IsDown() then
        self:SetNW2Bool("DrGBaseDown", true)
        self:PlaySequenceAndWait("downed")

        -- Revive after 5 seconds
        timer.Simple(5, function()
            if IsValid(self) then
                self:SetNW2Bool("DrGBaseDown", false)
                self:SetHealth(50)
            end
        end)
    end
end
```

**God Mode Boss Phase:**
```lua
function ENT:OnTakeDamage(dmg)
    if self:Health() < self:GetMaxHealth() * 0.25 then
        -- Enter invulnerable phase
        self:EnableGodMode()
        self:EmitSound("npc/scanner/scanner_combat_loop1.wav")

        -- Spawn minions, then disable
        timer.Simple(10, function()
            if IsValid(self) then
                self:DisableGodMode()
            end
        end)
    end
end
```

**Health Hooks:**
```lua
function ENT:OnHealthChange(oldHP, newHP)
    print("Health changed: " .. oldHP .. " -> " .. newHP)

    -- React to damage
    if newHP < oldHP then
        self:EmitSound("npc/zombie/zombie_pain" .. math.random(1,6) .. ".wav")
    end
end
```

## Best Practices

**Health Management:**
- Always use `SetHealth(hp, true)` to clamp within valid range
- Set `MaxHealth` in `SetupDataTables()` before `SetHealth()`
- Use `AddHealth()` / `RemoveHealth()` for relative changes
- Check `IsDead()` before applying health changes

**Regeneration:**
- Use `SetHealthRegen()` for passive regeneration
- Use `RegenHealth()` for scripted healing sequences
- Negative regen for damage-over-time effects
- Remember to reset regen to 0 when effect ends

**Scaling:**
- Scale health when scaling model size for consistency
- Use animated scaling (`SetScale(scale, duration)`) for visual feedback
- Be aware: large NPCs have larger collision and may get stuck
- Small NPCs may have trouble with navigation

**States:**
- Check `IsDead()` before processing behavior
- Use `IsDown()` for incapacitation mechanics
- `IsDying()` can block actions during death animation
- Networked states (`SetNW2Bool`) sync to clients

**Performance:**
- Health updates trigger `OnHealthChange()` hook
- Client health is synced via NW2Var
- Avoid excessive `SetHealth()` calls in Think hooks
- Regeneration is calculated once per second (internal timer)
