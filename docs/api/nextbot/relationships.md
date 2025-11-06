# Relationship & Faction Functions

Relationship management and faction system.

**File:** `lua/entities/drgbase_nextbot/relationships.lua` (831 lines)

---

## Faction Management

### ENT:SetFaction(faction)

Sets the NPC's primary faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (number) - Faction ID (FACTION_*)

**Returns:** None

**Example:**
```lua
self:SetFaction(FACTION_REBELS)
```

---

### ENT:GetFaction()

Gets primary faction.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `number` - Faction ID

---

### ENT:GetFactions()

Gets all factions NPC belongs to.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:**
- `table` - Array of faction IDs

---

### ENT:AddFaction(faction)

Adds NPC to a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (number) - Faction ID to add

**Returns:** None

---

### ENT:RemoveFaction(faction)

Removes NPC from a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (number) - Faction ID to remove

**Returns:** None

---

### ENT:HasFaction(faction)

Checks if NPC is in a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (number) - Faction ID

**Returns:**
- `boolean` - True if in faction

---

## Relationship Management

### ENT:SetRelationship(entity, disposition, priority)

Sets relationship to a specific entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `entity` (Entity) - Target entity
- `disposition` (number) - Relationship type (D_LI, D_HT, D_FR, D_NU)
- `priority` (number, optional) - Relationship priority

**Returns:** None

**Example:**
```lua
-- Make NPC like a specific player
self:SetRelationship(player, D_LI)

-- Make NPC hate an entity with high priority
self:SetRelationship(enemy, D_HT, 99)
```

**Priority System:**
The priority parameter determines which relationship takes precedence when multiple relationship rules apply to the same entity. Higher priority values override lower ones.

- Default priority: `1`
- Higher numbers = higher priority
- When priorities are equal, disposition priority is used: `D_LI (4) > D_FR (3) > D_HT (2) > D_NU (1)`
- Useful for making specific relationships override general ones

Example:
```lua
-- Set all zombies as neutral (priority 1)
self:SetClassRelationship("npc_zombie", D_NU, 1)

-- Make a specific zombie an enemy (priority 5 overrides class relationship)
self:SetEntityRelationship(specificZombie, D_HT, 5)
```

---

### ENT:GetRelationship(entity)

Gets relationship to an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `entity` (Entity) - Target entity

**Returns:**
- `number` - Disposition (D_LI, D_HT, D_FR, D_NU, D_ER)

**Example:**
```lua
local rel = self:GetRelationship(player)
if rel == D_HT then
    -- Player is enemy
elseif rel == D_LI then
    -- Player is friend
end
```

---

### ENT:RemoveRelationship(entity)

Removes specific relationship.

**Realm:** 🔴 SERVER

**Parameters:**
- `entity` (Entity) - Entity to remove relationship with

**Returns:** None

---

### ENT:ClearRelationships()

Clears all relationships.

**Realm:** 🔴 SERVER

**Parameters:** None

**Returns:** None

---

## Class & Model Relationships

### ENT:SetClassRelationship(class, disposition)

Sets relationship to all entities of a class.

**Realm:** 🔴 SERVER

**Parameters:**
- `class` (string) - Entity class name
- `disposition` (number) - Disposition

**Returns:** None

**Example:**
```lua
-- Hate all zombies
self:SetClassRelationship("npc_zombie", D_HT)
```

---

### ENT:GetClassRelationship(class)

Gets relationship to a class.

**Realm:** 🔴 SERVER

**Parameters:**
- `class` (string) - Entity class

**Returns:**
- `number` - Disposition

---

### ENT:SetModelRelationship(model, disposition)

Sets relationship to all entities with a model.

**Realm:** 🔴 SERVER

**Parameters:**
- `model` (string) - Model path
- `disposition` (number) - Disposition

**Returns:** None

---

### ENT:GetModelRelationship(model)

Gets relationship to a model.

**Realm:** 🔴 SERVER

**Parameters:**
- `model` (string) - Model path

**Returns:**
- `number` - Disposition

---

## Faction Relationships

### ENT:SetFactionRelationship(faction, disposition)

Sets relationship to all members of a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (number) - Faction ID
- `disposition` (number) - Disposition

**Returns:** None

**Example:**
```lua
-- Hate all Combine
self:SetFactionRelationship(FACTION_COMBINE, D_HT)

-- Like all Rebels
self:SetFactionRelationship(FACTION_REBELS, D_LI)
```

---

### ENT:GetFactionRelationship(faction)

Gets relationship to a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (number) - Faction ID

**Returns:**
- `number` - Disposition

---

## Enemy Determination

### ENT:IsEnemy(entity)

Checks if entity is an enemy.

**Realm:** 🔴 SERVER

**Parameters:**
- `entity` (Entity) - Entity to check

**Returns:**
- `boolean` - True if enemy (D_HT or D_FR)

**Example:**
```lua
if self:IsEnemy(ent) then
    self:SetEnemy(ent)
end
```

---

### ENT:IsFriend(entity)

Checks if entity is a friend.

**Realm:** 🔴 SERVER

**Parameters:**
- `entity` (Entity) - Entity to check

**Returns:**
- `boolean` - True if friend (D_LI)

---

### ENT:IsNeutral(entity)

Checks if entity is neutral.

**Realm:** 🔴 SERVER

**Parameters:**
- `entity` (Entity) - Entity to check

**Returns:**
- `boolean` - True if neutral (D_NU)

---

## Damage Tolerance System

The damage tolerance system automatically changes relationships based on received damage. When a friendly, neutral, or feared entity attacks the NPC, the relationship gradually shifts toward hostility.

**How It Works:**
1. Each time an ally/neutral/feared entity damages the NPC, a tolerance counter accumulates
2. The accumulated tolerance value is used as the priority for an enemy (D_HT) relationship
3. Once the tolerance exceeds existing relationship priorities, the entity becomes an enemy

**Configuration Properties:**
- `ENT.AllyDamageTolerance` (default: 0.33) - Tolerance increase per damage from allies
- `ENT.AfraidDamageTolerance` (default: 0.33) - Tolerance increase per damage from feared entities
- `ENT.NeutralDamageTolerance` (default: 0.33) - Tolerance increase per damage from neutrals

**Example:**
```lua
-- NPC tolerates 3 hits from allies before becoming hostile (0.33 * 3 ≈ 1.0 priority)
ENT.AllyDamageTolerance = 0.33

-- NPC becomes hostile after first hit from neutrals
ENT.NeutralDamageTolerance = 1.0

-- NPC never turns hostile from ally damage
ENT.AllyDamageTolerance = 0
```

**Notes:**
- Tolerance accumulates per entity (tracked separately for each attacker)
- The tolerance value becomes the priority of the hostile relationship
- Entities already set as enemies (D_HT) are not affected by this system
- Tolerance is not reduced over time - it's permanent once accumulated

---

## Relationship Priority System

The relationship priority system determines which relationship rule applies when multiple rules could affect the same entity. This allows for complex, layered relationship logic.

### How Relationships Are Calculated

When determining the relationship to an entity, DrGBase evaluates rules in this order:

1. **Team Relationships** (Absolute Override)
   - If both NPCs share the same team (via `ENT:SetTeam()`), they're automatically allies (D_LI)
   - Team relationships cannot be overridden by other rules
   - Team 0 = no team (normal relationship rules apply)

2. **Relationship Definers** (Priority-Based)
   The system collects all applicable relationship rules and selects the highest priority one:

   - **Entity-specific** relationships (set via `SetEntityRelationship()`)
   - **Class-specific** relationships (set via `SetClassRelationship()`)
   - **Model-specific** relationships (set via `SetModelRelationship()`)
   - **Faction** relationships (set via `SetFactionRelationship()`)
   - **Custom** relationships (returned by `CustomRelationship()` hook)
   - **Default** relationship (`ENT.DefaultRelationship` or D_NU)

3. **Priority Resolution**
   - Higher priority numbers take precedence (default is 1)
   - If priorities are equal, disposition priority is used: `D_LI (4) > D_FR (3) > D_HT (2) > D_NU (1)`
   - All matching faction relationships are considered and the highest priority wins

### Examples

**Example 1: Layered Relationships**
```lua
-- Set default: hate all players
self:SetClassRelationship("player", D_HT, 1)

-- But like players in the "FRIENDLY" faction (higher priority)
self:SetFactionRelationship("FRIENDLY", D_LI, 2)

-- But hate this specific friendly player (highest priority)
self:SetEntityRelationship(specificPlayer, D_HT, 10)
```

**Example 2: Faction Overrides**
```lua
-- Hate all Combine
self:SetFactionRelationship(FACTION_COMBINE, D_HT, 5)

-- Like a specific Combine soldier (higher priority overrides faction rule)
self:SetEntityRelationship(combineSoldier, D_LI, 10)
```

**Example 3: Damage Tolerance Changing Relationships**
```lua
-- Initially set as ally with priority 5
self:SetEntityRelationship(friendlyNPC, D_LI, 5)

-- After 15 hits (with AllyDamageTolerance = 0.33):
-- Accumulated tolerance = 0.33 * 15 = 4.95
-- This creates a D_HT relationship with priority 4.95
-- Since 4.95 < 5, the entity remains an ally

-- After 16 hits:
-- Accumulated tolerance = 0.33 * 16 = 5.28
-- Since 5.28 > 5, the entity becomes an enemy
```

### Priority Level Summary

From highest to lowest (when using same priority number):

1. Entity-specific relationships
2. Class-specific relationships
3. Model-specific relationships
4. Faction relationships
5. Custom relationships (hook)
6. Default relationship

**Note:** In practice, the priority **number** determines precedence, not the type. An entity relationship with priority 1 will be overridden by a faction relationship with priority 5.

---

## Related Hooks

- `ENT:OnRelationshipChanged(entity, oldDisp, newDisp)` - When relationship changes
- `ENT:OnFactionChanged(oldFaction, newFaction)` - When faction changes

---

## See Also

- [Enumerations](../core/enumerations.md) - Faction and disposition constants
- [AI Functions](./ai.md) - Enemy management
- [Faction Guide](../../guides/factions.md)
- [Relationship System](../../systems/relationships/README.md)
