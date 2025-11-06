# PhysObj Metatable Extensions

Extensions to the PhysObj (physics object) metatable for trajectory calculations.

**File:** `lua/drgbase/meta/phys.lua`

---

## Trajectory Functions

### PhysObj:DrG_AimAt(target, speed, feet)

Calculates and applies the velocity needed to aim at a target.

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Target position or entity
- `speed` (number) - Projectile/object speed
- `feet` (boolean, optional) - Aim at target's feet instead of center

**Returns:**
- `Vector` - Applied velocity vector
- `table` - Trajectory info:
  - `duration` (number) - Time to reach target (-1 if impossible)
  - `highest` (number, optional) - Highest point in arc (ballistic only)
  - `ballistic` (boolean) - Whether trajectory uses gravity

**Example:**
```lua
local phys = ent:GetPhysicsObject()
if IsValid(phys) then
    local vel, info = phys:DrG_AimAt(target:GetPos(), 1000)

    if info.duration > 0 then
        print("Will hit in", info.duration, "seconds")
    else
        print("Can't reach target")
    end
end
```

**Notes:**
- Automatically detects if physics object has gravity enabled
- **With gravity:** Uses ballistic trajectory calculation
- **Without gravity:** Uses straight-line trajectory
- Automatically disables drag for accurate trajectory
- Velocity is applied automatically
- Debug visualization available with `drgbase_debug_trajectories` convar

---

### PhysObj:DrG_ThrowAt(target, options, feet)

Calculates and applies a ballistic throw trajectory to reach a target.

**Realm:** 🔴 SERVER

**Parameters:**
- `target` (Vector or Entity) - Target position or entity
- `options` (table) - Throw options:
  - `magnitude` (number) - Initial velocity magnitude
  - `recursive` (boolean, optional) - Use recursive calculation for accuracy
  - Other trajectory options
- `feet` (boolean, optional) - Aim at target's feet

**Returns:**
- `Vector` - Applied velocity vector
- `table` - Trajectory info (same as `DrG_AimAt`)

**Example:**
```lua
local phys = ent:GetPhysicsObject()
if IsValid(phys) then
    -- Throw with 800 units/sec initial velocity
    local vel, info = phys:DrG_ThrowAt(target, {
        magnitude = 800,
        recursive = true
    })

    print("Arc height:", info.highest)
    print("Flight time:", info.duration)
end
```

**Notes:**
- Always uses ballistic (gravity-affected) trajectory
- Automatically disables drag
- Velocity is applied automatically
- `recursive` option improves accuracy for long-distance throws
- Perfect for grenades, throwable objects, and projectile weapons

---

## Trajectory Calculation

Both functions use advanced trajectory calculation:

1. **Ballistic Trajectory** (with gravity):
   - Calculates parabolic arc to target
   - Accounts for gravity (default: 600 units/sec²)
   - Returns highest point in arc
   - May be impossible for distant targets

2. **Line Trajectory** (no gravity):
   - Simple straight-line path
   - Always possible if speed > 0

---

## Debug Visualization

Set the `drgbase_debug_trajectories` convar to visualize trajectories:

```lua
-- Enable trajectory debug overlay for 5 seconds
GetConVar("drgbase_debug_trajectories"):SetFloat(5)
```

**Color coding:**
- 🟢 **Green:** Before launch
- ⚪ **White:** Normal flight path
- 🔴 **Red:** After impact/end

---

## ConVars

### drgbase_debug_trajectories

Duration to show trajectory debug overlays.

- **Default:** `0` (disabled)
- **Type:** Float (seconds)
- **Flags:** NONE

---

## Examples

### Throwing a Grenade
```lua
function SWEP:PrimaryAttack()
    local grenade = ents.Create("npc_grenade_frag")
    grenade:SetPos(self:GetOwner():GetShootPos())
    grenade:Spawn()

    local phys = grenade:GetPhysicsObject()
    if IsValid(phys) then
        local target = self:GetOwner():GetEyeTrace().HitPos
        phys:DrG_ThrowAt(target, {magnitude = 600})
    end
end
```

### Projectile Aiming
```lua
function ENT:FireProjectile()
    local proj = ents.Create("proj_drg_default")
    proj:SetPos(self:GetPos())
    proj:Spawn()

    local phys = proj:GetPhysicsObject()
    if IsValid(phys) then
        -- Aim at enemy
        local vel, info = phys:DrG_AimAt(self:GetEnemy(), 1500)

        if info.duration < 0 then
            -- Can't reach, fire straight at them anyway
            local dir = (self:GetEnemy():GetPos() - self:GetPos()):GetNormalized()
            phys:SetVelocity(dir * 1500)
        end
    end
end
```

---

## See Also

- [Vector Extensions](./vector.md) - Vector trajectory calculation functions
- [Entity Extensions](./entity.md) - Entity-level aim/throw functions
- [Projectile API](../projectile/functions.md) - Projectile system
