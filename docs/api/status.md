# Status System API Reference

The status system manages entity health, scaling, status effects, and life state.

**Source Files:**
- `lua/entities/drgbase_nextbot/status.lua`

---

## Health Management

### ENT:Health
```lua
number ENT:Health()
```
Gets the current health. Networked for clients.

**Realm:** SHARED
**Returns:** (number) Current health
**Example:**
```lua
if self:Health() < self:GetMaxHealth() / 2 then
    print("Low health!")
end
```

---

### ENT:GetMaxHealth
```lua
number ENT:GetMaxHealth()
```
Gets the maximum health. Networked for clients.

**Realm:** SHARED
**Returns:** (number) Maximum health
**Example:**
```lua
local healthPercent = self:Health() / self:GetMaxHealth()
```

---

### ENT:SetHealth
```lua
ENT:SetHealth(number health, [boolean clamp])
```
Sets the current health.

**Realm:** SERVER
**Parameters:**
- `health` (number) - New health value
- `clamp` (boolean) - If true, clamps to [0, MaxHealth] range (default: false)
**Example:**
```lua
-- Set exact health
self:SetHealth(100)

-- Set health with clamping
self:SetHealth(9999, true) -- Will not exceed MaxHealth
```

---

### ENT:SetMaxHealth
```lua
ENT:SetMaxHealth(number maxHealth)
```
Sets the maximum health.

**Realm:** SERVER
**Parameters:**
- `maxHealth` (number) - New maximum health
**Example:**
```lua
self:SetMaxHealth(200)
```

---

### ENT:AddHealth
```lua
ENT:AddHealth(number health)
```
Adds health (automatically clamped to MaxHealth).

**Realm:** SERVER
**Parameters:**
- `health` (number) - Amount to add
**Example:**
```lua
self:AddHealth(25) -- Heal 25 HP
```

---

### ENT:RemoveHealth
```lua
ENT:RemoveHealth(number health)
```
Removes health (automatically clamped to 0).

**Realm:** SERVER
**Parameters:**
- `health` (number) - Amount to remove
**Example:**
```lua
self:RemoveHealth(10) -- Take 10 damage
```

---

### ENT:ScaleHealth
```lua
ENT:ScaleHealth(number scale)
```
Scales both current and maximum health by a multiplier.

**Realm:** SERVER
**Parameters:**
- `scale` (number) - Scale multiplier (clamped to [0, infinity])
**Example:**
```lua
-- Double health
self:ScaleHealth(2)

-- Halve health
self:ScaleHealth(0.5)
```

---

## Health Regeneration

### ENT:GetHealthRegen
```lua
number ENT:GetHealthRegen()
```
Gets the current health regeneration rate per second.

**Realm:** SHARED
**Returns:** (number) HP per second (can be negative for degeneration)
**Example:**
```lua
if self:GetHealthRegen() > 0 then
    print("Regenerating health!")
end
```

---

### ENT:SetHealthRegen
```lua
ENT:SetHealthRegen(number regen)
```
Sets the health regeneration rate per second.

**Realm:** SERVER
**Parameters:**
- `regen` (number) - HP per second (negative = degeneration)
**Example:**
```lua
-- Regenerate 5 HP per second
self:SetHealthRegen(5)

-- Take 2 damage per second
self:SetHealthRegen(-2)

-- Stop regeneration
self:SetHealthRegen(0)
```

---

### ENT:RegenHealth
```lua
ENT:RegenHealth(number health, number duration, [function callback])
```
Regenerates health to a target value over time. Blocks until complete or interrupted.

**Realm:** SERVER
**Parameters:**
- `health` (number) - Target health (clamped to MaxHealth)
- `duration` (number) - Time in seconds (0 = instant)
- `callback` (function) - Called each frame: `function(ENT self, number currentHealth)`. Return true to stop early.
**Example:**
```lua
-- Regenerate to full health over 5 seconds
self:RegenHealth(self:GetMaxHealth(), 5)

-- With callback
self:RegenHealth(100, 3, function(self, hp)
    if self:GetEnemy() then
        return true -- Stop if we see an enemy
    end
end)

-- Instant heal
self:RegenHealth(self:GetMaxHealth(), 0)
```

**Note:** Restores previous health regen rate after completion.

---

## Life State

### ENT:IsAlive
```lua
boolean ENT:IsAlive()
```
Checks if the entity is alive.

**Realm:** SHARED
**Returns:** (boolean) True if not dead or dying
**Example:**
```lua
if self:IsAlive() then
    self:Think()
end
```

---

### ENT:Alive
```lua
boolean ENT:Alive()
```
Alias for `IsAlive()`.

**Realm:** SHARED
**Returns:** (boolean) True if not dead or dying

---

### ENT:IsDead
```lua
boolean ENT:IsDead()
```
Checks if the entity is dead or currently dying.

**Realm:** SHARED
**Returns:** (boolean) True if dead or in death animation
**Example:**
```lua
if not self:IsDead() then
    self:TakeDamage(10)
end
```

---

### ENT:IsDying
```lua
boolean ENT:IsDying()
```
Checks if the entity is currently playing death animation.

**Realm:** SHARED
**Returns:** (boolean) True if in death animation (before becoming fully dead)
**Example:**
```lua
if self:IsDying() then
    -- Can't be interrupted during death animation
    return
end
```

---

### ENT:IsDown
```lua
boolean ENT:IsDown()
```
Checks if the entity is downed (incapacitated but not dead).

**Realm:** SHARED
**Returns:** (boolean) True if downed
**Example:**
```lua
if self:IsDown() then
    -- Play downed animation
    return ACT_DIEBACKWARD
end
```

---

### ENT:IsDowned
```lua
boolean ENT:IsDowned()
```
Alias for `IsDown()`.

**Realm:** SHARED
**Returns:** (boolean) True if downed

---

### ENT:GetDowned
```lua
number ENT:GetDowned()
```
Gets the number of times the entity has been downed.

**Realm:** SHARED
**Returns:** (number) Down count
**Example:**
```lua
local downCount = self:GetDowned()
if downCount >= 3 then
    -- Permanently kill after 3 downs
    self:Remove()
end
```

---

## Scaling

### ENT:GetScale
```lua
number ENT:GetScale()
```
Gets the entity's scale multiplier.

**Realm:** SHARED
**Returns:** (number) Current scale (1 = normal size)
**Example:**
```lua
local scale = self:GetScale()
print("Entity is " .. (scale * 100) .. "% normal size")
```

---

### ENT:SetScale
```lua
ENT:SetScale(number scale, [number delta])
```
Sets the entity's scale. Updates model scale, physics, and speed.

**Realm:** SERVER
**Parameters:**
- `scale` (number) - New scale multiplier
- `delta` (number) - Interpolation time in seconds (default: instant)
**Example:**
```lua
-- Instant scale
self:SetScale(2) -- Double size

-- Smooth scale over 1 second
self:SetScale(0.5, 1) -- Shrink to half size
```

**Side Effects:**
- Updates model scale: `ModelScale * scale`
- Recalculates physics shadow
- Updates movement speed

---

### ENT:Scale
```lua
ENT:Scale(number mult, [number delta])
```
Multiplies the current scale by a factor.

**Realm:** SERVER
**Parameters:**
- `mult` (number) - Scale multiplier
- `delta` (number) - Interpolation time in seconds
**Example:**
```lua
-- Double current size
self:Scale(2)

-- Shrink to 75% over 0.5 seconds
self:Scale(0.75, 0.5)
```

---

## God Mode

### ENT:GetGodMode
```lua
boolean ENT:GetGodMode()
```
Checks if god mode is enabled.

**Realm:** SHARED
**Returns:** (boolean) True if invulnerable
**Example:**
```lua
if self:GetGodMode() then
    print("Invulnerable!")
end
```

---

### ENT:SetGodMode
```lua
ENT:SetGodMode(boolean enabled)
```
Sets god mode state.

**Realm:** SERVER
**Parameters:**
- `enabled` (boolean) - Enable/disable god mode
**Example:**
```lua
self:SetGodMode(true) -- Enable invulnerability
```

---

### ENT:EnableGodMode
```lua
ENT:EnableGodMode()
```
Enables god mode (invulnerability).

**Realm:** SERVER
**Example:**
```lua
self:EnableGodMode()
```

---

### ENT:DisableGodMode
```lua
ENT:DisableGodMode()
```
Disables god mode.

**Realm:** SERVER
**Example:**
```lua
self:DisableGodMode()
```

---

## Ground Detection (CLIENT)

### ENT:OnGround / ENT:IsOnGround
```lua
boolean ENT:OnGround()
boolean ENT:IsOnGround()
```
Checks if the entity is on the ground. Networked for clients.

**Realm:** CLIENT (overridden for DrGBase nextbots)
**Returns:** (boolean) True if on ground
**Example:**
```lua
if CLIENT and self:IsOnGround() then
    -- Draw ground shadow
end
```

---

## Hooks

### ENT:OnHealthChange
```lua
ENT:OnHealthChange(number oldHealth, number newHealth)
```
**Hook** called when health changes.

**Realm:** SERVER
**Parameters:**
- `oldHealth` (number) - Previous health value
- `newHealth` (number) - New health value
**Example:**
```lua
function ENT:OnHealthChange(oldHealth, newHealth)
    if newHealth < oldHealth then
        print("Took " .. (oldHealth - newHealth) .. " damage!")
    else
        print("Healed " .. (newHealth - oldHealth) .. " HP!")
    end

    -- Trigger low health behavior
    if newHealth < self:GetMaxHealth() * 0.25 then
        self:EmitSound("npc.low_health")
    end
end
```

---

## Internal Handlers

### ENT:_RegenHealth
Internal handler that applies health regeneration/degeneration every second.
- Positive regen: Calls `AddHealth()`
- Negative regen: Creates damage info with DMG_DIRECT

---

### ENT:_UpdateHealth
Internal handler that detects health changes and triggers `OnHealthChange` hook.

---

### ENT:_UpdateMaxHealth
Internal handler that networks max health to clients.

---

## Networking

All status values are automatically networked using NW2 variables:

| Variable | Type | Description |
|----------|------|-------------|
| `DrGBaseHealth` | Int | Current health |
| `DrGBaseMaxHealth` | Int | Maximum health |
| `DrGBaseHealthRegen` | Float | HP regen per second |
| `DrGBaseScale` | Float | Scale multiplier |
| `DrGBaseDown` | Bool | Downed state |
| `DrGBaseDying` | Bool | Currently dying |
| `DrGBaseDead` | Bool | Dead state |
| `DrGBaseDowned` | Int | Down count |
| `DrGBaseGodMode` | Bool | God mode state |
| `DrGBaseOnGround` | Bool | On ground state |

---

## Usage Examples

### Basic Health Management
```lua
-- Setup in Initialize
function ENT:Initialize()
    self:SetHealth(100)
    self:SetMaxHealth(100)
end

-- Damage handling
function ENT:OnTakeDamage(dmg)
    if self:GetGodMode() then return end

    local newHealth = self:Health() - dmg:GetDamage()
    self:SetHealth(newHealth, true) -- Clamp to valid range

    if self:Health() <= 0 then
        self:Die()
    end
end
```

---

### Regeneration Over Time
```lua
function ENT:EnableRegeneration()
    self:SetHealthRegen(5) -- 5 HP per second
end

function ENT:TakePoisonDamage()
    self:SetHealthRegen(-2) -- 2 HP loss per second

    self:Timer(10, function()
        self:SetHealthRegen(0) -- Remove poison after 10 seconds
    end)
end
```

---

### Heal to Full
```lua
function ENT:HealToFull()
    -- Instant heal
    self:SetHealth(self:GetMaxHealth())

    -- Or regenerate over 3 seconds
    self:RegenHealth(self:GetMaxHealth(), 3)
end
```

---

### Dynamic Difficulty Scaling
```lua
function ENT:ApplyDifficultyScale(difficulty)
    local scale = 1
    if difficulty == "easy" then
        scale = 0.75
    elseif difficulty == "hard" then
        scale = 1.5
    elseif difficulty == "nightmare" then
        scale = 2
    end

    self:ScaleHealth(scale)
    self:SetScale(scale ^ 0.5) -- Square root for visual size
end
```

---

### Boss Phases
```lua
function ENT:OnHealthChange(oldHealth, newHealth)
    local healthPercent = newHealth / self:GetMaxHealth()

    if healthPercent <= 0.5 and not self.phase2 then
        self.phase2 = true
        self:EnterPhase2()
    end

    if healthPercent <= 0.25 and not self.phase3 then
        self.phase3 = true
        self:EnterPhase3()
    end
end

function ENT:EnterPhase2()
    print("Boss entering phase 2!")
    self:SetHealthRegen(10) -- Start regenerating
    self:Scale(1.2, 1) -- Grow larger
end

function ENT:EnterPhase3()
    print("Boss entering final phase!")
    self:SetHealthRegen(0)
    self:EnableGodMode()

    -- Become vulnerable after animation
    self:Timer(5, function()
        self:DisableGodMode()
    end)
end
```

---

## See Also

- [Animation System](animations.md) - Animation control
- [Combat System](../guides/combat.md) - Damage and death handling
- [AI System](../guides/ai.md) - AI behaviors affected by status
