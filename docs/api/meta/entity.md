# Entity Metatable Extensions

Extensions to the Entity metatable for additional utility functions.

**File:** `lua/drgbase/meta/entity.lua`

---

## Utility Functions

### Entity:DrG_IsSanic()

Checks if an entity is a SANIC nextbot (specific nextbot framework).

**Realm:** 🟣 SHARED

**Returns:**
- `boolean` - True if entity is a SANIC nextbot

**Example:**
```lua
if ent:DrG_IsSanic() then
    -- Handle SANIC nextbot differently
end
```

---

### Entity:DrG_IsDoor()

Checks if an entity is a door.

**Realm:** 🟣 SHARED

**Returns:**
- `boolean` - True if entity is a door (prop_door_rotating, func_door, func_door_rotating)

**Example:**
```lua
if ent:DrG_IsDoor() then
    ent:Fire("Open")
end
```

---

### Entity:DrG_SearchBone(searchBone)

Searches for a bone by name (supports partial matching).

**Realm:** 🟣 SHARED

**Parameters:**
- `searchBone` (string) - Bone name or partial name to search for

**Returns:**
- `number` - Bone ID, or nil if not found

**Example:**
```lua
-- Find head bone
local headBone = self:DrG_SearchBone("head")

-- Partial matching works
local chestBone = self:DrG_SearchBone("chest") -- Finds "ValveBiped.Bip01_Spine2"
```

---

## Trace Functions

### Entity:DrG_TraceLine(vec, data)

Performs a line trace from the entity.

**Realm:** 🟣 SHARED

**Parameters:**
- `vec` (Vector) - Direction vector to trace
- `data` (table, optional) - Trace options:
  - `start` (Vector) - Start position (default: entity position + OBB center)
  - `endpos` (Vector) - End position (default: start + vec)
  - `collisiongroup` (number) - Collision group (default: entity's collision group)
  - `mask` (number) - Trace mask (default: entity's solid mask on SERVER)
  - `filter` (table or Entity) - Entities to ignore (default: self, weapon, possessor)

**Returns:**
- `table` - Trace result

**Example:**
```lua
-- Trace forward 100 units
local tr = self:DrG_TraceLine(self:GetForward() * 100)
if tr.Hit then
    print("Hit:", tr.Entity)
end

-- Custom trace
local tr = self:DrG_TraceLine(Vector(0, 0, -100), {
    start = self:GetPos(),
    mask = MASK_SOLID_BRUSHONLY
})
```

---

### Entity:DrG_TraceHull(vec, data)

Performs a hull trace using the entity's collision bounds.

**Realm:** 🟣 SHARED

**Parameters:**
- `vec` (Vector) - Direction vector to trace
- `data` (table, optional) - Trace options (same as `DrG_TraceLine`):
  - `start` (Vector) - Start position (default: entity position)
  - `endpos` (Vector) - End position
  - `step` (boolean) - Use step height for mins (for DrGBase nextbots)
  - `maxs` (Vector) - Maximum bounds (default: entity's collision bounds)
  - `mins` (Vector) - Minimum bounds (default: entity's collision bounds)
  - `collisiongroup`, `mask`, `filter` - Same as TraceLine

**Returns:**
- `table` - Trace result

**Example:**
```lua
-- Check if entity can move forward
local tr = self:DrG_TraceHull(self:GetForward() * 50)
if not tr.Hit then
    -- Path is clear
end

-- Check for step
local tr = self:DrG_TraceHull(Vector(50, 0, 0), {step = true})
```

---

### Entity:DrG_TraceLineRadial(distance, precision, data)

Performs multiple line traces in a 360-degree pattern around the entity.

**Realm:** 🟣 SHARED

**Parameters:**
- `distance` (number) - Trace distance
- `precision` (number) - Number of traces to perform (evenly distributed)
- `data` (table, optional) - Trace options (same as `DrG_TraceLine`)

**Returns:**
- `table` - Array of trace results, sorted by distance (closest first)

**Example:**
```lua
-- Find nearest obstacle in 360 degrees
local traces = self:DrG_TraceLineRadial(500, 16)
local closestHit = traces[1] -- Closest hit
```

---

### Entity:DrG_TraceHullRadial(distance, precision, data)

Performs multiple hull traces in a 360-degree pattern around the entity.

**Realm:** 🟣 SHARED

**Parameters:**
- `distance` (number) - Trace distance
- `precision` (number) - Number of traces to perform
- `data` (table, optional) - Trace options (same as `DrG_TraceHull`)

**Returns:**
- `table` - Array of trace results, sorted by distance (closest first)

---

## Timer Functions

### Entity:DrG_Timer(duration, callback, ...)

Creates a simple timer that calls a function after a delay. Auto-validates entity.

**Realm:** 🟣 SHARED

**Parameters:**
- `duration` (number) - Delay in seconds
- `callback` (function) - Function to call: `callback(self, ...)`
- `...` - Additional arguments to pass to callback

**Example:**
```lua
-- Explode after 5 seconds
self:DrG_Timer(5, function(self)
    self:Explode()
end)

-- With arguments
self:DrG_Timer(3, function(self, damage, radius)
    self:DealAOEDamage(damage, radius)
end, 50, 200)
```

---

### Entity:DrG_LoopTimer(delay, callback, ...)

Creates a looping timer. Callback returns `false` to stop.

**Realm:** 🟣 SHARED

**Parameters:**
- `delay` (number) - Delay between iterations in seconds
- `callback` (function) - Function to call: `callback(self, ...)`, return `false` to stop
- `...` - Additional arguments

**Example:**
```lua
-- Pulse effect every second for 10 seconds
local count = 0
self:DrG_LoopTimer(1, function(self)
    self:EmitSound("ambient/pulse.wav")
    count = count + 1
    return count < 10 -- Stop after 10 pulses
end)
```

---

## Server-Only Functions

### Entity:DrG_RandomPos(min, max)

Gets a random position around the entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `min` (number) - Minimum distance
- `max` (number, optional) - Maximum distance (if omitted, `min` becomes max and min becomes 0)

**Returns:**
- `Vector` - Random valid position

**Example:**
```lua
-- Random position 100-500 units away
local pos = self:DrG_RandomPos(100, 500)

-- Random position 0-200 units away
local pos = self:DrG_RandomPos(200)
```

**Notes:**
- Uses navmesh if available
- Falls back to world trace if navmesh unavailable
- Ensures position is in world bounds

---

### Entity:DrG_Dissolve(type)

Dissolves the entity with a visual effect.

**Realm:** 🔴 SERVER

**Parameters:**
- `type` (number, optional) - Dissolve type (0-3, default: 0)
  - 0 = Energy
  - 1 = Heavy electrical
  - 2 = Light electrical
  - 3 = Core effect

**Returns:**
- `boolean` - True if dissolution started successfully

**Example:**
```lua
-- Dissolve with energy effect
self:DrG_Dissolve(0)

-- Heavy electrical dissolve
self:DrG_Dissolve(1)
```

---

### Entity:DrG_DeathNotice(attacker, inflictor)

Triggers death notification hooks.

**Realm:** 🔴 SERVER

**Parameters:**
- `attacker` (Entity) - The attacker
- `inflictor` (Entity, optional) - Weapon/inflictor (defaults to attacker)

**Notes:**
- Calls `PlayerDeath` hook for players
- Calls `OnNPCKilled` hook for NPCs

---

### Entity:DrG_CreateRagdoll(dmg)

Creates a ragdoll of the entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `dmg` (CTakeDamageInfo, optional) - Damage info for force application

**Returns:**
- `Entity` - Ragdoll entity, or NULL if failed

**Example:**
```lua
local ragdoll = self:DrG_CreateRagdoll(dmg)
if IsValid(ragdoll) then
    -- Ragdoll created successfully
end
```

**Notes:**
- Copies model, skin, bodygroups, color, scale
- Applies damage force to physics bones
- Handles fire and dissolve effects
- Only works if model has a valid ragdoll

---

### Entity:DrG_RagdollDeath(dmg)

Kills the entity and creates a ragdoll.

**Realm:** 🔴 SERVER

**Parameters:**
- `dmg` (CTakeDamageInfo, optional) - Damage info

**Returns:**
- `Entity` - Ragdoll entity, or NULL if failed

**Example:**
```lua
function ENT:OnKilled(dmg)
    self:DrG_RagdollDeath(dmg)
end
```

**Notes:**
- Calls death notice hooks
- Removes entity after creating ragdoll
- Handles undo/cleanup replacement for non-players

---

### Entity:DrG_AimAt(target, speed, feet)

Calculates the velocity vector needed to aim at a target.

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Target position or entity
- `speed` (number) - Projectile speed
- `feet` (boolean, optional) - Aim at feet instead of center

**Returns:**
- `Vector` - Aim direction
- `table` - Info table with trajectory data

**Example:**
```lua
-- Aim at target
local dir = self:DrG_AimAt(target:GetPos(), 1000)
self:SetVelocity(dir)
```

---

### Entity:DrG_ThrowAt(target, options, feet)

Calculates the velocity to throw at a target (with gravity).

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Target position or entity
- `options` (table) - Throw options:
  - `magnitude` (number) - Initial velocity magnitude (default: 1000)
  - Other physics options
- `feet` (boolean, optional) - Aim at feet

**Returns:**
- `Vector` - Throw direction
- `table` - Info table with trajectory data

**Example:**
```lua
-- Throw at target
local dir = self:DrG_ThrowAt(target:GetPos(), {magnitude = 800})
self:SetVelocity(dir)
```

---

## Effect Functions

### Entity:DrG_ParticleEffect(effect, ...)

Creates a particle effect attached to the entity.

**Realm:** 🔴 SERVER

**Parameters:**
- `effect` (string) - Particle effect name
- `...` - Attachment configuration:
  - `string` - Attachment point name
  - `Entity` - Control point entity
  - `Vector` - Control point position

**Returns:**
- `table` - Particle effect data

**Example:**
```lua
-- Simple effect
self:DrG_ParticleEffect("fire_small")

-- Effect on attachment
self:DrG_ParticleEffect("fire_small", "eyes")

-- Effect with control points
self:DrG_ParticleEffect("beam_effect", target, "attach_point")
```

---

### Entity:DrG_DynamicLight(color, radius, brightness, style, attachment)

Creates a dynamic light attached to the entity.

**Realm:** 🟣 SHARED

**Parameters:**
- `color` (Color, optional) - Light color (default: white)
- `radius` (number, optional) - Light radius (default: 1000)
- `brightness` (number, optional) - Brightness 0-1 (default: 1)
- `style` (number, optional) - Light style (flicker pattern)
- `attachment` (string, optional) - Attachment point name

**Returns:**
- `Entity` - Light entity (SERVER)
- `table` - Light data (CLIENT)

**Example:**
```lua
-- Red glow
self:DrG_DynamicLight(Color(255, 0, 0), 300, 1)

-- Light on eye attachment
self:DrG_DynamicLight(Color(0, 255, 0), 200, 1, nil, "eyes")
```

---

## See Also

- [Entity Helpers](../core/entity-helpers.md) - Global entity helper functions
- [Physics Metatable](./physobj.md) - Physics object extensions
- [Vector Metatable](./vector.md) - Vector extensions
