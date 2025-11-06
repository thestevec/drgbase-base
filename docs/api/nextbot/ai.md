# AI System Functions

AI management, enemy handling, and decision making.

**File:** `lua/entities/drgbase_nextbot/ai.lua` (187 lines)

---

## Overview

The AI system manages enemy targeting, behavior states, and decision making. It works in conjunction with the detection and awareness systems to provide intelligent NPC behavior. The AI system handles:

- Enemy selection and tracking
- Nemesis system (priority targets)
- AI enable/disable states
- Behavior hooks for combat and patrol
- Integration with the ConVar `ai_disabled`

---

## AI State Management

### ENT:IsAIDisabled()

Checks if AI is currently disabled.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if AI is disabled

**Description:**
Returns true if AI is disabled either by the entity's internal state or by the global `ai_disabled` ConVar. When AI is disabled, the NPC will not update enemies, detect hostiles, or make decisions.

**Example:**
```lua
function ENT:Think()
    if not self:IsAIDisabled() then
        -- Perform AI logic
        self:UpdateAI()
    end
end
```

---

### ENT:SetAIDisabled(bool)

Enables or disables AI for this NPC.

**Realm:** 🔴 SERVER

**Parameters:**
- `bool` (boolean) - True to disable AI, false to enable

**Returns:** None

**Description:**
Sets the AI disabled state. When AI is re-enabled (changed from disabled to enabled), automatically calls `ENT:UpdateAI()` to refresh the AI state.

**Example:**
```lua
-- Disable AI temporarily
self:SetAIDisabled(true)

-- Wait 5 seconds then re-enable
timer.Simple(5, function()
    if IsValid(self) then
        self:SetAIDisabled(false) -- Auto-calls UpdateAI()
    end
end)
```

---

### ENT:DisableAI()

Disables AI for this NPC.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Convenience function that calls `ENT:SetAIDisabled(true)`.

**Example:**
```lua
function ENT:OnTakeDamage(dmg)
    if dmg:GetDamage() > 50 then
        -- Stun the NPC
        self:DisableAI()
        timer.Simple(3, function()
            if IsValid(self) then
                self:EnableAI()
            end
        end)
    end
end
```

---

### ENT:EnableAI()

Enables AI for this NPC.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Convenience function that calls `ENT:SetAIDisabled(false)`. Automatically triggers an AI update.

---

## Enemy Management

### ENT:GetEnemy()

Gets the current enemy target.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `Entity` - Current enemy entity, or NULL if none

**Description:**
Returns the NPC's current enemy. This is synchronized across client and server via networked variables.

**Example:**
```lua
local enemy = self:GetEnemy()
if IsValid(enemy) then
    print("Current enemy:", enemy:GetClass())
end
```

---

### ENT:SetEnemy(enemy)

Sets the current enemy target.

**Realm:** 🔴 SERVER

**Parameters:**
- `enemy` (Entity) - Entity to set as enemy, or NULL to clear

**Returns:** None

**Description:**
Sets the NPC's current enemy and clears any nemesis flag. This triggers enemy change hooks:
- If this is the first enemy: `ENT:OnNewEnemy(enemy)`
- If changing enemies: `ENT:OnEnemyChange(old, new)`
- If clearing last enemy: `ENT:OnLastEnemy(old)`

**Example:**
```lua
function ENT:CustomInitialize()
    -- Set a specific entity as initial enemy
    local target = ents.FindByClass("npc_zombie")[1]
    if IsValid(target) then
        self:SetEnemy(target)
    end
end
```

---

### ENT:HasEnemy()

Checks if NPC has a valid enemy.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if has valid enemy

**Description:**
Returns true if the NPC currently has a valid enemy entity.

**Example:**
```lua
function ENT:CustomThink()
    if self:HasEnemy() then
        self:ChaseEnemy()
    else
        self:Patrol()
    end
end
```

---

### ENT:HaveEnemy()

Alias for `ENT:HasEnemy()`.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if has valid enemy

---

### ENT:HadEnemy()

Checks if NPC has ever had an enemy.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if NPC has had an enemy at any point

**Description:**
Returns true if the NPC has ever had an enemy during its lifetime, even if it doesn't currently have one. Useful for checking if the NPC has been in combat before.

**Example:**
```lua
function ENT:OnIdle()
    if self:HadEnemy() then
        -- Alert behavior after combat
        self:LookAround()
    else
        -- Peaceful idle behavior
        self:Wander()
    end
end
```

---

## Nemesis System

The nemesis system allows you to designate a priority enemy that cannot be automatically changed by the AI system.

### ENT:SetNemesis(nemesis)

Sets a nemesis (priority enemy that won't be automatically changed).

**Realm:** 🔴 SERVER

**Parameters:**
- `nemesis` (Entity) - Entity to set as nemesis

**Returns:** None

**Description:**
Sets the NPC's enemy to the specified entity and marks it as a nemesis. Unlike regular enemies, a nemesis will not be automatically changed by `ENT:UpdateEnemy()`. The nemesis can only be cleared by calling `ENT:SetEnemy(NULL)` or setting a different nemesis.

**Example:**
```lua
function ENT:OnTakeDamage(dmg)
    local attacker = dmg:GetAttacker()
    if IsValid(attacker) and attacker:IsPlayer() then
        -- Player attacked us, make them our nemesis
        self:SetNemesis(attacker)
    end
end
```

---

### ENT:GetNemesis()

Gets the current nemesis.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `Entity` - Current nemesis entity, or NULL if no nemesis

**Description:**
Returns the current nemesis if one is set, otherwise returns NULL. Note: This only returns a valid entity if the NPC both has an enemy AND that enemy is marked as a nemesis.

---

### ENT:HasNemesis()

Checks if NPC has a nemesis.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if has a valid nemesis

**Example:**
```lua
function ENT:UpdateEnemy()
    if self:HasNemesis() then
        -- Nemesis takes priority over all other targets
        return
    end
    -- Normal enemy selection logic
end
```

---

### ENT:HaveNemesis()

Alias for `ENT:HasNemesis()`.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if has a valid nemesis

---

### ENT:HadNemesis()

Checks if NPC has ever had a nemesis.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if NPC has had a nemesis at any point

**Description:**
Returns true if the NPC has ever had a nemesis during its lifetime, even if it doesn't currently have one.

---

## AI Updates

### ENT:UpdateAI()

Updates all AI systems.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Performs a full AI update, including:
1. Updates hostile entity sight detection (`ENT:UpdateHostilesSight()`)
2. Updates current enemy target (`ENT:UpdateEnemy()`)

This is automatically called when AI is re-enabled via `ENT:SetAIDisabled(false)` or when the `ai_disabled` ConVar changes.

**Example:**
```lua
function ENT:OnTakeDamage(dmg)
    -- Force AI update after taking damage
    self:UpdateAI()
end
```

---

### ENT:UpdateEnemy()

Updates the current enemy based on AI logic.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `Entity` - The selected enemy (may be NULL)

**Description:**
Updates the NPC's current enemy based on the following logic:
1. If possessed by a player, clears enemy
2. If has a nemesis, keeps nemesis as enemy
3. Calls `ENT:OnUpdateEnemy()` to get enemy choice (defaults to `ENT:FetchEnemy()`)
4. Validates enemy is within range (uses `drgbase_ai_radius` ConVar)
5. For "afraid of" entities, validates they're within `WatchAfraidOfRange`
6. Sets the enemy via `ENT:SetEnemy()`

**Example:**
```lua
function ENT:CustomThink()
    -- Manually trigger enemy update
    self:UpdateEnemy()
end
```

---

### ENT:FetchEnemy()

Finds the best enemy from all hostile entities.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `Entity` - Best enemy target, or NULL if none found

**Description:**
Iterates through all hostile entities (via `ENT:HostileIterator(true)`) and selects the best enemy based on:
1. Custom logic from `ENT:OnFetchEnemy(ent1, ent2)` if it returns a boolean
2. Entity priority (from `ENT:GetPriority(ent)`)
3. Distance (closer enemies preferred if priority is equal)

This is called by `ENT:OnUpdateEnemy()` by default.

**Example:**
```lua
-- Override to add custom enemy selection
function ENT:OnFetchEnemy(ent1, ent2)
    -- Prefer players over NPCs
    if ent1:IsPlayer() and not ent2:IsPlayer() then
        return true
    elseif ent2:IsPlayer() and not ent1:IsPlayer() then
        return false
    end
    -- Otherwise use default logic
end
```

---

## Behavior Hooks

### ENT:OnNewEnemy(enemy)

Called when the NPC acquires its first enemy.

**Realm:** 🔵 SHARED

**Parameters:**
- `enemy` (Entity) - The new enemy

**Returns:** None

**Description:**
Called when the NPC gets an enemy for the first time (transitioning from no enemy to having one). This is only called once when first entering combat.

**Example:**
```lua
function ENT:OnNewEnemy(enemy)
    self:EmitSound("npc/zombie/zombie_alert1.wav")
    print("Entering combat!")
end
```

---

### ENT:OnEnemyChange(old, new)

Called when the enemy changes.

**Realm:** 🔵 SHARED

**Parameters:**
- `old` (Entity) - Previous enemy
- `new` (Entity) - New enemy

**Returns:** None

**Description:**
Called when the enemy changes from one valid entity to another valid entity (not called when going from enemy to no enemy, or no enemy to enemy).

**Example:**
```lua
function ENT:OnEnemyChange(old, new)
    print("Switching target from", old:GetClass(), "to", new:GetClass())
end
```

---

### ENT:OnLastEnemy(old)

Called when the NPC loses its last enemy.

**Realm:** 🔵 SHARED

**Parameters:**
- `old` (Entity) - The last enemy

**Returns:** None

**Description:**
Called when the NPC loses its enemy (transitioning from having an enemy to having none).

**Example:**
```lua
function ENT:OnLastEnemy(old)
    self:EmitSound("npc/zombie/zombie_idle1.wav")
    print("Lost enemy, returning to patrol")
end
```

---

### ENT:OnRangeAttack()

Called during range attack behavior.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called by the behavior tree when performing range attack behavior. Override this to implement custom range attack logic.

**Example:**
```lua
function ENT:OnRangeAttack()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    -- Fire projectile at enemy
    self:ShootProjectile(enemy:WorldSpaceCenter())
end
```

---

### ENT:OnMeleeAttack()

Called during melee attack behavior.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called by the behavior tree when performing melee attack behavior. Override this to implement custom melee attack logic.

**Example:**
```lua
function ENT:OnMeleeAttack()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    -- Perform melee damage
    enemy:TakeDamage(20, self, self)
    self:EmitSound("Zombie.AttackHit")
end
```

---

### ENT:OnChaseEnemy()

Called when chasing an enemy.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called by the behavior tree when actively chasing an enemy. Override to add custom chase behavior.

**Example:**
```lua
function ENT:OnChaseEnemy()
    -- Make angry sounds while chasing
    if math.random(1, 100) == 1 then
        self:EmitSound("npc/zombie/zombie_alert2.wav")
    end
end
```

---

### ENT:OnAvoidEnemy()

Called when avoiding an enemy.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called by the behavior tree when avoiding an enemy (typically for timid NPCs or when outnumbered).

**Example:**
```lua
function ENT:OnAvoidEnemy()
    -- Play scared animation
    self:PlayAnimation("cower")
end
```

---

### ENT:OnIdleEnemy()

Called when idling with an enemy present.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when the NPC has an enemy but is in an idle state (e.g., enemy is out of range but still tracked).

---

### ENT:OnEnemyUnreachable()

Called when the enemy cannot be reached.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called by the behavior tree when the pathfinder determines the enemy is unreachable.

**Example:**
```lua
function ENT:OnEnemyUnreachable()
    -- Give up after a while
    self:SetEnemy(NULL)
end
```

---

### ENT:OnAllyEnemy()

Called when an ally has an enemy.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when processing ally behavior in relation to their enemies.

---

### ENT:OnNeutralEnemy()

Called when interacting with neutral entities.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when processing neutral entity behavior.

---

## Afraid Of Behavior

### ENT:OnAvoidAfraidOf()

Called when avoiding an entity the NPC is afraid of.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called by the behavior tree when avoiding an entity marked with "afraid of" disposition.

**Example:**
```lua
function ENT:OnAvoidAfraidOf()
    -- Run away faster when afraid
    self.MoveSpeed = 200
end
```

---

### ENT:OnIdleAfraidOf()

Called when idling near an entity the NPC is afraid of.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when the NPC has an "afraid of" entity present but is in an idle state.

---

## Patrol Behavior

### ENT:OnReachedPatrol()

Called when reaching a patrol destination.

**Realm:** �004 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when the NPC reaches a patrol waypoint. Default behavior is to wait 3-7 seconds before continuing.

**Example:**
```lua
function ENT:OnReachedPatrol()
    -- Look around before continuing
    self:PlayAnimation("alert_idle")
    self:Wait(5)
end
```

---

### ENT:OnPatrolUnreachable()

Called when a patrol destination cannot be reached.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when the pathfinder determines a patrol waypoint is unreachable. Default behavior is to wait 3-7 seconds before selecting a new destination.

---

### ENT:OnPatrolling(...)

Called while patrolling.

**Realm:** 🔴 SERVER

**Parameters:**
- `...` - Forwarded to `ENT:WhilePatrolling()`

**Returns:**
- Any - Returns the result of `ENT:WhilePatrolling()`

**Description:**
Hook called by the behavior tree during patrol behavior. By default, forwards all arguments to `ENT:WhilePatrolling()`.

---

### ENT:WhilePatrolling()

Custom patrol behavior hook.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Override this hook to add custom behavior while patrolling.

**Example:**
```lua
function ENT:WhilePatrolling()
    -- Occasionally look around while patrolling
    if math.random(1, 100) == 1 then
        self:Wait(2)
        self:PlayAnimation("alert_idle")
    end
end
```

---

## Idle Behavior

### ENT:OnIdle()

Called when the NPC is idle (no enemy, no patrol).

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Description:**
Hook called when the NPC has nothing to do. Default behavior is to add a random patrol position within 1500 units.

**Example:**
```lua
function ENT:OnIdle()
    -- Custom idle behavior
    self:PlayAnimation("idle_subtle")
    self:Wait(math.random(5, 10))
    self:AddPatrolPos(self:RandomPos(2000))
end
```

---

## Enemy Selection Overrides

### ENT:OnUpdateEnemy()

Override enemy update logic.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `Entity` - The enemy to set, or nil to keep current enemy

**Description:**
Hook called by `ENT:UpdateEnemy()` to determine the new enemy. By default, returns `ENT:FetchEnemy()`. Return nil to keep the current enemy unchanged, or return an entity (including NULL) to change the enemy.

**Example:**
```lua
function ENT:OnUpdateEnemy()
    -- Only target players
    for _, ply in ipairs(player.GetAll()) do
        if self:IsInSight(ply) and self:Visible(ply) then
            return ply
        end
    end
    return NULL
end
```

---

### ENT:OnFetchEnemy(ent1, ent2)

Custom enemy comparison logic.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent1` (Entity) - First entity to compare
- `ent2` (Entity) - Second entity to compare

**Returns:**
- `boolean` (optional) - True if ent1 is better than ent2, false if ent2 is better, nil to use default logic

**Description:**
Hook called by `ENT:FetchEnemy()` when comparing two potential enemies. Return true if ent1 should be preferred, false if ent2 should be preferred, or nil to use the default priority/distance logic.

**Example:**
```lua
function ENT:OnFetchEnemy(ent1, ent2)
    -- Always prefer the entity that damaged us most recently
    local time1 = self.LastDamageTime[ent1] or 0
    local time2 = self.LastDamageTime[ent2] or 0

    if time1 > time2 then
        return true
    elseif time2 > time1 then
        return false
    end
    -- Use default logic if times are equal
end
```

---

## Movement Override

### ENT:ShouldRun()

Determines if the NPC should run.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `boolean` - True if NPC should run

**Description:**
Called to determine if the NPC should use run speed instead of walk speed. By default, returns true if the NPC has an enemy, or if the current patrol destination has its run flag set.

**Example:**
```lua
function ENT:ShouldRun()
    -- Always run if health is low
    if self:Health() < self:GetMaxHealth() * 0.3 then
        return true
    end
    -- Otherwise use default logic
    return self:HasEnemy()
end
```

---

## ConVars

### drgbase_ai_radius

**Type:** Number
**Default:** 5000
**Flags:** ARCHIVE, NOTIFY, REPLICATED

Maximum distance at which NPCs can have enemies. Enemies beyond this range will be automatically cleared.

---

### ai_disabled

**Type:** Boolean
**Default:** 0
**Flags:** Built-in Source engine ConVar

When set to 1, disables AI for all NPCs. DrGBase NPCs respect this ConVar and will not update their AI when it's enabled.

---

## Architecture Notes

The AI system operates on a hierarchical update model:

1. **AI Update Cycle**: `ENT:UpdateAI()` is called periodically by the behavior tree
2. **Detection**: Updates which entities are in sight
3. **Enemy Selection**: Calls `ENT:UpdateEnemy()` which uses `ENT:FetchEnemy()` to find the best target
4. **Behavior Execution**: Based on the current enemy and state, appropriate behavior hooks are called

The system integrates with several other systems:
- **Awareness System**: Tracks which entities have been spotted
- **Detection System**: Provides sight and hearing information
- **Relationship System**: Determines which entities are hostile, allies, or neutral
- **Behavior Tree**: Executes actions based on AI state

---

## See Also

- [Detection System](./detection.md) - Vision and sound detection
- [Awareness System](./awareness.md) - Entity perception and tracking
- [Relationship System](./relationships.md) - Entity disposition management
- [Behavior System](./behavior.md) - Behavior tree and actions
