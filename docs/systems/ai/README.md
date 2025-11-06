# AI System

Enemy detection, awareness, and decision making.

**Files:** `ai.lua`, `detection.lua`, `awareness.lua`

## Overview

The AI system manages enemy selection, detection (vision and hearing), awareness (entity memory), and provides hooks for decision-making. It determines what entities the NPC is aware of, which entities are enemies, and triggers appropriate behavioral responses. The system includes:

- **Enemy management** - Automatic selection and tracking of hostile targets
- **Vision** - Field of view, sight range, and luminosity-based detection
- **Hearing** - Sound-based detection with distance and volume calculations
- **Awareness** - Entity memory system that tracks spotted and lost entities
- **ConVars** - Server-side configuration for global AI behavior

## Components

### Enemy Management

Enemies are selected automatically based on several factors:

**Enemy Selection Process:**
1. `UpdateEnemy()` is called automatically by the AI system
2. `OnUpdateEnemy()` hook returns potential enemy (defaults to `FetchEnemy()`)
3. `FetchEnemy()` iterates through all hostile entities (D_HT and D_FR)
4. `OnFetchEnemy()` hook can customize priority comparison
5. System picks highest priority enemy within `drgbase_ai_radius` (default: 5000 units)

**Priority System:**
- Entities with higher relationship priority are favored
- If priority is equal, closer entities are favored
- Enemies marked as "afraid of" must be within `WatchAfraidOfRange` property

**Nemesis System:**
- A nemesis is a locked-in enemy that won't change automatically
- Set via `SetNemesis(entity)` instead of `SetEnemy(entity)`
- Useful for scripted encounters or boss fights

**Key Functions:**
- `GetEnemy()` - Returns current enemy
- `HasEnemy()` / `HaveEnemy()` - Check if enemy is valid
- `SetEnemy(enemy)` - Manually set enemy
- `SetNemesis(nemesis)` - Lock enemy as nemesis
- `GetNemesis()` / `HasNemesis()` - Nemesis getters
- `FetchEnemy()` - Find best enemy from hostiles
- `UpdateEnemy()` - Refresh enemy selection

### Detection System

The detection system handles both vision and hearing.

**Vision Detection:**

The NPC can see entities based on:
- **Field of View (FOV)** - Angle in degrees (0-360), set via `SightFOV` property
- **Sight Range** - Maximum distance in units, set via `SightRange` property
- **Luminosity Range** - For players, detects based on darkness/light (0-1 range)
  - Set via `MinLuminosity` and `MaxLuminosity` properties
  - Players with flashlights are always visible (luminosity = 1)
- **Line of sight** - Must have unobstructed view using `Visible(ent)`

**Vision Functions:**
- `IsInSight(ent)` - Check if entity is currently visible
- `GetInSight(disposition, spotted)` - Get all visible entities matching disposition
- `GetAlliesInSight()` / `GetEnemiesInSight()` / etc.
- `UpdateSight(disposition)` - Check sight and trigger `OnSight` / `OnLostSight` hooks
- `UpdateHostilesSight()` - Update sight for enemies and afraid-of entities
- `SetSightFOV(angle)` - Change FOV dynamically
- `SetSightRange(range)` - Change sight range dynamically
- `IsBlind()` - Check if NPC cannot see
- `Blind(blindInfo)` - Temporarily blind the NPC

**Hearing Detection:**

NPCs can hear sounds from players and other sources:
- Distance based on sound level and volume: `distance = (SoundLevel/2)^2 * Volume`
- Multiplied by `HearingCoefficient` property (default: 1)
- Further multiplied by 0.5 if no line of sight to sound source
- Triggers `OnSound(entity, soundData)` hook when heard

**Hearing Functions:**
- `IsDeaf()` - Check if NPC cannot hear
- `SetHearingCoefficient(coeff)` - Change hearing sensitivity

**ConVars:**
- `drgbase_ai_sight` (1) - Enable/disable vision globally
- `drgbase_ai_hearing` (1) - Enable/disable hearing globally

### Awareness System

The awareness system tracks which entities the NPC has "spotted" (seen or heard) and remembers them for a duration even after losing sight.

**Spotting:**
- Entities are spotted when seen (`OnSight`) or heard (`OnSound`) or touched (`OnContact`)
- Spotted status lasts for `SpotDuration` property seconds (default varies by NPC)
- Timer refreshes each time entity is detected again
- Last known position is stored via `UpdateKnownPosition(ent, pos)`

**Awareness Functions:**
- `HasSpotted(ent)` - Check if entity is currently spotted
- `HasLost(ent)` - Check if entity was spotted but is now lost
- `SpotEntity(ent)` - Manually mark entity as spotted
- `LoseEntity(ent)` - Manually mark entity as lost
- `LastTimeSpotted(ent)` - Get timestamp of last detection
- `LastKnownPosition(ent)` - Get last known position vector
- `GetSpotted()` - Get table of all spotted entities
- `SpottedEntities()` - Iterator for spotted entities
- `AlertAllies(ent)` - Make nearby allies spot the same entity

**Omniscient Mode:**
- Set via `Omniscient` property or `SetOmniscient(bool)`
- When enabled, NPC knows about all entities automatically
- ConVar `drgbase_ai_omniscient` makes all NPCs omniscient

**Awareness Hooks:**
- `OnSpotted(ent)` - Called when entity first spotted
- `OnLost(ent)` - Called when spotted entity is lost
- `OnAlert(ent, alerted)` - Called when allies are alerted
- `OnAlerted(ent, alerter)` - Called when alerted by ally

### Decision Making

The AI system provides hooks that are called automatically to drive NPC behavior. These are called by the behavior tree (BT) system.

**Combat Decision Hooks:**
- `OnRangeAttack()` - Called when in range attack state
- `OnMeleeAttack()` - Called when in melee attack state
- `OnChaseEnemy()` - Called when chasing enemy
- `OnAvoidEnemy()` - Called when avoiding enemy (fleeing)
- `OnIdleEnemy()` - Called when enemy is unreachable or out of range
- `OnEnemyUnreachable()` - Called when pathfinding to enemy fails
- `OnAllyEnemy()` - Called when handling ally's enemy
- `OnNeutralEnemy()` - Called when handling neutral entity

**Afraid-Of Decision Hooks:**
- `OnAvoidAfraidOf()` - Called when fleeing from frightening entity
- `OnIdleAfraidOf()` - Called when frightening entity is out of range

**Patrol Decision Hooks:**
- `OnReachedPatrol()` - Called when patrol point reached (default: wait 3-7 seconds)
- `OnPatrolUnreachable()` - Called when patrol point unreachable
- `OnPatrolling()` - Called while moving to patrol point

**Idle Decision Hook:**
- `OnIdle()` - Called when no enemies or patrols (default: add random patrol point)

**Enemy Selection Hooks:**
- `OnUpdateEnemy()` - Override enemy selection logic (default: `FetchEnemy()`)
- `OnFetchEnemy(ent1, ent2)` - Compare two enemies to pick best one

**Detection Hooks:**
- `OnSight(ent)` - Entity enters sight (default: spot entity)
- `OnLostSight(ent)` - Entity leaves sight
- `OnSound(ent, sound)` - Sound heard from entity (default: spot entity)
- `OnContact(ent)` - Physical contact with entity (default: spot entity)
- `OnBlind(blindInfo)` - Called when blinded
- `OnBlinded()` - Coroutine called during blind effect

**Enemy Change Hooks:**
- `OnNewEnemy(enemy)` - First enemy ever acquired
- `OnEnemyChange(old, new)` - Enemy changed
- `OnLastEnemy(enemy)` - Last enemy lost (now has no enemy)

## Configuration

**AI Properties:**
```lua
ENT.SightFOV = 90  -- Field of view in degrees (0-360)
ENT.SightRange = 8192  -- Maximum sight distance in units
ENT.MinLuminosity = 0  -- Minimum light level to see players (0-1)
ENT.MaxLuminosity = 1  -- Maximum light level to see players (0-1)
ENT.HearingCoefficient = 1  -- Hearing sensitivity multiplier
ENT.SpotDuration = 20  -- How long entities stay "spotted" after losing sight
ENT.Omniscient = false  -- If true, always knows about all entities
ENT.WatchAfraidOfRange = 4096  -- Max range to watch frightening entities
```

**ConVars:**
- `drgbase_ai_radius` (5000) - Maximum enemy search radius
- `drgbase_ai_sight` (1) - Enable/disable vision globally
- `drgbase_ai_hearing` (1) - Enable/disable hearing globally
- `drgbase_ai_omniscient` (0) - Make all NPCs omniscient
- `ai_disabled` (0) - Vanilla GMod ConVar to disable AI
- `ai_ignoreplayers` (0) - Vanilla GMod ConVar to ignore players

## Usage Examples

**Basic Enemy Detection:**
```lua
function ENT:OnNewEnemy(enemy)
    self:EmitSound("npc/zombie/zombie_alert1.wav")
end

function ENT:OnEnemyChange(old, new)
    if IsValid(new) then
        print("Now targeting: " .. tostring(new))
    end
end

function ENT:OnChaseEnemy()
    -- Called when chasing enemy
    self:ChaseEntity(self:GetEnemy())
end
```

**Custom Enemy Selection:**
```lua
function ENT:OnFetchEnemy(ent1, ent2)
    -- Prefer players over NPCs
    if ent1:IsPlayer() and not ent2:IsPlayer() then
        return true
    elseif ent2:IsPlayer() and not ent1:IsPlayer() then
        return false
    end
    -- Otherwise use default (priority then distance)
end
```

**Awareness and Alerting:**
```lua
function ENT:OnSpotted(ent)
    if ent:IsPlayer() then
        self:EmitSound("npc/combine_soldier/vo/alert1.wav")
        self:AlertAllies(ent)  -- Tell nearby allies
    end
end

function ENT:OnAlerted(ent, alertedBy)
    print("I was alerted to " .. tostring(ent) .. " by " .. tostring(alertedBy))
end
```

**Custom Sight Configuration:**
```lua
function ENT:Initialize()
    -- Narrow FOV, long range (sniper-like)
    self:SetSightFOV(30)
    self:SetSightRange(16384)

    -- Only see in complete darkness
    self:SetSightLuminosityRange(0, 0.1)
end
```

**Sound-Based Detection:**
```lua
function ENT:OnSound(ent, sound)
    if sound.Channel == CHAN_WEAPON then
        -- Weapon fire detected!
        print("Heard gunshot from " .. tostring(ent))
        self:AddPatrolSound(sound)  -- Investigate the sound
    end
end
```

**AI Disabling:**
```lua
-- Temporarily disable AI
self:DisableAI()

-- Re-enable AI later
timer.Simple(5, function()
    self:EnableAI()
end)
```

## API Reference

- [AI Functions](../../api/nextbot/ai.md)
- [Detection Functions](../../api/nextbot/detection.md)
- [Awareness Functions](../../api/nextbot/awareness.md)

## Best Practices

**Performance:**
- Use `UpdateHostilesSight()` instead of `UpdateSight()` to only check hostile entities
- Set appropriate `SightRange` to avoid checking distant entities
- Use `SpotDuration` to prevent constant re-detection overhead
- Limit `drgbase_ai_radius` if you have many NPCs in large maps

**Behavior Design:**
- Always implement `OnChaseEnemy()` for combat NPCs
- Use `OnIdleEnemy()` to handle enemy being temporarily unreachable
- Implement `OnLost(ent)` to add last known position as patrol point
- Use nemesis system sparingly (for bosses or scripted sequences)

**Detection Tuning:**
- Lower FOV = more focused, easier to flank
- Higher `MinLuminosity` = only see players in light (stealth gameplay)
- Lower `HearingCoefficient` = deaf or less sensitive to sound
- Use `Omniscient = true` for testing or omniscient NPCs (e.g., zombies)

**Debugging:**
- Enable `drgbase_debug_relationships` to see relationship changes
- Use `drgbase_ai_omniscient 1` for testing to make all NPCs aware
- Check `IsBlind()` and `IsDeaf()` if detection isn't working
- Use `GetInSight()` to debug what NPC can currently see
