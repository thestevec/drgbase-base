# Patrol System Functions

Priority-based patrol and investigation system.

**File:** `lua/entities/drgbase_nextbot/patrol.lua` (227 lines)

---

## Overview

The patrol system provides a flexible, priority-based system for NPCs to investigate points of interest. Unlike traditional waypoint patrols, this system uses a dynamic priority queue where different types of patrols (positions, sounds, searches) can be added and automatically sorted by importance.

**Key Features:**
- Priority-based patrol system (sounds > searches > positions)
- Three patrol types: positions, sounds, and enemy searches
- Automatic sorting by priority and time
- Custom sorting via hooks
- Automatic validation and cleanup

**Patrol Types (Priority Order):**
1. `PATROL_SOUND` (200) - Investigate heard sounds
2. `PATROL_SEARCH` (100) - Search for lost enemies
3. `PATROL_POS` (0) - Visit specific positions

---

## Patrol Constants

```lua
PATROL_POS = 0      -- Basic position patrol
PATROL_SEARCH = 100 -- Search for lost enemy
PATROL_SOUND = 200  -- Investigate sound
```

---

## Patrol Management

### ENT:AddPatrol(patrol)

Adds a patrol object to the queue.

**Realm:** 🔴 SERVER

**Parameters:**
- `patrol` (Patrol) - Patrol object (created via `DrGBase.Patrol`)

**Returns:**
- `boolean` - True if patrol was added, false if rejected

**Note:** Low-level function. Use `AddPatrolPos`, `AddPatrolSound`, or `AddPatrolSearch` instead.

---

### ENT:AddPatrolPos(pos, run)

Adds a position patrol to investigate a location.

**Realm:** 🔴 SERVER

**Parameters:**
- `pos` (Vector) - Position to patrol to
- `run` (boolean, optional) - Whether to run to this position (default false)

**Returns:**
- `boolean` - True if patrol was added

**Example:**
```lua
function ENT:OnIdle()
    -- Add a patrol point
    self:AddPatrolPos(Vector(100, 200, 0), false)

    local patrol = self:GetPatrol()
    if patrol then
        if self:GoTo(patrol:FetchPos()) then
            patrol:OnReached(self)
            self:RemovePatrol(patrol)
        end
    end
end
```

---

### ENT:AddPatrolSound(sound)

Adds a sound patrol to investigate a heard sound.

**Realm:** 🔴 SERVER

**Parameters:**
- `sound` (table) - Sound info table from `OnHearSound` hook

**Returns:**
- `boolean` - True if patrol was added

**Sound Table Structure:**
```lua
{
    Pos = Vector,        -- Sound origin position
    Channel = CHAN_*,    -- Sound channel
    SoundLevel = number, -- Sound level
    Volume = number,     -- Volume (0-1)
    Entity = Entity      -- Entity that made the sound
}
```

**Behavior:**
- Weapon sounds (CHAN_WEAPON) always override other sounds
- Louder sounds replace quieter sounds
- Only one sound patrol can exist at a time

**Example:**
```lua
function ENT:OnHearSound(sound)
    if sound.Channel == CHAN_WEAPON then
        self:AddPatrolSound(sound)
    end
end

function ENT:OnIdle()
    local patrol = self:GetPatrol()
    if patrol and patrol:GetType() == PATROL_SOUND then
        if self:GoTo(patrol:FetchPos()) then
            print("Investigated sound")
            self:RemovePatrol(patrol)
        end
    end
end
```

---

### ENT:AddPatrolSearch(enemy)

Adds a search patrol to look for a lost enemy at their last known position.

**Realm:** 🔴 SERVER

**Parameters:**
- `enemy` (Entity) - Enemy entity to search for

**Returns:**
- `boolean` - True if patrol was added

**Note:** The patrol stores the enemy's position when added. Use when losing sight of an enemy.

**Example:**
```lua
function ENT:OnLostEnemy(enemy)
    -- Search for the enemy at their last known position
    self:AddPatrolSearch(enemy)
end

function ENT:OnIdle()
    local patrol = self:GetPatrol()
    if patrol and patrol:GetType() == PATROL_SEARCH then
        if self:GoTo(patrol:FetchPos()) then
            print("Searched last known position")
            self:RemovePatrol(patrol)
        end
    end
end
```

---

### ENT:RemovePatrol(patrol)

Removes a patrol from the queue.

**Realm:** 🔴 SERVER

**Parameters:**
- `patrol` (Patrol) - Patrol object to remove

**Returns:**
- `boolean` - True if patrol was removed

**Example:**
```lua
function ENT:OnIdle()
    local patrol = self:GetPatrol()
    if patrol then
        if self:GoTo(patrol:FetchPos()) then
            self:RemovePatrol(patrol) -- Remove after reaching
        end
    end
end
```

---

### ENT:GetPatrol()

Gets the highest priority valid patrol.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `Patrol` - Highest priority patrol, or nil if none exist

**Note:** Automatically validates and removes invalid patrols.

**Example:**
```lua
function ENT:OnIdle()
    local patrol = self:GetPatrol()
    if patrol then
        local pos = patrol:FetchPos()
        local shouldRun = patrol:ShouldRun()
        print("Patrolling to " .. tostring(pos))
    end
end
```

---

### ENT:HasPatrol(patrol)

Checks if a patrol exists in the queue.

**Realm:** 🔴 SERVER

**Parameters:**
- `patrol` (Patrol, optional) - Specific patrol to check. If omitted, checks if any valid patrol exists

**Returns:**
- `boolean` - True if patrol exists (or any valid patrol exists)

**Example:**
```lua
function ENT:OnIdle()
    if self:HasPatrol() then
        print("I have somewhere to go!")
    else
        print("Nothing to do")
    end
end
```

---

### ENT:ClearPatrols()

Removes all patrols from the queue.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Example:**
```lua
function ENT:OnNewEnemy(enemy)
    -- Cancel all patrols when enemy found
    self:ClearPatrols()
end
```

---

### ENT:SortPatrols()

Re-sorts the patrol queue by priority.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

**Note:** Called automatically when patrols are added/removed. Uses `OnSortPatrols` hook for custom sorting.

---

## Patrol Hooks

### ENT:OnSortPatrols(patrol1, patrol2)

Hook to customize patrol sorting/priority.

**Realm:** 🔴 SERVER

**Parameters:**
- `patrol1` (Patrol) - First patrol to compare
- `patrol2` (Patrol) - Second patrol to compare

**Returns:**
- `boolean` - True if patrol1 should come before patrol2
- `nil` - Use default sorting

**Default Sorting:**
1. Valid patrols before invalid patrols
2. Higher type values before lower (SOUND > SEARCH > POS)
3. Older patrols before newer (by creation time)

**Example:**
```lua
function ENT:OnSortPatrols(patrol1, patrol2)
    -- Always prioritize sounds from players
    local type1, type2 = patrol1:GetType(), patrol2:GetType()

    if type1 == PATROL_SOUND and type2 ~= PATROL_SOUND then
        local sound = patrol1:GetSound()
        if IsValid(sound.Entity) and sound.Entity:IsPlayer() then
            return true -- patrol1 comes first
        end
    end

    -- Use default sorting
    return nil
end
```

---

## Patrol Class Methods

Patrol objects have these methods available:

### patrol:GetType()

Gets the patrol type constant.

**Returns:**
- `number` - PATROL_POS, PATROL_SEARCH, or PATROL_SOUND

---

### patrol:GetTime()

Gets the time the patrol was created.

**Returns:**
- `number` - CurTime() when patrol was created

---

### patrol:FetchPos()

Gets the target position for this patrol.

**Returns:**
- `Vector` - Position to patrol to

---

### patrol:ShouldRun()

Checks if the NPC should run to this patrol.

**Returns:**
- `boolean` - True if should run

**Default Behavior:**
- Position patrols: Returns the `run` parameter value
- Sound patrols: Returns true for weapon sounds (CHAN_WEAPON)
- Search patrols: Returns true if target not visible

---

### patrol:IsValid(nextbot)

Checks if the patrol is still valid.

**Parameters:**
- `nextbot` (Entity) - The NPC entity

**Returns:**
- `boolean` - True if valid

**Validity Rules:**
- Position patrols: Always valid
- Sound patrols: Always valid (sound already happened)
- Search patrols: Valid only if enemy is still an enemy

---

### patrol:OnAdded(nextbot)

Called when patrol is added to the queue.

**Parameters:**
- `nextbot` (Entity) - The NPC entity

**Returns:**
- `boolean` - Return false to reject the patrol

**Note:** Sound patrols use this to replace lower-priority sounds.

---

### patrol:OnRemoved(nextbot)

Called when patrol is removed from the queue.

**Parameters:**
- `nextbot` (Entity) - The NPC entity

---

### patrol:OnReached(nextbot)

Called when the NPC reaches the patrol destination.

**Parameters:**
- `nextbot` (Entity) - The NPC entity

---

### patrol:OnUnreachable(nextbot)

Called when the patrol destination is unreachable.

**Parameters:**
- `nextbot` (Entity) - The NPC entity

---

## Usage Examples

### Basic Patrol System

```lua
function ENT:CustomInitialize()
    -- Add patrol points
    self:AddPatrolPos(Vector(0, 0, 0))
    self:AddPatrolPos(Vector(500, 0, 0))
    self:AddPatrolPos(Vector(500, 500, 0))
end

function ENT:OnIdle()
    local patrol = self:GetPatrol()
    if patrol then
        local result = self:GoTo(patrol:FetchPos())
        self:RemovePatrol(patrol)

        if result then
            patrol:OnReached(self)
            self:Timer(2, function()
                -- Add this patrol back after a delay
                self:AddPatrolPos(patrol:FetchPos())
            end)
        else
            patrol:OnUnreachable(self)
        end
    end
end
```

### Sound Investigation

```lua
function ENT:OnHearSound(sound)
    -- Only investigate loud sounds
    if sound.SoundLevel > 70 then
        self:AddPatrolSound(sound)
    end
end

function ENT:OnIdle()
    if not self:HasEnemy() then
        local patrol = self:GetPatrol()
        if patrol then
            if patrol:GetType() == PATROL_SOUND then
                print("Investigating sound...")
            end

            if self:GoTo(patrol:FetchPos()) then
                self:RemovePatrol(patrol)
            end
        end
    end
end
```

### Enemy Search System

```lua
function ENT:OnLostEnemy(enemy)
    -- Search last known position when losing sight
    self:AddPatrolSearch(enemy)
    print("Lost sight of enemy, searching...")
end

function ENT:OnIdle()
    local patrol = self:GetPatrol()

    if patrol and patrol:GetType() == PATROL_SEARCH then
        print("Searching for lost enemy")

        if self:GoTo(patrol:FetchPos()) then
            -- Look around when reaching search location
            self:PlayAnimation("search")
            self:Timer(3, function()
                self:RemovePatrol(patrol)
            end)
        else
            self:RemovePatrol(patrol)
        end
    end
end
```

### Priority-Based Patrol

```lua
function ENT:CustomInitialize()
    -- Add low-priority patrols
    self:AddPatrolPos(Vector(0, 0, 0))
    self:AddPatrolPos(Vector(500, 0, 0))
end

function ENT:OnHearSound(sound)
    -- Sound patrols automatically get higher priority
    if sound.Channel == CHAN_WEAPON then
        self:AddPatrolSound(sound)
        print("Weapon sound detected! Investigating immediately")
    end
end

function ENT:OnLostEnemy(enemy)
    -- Search patrols have medium priority
    self:AddPatrolSearch(enemy)
    print("Lost enemy, will search after investigating sounds")
end

function ENT:OnIdle()
    local patrol = self:GetPatrol()
    if patrol then
        -- Automatically handles highest priority patrol
        if self:GoTo(patrol:FetchPos(), 50) then
            self:RemovePatrol(patrol)
        end
    end
end
```

### Custom Patrol Sorting

```lua
function ENT:OnSortPatrols(patrol1, patrol2)
    local type1 = patrol1:GetType()
    local type2 = patrol2:GetType()

    -- Prioritize search patrols for specific enemies
    if type1 == PATROL_SEARCH and type2 ~= PATROL_SEARCH then
        local enemy = patrol1:GetEnemy()
        if IsValid(enemy) and enemy:IsPlayer() then
            return true -- Search for players first
        end
    end

    -- Prioritize closer patrols when health is low
    if self:Health() < self:GetMaxHealth() * 0.3 then
        local dist1 = self:GetPos():Distance(patrol1:FetchPos())
        local dist2 = self:GetPos():Distance(patrol2:FetchPos())
        return dist1 < dist2
    end

    return nil -- Use default sorting
end
```

### Dynamic Patrol Routes

```lua
function ENT:CustomInitialize()
    self.patrolPoints = {
        Vector(0, 0, 0),
        Vector(500, 0, 0),
        Vector(500, 500, 0),
        Vector(0, 500, 0)
    }
    self.currentPatrolIndex = 1
end

function ENT:OnIdle()
    if not self:HasEnemy() and not self:HasPatrol() then
        -- Add next patrol point
        local pos = self.patrolPoints[self.currentPatrolIndex]
        self:AddPatrolPos(pos, false)

        self.currentPatrolIndex = self.currentPatrolIndex + 1
        if self.currentPatrolIndex > #self.patrolPoints then
            self.currentPatrolIndex = 1
        end
    end

    local patrol = self:GetPatrol()
    if patrol then
        if self:GoTo(patrol:FetchPos(), 30) then
            self:RemovePatrol(patrol)
            self:Timer(math.random(2, 5), function()
                -- Wait before moving to next point
            end)
        end
    end
end
```

---

## Advanced: Creating Custom Patrol Types

You can create custom patrol types using the `DrGBase.Patrol` function:

```lua
-- Define custom patrol type
PATROL_INVESTIGATE = 150 -- Between SEARCH and SOUND

local PatrolInvestigate = DrGBase.Patrol(PATROL_INVESTIGATE)

function PatrolInvestigate:New(entity, reason)
    local patrol = {}
    patrol._entity = entity
    patrol._pos = entity:GetPos()
    patrol._reason = reason
    setmetatable(patrol, self)
    return patrol
end

function PatrolInvestigate:FetchPos()
    return self._pos
end

function PatrolInvestigate:ShouldRun()
    return true -- Always run to investigations
end

function PatrolInvestigate:IsValid(nextbot)
    return IsValid(self._entity)
end

function PatrolInvestigate:OnReached(nextbot)
    print("Investigated " .. self._reason)
end

function PatrolInvestigate:__tostring()
    return "PatrolInvestigate [" .. self._reason .. "]"
end

-- Usage
function ENT:AddPatrolInvestigate(entity, reason)
    local patrol = PatrolInvestigate:New(entity, reason)
    return self:AddPatrol(patrol)
end
```

---

## Technical Details

### Patrol Priority

Default priority order (higher numbers = higher priority):
1. **PATROL_SOUND (200)** - Immediate response to sounds
2. **PATROL_SEARCH (100)** - Active searching for lost enemies
3. **PATROL_POS (0)** - Routine patrol routes

Within same priority, older patrols come first (FIFO).

### Sound Patrol Replacement

When adding a sound patrol:
- If existing sound is a weapon sound, new non-weapon sounds are rejected
- Otherwise, louder sounds replace quieter sounds
- Loudness calculated as: `(SoundLevel/2)² × Volume`

### Automatic Cleanup

Invalid patrols are automatically removed when:
- `GetPatrol()` is called
- `SortPatrols()` is called
- Search patrol's enemy is no longer valid

---

## See Also

- [Movement Functions](./movement.md) - Path following and navigation
- [Path Functions](./path.md) - Pathfinding system
- [Sound Functions](./perception.md#sound-detection) - Hearing sounds
- [Enemy Functions](./enemy.md) - Enemy tracking
