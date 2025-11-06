# Enumerations Reference

Quick reference for all DrGBase constants and enumerations.

For detailed documentation, see [Enumerations API](../api/core/enumerations.md).

---

## Quick Lookup Tables

### Factions

#### Half-Life 2 Factions

| Constant | Value | Members | Typical Enemies |
|----------|-------|---------|-----------------|
| `FACTION_REBELS` | `"FACTION_HL2_REBELS"` | Citizens, Barney, Alyx, Vortigaunts | Combine, Zombies |
| `FACTION_COMBINE` | `"FACTION_HL2_COMBINE"` | Soldiers, Metrocops, Striders, Manhacks | Rebels, Players |
| `FACTION_ZOMBIES` | `"FACTION_HL2_ZOMBIES"` | Zombies, Headcrabs (all types) | All living creatures |
| `FACTION_ANTLIONS` | `"FACTION_HL2_ANTLIONS"` | Antlions, Antlion Guards | Most factions |
| `FACTION_ANIMALS` | `"FACTION_HL2_ANIMALS"` | Crows, Pigeons, Seagulls | Neutral/Defensive |
| `FACTION_BARNACLES` | `"FACTION_HL2_BARNACLES"` | Barnacles | All (ceiling trap) |
| `FACTION_GMAN` | `"FACTION_HL2_GMAN"` | G-Man | Neutral/Special |

#### Half-Life 1 Factions

| Constant | Value | Members |
|----------|-------|---------|
| `FACTION_XEN_ARMY` | `"FACTION_HL_XEN_ARMY"` | Alien Grunts, Controllers, Gargantua |
| `FACTION_XEN_WILDLIFE` | `"FACTION_HL_XEN_WILDLIFE"` | Bullsquids, Houndeyes, Snarks |
| `FACTION_HECU` | `"FACTION_HL_HECU"` | Marines, Assassins |

#### Special

| Constant | Value | Purpose |
|----------|-------|---------|
| `FACTION_SANIC` | `"FACTION_SHITTY_SANIC_CLONES"` | Easter egg faction |

---

### Relationship Dispositions

| Constant | Value (Server) | Value (Client) | Behavior |
|----------|----------------|----------------|----------|
| `D_LI` | GMod D_LI | 3 | Like/Ally - Will not attack, may assist |
| `D_HT` | GMod D_HT | 1 | Hate/Enemy - Attack on sight |
| `D_FR` | GMod D_FR | 2 | Fear - Flee and avoid |
| `D_NU` | GMod D_NU | 4 | Neutral - Ignore unless provoked |
| `D_ER` | N/A | 0 | Error/Invalid |

**Usage:**
```lua
self:SetDefaultRelationship(D_HT) -- Hostile to all
self:AddRelationship(player, D_LI) -- Friendly to specific player
```

---

### Possession Modes

| Constant | Value | Aliases | Description |
|----------|-------|---------|-------------|
| `POSSESSION_MOVE_8DIR` | 1 | `POSSESSION_MOVE_NSEW`, `POSSESSION_MOVE_COMPASS` | 8-directional with diagonals (default) |
| `POSSESSION_MOVE_1DIR` | 2 | `POSSESSION_MOVE_FORWARD` | Forward-only movement |
| `POSSESSION_MOVE_4DIR` | 3 | None | 4 cardinal directions only |
| `POSSESSION_MOVE_CUSTOM` | 0 | None | Custom implementation |

**Usage:**
```lua
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
```

---

### AI Behavior Types

| Constant | Value | Description | Usage |
|----------|-------|-------------|-------|
| `AI_BEHAV_CUSTOM` | 0 | Custom AI behavior | Implement all AI through hooks |
| `AI_BEHAV_BASE` | 1 | Base AI (default) | Standard DrGBase AI for creatures |
| `AI_BEHAV_HUMAN` | 2 | Human AI | Enhanced AI for humanoids with weapons |

**Usage:**
```lua
ENT.AIBehaviour = AI_BEHAV_HUMAN
```

---

### Patrol Types

| Constant | Value | Priority | Description |
|----------|-------|----------|-------------|
| `PATROL_POS` | 0 | Lowest | Regular patrol points |
| `PATROL_SEARCH` | 100 | Medium | Search/investigation locations |
| `PATROL_SOUND` | 200 | Highest | Sound origin locations |

**Usage:**
```lua
self:AddPatrolPos(pos, PATROL_SOUND) -- High priority
```

---

### Node Types

| Constant | Value | Description |
|----------|-------|-------------|
| `NODE_TYPE_GROUND` | 2 | Ground/walking navigation |
| `NODE_TYPE_AIR` | 3 | Air/flying navigation |
| `NODE_TYPE_CLIMB` | 4 | Climbing navigation |
| `NODE_TYPE_WATER` | 5 | Water/swimming navigation |

---

### Input Flags

| Constant | Value | Description |
|----------|-------|-------------|
| `IN_ATTACK3` | 33554432 | Third attack button (possession) |

**Usage:**
```lua
ENT.PossessionBinds = {
    [IN_ATTACK3] = {{
        onkeydown = function(self)
            -- Special attack
        end
    }}
}
```

---

## Common Value Tables

### NPC States (Garry's Mod Built-in)

| State | Description |
|-------|-------------|
| `NPC_STATE_IDLE` | NPC is idle, no enemy |
| `NPC_STATE_ALERT` | NPC is alert, searching for enemy |
| `NPC_STATE_COMBAT` | NPC is in active combat |
| `NPC_STATE_SCRIPT` | NPC is running scripted sequence |
| `NPC_STATE_PLAYDEAD` | NPC is playing dead |
| `NPC_STATE_PRONE` | NPC is prone |
| `NPC_STATE_DEAD` | NPC is dead |

### Damage Types (Garry's Mod Built-in)

| Type | Common Uses |
|------|-------------|
| `DMG_GENERIC` | Generic damage |
| `DMG_CRUSH` | Crushing damage |
| `DMG_BULLET` | Bullet damage |
| `DMG_SLASH` | Melee/slash attacks |
| `DMG_BURN` | Fire damage |
| `DMG_VEHICLE` | Vehicle collision |
| `DMG_FALL` | Fall damage |
| `DMG_BLAST` | Explosive damage |
| `DMG_CLUB` | Blunt weapon damage |
| `DMG_SHOCK` | Electrical damage |
| `DMG_SONIC` | Sonic damage |
| `DMG_ENERGYBEAM` | Energy beam damage |
| `DMG_POISON` | Poison damage |
| `DMG_RADIATION` | Radiation damage |
| `DMG_ACID` | Acid damage |

**Usage:**
```lua
local dmg = DamageInfo()
dmg:SetDamageType(DMG_SLASH)
```

### Trace Masks (Garry's Mod Built-in)

| Mask | Description |
|------|-------------|
| `MASK_SOLID` | Everything solid |
| `MASK_PLAYERSOLID` | Solid for players |
| `MASK_NPCSOLID` | Solid for NPCs |
| `MASK_WATER` | Water only |
| `MASK_OPAQUE` | Blocks line of sight |
| `MASK_VISIBLE` | Blocks visibility |
| `MASK_SHOT` | Bullets |
| `MASK_SHOT_HULL` | Bullet hull traces |

**Usage:**
```lua
local tr = util.TraceLine({
    start = start,
    endpos = endpos,
    filter = self,
    mask = MASK_SHOT
})
```

### Hit Groups (Garry's Mod Built-in)

| Hit Group | Value | Description |
|-----------|-------|-------------|
| `HITGROUP_GENERIC` | 0 | Generic hit |
| `HITGROUP_HEAD` | 1 | Head shot |
| `HITGROUP_CHEST` | 2 | Chest hit |
| `HITGROUP_STOMACH` | 3 | Stomach hit |
| `HITGROUP_LEFTARM` | 4 | Left arm |
| `HITGROUP_RIGHTARM` | 5 | Right arm |
| `HITGROUP_LEFTLEG` | 6 | Left leg |
| `HITGROUP_RIGHTLEG` | 7 | Right leg |
| `HITGROUP_GEAR` | 10 | Gear/equipment |

**Usage:**
```lua
function ENT:OnDeath(dmg, delay, hitgroup)
    if hitgroup == HITGROUP_HEAD then
        -- Headshot death
    end
end
```

---

## Usage Examples

### Setting Up Factions

```lua
function ENT:CustomInitialize()
    -- Join Combine faction
    self:SetFaction(FACTION_COMBINE)

    -- Hostile to rebels
    self:SetFactionRelationship(FACTION_REBELS, D_HT)

    -- Neutral to animals
    self:SetFactionRelationship(FACTION_ANIMALS, D_NU)
end
```

### Possession Configuration

```lua
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:MeleeAttack()
        end
    }},
    [IN_ATTACK2] = {{
        coroutine = true,
        onkeydown = function(self)
            self:RangedAttack()
        end
    }}
}
```

### Patrol System

```lua
function ENT:OnHearSound(sound)
    -- Add high-priority patrol point at sound location
    self:AddPatrolPos(sound.pos, PATROL_SOUND)
end

function ENT:OnLostEnemy(enemy)
    -- Search last known position
    self:AddPatrolPos(enemy:GetPos(), PATROL_SEARCH)
end
```

---

## See Also

- [Enumerations API (Detailed)](../api/core/enumerations.md)
- [Relationship System](../../systems/relationships/README.md)
- [Faction Guide](../guides/factions.md)
- [Possession System](../../systems/possession/README.md)
