# Detection System Functions

Vision and sound detection for perceiving entities.

**File:** `lua/entities/drgbase_nextbot/detection.lua` (244 lines)

---

## Overview

The detection system provides sight and hearing capabilities for NPCs. It handles:

- Field of view (FOV) based vision detection
- Range-based line of sight checks
- Luminosity-based player detection (darkness/flashlight)
- Sound detection and hearing range
- Blind and deaf states
- Integration with awareness system for entity spotting

The detection system works in conjunction with the awareness system to track which entities have been detected.

---

## Vision System

### ENT:GetSightFOV()

Gets the NPC's field of view angle for vision.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `number` - Field of view angle in degrees (0-360)

**Description:**
Returns the NPC's vision FOV. An FOV of 360 means the NPC can see in all directions, while 90 means a narrow forward cone. An FOV of 0 makes the NPC effectively blind.

**Example:**
```lua
local fov = self:GetSightFOV()
print("NPC can see", fov, "degrees")
```

---

### ENT:SetSightFOV(angle)

Sets the NPC's field of view angle.

**Realm:** 🔴 SERVER

**Parameters:**
- `angle` (number) - FOV angle in degrees (automatically clamped to 0-360)

**Returns:** None

**Description:**
Sets the NPC's vision FOV. The angle is automatically clamped between 0 and 360 degrees.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Set narrow FOV for focused vision
    self:SetSightFOV(90)

    -- Set wide FOV for alert NPC
    self:SetSightFOV(270)

    -- Set omnidirectional vision
    self:SetSightFOV(360)
end
```

---

### ENT:GetSightRange()

Gets the NPC's vision range.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `number` - Vision range in Hammer units

**Description:**
Returns the maximum distance at which the NPC can see entities.

**Example:**
```lua
local range = self:GetSightRange()
print("NPC can see up to", range, "units away")
```

---

### ENT:SetSightRange(range)

Sets the NPC's vision range.

**Realm:** 🔴 SERVER

**Parameters:**
- `range` (number) - Vision range in Hammer units (automatically clamped to minimum 0)

**Returns:** None

**Description:**
Sets the maximum distance at which the NPC can see entities. The range is automatically clamped to a minimum of 0.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Short-sighted NPC
    self:SetSightRange(500)

    -- Far-sighted NPC
    self:SetSightRange(3000)
end
```

---

### ENT:GetSightLuminosityRange()

Gets the luminosity range for player detection.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `number` - Minimum luminosity (0-1)
- `number` - Maximum luminosity (0-1)

**Description:**
Returns the range of player luminosity values that the NPC can detect. Players in darkness (low luminosity) may be harder to spot, while players with flashlights (high luminosity) are easier to detect.

**Example:**
```lua
local min, max = self:GetSightLuminosityRange()
print("Can see players with luminosity between", min, "and", max)
```

---

### ENT:SetSightLuminosityRange(min, max)

Sets the luminosity range for player detection.

**Realm:** 🔴 SERVER

**Parameters:**
- `min` (number) - Minimum luminosity (0-1)
- `max` (number, optional) - Maximum luminosity (0-1). If not provided, min is used as max and min becomes 0

**Returns:** None

**Description:**
Sets the range of player luminosity values that the NPC can detect. Values are automatically clamped to 0-1. This allows you to create NPCs that:
- Only see players in darkness: `SetSightLuminosityRange(0, 0.3)`
- Only see players with flashlights: `SetSightLuminosityRange(0.7, 1)`
- See all players: `SetSightLuminosityRange(0, 1)`

If only one argument is provided, it's treated as the max value with min set to 0.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Can see players in any lighting
    self:SetSightLuminosityRange(0, 1)

    -- Only see players in bright light or with flashlights
    self:SetSightLuminosityRange(0.5, 1)

    -- Only see players in darkness (avoid flashlights)
    self:SetSightLuminosityRange(0, 0.3)
end
```

---

### ENT:IsBlind()

Checks if the NPC is blind.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if the NPC cannot see

**Description:**
Returns true if the NPC is blind due to any of the following:
- Global sight ConVar is disabled (`drgbase_ai_sight` = 0)
- NPC has "DrGBaseBlind" cooldown active (temporary blindness)
- Sight FOV is 0 or less
- Sight range is 0 or less

**Example:**
```lua
function ENT:CustomThink()
    if self:IsBlind() then
        -- Rely on hearing instead
        return
    end
    self:UpdateSight()
end
```

---

### ENT:IsInSight(ent)

Checks if an entity is currently in the NPC's field of vision.

**Realm:** 🔴 SERVER (🔵 CLIENT with callback)

**Parameters:**
- `ent` (Entity) - Entity to check

**Returns:**
- `boolean` - True if entity is in sight (SERVER)
- `boolean` - WasInSight value (CLIENT, use callback for current value)

**Description:**
Performs a comprehensive vision check including:
1. Entity validity
2. Blind state
3. Distance (within sight range)
4. Player luminosity (if entity is a player)
5. FOV angle check
6. Line of sight trace (via `ENT:Visible()`)

For players, also handles possession detection and flashlight/luminosity checks.

On CLIENT, you can pass a callback as the second parameter to get the actual server-side result asynchronously.

**Example:**
```lua
-- SERVER
if self:IsInSight(enemy) then
    print("I can see the enemy!")
end

-- CLIENT with callback
self:IsInSight(entity, function(self, inSight)
    if inSight then
        print("Entity is in sight on server!")
    end
end)
```

---

### ENT:GetInSight(disp, spotted)

Gets all entities of a certain disposition that are in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `disp` (number or table, optional) - Disposition or table of dispositions to check. Defaults to all dispositions
- `spotted` (boolean, optional) - If true, only returns spotted entities

**Returns:**
- `table` - Array of entities in sight

**Description:**
Returns a table of all entities matching the specified disposition(s) that are currently in the NPC's field of vision. Disposition constants:
- `D_LI` - Allies
- `D_HT` - Enemies/Hostile
- `D_FR` - Afraid of
- `D_NU` - Neutral

If `spotted` is true, only returns entities that have been spotted (tracked by awareness system).

**Example:**
```lua
-- Get all hostiles in sight
local hostiles = self:GetInSight({D_HT, D_FR})
for _, ent in ipairs(hostiles) do
    print("Hostile in sight:", ent)
end

-- Get all allies in sight that we've spotted
local spottedAllies = self:GetInSight(D_LI, true)
```

---

### ENT:GetAlliesInSight(spotted)

Gets all allied entities in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only returns spotted allies

**Returns:**
- `table` - Array of allied entities in sight

**Description:**
Convenience function that calls `ENT:GetInSight(D_LI, spotted)`.

**Example:**
```lua
local allies = self:GetAlliesInSight()
print("Can see", #allies, "allies")
```

---

### ENT:GetEnemiesInSight(spotted)

Gets all enemy entities in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only returns spotted enemies

**Returns:**
- `table` - Array of enemy entities in sight

**Description:**
Convenience function that calls `ENT:GetInSight(D_HT, spotted)`.

**Example:**
```lua
local enemies = self:GetEnemiesInSight(true)
for _, enemy in ipairs(enemies) do
    self:FireAt(enemy)
end
```

---

### ENT:GetAfraidOfInSight(spotted)

Gets all entities the NPC is afraid of that are in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only returns spotted entities

**Returns:**
- `table` - Array of feared entities in sight

**Description:**
Convenience function that calls `ENT:GetInSight(D_FR, spotted)`.

---

### ENT:GetHostilesInSight(spotted)

Gets all hostile entities (enemies and feared) in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only returns spotted hostiles

**Returns:**
- `table` - Array of hostile entities in sight

**Description:**
Convenience function that calls `ENT:GetInSight({D_HT, D_FR}, spotted)`. Returns both enemies and entities the NPC is afraid of.

**Example:**
```lua
local hostiles = self:GetHostilesInSight()
print("Total hostile entities in sight:", #hostiles)
```

---

### ENT:GetNeutralInSight(spotted)

Gets all neutral entities in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only returns spotted neutrals

**Returns:**
- `table` - Array of neutral entities in sight

**Description:**
Convenience function that calls `ENT:GetInSight(D_NU, spotted)`.

---

## Sight Updates

### ENT:UpdateSight(disp, spotted)

Updates sight state for entities of a certain disposition.

**Realm:** 🔴 SERVER

**Parameters:**
- `disp` (number or table, optional) - Disposition or table of dispositions to update. Defaults to all dispositions
- `spotted` (boolean, optional) - If true, only updates spotted entities

**Returns:** None

**Description:**
Checks sight for all entities matching the specified disposition(s) and triggers appropriate hooks:
- `ENT:OnSight(ent)` - Called when entity comes into sight or remains in sight
- `ENT:OnLostSight(ent)` - Called when entity leaves sight

This function is called automatically by the AI update cycle. It tracks which entities were in sight on the previous update to detect sight changes.

Does nothing if AI is disabled.

**Example:**
```lua
-- Manually update sight for enemies
self:UpdateSight(D_HT)

-- Update sight for all entity types
self:UpdateSight()
```

---

### ENT:UpdateAlliesSight(spotted)

Updates sight state for allied entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only updates spotted allies

**Returns:** None

**Description:**
Convenience function that calls `ENT:UpdateSight(D_LI, spotted)`.

---

### ENT:UpdateEnemiesSight(spotted)

Updates sight state for enemy entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only updates spotted enemies

**Returns:** None

**Description:**
Convenience function that calls `ENT:UpdateSight(D_HT, spotted)`.

---

### ENT:UpdateAfraidOfSight(spotted)

Updates sight state for feared entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only updates spotted feared entities

**Returns:** None

**Description:**
Convenience function that calls `ENT:UpdateSight(D_FR, spotted)`.

---

### ENT:UpdateHostilesSight(spotted)

Updates sight state for all hostile entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only updates spotted hostiles

**Returns:** None

**Description:**
Convenience function that calls `ENT:UpdateSight({D_HT, D_FR}, spotted)`. Updates both enemies and feared entities.

---

### ENT:UpdateNeutralSight(spotted)

Updates sight state for neutral entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `spotted` (boolean, optional) - If true, only updates spotted neutrals

**Returns:** None

**Description:**
Convenience function that calls `ENT:UpdateSight(D_NU, spotted)`.

---

## Blind Status

### ENT:Blind(blind)

Temporarily blinds the NPC.

**Realm:** 🔴 SERVER

**Parameters:**
- `blind` - Blind effect object with duration information

**Returns:** None

**Description:**
Applies a temporary blind effect to the NPC. The blind duration is controlled by a cooldown ("DrGBaseBlind"). The function:
1. Returns early if already blind
2. Calls `ENT:OnBlind(blind)` hook - return true to cancel, return number to scale duration
3. Sets the blind cooldown
4. Calls `ENT:OnBlinded(blind)` in a coroutine if the hook exists

**Example:**
```lua
-- Apply blind effect (assuming you have a blind object from damage system)
self:Blind(blindEffect)
```

---

## Hearing System

### ENT:GetHearingCoefficient()

Gets the NPC's hearing multiplier.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `number` - Hearing coefficient (multiplier for hearing range)

**Description:**
Returns the hearing coefficient, which multiplies the base sound distance to determine if the NPC can hear a sound. Higher values mean better hearing. A value of 0 makes the NPC deaf.

**Example:**
```lua
local hearing = self:GetHearingCoefficient()
print("Hearing coefficient:", hearing)
```

---

### ENT:SetHearingCoefficient(coeff)

Sets the NPC's hearing multiplier.

**Realm:** 🔴 SERVER

**Parameters:**
- `coeff` (number) - Hearing coefficient (automatically clamped to minimum 0)

**Returns:** None

**Description:**
Sets the hearing coefficient. Higher values increase hearing range. The value is automatically clamped to a minimum of 0.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Normal hearing
    self:SetHearingCoefficient(1)

    -- Enhanced hearing (2x range)
    self:SetHearingCoefficient(2)

    -- Poor hearing (half range)
    self:SetHearingCoefficient(0.5)

    -- Deaf
    self:SetHearingCoefficient(0)
end
```

---

### ENT:IsDeaf()

Checks if the NPC is deaf.

**Realm:** 🔵 SHARED

**Parameters:** None

**Returns:**
- `boolean` - True if the NPC cannot hear

**Description:**
Returns true if the NPC is deaf due to:
- Global hearing ConVar is disabled (`drgbase_ai_hearing` = 0)
- Hearing coefficient is 0 or less

**Example:**
```lua
if not self:IsDeaf() then
    -- Process sound detection
end
```

---

## Detection Hooks

### ENT:OnContact(ent)

Called when the NPC physically touches an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity that was contacted

**Returns:** None

**Description:**
Called by the engine when the NPC's collision hull touches another entity. Default behavior is to spot the entity via `ENT:SpotEntity(ent)`.

**Example:**
```lua
function ENT:OnContact(ent)
    if ent:IsPlayer() then
        print("Touched a player!")
    end
    -- Call default to spot the entity
    self:SpotEntity(ent)
end
```

---

### ENT:OnSight(ent)

Called when an entity is in sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity in sight

**Returns:** None

**Description:**
Called continuously while an entity remains in sight (every `ENT:UpdateSight()` call). Default behavior is to spot the entity via `ENT:SpotEntity(ent)`.

Note: This is called every update while in sight, not just when first seen. To detect first sight, track previous state or use the awareness system's `ENT:OnSpotted()`.

**Example:**
```lua
function ENT:OnSight(ent)
    if ent:IsPlayer() and not self:HasEnemy() then
        print("Player spotted!")
        self:EmitSound("npc/zombie/zombie_alert1.wav")
    end
end
```

---

### ENT:OnLostSight(ent)

Called when an entity leaves sight.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity that left sight

**Returns:** None

**Description:**
Called when an entity that was previously in sight is no longer visible. Default behavior is empty (no action).

**Example:**
```lua
function ENT:OnLostSight(ent)
    if ent == self:GetEnemy() then
        print("Lost sight of enemy!")
        -- Go to last known position
        local lastPos = self:LastKnownPosition(ent)
        if lastPos then
            self:MoveTo(lastPos)
        end
    end
end
```

---

### ENT:OnSound(ent, sound)

Called when the NPC hears a sound.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity that made the sound
- `sound` (table) - Sound information table with fields:
  - `Pos` (Vector) - Sound origin position
  - `SoundLevel` (number) - Sound level/volume
  - `Volume` (number) - Volume multiplier
  - Other EntityEmitSound fields

**Returns:** None

**Description:**
Called when a sound is within the NPC's hearing range. Default behavior is to spot the entity via `ENT:SpotEntity(ent)`.

The hearing system calculates audible distance based on sound level, volume, and the NPC's hearing coefficient. Line of sight to the sound also affects hearing (sounds behind walls are quieter).

**Example:**
```lua
function ENT:OnSound(ent, sound)
    if ent:IsPlayer() then
        print("Heard player at", sound.Pos)
        -- Investigate the sound
        if not self:HasEnemy() then
            self:AddPatrolPos(sound.Pos)
        end
    end
end
```

---

### ENT:OnBlind(blind)

Called when a blind effect is being applied.

**Realm:** 🔴 SERVER

**Parameters:**
- `blind` - Blind effect object

**Returns:**
- `boolean` (optional) - Return true to cancel the blind effect
- `number` (optional) - Return a number to scale the blind duration

**Description:**
Hook called before applying a blind effect. You can:
- Return true to completely prevent the blind effect
- Return a number to scale the duration (e.g., 0.5 for half duration, 2 for double)
- Return nothing to apply the effect normally

**Example:**
```lua
function ENT:OnBlind(blind)
    if self:Health() > self:GetMaxHealth() * 0.8 then
        -- Healthy NPCs resist blind
        return 0.5 -- Half duration
    end
end
```

---

### ENT:OnBlinded(blind)

Called after the NPC becomes blinded.

**Realm:** 🔴 SERVER

**Parameters:**
- `blind` - Blind effect object

**Returns:** None

**Description:**
Coroutine hook called after a blind effect is successfully applied. This runs in a coroutine, so you can use coroutine functions like `self:Wait()`.

Only called if you define this function (it's not defined by default).

**Example:**
```lua
function ENT:OnBlinded(blind)
    self:EmitSound("npc/zombie/zombie_pain1.wav")
    self:PlayAnimation("flinch")
    self:Wait(2)
    -- Become aggressive after recovering
    if self:HasEnemy() then
        self:SetNemesis(self:GetEnemy())
    end
end
```

---

## Client-Side Detection

### ENT:IsInSight(ent, callback)

Client-side sight check with server callback.

**Realm:** 🔵 CLIENT

**Parameters:**
- `ent` (Entity) - Entity to check
- `callback` (function, optional) - Callback function(self, inSight)

**Returns:**
- `boolean` - Previous cached result (may be outdated)

**Description:**
On the client, `IsInSight` queries the server for the actual sight state. The callback receives the server's response asynchronously.

The immediate return value is the last cached result from a previous check.

**Example:**
```lua
-- CLIENT
if CLIENT then
    self:IsInSight(LocalPlayer(), function(self, inSight)
        if inSight then
            -- Server confirms NPC can see the player
            surface.PlaySound("buttons/button17.wav")
        end
    end)
end
```

---

### ENT:WasInSight(ent)

Gets the last known sight state.

**Realm:** 🔵 CLIENT

**Parameters:**
- `ent` (Entity) - Entity to check

**Returns:**
- `boolean` - Whether entity was in sight last time it was checked

**Description:**
Returns the cached result from the last `IsInSight` check. Also triggers a new check to update the cache.

---

## ConVars

### drgbase_ai_sight

**Type:** Boolean
**Default:** 1
**Flags:** ARCHIVE, NOTIFY, REPLICATED

Global sight enable/disable. When set to 0, all NPCs become blind.

---

### drgbase_ai_hearing

**Type:** Boolean
**Default:** 1
**Flags:** ARCHIVE, NOTIFY, REPLICATED

Global hearing enable/disable. When set to 0, all NPCs become deaf.

---

## Configuration Properties

These properties should be set in `ENT:SetupDataTables()` or on initialization:

### self.SightFOV

**Type:** Number
**Default:** Varies by NPC

Initial field of view angle in degrees.

---

### self.SightRange

**Type:** Number
**Default:** Varies by NPC

Initial vision range in Hammer units.

---

### self.MinLuminosity

**Type:** Number (0-1)
**Default:** 0

Minimum player luminosity required to see them.

---

### self.MaxLuminosity

**Type:** Number (0-1)
**Default:** 1

Maximum player luminosity at which they can be seen.

---

### self.HearingCoefficient

**Type:** Number
**Default:** 1

Initial hearing range multiplier.

---

## Architecture Notes

The detection system operates through these mechanisms:

1. **Vision Detection**: `ENT:IsInSight()` performs comprehensive checks:
   - Distance calculation using sight range
   - Angle calculation using FOV
   - Player luminosity validation
   - Line of sight trace

2. **Hearing Detection**: Global hook listens to all sounds:
   - Calculates audible distance from sound properties
   - Applies hearing coefficient multiplier
   - Reduces range by 50% if no line of sight
   - Triggers `ENT:OnSound()` for NPCs in range

3. **Update Cycle**: `ENT:UpdateSight()` is called by AI system:
   - Checks all relevant entities
   - Tracks sight state changes
   - Triggers sight hooks
   - Integrates with awareness for entity spotting

4. **Integration**: Detection feeds into other systems:
   - **Awareness**: Spotted entities are tracked
   - **AI**: Enemy updates use sight information
   - **Behavior**: Actions respond to detected entities

---

## See Also

- [Awareness System](./awareness.md) - Entity tracking and spotting
- [AI System](./ai.md) - Enemy selection and behavior
- [Relationship System](./relationships.md) - Entity disposition
