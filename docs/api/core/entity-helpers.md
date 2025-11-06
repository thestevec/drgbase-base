# Entity Helper Functions

Utility functions for entity operations, providing convenient shortcuts for common entity tasks like printing debug information, managing timers, performing traces, and networking.

**File:** `lua/drgbase/entity_helpers.lua`

## Print Functions

These functions output debug information about the entity to the console. Useful for development and troubleshooting.

### ENT:PrintPoseParameters()

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:** None

**Description:**
Prints all pose parameters of the entity with their min/max ranges to the console.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Print pose parameters for debugging
    self:PrintPoseParameters()
end
```

---

### ENT:PrintAnimations()

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:** None

**Description:**
Prints all animation sequences with their associated activities to the console. Shows sequence index, name, activity ID, and activity name.

**Example:**
```lua
function ENT:CustomInitialize()
    -- See all available animations
    self:PrintAnimations()
end
```

---

### ENT:PrintBones()

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:** None

**Description:**
Prints all bone names and their indices to the console.

**Example:**
```lua
function ENT:CustomInitialize()
    -- List all bones for attachment purposes
    self:PrintBones()
end
```

---

### ENT:PrintAttachments()

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:** None

**Description:**
Prints all attachment points with their IDs and names to the console.

**Example:**
```lua
function ENT:CustomInitialize()
    -- See available attachment points
    self:PrintAttachments()
end
```

---

### ENT:PrintBodygroups()

**Realm:** 🟣 SHARED

**Parameters:** None

**Returns:** None

**Description:**
Prints all bodygroups with their IDs, names, and number of subgroups to the console.

**Example:**
```lua
function ENT:CustomInitialize()
    -- Check bodygroup configuration
    self:PrintBodygroups()
end
```

---

## Timer Functions

Convenience wrappers for entity-bound timers that automatically handle entity validity.

### ENT:Timer(duration, callback, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `duration` (number) - Time in seconds before callback executes
- `callback` (function) - Function to call, receives self as first parameter
- `...` (any, optional) - Additional arguments to pass to callback

**Returns:** None

**Description:**
Creates a timer that executes after a delay. The timer automatically checks entity validity before executing the callback. This is a wrapper for `ENT:DrG_Timer()`.

**Example:**
```lua
function ENT:OnMeleeAttack(target)
    print("Starting attack...")
    self:Timer(1, function(self)
        print("Attack complete!")
        -- Do damage after 1 second delay
    end)
end
```

**Notes:**
- Automatically validates entity before callback execution
- Callback will not run if entity becomes invalid

---

### ENT:LoopTimer(delay, callback, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `delay` (number) - Time in seconds between each callback execution
- `callback` (function) - Function to call repeatedly, receives self as first parameter
- `...` (any, optional) - Additional arguments to pass to callback

**Returns:** None

**Description:**
Creates a repeating timer that executes at regular intervals. The callback should return `false` to stop the loop. This is a wrapper for `ENT:DrG_LoopTimer()`.

**Example:**
```lua
function ENT:StartHealing()
    self:LoopTimer(1, function(self)
        if self:Health() >= self:GetMaxHealth() then
            return false -- Stop healing
        end
        self:SetHealth(self:Health() + 5)
        return true -- Continue healing
    end)
end
```

**Notes:**
- Return `false` from callback to stop the timer
- Automatically stops if entity becomes invalid

---

## Trace Functions

Wrapper functions for performing traces from the entity's perspective. These simplify trace setup by automatically handling entity position, collision bounds, and filters.

### ENT:TraceLine(vec, data)

**Realm:** 🟣 SHARED

**Parameters:**
- `vec` (Vector, optional) - Direction vector from entity center
- `data` (table, optional) - Additional trace parameters:
  - `start` (Vector) - Custom start position (default: entity center)
  - `endpos` (Vector) - Custom end position (default: start + vec)
  - `collisiongroup` (number) - Collision group to use
  - `mask` (number) - Trace mask (SERVER only)
  - `filter` (table/Entity) - Entities to ignore

**Returns:**
- `table` - Trace result

**Description:**
Performs a line trace from the entity. Automatically configures the trace with entity's position, collision group, and filters including the entity, its weapon, and possessor. This is a wrapper for `ENT:DrG_TraceLine()`.

**Example:**
```lua
function ENT:OnRangeAttack(target)
    -- Trace forward to check line of sight
    local trace = self:TraceLine(self:GetForward() * 1000)
    if trace.Entity == target then
        print("Clear shot!")
    end
end
```

---

### ENT:TraceHull(vec, data)

**Realm:** 🟣 SHARED

**Parameters:**
- `vec` (Vector, optional) - Direction vector from entity position
- `data` (table, optional) - Additional trace parameters:
  - `start` (Vector) - Custom start position (default: entity position)
  - `endpos` (Vector) - Custom end position (default: start + vec)
  - `step` (boolean) - If true, uses step height for mins.z (nextbots only)
  - `mins` (Vector) - Custom minimum bounds
  - `maxs` (Vector) - Custom maximum bounds
  - `collisiongroup` (number) - Collision group to use
  - `mask` (number) - Trace mask (SERVER only)
  - `filter` (table/Entity) - Entities to ignore

**Returns:**
- `table` - Trace result

**Description:**
Performs a hull trace using the entity's collision bounds. Useful for checking if the entity can fit through spaces. This is a wrapper for `ENT:DrG_TraceHull()`.

**Example:**
```lua
function ENT:CanMoveForward()
    local trace = self:TraceHull(self:GetForward() * 100)
    return not trace.Hit
end
```

**Notes:**
- Automatically uses entity collision bounds for hull size
- Set `step = true` in data to use step height for vertical clearance

---

### ENT:TraceLineRadial(distance, precision, data)

**Realm:** 🟣 SHARED

**Parameters:**
- `distance` (number) - Maximum distance to trace
- `precision` (number) - Number of traces to perform in a 360° circle
- `data` (table, optional) - Additional trace parameters (see TraceLine)

**Returns:**
- `table` - Array of trace results sorted by distance (closest first)

**Description:**
Performs multiple line traces in a circle around the entity, useful for finding the nearest obstacle in any direction. This is a wrapper for `ENT:DrG_TraceLineRadial()`.

**Example:**
```lua
function ENT:FindNearestWall()
    local traces = self:TraceLineRadial(500, 16)
    local closest = traces[1]
    print("Nearest wall at:", closest.HitPos)
end
```

---

### ENT:TraceHullRadial(distance, precision, data)

**Realm:** 🟣 SHARED

**Parameters:**
- `distance` (number) - Maximum distance to trace
- `precision` (number) - Number of traces to perform in a 360° circle
- `data` (table, optional) - Additional trace parameters (see TraceHull)

**Returns:**
- `table` - Array of trace results sorted by distance (closest first)

**Description:**
Performs multiple hull traces in a circle around the entity. This is a wrapper for `ENT:DrG_TraceHullRadial()`.

**Example:**
```lua
function ENT:CheckSurroundings()
    local traces = self:TraceHullRadial(300, 8)
    for i, trace in ipairs(traces) do
        if trace.Hit then
            print("Obstacle at angle:", i * 45)
        end
    end
end
```

---

## Cooldown Functions

Simple cooldown system for limiting ability usage frequency.

### ENT:GetCooldown(name)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Unique identifier for the cooldown

**Returns:**
- `number` - Time remaining on cooldown in seconds (0 if not on cooldown)

**Description:**
Gets the remaining time for a named cooldown. Returns 0 if the cooldown has expired or doesn't exist.

**Example:**
```lua
function ENT:OnRangeAttack(target)
    if self:GetCooldown("special_attack") > 0 then
        -- Still on cooldown
        return
    end

    -- Perform special attack
    self:SetCooldown("special_attack", 5)
end
```

---

### ENT:SetCooldown(name, delay)

**Realm:** 🔴 SERVER

**Parameters:**
- `name` (string) - Unique identifier for the cooldown
- `delay` (number) - Duration of cooldown in seconds

**Returns:** None

**Description:**
Sets a cooldown timer that prevents an ability from being used for a specified duration. The cooldown is networked to clients via NW2 vars.

**Example:**
```lua
function ENT:FireProjectile()
    if self:GetCooldown("fire") > 0 then
        return -- Still on cooldown
    end

    -- Fire the projectile
    local proj = self:CreateProjectile("proj_drg_grenade")

    -- Set 3 second cooldown
    self:SetCooldown("fire", 3)
end
```

**Notes:**
- Must be called on SERVER
- Cooldown is automatically networked to clients
- Use GetCooldown() to check remaining time

---

## Miscellaneous Functions

### ENT:ScreenShake(amplitude, frequency, duration, radius)

**Realm:** 🟣 SHARED

**Parameters:**
- `amplitude` (number) - Shake intensity
- `frequency` (number) - Shake speed
- `duration` (number) - How long to shake in seconds
- `radius` (number) - Area of effect radius

**Returns:**
- Return value from `util.ScreenShake()`

**Description:**
Creates a screen shake effect at the entity's position. Affects all players within the radius.

**Example:**
```lua
function ENT:Explode()
    -- Create screen shake on explosion
    self:ScreenShake(10, 5, 2, 500)
end
```

---

### ENT:RandomPos(min, max)

**Realm:** 🔴 SERVER

**Parameters:**
- `min` (number) - Minimum distance from entity
- `max` (number, optional) - Maximum distance from entity (if omitted, min becomes max)

**Returns:**
- `Vector` - Random valid position on navmesh or world

**Description:**
Finds a random position at a certain distance from the entity. Attempts to use navmesh for valid pathable positions, falls back to world traces. This is a wrapper for `ENT:DrG_RandomPos()`.

**Example:**
```lua
function ENT:OnIdle()
    -- Wander to a random nearby location
    local randomPos = self:RandomPos(100, 500)
    self:FollowPath(randomPos)
    self:PauseCoroutine(5)
end
```

---

### ENT:PushEntity(ent, force)

**Realm:** 🔴 SERVER

**Parameters:**
- `ent` (Entity/table) - Entity or table of entities to push
- `force` (Vector) - Force vector in local space (forward, right, up)

**Returns:**
- `Vector` or `table` - Velocity vector(s) applied

**Description:**
Pushes entity(ies) away from this entity using local directional force. The force vector is relative to the entity's facing direction (X=forward, Y=right, Z=up). Handles NextBots, physics objects, and players differently.

**Example:**
```lua
function ENT:KnockbackAttack(target)
    -- Push target forward and upward
    self:PushEntity(target, Vector(500, 0, 300))
end

function ENT:ExplosionPush()
    -- Push all nearby entities
    local nearby = self:FindInSphere(self:GetPos(), 300)
    self:PushEntity(nearby, Vector(800, 0, 400))
end
```

**Notes:**
- Force is in local space relative to entity's facing direction
- Automatically handles NextBots, physics objects, and players
- Can accept a single entity or table of entities

---

## Network Functions

Functions for client-server communication and remote procedure calls.

### ENT:NetMessage(name, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Message identifier
- `...` (any) - Data to send with the message

**Returns:**
- Return value from `net.DrG_Send()`

**Description:**
Sends a networked message associated with this entity. Messages are received by `ENT:OnNetMessage()` or handled internally by `ENT:_HandleNetMessage()`.

**Example:**
```lua
-- Server to client
if SERVER then
    function ENT:NotifyClient(message)
        self:NetMessage("ShowAlert", message)
    end
end

-- Client receives
if CLIENT then
    function ENT:OnNetMessage(name, ...)
        if name == "ShowAlert" then
            local message = ...
            chat.AddText(Color(255, 0, 0), message)
        end
    end
end
```

---

### ENT:CallOnClient(name, ...)

**Realm:** 🔴 SERVER

**Parameters:**
- `name` (string) - Name of the client-side function to call
- `...` (any) - Arguments to pass to the function

**Returns:**
- Return value from `NetMessage()`

**Description:**
Calls a function on the client side with the provided arguments. The function must exist on the entity's client-side code.

**Example:**
```lua
-- Server
function ENT:TriggerEffect()
    self:CallOnClient("PlayLocalEffect")
end

-- Client
function ENT:PlayLocalEffect()
    -- This runs on client
    local effectData = EffectData()
    effectData:SetEntity(self)
    util.Effect("my_effect", effectData)
end
```

**Notes:**
- SERVER only
- Function name must be a string matching an entity method
- Target function will be called on clients

---

### ENT:NetCallback(name, callback, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Unique callback identifier
- `callback` (function) - Function to execute, receives self as first parameter
- `ply` (Player) - Target player (SERVER only)
- `...` (any) - Additional arguments

**Returns:**
- Return value from networking function

**Description:**
Creates a networked callback that executes on the receiving realm. On server, you must specify the target player. The callback receives the entity as first parameter.

**Example:**
```lua
-- Server requests data from client
if SERVER then
    function ENT:RequestClientData(ply)
        self:NetCallback("GetData", function(self, data)
            print("Received from client:", data)
        end, ply, "some_data_id")
    end
end

-- Client responds
if CLIENT then
    function ENT:OnNetMessage(name, ...)
        if name == "GetData" then
            local dataId = ...
            local data = self:GetLocalData(dataId)
            -- Send data back to server
        end
    end
end
```

**Notes:**
- Automatically handles entity validity checks
- On SERVER, requires player parameter
- On CLIENT, player parameter is omitted

---

### ENT:OnNetMessage(name, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Message identifier
- `ply` (Player) - Sending player (SERVER only)
- `...` (any) - Message data

**Returns:** None (override this function)

**Description:**
Hook called when a net message is received for this entity. Override this to handle custom network messages.

**Example:**
```lua
function ENT:OnNetMessage(name, ...)
    if name == "CustomAction" then
        local data = ...
        print("Received custom action:", data)
    end
end
```

**Notes:**
- Override this function to handle messages
- On SERVER, second parameter is the sending player
- On CLIENT, no player parameter

---

### ENT:_HandleNetMessage(...)

**Realm:** 🟣 SHARED

**Parameters:** (varies)

**Returns:**
- `boolean` - Return true to prevent OnNetMessage from being called

**Description:**
Internal handler for network messages. Not intended for override - use `ENT:OnNetMessage()` instead.

---

## Effect Functions

Functions for creating visual effects.

### ENT:ParticleEffect(effect, ...)

**Realm:** 🔴 SERVER

**Parameters:**
- `effect` (string) - Particle effect name
- `...` - Variable parameters:
  - First string argument: attachment name
  - Entity arguments: control point entities
  - Vector arguments: control point positions
  - String after entity: control point attachment

**Returns:**
- Return value from `DrGBase.ParticleEffect()`

**Description:**
Creates a particle effect attached to this entity. Supports complex particle setups with control points and attachments. This is a wrapper for `ENT:DrG_ParticleEffect()`.

**Example:**
```lua
function ENT:OnSpawn()
    -- Simple particle on entity
    self:ParticleEffect("fire_medium_01")

    -- Particle on specific attachment
    self:ParticleEffect("fire_small_01", "chest")

    -- Particle with control point at target
    self:ParticleEffect("blood_impact", target)

    -- Complex particle with position control point
    self:ParticleEffect("laser_beam", Vector(0, 0, 100))
end
```

**Notes:**
- SERVER only
- Particle is automatically networked to clients
- Supports multiple control points and attachments

---

### ENT:DynamicLight(color, radius, brightness, style, attachment)

**Realm:** 🟣 SHARED

**Parameters:**
- `color` (Color, optional) - Light color (default: white)
- `radius` (number, optional) - Light radius (default: 1000)
- `brightness` (number, optional) - Light intensity 0-n (default: 1)
- `style` (string, optional) - Light style pattern (flicker, pulse, etc.)
- `attachment` (string/number, optional) - Attachment point name or ID

**Returns:**
- `Entity` - Light entity (SERVER)
- `table` - DynamicLight object (CLIENT)

**Description:**
Creates a dynamic light attached to the entity. On server, creates a light_dynamic entity. On client, creates a DynamicLight. This is a wrapper for `ENT:DrG_DynamicLight()`.

**Example:**
```lua
function ENT:OnSpawn()
    -- Red glow
    self:DynamicLight(Color(255, 0, 0), 300, 2)

    -- Flickering light on attachment
    self:DynamicLight(Color(255, 200, 100), 400, 3, "6", "eyes")
end
```

**Notes:**
- Works differently on SERVER vs CLIENT
- Light is parented to entity and removed with it
- Style patterns: 0-13 for various flicker/strobe effects
- Common styles: "0" (normal), "6" (flicker), "10" (strobe)

---

## See Also

- [DrGBase Global](./drgbase.md)
- [Entity Metatable](../meta/entity.md)
- [Nextbot Functions](../nextbot/README.md)
- [Timer Module](../modules/timer.md)
- [Net Module](../modules/net.md)
