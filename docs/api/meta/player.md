# Player Metatable Extensions

Extensions to the Player metatable for possession, button tracking, toolgun selection, and factions.

**File:** `lua/drgbase/meta/player.lua`

---

## Possession Functions

### Player:DrG_IsPossessing()

Checks if the player is currently possessing a nextbot.

**Realm:** 🟣 SHARED

**Returns:**
- `boolean` - True if possessing a nextbot

**Example:**
```lua
if ply:DrG_IsPossessing() then
    print("Player is possessing a nextbot")
end
```

---

### Player:DrG_Possessing()

Gets the nextbot the player is currently possessing.

**Realm:** 🟣 SHARED

**Returns:**
- `Entity` - Possessed nextbot, or NULL if not possessing

**Example:**
```lua
local npc = ply:DrG_Possessing()
if IsValid(npc) then
    print("Possessing:", npc)
end
```

**Alias:** `Player:DrG_GetPossessing()`

---

## Button State Functions

### Player:DrG_ButtonDown(button)

Checks if a button is currently held down.

**Realm:** 🟣 SHARED

**Parameters:**
- `button` (number) - Button constant (e.g., KEY_E, MOUSE_LEFT)

**Returns:**
- `boolean` - True if button is held down

**Example:**
```lua
if ply:DrG_ButtonDown(KEY_E) then
    print("E is held down")
end
```

---

### Player:DrG_ButtonPressed(button)

Checks if a button was just pressed this frame.

**Realm:** 🟣 SHARED

**Parameters:**
- `button` (number) - Button constant

**Returns:**
- `boolean` - True if button was just pressed

**Example:**
```lua
if ply:DrG_ButtonPressed(KEY_SPACE) then
    print("Space was just pressed!")
end
```

---

### Player:DrG_ButtonUp(button)

Checks if a button is currently not pressed.

**Realm:** 🟣 SHARED

**Parameters:**
- `button` (number) - Button constant

**Returns:**
- `boolean` - True if button is not pressed

---

### Player:DrG_ButtonReleased(button)

Checks if a button was just released this frame.

**Realm:** 🟣 SHARED

**Parameters:**
- `button` (number) - Button constant

**Returns:**
- `boolean` - True if button was just released

**Example:**
```lua
if ply:DrG_ButtonReleased(MOUSE_LEFT) then
    print("Left mouse button released")
end
```

---

## Toolgun Selection Functions

### Player:DrG_GetSelectionTable(mode)

Gets the selection table for a toolgun mode.

**Realm:** 🟣 SHARED

**Parameters:**
- `mode` (string, optional) - Tool mode name (defaults to current tool)

**Returns:**
- `table` - Table of selected entities

**Example:**
```lua
local selection = ply:DrG_GetSelectionTable("drgbase_tool_faction")
```

---

### Player:DrG_SelectedEntities(mode)

Returns an iterator for selected entities.

**Realm:** 🟣 SHARED

**Parameters:**
- `mode` (string, optional) - Tool mode name

**Returns:**
- `function` - Iterator function

**Example:**
```lua
for ent in ply:DrG_SelectedEntities() do
    print("Selected:", ent)
end
```

---

### Player:DrG_GetSelectedEntities(mode)

Gets an array of all selected entities.

**Realm:** 🟣 SHARED

**Parameters:**
- `mode` (string, optional) - Tool mode name

**Returns:**
- `table` - Array of entities

**Example:**
```lua
local entities = ply:DrG_GetSelectedEntities()
print("Selected", #entities, "entities")
```

---

### Player:DrG_IsEntitySelected(ent, mode)

Checks if an entity is selected.

**Realm:** 🟣 SHARED

**Parameters:**
- `ent` (Entity) - Entity to check
- `mode` (string, optional) - Tool mode name

**Returns:**
- `boolean` - True if entity is selected

---

### Player:DrG_SelectEntity(ent, mode)

Selects an entity with the toolgun.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to select
- `mode` (string, optional) - Tool mode name

**Example:**
```lua
ply:DrG_SelectEntity(npc, "drgbase_tool_faction")
```

---

### Player:DrG_DeselectEntity(ent, mode)

Deselects an entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to deselect
- `mode` (string, optional) - Tool mode name

---

### Player:DrG_ClearSelectedEntities(mode)

Clears all selected entities.

**Realm:** 🔴 SERVER

**Parameters:**
- `mode` (string, optional) - Tool mode name

**Example:**
```lua
ply:DrG_ClearSelectedEntities()
```

---

### Player:DrG_ToggleEntitySelect(ent, mode)

Toggles an entity's selection state.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to toggle
- `mode` (string, optional) - Tool mode name

---

### Player:DrG_CleverEntitySelect(ent, mode)

Smart selection: clears others unless SHIFT is held.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to select
- `mode` (string, optional) - Tool mode name

**Example:**
```lua
-- Clicking without SHIFT: selects only this entity
-- Clicking with SHIFT: adds to selection
ply:DrG_CleverEntitySelect(npc)
```

---

### Player:DrG_SingleEntitySelect(ent, mode)

Selects a single entity, clearing others if needed.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to select
- `mode` (string, optional) - Tool mode name

---

## Faction Functions

### Player:DrG_JoinFaction(faction)

Adds the player to a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (string) - Faction name (case-insensitive)

**Example:**
```lua
ply:DrG_JoinFaction("COMBINE")
ply:DrG_JoinFaction("rebels") -- case doesn't matter
```

**Notes:**
- Automatically updates relationships with all DrGBase nextbots
- Faction names are converted to uppercase

---

### Player:DrG_LeaveFaction(faction)

Removes the player from a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (string) - Faction name

**Example:**
```lua
ply:DrG_LeaveFaction("COMBINE")
```

---

### Player:DrG_IsInFaction(faction)

Checks if the player is in a faction.

**Realm:** 🔴 SERVER

**Parameters:**
- `faction` (string) - Faction name

**Returns:**
- `boolean` - True if player is in the faction

**Example:**
```lua
if ply:DrG_IsInFaction("COMBINE") then
    print("Player is Combine")
end
```

---

### Player:DrG_GetFactions()

Gets all factions the player is in.

**Realm:** 🔴 SERVER

**Returns:**
- `table` - Array of faction names

**Example:**
```lua
local factions = ply:DrG_GetFactions()
for _, faction in ipairs(factions) do
    print("In faction:", faction)
end
```

---

### Player:DrG_JoinFactions(factions)

Joins multiple factions at once.

**Realm:** 🔴 SERVER

**Parameters:**
- `factions` (table) - Array of faction names

**Example:**
```lua
ply:DrG_JoinFactions({"COMBINE", "OVERWATCH", "MILITARY"})
```

---

### Player:DrG_LeaveFactions(factions)

Leaves multiple factions at once.

**Realm:** 🔴 SERVER

**Parameters:**
- `factions` (table) - Array of faction names

---

### Player:DrG_LeaveAllFactions()

Removes the player from all factions.

**Realm:** 🔴 SERVER

**Example:**
```lua
ply:DrG_LeaveAllFactions()
```

---

## Miscellaneous Functions

### Player:DrG_Luminosity()

Gets the light level at the player's position.

**Realm:** 🟣 SHARED

**Returns:**
- `number` - Light level (0.0 = dark, 1.0 = bright)

**Example:**
```lua
local brightness = ply:DrG_Luminosity()
if brightness < 0.3 then
    print("Player is in darkness")
end
```

**Notes:**
- **CLIENT:** Calculated using render.GetLightColor() at eye position
- **SERVER:** Synced from client every second (requires `drgbase_update_luminosity` convar enabled)
- Useful for stealth mechanics and AI detection

---

### Player:DrG_Immobilize()

Immobilizes the player by placing them in an invisible vehicle.

**Realm:** 🔴 SERVER

**Example:**
```lua
-- Freeze player in place
ply:DrG_Immobilize()

-- Release later
if IsValid(ply:GetVehicle()) then
    ply:ExitVehicle()
end
```

**Notes:**
- Creates an invisible prop_vehicle_prisoner_pod
- Useful for cutscenes or temporary immobilization
- Player can still look around but cannot move

---

### Player:DrG_AddUndo(ent, type, text)

Adds an entity to the player's undo stack.

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity) - Entity to add to undo
- `type` (string) - Undo type identifier
- `text` (string, optional) - Custom undo text (default: "Undone #className")

**Example:**
```lua
local npc = ents.Create("npc_drg_zombie")
npc:Spawn()

ply:DrG_AddUndo(npc, "DrGBase NPC", "Undone Zombie NPC")
```

---

### Player:DrG_NetCallback(name, callback, ...)

Calls a client-side network callback for this player.

**Realm:** 🔴 SERVER

**Parameters:**
- `name` (string) - Callback name
- `callback` (function) - Callback function
- `...` - Arguments to pass

**Returns:**
- Varies based on callback

**See Also:**
- [Net Module](../modules/net.md) - Network callback system

---

## ConVars

### drgbase_update_luminosity

Controls whether player luminosity is synced to server.

- **Default:** `1` (enabled)
- **Type:** Boolean
- **Flags:** ARCHIVE, NOTIFY, REPLICATED

---

## See Also

- [Possession System](../nextbot/possession.md) - Nextbot possession mechanics
- [Faction System Guide](../../guides/factions.md) - Working with factions
- [Relationships API](../nextbot/relationships.md) - Faction relationships
