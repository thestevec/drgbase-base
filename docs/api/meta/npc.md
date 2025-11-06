# NPC Metatable Extensions

Extensions to the NPC metatable for relationship management.

**File:** `lua/drgbase/meta/npc.lua`

---

## Relationship Functions

### NPC:DrG_SetRelationship(ent, disp)

Sets the relationship between this NPC and another entity with automatic priority handling.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Target entity to set relationship with
- `disp` (number) - Disposition value (D_HT, D_FR, D_LI, D_NU)

**Example:**
```lua
-- Make NPC hate player
npc:DrG_SetRelationship(player, D_HT)

-- Make NPC like another NPC
npc:DrG_SetRelationship(otherNPC, D_LI)

-- Make NPC neutral to entity
npc:DrG_SetRelationship(ent, D_NU)
```

**Notes:**
- Automatically increments priority for each relationship set
- Compatible with CPTBase NPCs (uses priority 99)
- For standard NPCs, maintains a priority counter per entity
- Handles invalid entities gracefully

**Disposition Values:**
- `D_HT` (1) - Hate (will attack)
- `D_FR` (2) - Fear (will flee)
- `D_LI` (4) - Like (friendly)
- `D_NU` (3) - Neutral (ignore)

**See Also:**
- [Relationships API](../nextbot/relationships.md) - DrGBase nextbot relationship system
- [Enumerations](../core/enumerations.md) - All disposition constants
