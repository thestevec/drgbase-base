# Vector Metatable Extensions

Extensions to the Vector metatable for trajectory calculations and utility functions.

**File:** `lua/drgbase/meta/vector.lua`

---

## Trajectory Calculation

### Vector:DrG_CalcLineTrajectory(target, speed, feet)

Calculates a straight-line trajectory to a target.

**Realm:** 🟣 SHARED

**Parameters:**
- `target` (Vector or Entity) - Target position or moving entity
- `speed` (number) - Projectile speed
- `feet` (boolean, optional) - Aim at entity's feet instead of center

**Returns:**
- `Vector` - Velocity vector
- `table` - Trajectory info:
  - `duration` (number) - Time to reach target
  - `normal` (Vector) - Normalized direction
  - `forward` (Vector) - Forward component (XY plane)
  - `magnitude` (number) - Velocity magnitude
  - `pitch` (number) - Pitch angle in degrees
  - `Predict(t)` (function) - Predicts position at time `t`
  - `ballistic` (boolean) - Always `false` for line trajectories

**Example:**
```lua
local origin = Vector(0, 0, 100)
local target = Vector(500, 500, 100)
local vel, info = origin:DrG_CalcLineTrajectory(target, 1000)

print("Time to target:", info.duration)
print("Velocity:", vel)

-- Predict position after 0.5 seconds
local futurePos = info.Predict(0.5)
```

**Notes:**
- For moving entities, automatically leads the target based on velocity
- Always possible if speed > 0
- No gravity calculation

---

### Vector:DrG_CalcBallisticTrajectory(target, options, feet)

Calculates a gravity-affected ballistic trajectory.

**Realm:** 🟣 SHARED

**Parameters:**
- `target` (Vector or Entity) - Target position or entity
- `options` (table, optional) - Trajectory options:
  - `magnitude` (number) - Initial velocity magnitude
  - `pitch` (number) - Launch angle in degrees (-90 to 90)
  - `highest` (boolean) - Use high arc instead of low arc
  - `recursive` (boolean) - Retry with adjusted values if impossible
- `feet` (boolean, optional) - Aim at entity's feet

**Returns:**
- `Vector` - Velocity vector
- `table` - Trajectory info:
  - `duration` (number) - Flight time (-1 if impossible)
  - `highest` (number) - Time to reach peak height
  - `height` (number) - Maximum height above launch point
  - `ballistic` (boolean) - Always `true`
  - Other fields from line trajectory

**Example:**
```lua
local origin = Vector(0, 0, 0)
local target = Vector(500, 0, 100)

-- Calculate with specific velocity
local vel, info = origin:DrG_CalcBallisticTrajectory(target, {
    magnitude = 800,
    recursive = true
})

if info.duration > 0 then
    print("Flight time:", info.duration)
    print("Peak height:", info.height)
    print("Arc apex at:", info.highest, "seconds")
else
    print("Target unreachable with this velocity")
end

-- Use high arc instead of low arc
local vel2, info2 = origin:DrG_CalcBallisticTrajectory(target, {
    magnitude = 800,
    highest = true
})

-- Specific launch angle
local vel3, info3 = origin:DrG_CalcBallisticTrajectory(target, {
    pitch = 45
})
```

**Calculation Modes:**

1. **Magnitude only:** Calculates best pitch angle for given speed
2. **Pitch only:** Calculates required speed for given angle
3. **Both:** Uses exact values (may be impossible)
4. **Neither:** Defaults to 45° and calculates required speed

**Notes:**
- Automatically leads moving targets
- May be impossible for distant targets at low velocities
- `recursive` option retries with adjusted parameters
- Uses gravity from `physenv.GetGravity()`

---

### Vector:DrG_TrajectoryInfo(direction, ballistic)

Creates trajectory info from a velocity vector.

**Realm:** 🟣 SHARED

**Parameters:**
- `direction` (Vector) - Velocity/direction vector
- `ballistic` (boolean) - Whether trajectory includes gravity

**Returns:**
- `table` - Trajectory info table (same structure as calc functions)

**Example:**
```lua
local origin = Vector(0, 0, 0)
local velocity = Vector(100, 100, 50)
local info = origin:DrG_TrajectoryInfo(velocity, true)

-- Predict position after 2 seconds
local pos, vel = info.Predict(2)
print("Position:", pos)
print("Velocity:", vel)
```

---

### Vector:DrG_Data()

Extracts directional data from a vector.

**Realm:** 🟣 SHARED

**Returns:**
- `table` - Vector data:
  - `normal` (Vector) - Normalized vector
  - `forward` (Vector) - XY component normalized
  - `length` (number) - Vector magnitude
  - `pitch` (number) - Pitch angle in degrees

**Example:**
```lua
local vec = Vector(100, 100, 50)
local data = vec:DrG_Data()

print("Length:", data.length)
print("Pitch:", data.pitch)
print("Forward:", data.forward) -- XY direction
```

---

## Distance & Direction

### Vector:DrG_ManhattanDistance(pos)

Calculates Manhattan distance (taxicab distance) to another position.

**Realm:** 🟣 SHARED

**Parameters:**
- `pos` (Vector) - Target position

**Returns:**
- `number` - Manhattan distance (sum of absolute axis differences)

**Example:**
```lua
local a = Vector(0, 0, 0)
local b = Vector(10, 10, 10)

local euclidean = a:Distance(b) -- ~17.32
local manhattan = a:DrG_ManhattanDistance(b) -- 30
```

**Notes:**
- Faster than Euclidean distance (no square root)
- Useful for grid-based pathfinding
- Good approximation for quick distance checks

---

### Vector:DrG_Direction(pos)

Gets the direction vector from this position to another.

**Realm:** 🟣 SHARED

**Parameters:**
- `pos` (Vector) - Target position

**Returns:**
- `Vector` - Direction vector (not normalized)

**Example:**
```lua
local from = Vector(0, 0, 0)
local to = Vector(100, 50, 25)

local dir = from:DrG_Direction(to) -- Vector(100, 50, 25)
local normalized = dir:GetNormalized()
```

**Alias:** `Vector:DrG_Dir(pos)`

---

### Vector:DrG_Degrees(vec2, origin)

Calculates the angle between two vectors in degrees.

**Realm:** 🟣 SHARED

**Parameters:**
- `vec2` (Vector) - Second vector
- `origin` (Vector, optional) - Origin point (default: 0,0,0)

**Returns:**
- `number` - Angle in degrees (0-180)

**Example:**
```lua
local v1 = Vector(1, 0, 0)
local v2 = Vector(0, 1, 0)

local angle = v1:DrG_Degrees(v2) -- 90 degrees

-- With custom origin
local angle2 = v1:DrG_Degrees(v2, Vector(10, 10, 0))
```

---

### Vector:DrG_Away(pos)

Gets a position on the opposite side of this vector from a given point.

**Realm:** 🟣 SHARED

**Parameters:**
- `pos` (Vector) - Reference position

**Returns:**
- `Vector` - Position on opposite side

**Example:**
```lua
local center = Vector(0, 0, 0)
local point = Vector(100, 0, 0)

-- Get point on opposite side
local opposite = point:DrG_Away(center) -- Vector(-100, 0, 0)

-- Useful for retreat positions
local enemyPos = enemy:GetPos()
local myPos = self:GetPos()
local retreatPos = myPos:DrG_Away(enemyPos)
```

---

## Utility Functions

### Vector:DrG_Round(decimals)

Rounds vector components to a number of decimal places.

**Realm:** 🟣 SHARED

**Parameters:**
- `decimals` (number, optional) - Decimal places (default: 0)

**Returns:**
- `Vector` - New rounded vector

**Example:**
```lua
local vec = Vector(1.23456, 2.34567, 3.45678)

local rounded = vec:DrG_Round(2) -- Vector(1.23, 2.35, 3.46)
local integers = vec:DrG_Round() -- Vector(1, 2, 3)
```

---

### Vector:DrG_ToString(decimals)

Converts vector to a readable string.

**Realm:** 🟣 SHARED

**Parameters:**
- `decimals` (number, optional) - Decimal places for rounding

**Returns:**
- `string` - Formatted string "X / Y / Z"

**Example:**
```lua
local vec = Vector(1.23456, 2.34567, 3.45678)

print(vec:DrG_ToString(2)) -- "1.23 / 2.35 / 3.46"
print(vec:DrG_ToString())  -- "1 / 2 / 3"
```

---

### Vector:DrG_Copy()

Creates a deep copy of the vector.

**Realm:** 🟣 SHARED

**Returns:**
- `Vector` - New vector with same values

**Example:**
```lua
local original = Vector(1, 2, 3)
local copy = original:DrG_Copy()

copy.x = 10
print(original) -- Still Vector(1, 2, 3)
print(copy)     -- Vector(10, 2, 3)
```

---

### Vector:DrG_Join(other, ratio)

Linearly interpolates between two vectors.

**Realm:** 🟣 SHARED

**Parameters:**
- `other` (Vector) - Target vector
- `ratio` (number) - Interpolation ratio (0-1)

**Returns:**
- `Vector` - Interpolated vector

**Example:**
```lua
local start = Vector(0, 0, 0)
local finish = Vector(100, 100, 100)

local midpoint = start:DrG_Join(finish, 0.5) -- Vector(50, 50, 50)
local quarter = start:DrG_Join(finish, 0.25) -- Vector(25, 25, 25)
```

**Alias for:** `LerpVector(ratio, self, other)`

---

## Complete Examples

### Leading a Moving Target
```lua
function ENT:FireProjectile()
    local origin = self:GetPos()
    local target = self:GetEnemy()
    local speed = 1500

    -- Automatically leads target
    local vel, info = origin:DrG_CalcLineTrajectory(target, speed)

    local proj = ents.Create("proj_drg_default")
    proj:SetPos(origin)
    proj:SetVelocity(vel)
    proj:Spawn()

    print("Will hit in", info.duration, "seconds")
end
```

### Grenade Arc
```lua
function ENT:ThrowGrenade()
    local origin = self:GetPos()
    local target = self:GetEnemy():GetPos()

    -- Calculate high arc trajectory
    local vel, info = origin:DrG_CalcBallisticTrajectory(target, {
        magnitude = 600,
        highest = true, -- Use high arc
        recursive = true -- Adjust if impossible
    })

    if info.duration > 0 then
        local grenade = ents.Create("npc_grenade_frag")
        grenade:SetPos(origin)
        grenade:SetVelocity(vel)
        grenade:Spawn()

        print("Arc peak:", info.height, "units high")
        print("Flight time:", info.duration, "seconds")
    end
end
```

---

## See Also

- [PhysObj Extensions](./physobj.md) - Physics object trajectory functions
- [Entity Extensions](./entity.md) - Entity-level aim/throw functions
- [Math Module](../modules/math.md) - Additional math utilities
