# Awareness System Functions

Entity perception, tracking, and awareness state management.

**File:** `lua/entities/drgbase_nextbot/awareness.lua` (262 lines)

---

## Overview

The awareness system tracks which entities the NPC knows about and their last known locations. It provides a persistent memory layer on top of the detection system. Key features:

- Spotted/lost entity state tracking
- Last known position and time tracking
- Automatic timeout for entity memory
- Omniscient mode for developer testing
- Ally alerting system
- Client-server awareness synchronization for local player

The awareness system bridges detection (can I see it?) with AI decision making (do I know where it is?).

---

## Omniscient Mode

### ENT:IsOmniscient()

Checks if the NPC is in omniscient mode.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if omniscient

**Description:**
Returns true if the NPC is omniscient (can perceive all entities without detection). Omniscient mode is enabled either globally via the `drgbase_ai_omniscient` ConVar or per-entity via `ENT:SetOmniscient()`.

In omniscient mode:
- `ENT:HasSpotted()` always returns true for all entities
- Detection checks are bypassed
- Useful for debugging and testing AI behavior

**Example:**
```lua
if self:IsOmniscient() then
    print("I know where everyone is!")
end
```

---

### ENT:SetOmniscient(omniscient)

Enables or disables omniscient mode for this NPC.

**Realm:** 🔴 SERVER

**Parameters:**
- `omniscient` (boolean) - True to enable omniscient mode

**Returns:** None

**Description:**
Sets whether this NPC is omniscient. Omniscient NPCs automatically "spot" all entities without needing to see or hear them.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Make this NPC omniscient (always knows where enemies are)
    self:SetOmniscient(true)
end
```

---

## Spot Duration

### ENT:GetSpotDuration()

Gets how long the NPC remembers spotted entities.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `number` - Duration in seconds that entities remain spotted after detection

**Description:**
Returns the number of seconds an entity remains "spotted" after the NPC last detected it. After this duration expires without re-detection, the entity is "lost" and `ENT:OnLost()` is called.

A value of 0 disables the awareness system entirely (entities are never spotted).
A negative value means infinite memory (spotted entities are never forgotten).

**Example:**
```lua
local duration = self:GetSpotDuration()
print("Remembers enemies for", duration, "seconds")
```

---

### ENT:SetSpotDuration(duration)

Sets how long the NPC remembers spotted entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `duration` (number) - Duration in seconds (0 = disabled, negative = infinite)

**Returns:** None

**Description:**
Sets the spot duration. This controls the "memory" of the NPC:
- Short duration (3-10s): Forgetful, loses track quickly
- Medium duration (15-30s): Normal memory
- Long duration (60s+): Persistent memory
- Negative: Never forgets spotted entities

**Example:**
```lua
function ENT:CustomInitialize()
    -- Short memory (forgetful)
    self:SetSpotDuration(5)

    -- Long memory (persistent)
    self:SetSpotDuration(60)

    -- Infinite memory (never forgets)
    self:SetSpotDuration(-1)

    -- Disabled (no tracking)
    self:SetSpotDuration(0)
end
```

---

## Entity Awareness State

### ENT:HasSpotted(ent, absolute)

Checks if the NPC has spotted an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to check
- `absolute` (boolean, optional) - If true, ignores omniscient mode

**Returns:**
- `boolean` - True if entity has been spotted

**Description:**
Returns true if the NPC currently has the entity in its awareness state as "spotted". This includes entities currently being detected AND entities that were recently detected (within spot duration).

If `absolute` is false (default) and the NPC is omniscient, always returns true.

**Example:**
```lua
if self:HasSpotted(player) then
    print("I know where the player is!")
    local pos = self:LastKnownPosition(player)
end
```

---

### ENT:HasLost(ent, absolute)

Checks if the NPC has lost track of an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to check
- `absolute` (boolean, optional) - If true, ignores omniscient mode

**Returns:**
- `boolean` - True if entity was spotted but is now lost

**Description:**
Returns true if the entity was previously spotted but the spot duration has expired without re-detection. This is different from never having spotted the entity.

If `absolute` is false (default) and the NPC is omniscient, always returns false.

**Example:**
```lua
if self:HasLost(enemy) then
    print("Lost track of enemy, searching...")
    -- Search last known area
    self:AddPatrolPos(self:LastKnownPosition(enemy))
end
```

---

## Entity Iterators and Lists

### ENT:SpottedEntities()

Iterator for all spotted entities.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `function` - Iterator function for use in for loops

**Description:**
Returns an iterator that loops through all entities the NPC has currently spotted. Use in a for loop.

**Example:**
```lua
for ent in self:SpottedEntities() do
    print("Spotted:", ent:GetClass())
    local pos = self:LastKnownPosition(ent)
    local time = self:LastTimeSpotted(ent)
end
```

---

### ENT:LostEntities()

Iterator for all lost entities.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `function` - Iterator function for use in for loops

**Description:**
Returns an iterator that loops through all entities the NPC previously spotted but has now lost. Use in a for loop.

**Example:**
```lua
for ent in self:LostEntities() do
    print("Lost:", ent:GetClass())
end
```

---

### ENT:GetSpotted()

Gets a table of all spotted entities.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `table` - Array of spotted entities

**Description:**
Returns a table array containing all currently spotted entities. This is a convenience wrapper around `ENT:SpottedEntities()`.

**Example:**
```lua
local spotted = self:GetSpotted()
print("Currently tracking", #spotted, "entities")

for _, ent in ipairs(spotted) do
    print(ent:GetClass())
end
```

---

### ENT:GetLost()

Gets a table of all lost entities.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `table` - Array of lost entities

**Description:**
Returns a table array containing all entities that were spotted but are now lost. This is a convenience wrapper around `ENT:LostEntities()`.

**Example:**
```lua
local lost = self:GetLost()
if #lost > 0 then
    print("Lost track of", #lost, "entities")
end
```

---

## Position and Time Tracking

### ENT:LastTimeSpotted(ent)

Gets the last time an entity was spotted.

**Realm:** 🔴 SERVER (🔵 CLIENT for local player)

**Parameters:**
- `ent` (Entity) - Entity to check (SERVER only)

**Returns:**
- `number` - CurTime() when entity was last spotted, or -1 if never spotted

**Description:**
Returns the timestamp (from `CurTime()`) when the entity was last detected. Returns -1 if the entity has never been spotted.

On CLIENT, omit the parameter to get when the local player was last spotted by this NPC.

**Example:**
```lua
-- SERVER
local lastSeen = self:LastTimeSpotted(enemy)
if lastSeen > 0 then
    local timeSince = CurTime() - lastSeen
    print("Last saw enemy", timeSince, "seconds ago")
end

-- CLIENT
if CLIENT then
    local lastSeen = self:LastTimeSpotted()
    if lastSeen > 0 then
        print("NPC last saw me", CurTime() - lastSeen, "seconds ago")
    end
end
```

---

### ENT:LastKnownPosition(ent)

Gets the last known position of an entity.

**Realm:** 🔴 SERVER (🔵 CLIENT for local player)

**Parameters:**
- `ent` (Entity) - Entity to check (SERVER only)

**Returns:**
- `Vector` - Last known position, or nil if never spotted

**Description:**
Returns the last recorded position of an entity when it was spotted. This is updated each time the entity is detected.

On CLIENT, omit the parameter to get the local player's last known position from this NPC's perspective.

**Example:**
```lua
-- SERVER
local enemy = self:GetEnemy()
if not self:IsInSight(enemy) then
    -- Go to last known position
    local lastPos = self:LastKnownPosition(enemy)
    if lastPos then
        self:MoveTo(lastPos)
    end
end

-- CLIENT
if CLIENT then
    local lastPos = self:LastKnownPosition()
    if lastPos then
        -- Draw debug marker
        debugoverlay.Cross(lastPos, 10, 1, Color(255, 0, 0))
    end
end
```

---

### ENT:UpdateKnownPosition(ent, pos)

Updates the last known position of an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to update
- `pos` (Vector, optional) - Position to record. Defaults to `ent:GetPos()`

**Returns:** None

**Description:**
Manually updates the stored position for an entity. This is automatically called by `ENT:SpotEntity()`, but you can call it manually if needed.

**Example:**
```lua
-- Update with custom position (e.g., predicted position)
local predicted = enemy:GetPos() + enemy:GetVelocity() * 0.5
self:UpdateKnownPosition(enemy, predicted)
```

---

## Spotting and Losing Entities

### ENT:SpotEntity(ent)

Marks an entity as spotted.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to spot

**Returns:** None

**Description:**
Marks an entity as spotted by this NPC. This function:
1. Returns early if entity is invalid, dead player, or ignored player
2. Returns early if spot duration is 0 (awareness disabled)
3. Updates last spotted time to current time
4. Sets spotted state to true
5. Updates relationship caches for the entity
6. Updates last known position
7. Cancels sound-investigation patrol if this entity made the sound
8. Calls `ENT:OnSpotted(ent)` if first time spotting
9. Sends awareness network message to player if applicable
10. Creates/resets spot duration timer

This is automatically called by:
- `ENT:OnSight(ent)` when entity is seen
- `ENT:OnSound(ent, sound)` when entity is heard
- `ENT:OnContact(ent)` when entity is touched

**Example:**
```lua
-- Manually spot an entity (e.g., from ally alert)
function ENT:OnAlerted(ent, alertedBy)
    self:SpotEntity(ent)
    print("Ally alerted me to", ent:GetClass())
end
```

---

### ENT:LoseEntity(ent)

Marks an entity as lost.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to lose

**Returns:** None

**Description:**
Marks a spotted entity as lost. This function:
1. Returns early if entity is invalid or not spotted
2. Returns early if already lost
3. Sends awareness network message to player if applicable
4. Removes spot timer
5. Sets spotted state to false
6. Clears relationship caches for the entity
7. Calls `ENT:OnLost(ent)`

This is automatically called when the spot duration timer expires.

**Example:**
```lua
-- Manually lose track of an entity
function ENT:OnEnemyDeath(enemy)
    self:LoseEntity(enemy)
    print("Enemy died, losing track")
end
```

---

## Ally Communication

### ENT:AlertAllies(ent, spotted)

Alerts allies about a spotted entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to alert about
- `spotted` (boolean, optional) - If true, only alert spotted allies

**Returns:** None

**Description:**
Notifies all allied DrGBase nextbots about a spotted entity. For each ally:
- If the ally hasn't spotted the entity: Calls `ENT:OnAlerted(ent, self)` on the ally
- If the ally has spotted it: Refreshes their spot timer via `ally:SpotEntity(ent)`

If any allies were newly alerted, calls `ENT:OnAlert(ent, alerted)` on this NPC with the list of alerted allies.

**Example:**
```lua
function ENT:OnSpotted(ent)
    if ent:IsPlayer() then
        -- Alert nearby allies about player
        self:AlertAllies(ent, true)
        print("Alerting allies to player!")
    end
end
```

---

## Awareness Hooks

### ENT:OnSpotted(ent)

Called when an entity is first spotted.

**Realm:** 🔴 SERVER (🔵 CLIENT for local player)

**Parameters:**
- `ent` (Entity) - Entity that was spotted (SERVER only, CLIENT is always local player)

**Returns:** None

**Description:**
Called the first time an entity transitions from unknown/lost to spotted. This is only called once per spot cycle (not continuously while spotted).

On CLIENT, called when the local player is spotted by this NPC (no parameter).

**Example:**
```lua
-- SERVER
function ENT:OnSpotted(ent)
    if ent:IsPlayer() then
        self:EmitSound("npc/zombie/zombie_alert1.wav")
        print("Spotted player for the first time!")

        -- Alert allies
        self:AlertAllies(ent)
    end
end

-- CLIENT
if CLIENT then
    function ENT:OnSpotted()
        -- Local player was spotted
        surface.PlaySound("ambient/alarms/klaxon1.wav")
        chat.AddText(Color(255, 0, 0), "You've been spotted!")
    end
end
```

---

### ENT:OnLost(ent)

Called when a spotted entity is lost.

**Realm:** 🔴 SERVER (🔵 CLIENT for local player)

**Parameters:**
- `ent` (Entity) - Entity that was lost (SERVER only, CLIENT is always local player)

**Returns:** None

**Description:**
Called when the spot duration expires without re-detecting an entity. The entity transitions from spotted to lost.

On CLIENT, called when the local player is lost by this NPC (no parameter).

**Example:**
```lua
-- SERVER
function ENT:OnLost(ent)
    if ent == self:GetEnemy() then
        print("Lost track of enemy!")
        -- Search last known position
        local lastPos = self:LastKnownPosition(ent)
        if lastPos then
            self:AddPatrolPos(lastPos)
        end
    end
end

-- CLIENT
if CLIENT then
    function ENT:OnLost()
        -- Local player evaded NPC
        notification.AddLegacy("Evaded!", NOTIFY_HINT, 3)
    end
end
```

---

### ENT:OnAlert(ent, alerted)

Called when alerting allies about an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity that was spotted
- `alerted` (table) - Array of allies that were newly alerted

**Returns:** None

**Description:**
Called after successfully alerting one or more allies about a spotted entity via `ENT:AlertAllies()`. The `alerted` table contains only allies that didn't already know about the entity.

**Example:**
```lua
function ENT:OnAlert(ent, alerted)
    print("Alerted", #alerted, "allies about", ent:GetClass())
    self:EmitSound("npc/combine_soldier/vo/alert1.wav")
end
```

---

### ENT:OnAlerted(ent, alertedBy)

Called when alerted by an ally.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to be aware of
- `alertedBy` (Entity) - The ally that alerted this NPC

**Returns:** None

**Description:**
Called when an ally calls `AlertAllies()` and this NPC didn't already know about the entity. Default behavior is to spot the entity via `ENT:SpotEntity(ent)`.

**Example:**
```lua
function ENT:OnAlerted(ent, alertedBy)
    print("Ally", alertedBy:GetClass(), "alerted me to", ent:GetClass())

    -- Default behavior
    self:SpotEntity(ent)

    -- Custom behavior
    if ent:IsPlayer() then
        self:EmitSound("npc/combine_soldier/vo/copy.wav")
    end
end
```

---

## Client-Side Awareness

### ENT:HasSpottedLocalPlayer()

Checks if NPC has spotted the local player.

**Realm:** 🔵 CLIENT

**Parameters:** None

**Returns:**
- `boolean` - True if local player is spotted

**Description:**
Client-side function to check if this NPC currently has the local player spotted. Updated via network messages from the server.

If the NPC is omniscient, always returns true.

**Example:**
```lua
if CLIENT then
    hook.Add("HUDPaint", "CheckSpotted", function()
        for _, nb in ipairs(ents.FindByClass("drgbase_nextbot")) do
            if nb:HasSpottedLocalPlayer() then
                draw.SimpleText("SPOTTED", "DermaLarge", ScrW()/2, 100, Color(255, 0, 0), TEXT_ALIGN_CENTER)
            end
        end
    end)
end
```

---

### ENT:HasLostLocalPlayer()

Checks if NPC has lost the local player.

**Realm:** 🔵 CLIENT

**Parameters:** None

**Returns:**
- `boolean` - True if local player was spotted but is now lost

**Description:**
Client-side function to check if this NPC previously spotted the local player but has now lost track. Updated via network messages from the server.

If the NPC is omniscient, always returns false.

**Example:**
```lua
if CLIENT then
    function ENT:OnLost()
        -- Successfully evaded
        notification.AddLegacy("Evaded!", NOTIFY_HINT, 3)
    end
end
```

---

## ConVars

### drgbase_ai_omniscient

**Type:** Boolean
**Default:** 0
**Flags:** ARCHIVE, NOTIFY, REPLICATED

Global omniscient mode. When set to 1, all DrGBase NPCs become omniscient and can perceive all entities without detection.

Useful for testing AI behavior without worrying about sight/sound detection.

---

### ai_ignoreplayers

**Type:** Boolean
**Default:** 0
**Flags:** Built-in Source engine ConVar

When set to 1, NPCs ignore players. DrGBase NPCs will automatically lose awareness of all players and won't spot them.

---

## Configuration Properties

These properties should be set in your NPC's initialization:

### self.Omniscient

**Type:** Boolean
**Default:** false

Initial omniscient state for this NPC.

---

### self.SpotDuration

**Type:** Number
**Default:** Varies by NPC

Initial spot duration in seconds. How long the NPC remembers spotted entities.

---

## Architecture Notes

The awareness system provides a memory layer between detection and AI:

1. **Detection Layer**: Detection system determines what can currently be seen/heard
2. **Awareness Layer**: Tracks which entities are known and for how long
3. **AI Layer**: Makes decisions based on what entities are known

**Flow:**
```
Detection (IsInSight/OnSound) → SpotEntity() → Awareness State (HasSpotted)
                                      ↓
                                 Start Timer
                                      ↓
                              (Duration Expires)
                                      ↓
                                 LoseEntity() → Lost State (HasLost)
```

**Key Benefits:**
- NPCs don't immediately forget enemies when they leave sight
- Last known position allows searching behavior
- Spot duration creates realistic "search time" after losing track
- Omniscient mode allows testing AI without detection concerns

**Integration Points:**
- Detection hooks (`OnSight`, `OnSound`, `OnContact`) automatically call `SpotEntity()`
- AI system uses `HasSpotted()` to filter entity iterators
- Relationship system caches include awareness state for performance

**Network Synchronization:**
- Server tracks all entity awareness states
- Local player's spotted/lost state is networked to client
- Client can show UI elements based on awareness state

---

## See Also

- [Detection System](./detection.md) - Vision and sound detection
- [AI System](./ai.md) - Enemy selection and behavior
- [Relationship System](./relationships.md) - Entity disposition management
