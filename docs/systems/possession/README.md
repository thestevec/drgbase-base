# Possession System

Player control of NPCs.

**File:** `possession.lua` (420 lines)

## Overview

The possession system allows players to take direct control of NPCs, enabling player input to drive NPC movement, aiming, and actions. This creates a unique gameplay mechanic where players can "possess" NPCs and control them like characters.

**Key Features:**
- **Player Control** - WASD movement, mouse aiming, key bindings
- **View Presets** - Multiple camera angles (first-person, third-person, custom)
- **Movement Modes** - 8-directional, 4-directional, 1-directional, or custom
- **Lock-On System** - Target locking for aiming
- **Climbing Support** - Possess NPCs and climb ladders manually
- **Customizable Controls** - Define custom keybinds for special actions

## How It Works

**Possession Flow:**
1. Player initiates possession (via `DrG_Possess()` player method)
2. NPC's `PossessionEnabled` property must be `true`
3. Player's view is set to the NPC's camera position
4. Player input (`IN_FORWARD`, `IN_BACK`, etc.) drives NPC movement
5. NPC behavior hooks (e.g., `OnPossession()`) handle custom actions
6. Player can cycle camera views with configured keybind
7. Dispossession returns control to player

**Possession Query Functions:**
```lua
-- Check if NPC is possessed
if self:IsPossessed() then end

-- Get possessing player
local ply = self:GetPossessor()

-- Check if specific player is possessing
if self:IsPossessor(ply) then end

-- Check if possession is enabled
if self:IsPossessionEnabled() then end
```

## Configuration

**Enable Possession:**
```lua
ENT.PossessionEnabled = true  -- Allow players to possess this NPC
```

**Movement Modes:**
```lua
-- 8-directional (WASD strafe + forward/back)
ENT.PossessionMovement = POSSESSION_MOVE_8DIR  -- Default

-- 4-directional (WASD, one direction at a time)
ENT.PossessionMovement = POSSESSION_MOVE_4DIR

-- 1-directional (relative to camera)
ENT.PossessionMovement = POSSESSION_MOVE_1DIR

-- Custom (implement your own)
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM
```

**View Presets:**
```lua
ENT.PossessionViews = {
    {
        offset = Vector(-200, 0, 50),  -- Camera offset from origin
        distance = 150,                -- Distance back from offset
    },
    {
        eyepos = true,                 -- Use eye position as origin
        offset = Vector(0, 0, 0),
        distance = 0,                  -- First-person view
    },
    {
        bone = "ValveBiped.Bip01_Head1",  -- Use specific bone
        offset = Vector(-50, 0, 10),
        distance = 100,
    },
    {
        auto = true,  -- Automatic camera (centers on model)
    }
}
```

**Player Cycling Views:**

Players cycle views using the configured keybind (default: V key, `drgbase_possession_view` console variable).

## Usage Examples

**Basic Possession Setup:**
```lua
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionViews = {
    {auto = true},  -- Auto third-person
    {eyepos = true, distance = 0},  -- First-person
}
```

**Custom Controls:**
```lua
function ENT:OnPossession()
    local possessor = self:GetPossessor()

    -- Jump with SPACE
    if possessor:KeyDown(IN_JUMP) then
        self.loco:Jump()
    end

    -- Shoot with LEFT MOUSE
    if possessor:DrG_ButtonDown(MOUSE_LEFT) then
        self:WeaponPrimaryFire()
    end

    -- Don't use default possession movement
    return true
end
```

**Movement Hooks:**

When using `POSSESSION_MOVE_CUSTOM`:
```lua
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM

function ENT:PossessionControls(forward, backward, right, left)
    if forward then
        self:MoveForward()
    elseif backward then
        self:MoveBackward()
    end

    if right then
        self:TurnRight()
    elseif left then
        self:TurnLeft()
    end
end
```

**Possession Hooks:**
```lua
function ENT:OnPossessed(possessor)
    print(tostring(possessor) .. " is now controlling me!")
    self:EmitSound("npc/scanner/scanner_scan2.wav")
end

function ENT:OnDispossessed(possessor)
    print(tostring(possessor) .. " released control")
end
```

**Lock-On System:**
```lua
-- Get locked-on target
local target = self:PossessionGetLockedOn()

if IsValid(target) then
    -- NPC automatically aims at locked target
    print("Locked onto: " .. tostring(target))
end
```

**Camera Functions:**
```lua
-- Get current camera position and angles
local pos, ang = self:PossessorView()

-- Trace from camera
local tr = self:PossessorTrace()
print("Player is looking at: " .. tostring(tr.Entity))

-- Get camera direction vectors
local forward = self:PossessorForward()  -- Camera forward (ignores Z)
local right = self:PossessorRight()      -- Camera right
local normal = self:PossessorNormal()    -- Raw camera forward
```

**Cycle Views Programmatically:**
```lua
-- Change to next view preset
self:CycleViewPresets()

-- Get current view
local index, viewData = self:CurrentViewPreset()
```

**Custom Possession Movement:**
```lua
function ENT:PossessionMoveForward()
    -- Custom forward movement
    self:MoveTowards(self:GetPos() + self:PossessorForward() * 100)
end

function ENT:PossessionMoveBackward()
    -- Custom backward movement
    self:MoveTowards(self:GetPos() - self:PossessorForward() * 100)
end

-- Similar for PossessionMoveLeft, PossessionMoveRight
-- Similar for PossessionFaceForward
```

**Disable Possession Dynamically:**
```lua
-- Disable possession (kicks out player if possessed)
self:SetPossessionEnabled(false)

-- Re-enable
self:SetPossessionEnabled(true)
```

## Player-Side API

**Possessing an NPC (Player):**
```lua
-- Player possesses NPC
ply:DrG_Possess(npc)

-- Check if player is possessing
if ply:DrG_IsPossessing() then end

-- Get possessed NPC
local npc = ply:DrG_GetPossessing()

-- Dispossess
ply:DrG_Dispossess()
```

## Best Practices

**View Configuration:**
- Provide multiple view presets for player preference
- Include at least one third-person and one first-person view
- Test camera collision with walls (camera may clip)
- Use `auto = true` for quick default camera

**Movement Design:**
- `POSSESSION_MOVE_8DIR` - Best for combat/action NPCs
- `POSSESSION_MOVE_4DIR` - Good for grid-based or simple movement
- `POSSESSION_MOVE_1DIR` - Tank controls, good for vehicles
- `POSSESSION_MOVE_CUSTOM` - Full control, implement your own

**Controls:**
- Use `DrG_ButtonDown()` for one-time actions (shoot, jump)
- Use `KeyDown()` for held actions (aiming, crouching)
- Bind climb to `drgbase_possession_climb` ConVar (default: C key)
- Don't block default possession movement unless necessary

**Performance:**
- Possession is fully networked (works in multiplayer)
- Camera position is calculated client-side
- Movement is handled server-side
- Lock-on system updates automatically

**Gameplay:**
- Disable possession for boss NPCs or important encounters
- Use `OnPossessed()` to restrict certain abilities
- Consider balance when allowing weapon use
- Provide feedback when possession starts/ends (sounds, effects)
