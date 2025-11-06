# Implementing Possession

A comprehensive guide to implementing and using the possession system in DrGBase NPCs.

## Table of Contents

- [Introduction](#introduction)
- [Enabling Possession](#enabling-possession)
- [Player Controls](#player-controls)
- [Camera Views](#camera-views)
- [Lock-On System](#lock-on-system)
- [Key Bindings](#key-bindings)
- [Possession Hooks](#possession-hooks)
- [Custom Movement](#custom-movement)
- [Best Practices](#best-practices)
- [Complete Examples](#complete-examples)

## Introduction

The possession system allows players to take direct control of NPCs, controlling their movement, actions, and abilities. When possessing an NPC, the player:

- Controls the NPC's movement using WASD keys
- Can trigger NPC abilities via custom key bindings
- Views the world through configurable camera positions
- Can cycle between multiple camera views
- Can lock onto targets for easier combat

The possession system is fully integrated with DrGBase's behavior system, allowing NPCs to seamlessly transition between AI control and player control.

**Source:** `/home/user/drgbase-base/lua/entities/drgbase_nextbot/possession.lua`

## Enabling Possession

### Basic Configuration

To enable possession for an NPC, set these properties in your entity file:

```lua
ENT.PossessionEnabled = true           -- Allow this NPC to be possessed
ENT.PossessionPrompt = true            -- Show "Press E to possess" prompt
ENT.PossessionCrosshair = false        -- Show custom crosshair when possessed
ENT.PossessionMovement = POSSESSION_MOVE_8DIR  -- Movement type
```

### Global Settings

Possession can be controlled globally via console variables:

```lua
drgbase_possession_enable 1            -- Enable/disable possession system
drgbase_possession_allow_lockon 1      -- Enable/disable lock-on targeting
drgbase_possession_targetall 1         -- Allow possessing all entities
```

**Important:** Even if `PossessionEnabled = true`, possession won't work if `drgbase_possession_enable` is set to 0.

### How to Possess

Players can possess NPCs in two ways:

1. **Using the Possessor Tool:**
   - Equip the "Possessor" tool from the toolgun menu
   - Aim at an NPC with `PossessionEnabled = true`
   - Left-click to possess

2. **Using Context Menu:**
   - Aim at an NPC
   - Hold C (context menu)
   - Click "Possess" if available

## Player Controls

### Default Controls

When possessing an NPC, these controls are available:

| Key | Action | ConVar |
|-----|--------|--------|
| **WASD** | Movement | Built-in |
| **Mouse** | Camera rotation | Built-in |
| **E** | Exit possession | `drgbase_possession_exit` |
| **V** | Cycle camera views | `drgbase_possession_view` |
| **C** | Climb ladders/ledges | `drgbase_possession_climb` |
| **L** | Toggle lock-on | `drgbase_possession_lockon` |
| **Space** | Jump (if NPC supports it) | Built-in |

### Rebinding Controls

Players can rebind possession controls via console:

```lua
bind "e" "+drgbase_possession_exit"
bind "v" "+drgbase_possession_view"
bind "c" "+drgbase_possession_climb"
bind "l" "+drgbase_possession_lockon"
```

Or through the spawn menu under "DrGBase" > "Options".

## Camera Views

### View Configuration

Define camera views in the `PossessionViews` table:

```lua
ENT.PossessionViews = {
    -- View 1: Third person
    {
        offset = Vector(0, 30, 20),    -- X, Y, Z offset from center
        distance = 100,                 -- Distance behind NPC
    },
    -- View 2: First person
    {
        offset = Vector(0, 0, 0),      -- No offset
        distance = 0,                   -- No distance
        eyepos = true,                  -- Use eye position
    },
    -- View 3: Custom bone position
    {
        bone = "ValveBiped.Bip01_Head1",  -- Bone name
        offset = Vector(0, 20, 10),
        distance = 50,
    }
}
```

### View Properties

Each view can have these properties:

- **`offset`** (Vector): Offset from the reference point
  - X = Forward/backward
  - Y = Left/right
  - Z = Up/down
  - Scaled by NPC model scale

- **`distance`** (number): Distance the camera pulls back from the offset point

- **`eyepos`** (boolean): Use NPC's eye position as reference point instead of center

- **`bone`** (string): Use a specific bone position as reference point

- **`auto`** (boolean): Automatically calculate view based on NPC size

### Auto View

If no views are defined or a view has `auto = true`, the system automatically calculates a view:

```lua
-- Automatic calculation
origin = NPC:WorldSpaceCenter() + Vector(0, 0, NPC:Height() / 3)
distance = NPC:Length() * 3
```

### View Cycling

Players can cycle through views by pressing **V** (default). The current view index wraps around:

```lua
-- Programmatically cycle views
npc:CycleViewPresets()

-- Get current view
local index, view = npc:CurrentViewPreset()
```

## Lock-On System

The lock-on system helps players maintain focus on targets during combat.

### Enabling Lock-On

Lock-on is enabled by default if `drgbase_possession_allow_lockon` is 1.

### Using Lock-On

1. Press **L** (default) while possessing
2. The system automatically targets the closest hostile entity
3. Camera smoothly tracks the target
4. NPC faces the locked target when moving
5. Press **L** again to release lock-on

### Lock-On Speed

Control how quickly the camera tracks targets:

```lua
drgbase_possession_lockon_speed 0.05   -- Range: 0.01 to 1.0
```

Lower values = slower, smoother tracking
Higher values = faster, more responsive tracking

### Custom Lock-On Logic

Override the lock-on target selection:

```lua
function ENT:PossessionFetchLockOn()
    -- Return the entity to lock onto
    local closest = self:GetClosestHostile()
    if not IsValid(closest) then return nil end
    if self:Visible(closest) then
        return closest
    end
    return nil
end
```

### Programmatic Lock-On Control

```lua
-- SERVER only
npc:PossessionLockOn(target_entity)    -- Lock onto specific entity
npc:PossessionLockOn(NULL)             -- Clear lock-on

-- Get current target
local target = npc:PossessionGetLockedOn()
if IsValid(target) then
    print("Locked onto:", target)
end
```

## Key Bindings

### Binding Structure

Add custom key bindings via the `PossessionBinds` table:

```lua
ENT.PossessionBinds = {
    [KEY_OR_BUTTON] = {{
        -- Execution context
        coroutine = false,      -- Run in coroutine (for movement/animation)
        client = false,         -- Run on client side

        -- Key event callbacks
        onkeydown = function(self, ply) end,      -- Held down
        onkeyup = function(self, ply) end,        -- Not held down
        onkeypressed = function(self, ply) end,   -- Just pressed
        onkeyreleased = function(self, ply) end,  -- Just released
        onkeydownlast = function(self, ply) end,  -- Was down last tick

        -- Button event callbacks (DrGBase button system)
        onbuttondown = function(self, ply) end,
        onbuttonup = function(self, ply) end,
        onbuttonpressed = function(self, ply) end,
        onbuttonreleased = function(self, ply) end,
    }}
}
```

### Available Keys

Use Garry's Mod input constants:

```lua
-- Mouse buttons
IN_ATTACK       -- Left mouse button
IN_ATTACK2      -- Right mouse button
IN_ATTACK3      -- Middle mouse (rarely used)

-- Action keys
IN_RELOAD       -- R key
IN_USE          -- E key
IN_JUMP         -- Space bar
IN_DUCK         -- Ctrl key
IN_SPEED        -- Shift key
IN_WALK         -- Alt key

-- Keyboard keys
KEY_A, KEY_B, KEY_C, ... KEY_Z
KEY_0, KEY_1, ... KEY_9
KEY_F1, KEY_F2, ... KEY_F12

-- Special keys
KEY_SPACE
KEY_LSHIFT, KEY_RSHIFT
KEY_LCONTROL, KEY_RCONTROL
KEY_LALT, KEY_RALT
```

### Coroutine vs Non-Coroutine

**Coroutine (`coroutine = true`):**
- Runs in the NPC's behavior coroutine
- Can use movement functions: `PlayActivityAndMove`, `MoveTowards`, etc.
- Blocks until movement/animation completes
- Best for attacks, complex actions

**Non-Coroutine (`coroutine = false` or omitted):**
- Runs every tick while key is held
- Cannot use movement functions
- Best for continuous effects, sounds, instant actions

### Custom Key Configuration

Allow players to configure keys via ConVars:

```lua
ENT.PossessionBinds = {
    ["drgbase_mymod_firekey"] = {{
        coroutine = true,
        onkeydown = function(self, ply)
            self:FireWeapon()
        end
    }}
}
```

Create the ConVar:

```lua
if CLIENT then
    CreateClientConVar("drgbase_mymod_firekey", tostring(IN_ATTACK), true, true)
end
```

## Possession Hooks

### Core Hooks

Override these hooks to customize possession behavior:

```lua
-- SERVER: Called when checking if player can possess
function ENT:CanPossess(ply)
    -- Return false to prevent possession
    if ply:IsAdmin() then
        return true
    end
    return false
end

-- SERVER: Called when checking if player can exit possession
function ENT:CanDispossess(ply)
    -- Return false to prevent exiting
    if self:Health() < 50 then
        return false  -- Trapped until health restored!
    end
    return true
end

-- SERVER: Called immediately after being possessed
function ENT:OnPossessed(ply)
    print(ply:Nick(), "possessed", self:GetClass())
    self:EmitSound("npc/scanner/scanner_talk1.wav")
    self:SetHealth(self:GetMaxHealth())  -- Heal on possess
end

-- SERVER: Called immediately after being dispossessed
function ENT:OnDispossessed(ply)
    print(ply:Nick(), "stopped possessing", self:GetClass())
    self:EmitSound("npc/scanner/scanner_pain1.wav")
end

-- CLIENT: Called when local player starts possessing this NPC
function ENT:OnPossessed(ply)
    chat.AddText("You are now controlling ", self:GetClass())
end

-- CLIENT: Called when local player stops possessing this NPC
function ENT:OnDispossessed(ply)
    chat.AddText("You stopped controlling ", self:GetClass())
end
```

### Advanced Hooks

```lua
-- SERVER: Called every tick during possession (in coroutine)
-- Return true to override default movement
function ENT:OnPossession()
    if self:WaterLevel() > 2 then
        -- Custom swimming behavior
        self:SetVelocity(Vector(0, 0, 10))
        return true  -- Skip default movement
    end
    return false  -- Use default movement
end

-- CLIENT: Custom HUD rendering
function ENT:PossessionHUD()
    -- Draw custom HUD
    draw.SimpleText("Health: " .. self:Health(), "DermaDefault",
        ScrW()/2, ScrH() - 50, Color(255,255,255), TEXT_ALIGN_CENTER)

    -- Return true to override default HUD
    return false
end

-- CLIENT: Screen effects
function ENT:PossessionRender()
    -- Apply screen effects
    DrawColorModify({
        ["$pp_colour_addr"] = 0.02,
        ["$pp_colour_addg"] = 0,
        ["$pp_colour_addb"] = 0,
        ["$pp_colour_brightness"] = 0,
        ["$pp_colour_contrast"] = 1,
        ["$pp_colour_colour"] = 1,
        ["$pp_colour_mulr"] = 0,
        ["$pp_colour_mulg"] = 0,
        ["$pp_colour_mulb"] = 0
    })
end

-- CLIENT: Draw halos around entities
function ENT:PossessionHalos()
    local enemies = {}
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
        if ent:IsNPC() and self:Disposition(ent) == D_HT then
            table.insert(enemies, ent)
        end
    end

    halo.Add(enemies, Color(255, 0, 0), 2, 2, 2, true, true)
end
```

## Custom Movement

### Movement Modes

Set the movement mode via `PossessionMovement`:

```lua
POSSESSION_MOVE_8DIR     -- 8-directional (default for humanoids)
POSSESSION_MOVE_NSEW     -- Alias for 8DIR
POSSESSION_MOVE_COMPASS  -- Alias for 8DIR

POSSESSION_MOVE_1DIR     -- Forward only
POSSESSION_MOVE_FORWARD  -- Alias for 1DIR

POSSESSION_MOVE_4DIR     -- 4-directional (no diagonals)

POSSESSION_MOVE_CUSTOM   -- Custom movement logic
```

### 8-Directional Movement (POSSESSION_MOVE_8DIR)

Standard movement with 8 directions including diagonals:

```lua
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
```

- **W** = Forward
- **S** = Backward
- **A** = Left
- **D** = Right
- **W+A** = Forward-left diagonal
- **W+D** = Forward-right diagonal
- Etc.

NPC automatically faces the camera direction and moves relative to it.

### Forward-Only Movement (POSSESSION_MOVE_1DIR)

Camera-relative movement in any direction:

```lua
ENT.PossessionMovement = POSSESSION_MOVE_1DIR
```

- NPC moves toward combined WASD input direction
- W = forward (camera direction)
- S = backward
- A = left
- D = right
- Smooth omnidirectional movement

Best for: Flying creatures, smooth movement

### 4-Directional Movement (POSSESSION_MOVE_4DIR)

Restricts movement to 4 cardinal directions (no diagonals):

```lua
ENT.PossessionMovement = POSSESSION_MOVE_4DIR
```

- Only one direction at a time
- W, A, S, or D - no combinations
- Last pressed key takes priority

Best for: Retro-style movement, grid-based games

### Custom Movement (POSSESSION_MOVE_CUSTOM)

Implement your own movement logic:

```lua
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM

function ENT:PossessionControls(forward, backward, right, left)
    -- forward, backward, right, left are booleans

    -- Example: Tank controls
    if forward then
        self:SetVelocity(self:GetForward() * 200)
    end

    if left then
        self:SetAngles(self:GetAngles() + Angle(0, -2, 0))
    elseif right then
        self:SetAngles(self:GetAngles() + Angle(0, 2, 0))
    end
end
```

### Movement Utilities

The possession system provides direction helpers:

```lua
-- SERVER only, available during possession

-- Get camera forward direction (accounts for lock-on)
local forward = self:PossessorForward()  -- Vector, Z = 0

-- Get camera right direction
local right = self:PossessorRight()      -- Vector, Z = 0

-- Get up direction
local up = self:PossessorUp()            -- Always Vector(0, 0, 1)

-- Get raw camera direction (no lock-on)
local normal = self:PossessorNormal()    -- Vector from camera angles

-- Face where possessor is looking (respects lock-on)
self:PossessionFaceForward()

-- Move in direction
self:PossessionMoveForward()   -- Move forward
self:PossessionMoveBackward()  -- Move backward
self:PossessionMoveRight()     -- Move right
self:PossessionMoveLeft()      -- Move left

-- Trace from camera
local tr = self:PossessorTrace({
    filter = self,
    mask = MASK_SHOT
})
print("Camera looking at:", tr.HitPos)
```

## Best Practices

### 1. Always Define Views

Don't rely on auto-calculated views. Define at least one view:

```lua
-- Bad
ENT.PossessionViews = {}

-- Good
ENT.PossessionViews = {
    { offset = Vector(0, 30, 20), distance = 100 }
}
```

### 2. Use Coroutines for Actions

Any action that plays an animation or moves the NPC should use `coroutine = true`:

```lua
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,  -- Required!
        onkeydown = function(self)
            self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1)
        end
    }}
}
```

### 3. Provide Multiple Camera Views

Give players options:

```lua
ENT.PossessionViews = {
    { offset = Vector(0, 30, 20), distance = 100 },  -- Third person
    { offset = Vector(0, 0, 0), distance = 0, eyepos = true },  -- First person
    { offset = Vector(0, 50, 30), distance = 150 },  -- Far third person
}
```

### 4. Handle Edge Cases

```lua
function ENT:CanPossess(ply)
    -- Don't allow possession during certain states
    if self:IsAttacking() then return false end
    if self:Health() <= 0 then return false end
    return true
end

function ENT:OnPossessed(ply)
    -- Clean up state
    self:ClearEnemyMemory()
    self:SetEnemy(NULL)
end
```

### 5. Respect Movement Modes

Choose the right movement mode for your NPC:

- **Humanoid NPCs:** `POSSESSION_MOVE_8DIR`
- **Flying/Swimming:** `POSSESSION_MOVE_1DIR`
- **Grid-based:** `POSSESSION_MOVE_4DIR`
- **Vehicles/Special:** `POSSESSION_MOVE_CUSTOM`

### 6. Balance Abilities

Don't make possessed NPCs overpowered:

```lua
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeypressed = function(self)  -- Only on press, not hold
            self:Attack()
        end
    }}
}
```

### 7. Provide Feedback

Use sounds and effects:

```lua
function ENT:OnPossessed(ply)
    self:EmitSound("npc/scanner/scanner_talk1.wav")

    -- Visual effect
    local effectdata = EffectData()
    effectdata:SetOrigin(self:GetPos())
    util.Effect("ManhackSparks", effectdata)
end
```

## Complete Examples

### Example 1: Possessable Zombie

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Zombie"

-- Enable possession
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR

-- Camera views
ENT.PossessionViews = {
    -- Third person
    {
        offset = Vector(0, 30, 20),
        distance = 100
    },
    -- First person
    {
        offset = Vector(7.5, 0, 0),
        distance = 0,
        eyepos = true
    }
}

-- Key bindings
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:EmitSound("Zombie.Attack")
            self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.PossessionFaceForward)
        end
    }},
    [IN_RELOAD] = {{
        onkeypressed = function(self)
            self:EmitSound("Zombie.Idle")
        end
    }}
}

if SERVER then
    function ENT:OnPossessed(ply)
        self:EmitSound("Zombie.Alert")
        -- Heal on possess
        self:SetHealth(self:GetMaxHealth())
    end
end
```

### Example 2: Flying Drone

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Drone"

-- Flight setup
ENT.CanFly = true
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_1DIR  -- Smooth omnidirectional

-- Distant camera for better flight view
ENT.PossessionViews = {
    {
        offset = Vector(0, 0, 30),
        distance = 200
    }
}

ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        onkeydown = function(self, ply)
            -- Continuous laser
            local tr = self:PossessorTrace()
            local effectdata = EffectData()
            effectdata:SetOrigin(self:EyePos())
            effectdata:SetStart(tr.HitPos)
            util.Effect("ToolTracer", effectdata)

            if IsValid(tr.Entity) then
                tr.Entity:TakeDamage(1, self, self)
            end
        end
    }},
    [IN_JUMP] = {{
        onkeydown = function(self)
            -- Ascend
            self:SetVelocity(Vector(0, 0, 50))
        end
    }},
    [IN_DUCK] = {{
        onkeydown = function(self)
            -- Descend
            self:SetVelocity(Vector(0, 0, -50))
        end
    }}
}

if SERVER then
    function ENT:OnPossession()
        -- Custom flight movement
        local ply = self:GetPossessor()

        -- Maintain altitude
        if not ply:KeyDown(IN_JUMP) and not ply:KeyDown(IN_DUCK) then
            local targetZ = self:GetPos().z
            local currentZ = self:GetPos().z
            if math.abs(targetZ - currentZ) > 5 then
                self:SetVelocity(Vector(0, 0, (targetZ - currentZ) * 0.1))
            end
        end

        return false  -- Allow default horizontal movement
    end
end
```

### Example 3: Stealth Spy

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Spy"

ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR

-- Close camera for stealth
ENT.PossessionViews = {
    {
        offset = Vector(0, 20, 15),
        distance = 60
    }
}

ENT.PossessionBinds = {
    [IN_ATTACK2] = {{
        client = false,
        coroutine = false,
        onkeydown = function(self, ply)
            -- Hold to go invisible
            self:SetRenderMode(RENDERMODE_TRANSALPHA)
            self:SetColor(Color(255, 255, 255, 50))
            self:SetMaterial("models/effects/vol_light001")
        end,
        onkeyup = function(self, ply)
            -- Release to become visible
            self:SetRenderMode(RENDERMODE_NORMAL)
            self:SetColor(Color(255, 255, 255, 255))
            self:SetMaterial("")
        end
    }},
    [IN_ATTACK] = {{
        coroutine = true,
        onkeypressed = function(self)
            -- Backstab
            local tr = self:PossessorTrace({filter = self})
            if IsValid(tr.Entity) and tr.Entity:Health() > 0 then
                local dmg = DamageInfo()
                dmg:SetDamage(999)
                dmg:SetAttacker(self:GetPossessor())
                dmg:SetInflictor(self)
                dmg:SetDamageType(DMG_SLASH)
                tr.Entity:TakeDamageInfo(dmg)

                self:EmitSound("weapons/knife/knife_stab.wav")
            end
        end
    }}
}

if CLIENT then
    function ENT:PossessionHUD()
        -- Draw crosshair
        surface.SetDrawColor(255, 0, 0, 200)
        local x, y = ScrW()/2, ScrH()/2
        surface.DrawLine(x - 10, y, x + 10, y)
        surface.DrawLine(x, y - 10, x, y + 10)

        -- Draw stealth meter
        local alpha = self:GetColor().a
        local stealthPercent = (255 - alpha) / 255
        draw.SimpleText("Stealth: " .. math.Round(stealthPercent * 100) .. "%",
            "DermaLarge", x, y + 30, Color(255,255,255), TEXT_ALIGN_CENTER)
    end

    function ENT:PossessionHalos()
        -- Highlight enemies through walls
        local enemies = {}
        for _, ent in ipairs(ents.GetAll()) do
            if ent:IsNPC() and ent ~= self then
                table.insert(enemies, ent)
            end
        end
        halo.Add(enemies, Color(255, 50, 50), 2, 2, 2, true, true)
    end
end
```

### Example 4: Tank with Custom Controls

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Tank"

ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM  -- Tank controls

ENT.PossessionViews = {
    {
        offset = Vector(-50, 0, 40),
        distance = 150
    }
}

ENT.TankSpeed = 100
ENT.TankTurnSpeed = 2

ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeypressed = function(self)
            -- Fire cannon
            self:EmitSound("weapons/airboat/airboat_gun_energy" .. math.random(1,2) .. ".wav")

            local forward = self:GetForward()
            local shell = ents.Create("prop_physics")
            shell:SetModel("models/props_phx/misc/smallcannonball.mdl")
            shell:SetPos(self:EyePos() + forward * 50)
            shell:Spawn()
            shell:GetPhysicsObject():SetVelocity(forward * 2000)

            -- Recoil
            timer.Simple(0.5, function()
                if IsValid(self) then
                    self:SetVelocity(-forward * 200)
                end
            end)
        end
    }}
}

if SERVER then
    function ENT:PossessionControls(forward, backward, right, left)
        local ply = self:GetPossessor()

        -- Forward/backward movement
        if forward then
            self:SetVelocity(self:GetForward() * self.TankSpeed)
        elseif backward then
            self:SetVelocity(self:GetForward() * -self.TankSpeed * 0.5)
        end

        -- Turning
        if left then
            local ang = self:GetAngles()
            ang:RotateAroundAxis(Vector(0, 0, 1), self.TankTurnSpeed)
            self:SetAngles(ang)
        elseif right then
            local ang = self:GetAngles()
            ang:RotateAroundAxis(Vector(0, 0, 1), -self.TankTurnSpeed)
            self:SetAngles(ang)
        end
    end
end
```

## Troubleshooting

### Possession Not Working

1. Check global ConVar: `drgbase_possession_enable 1`
2. Verify `ENT.PossessionEnabled = true`
3. Ensure at least one view is defined in `PossessionViews`
4. Check server console for errors

### Camera Issues

1. Define explicit views rather than relying on auto-calculation
2. Adjust `distance` and `offset` values
3. Test with multiple camera views
4. Check for bone name typos in bone-based views

### Movement Not Responding

1. Verify `PossessionMovement` is set to valid constant
2. For custom movement, check `PossessionControls` implementation
3. Ensure NPC has pathfinding enabled
4. Check for coroutine blocking (don't use `Wait()` in `OnPossession`)

### Key Bindings Not Working

1. Use `coroutine = true` for movement/animation actions
2. Check for key constant typos
3. Verify callbacks are defined correctly
4. Test with simple `print()` statements first

## See Also

- [Possession API Reference](/home/user/drgbase-base/docs/api/nextbot/possession.md)
- [Base Configuration](/home/user/drgbase-base/docs/api/base-configuration.md#possession-properties)
- [Enumerations](/home/user/drgbase-base/docs/api/enumerations.md#possession-movement-modes)
- [Advanced Features](/home/user/drgbase-base/docs/getting-started/05-advanced-features.md#possession-system)
