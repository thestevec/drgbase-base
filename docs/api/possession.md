# Possession System API Reference

The possession system allows players to take direct control of nextbots, controlling their movement, viewing through custom camera positions, and using custom input bindings.

**Source Files:**
- `lua/entities/drgbase_nextbot/possession.lua` - Nextbot possession functions
- `lua/drgbase/possession.lua` - Global possession handlers and hooks

---

## Configuration

### ConVars

#### Server/Shared ConVars
```lua
drgbase_possession_enable -- Enable/disable possession system
drgbase_possession_allow_lockon -- Enable/disable lock-on targeting
```

#### Client ConVars
```lua
drgbase_possession_exit -- Key to exit possession (default: KEY_E)
drgbase_possession_view -- Key to cycle camera views (default: KEY_V)
drgbase_possession_climb -- Key to climb ledges/ladders (default: KEY_C)
drgbase_possession_lockon -- Key to lock onto enemies (default: KEY_L)
drgbase_possession_lockon_speed -- Lock-on camera lerp speed (default: 0.05)
drgbase_possession_teleport -- Teleport to NPC on dispossess instead of returning (default: 0)
```

**Example:**
```lua
-- Disable possession globally
RunConsoleCommand("drgbase_possession_enable", "0")

-- Change exit key to F
RunConsoleCommand("drgbase_possession_exit", tostring(KEY_F))
```

---

## Entity Configuration

### ENT.PossessionEnabled
```lua
ENT.PossessionEnabled = true
```
**Type:** boolean
**Default:** true
**Description:** Whether this entity can be possessed.

---

### ENT.PossessionPrompt
```lua
ENT.PossessionPrompt = true
```
**Type:** boolean
**Default:** true
**Description:** Whether to show the "Possess" option in the context menu.

---

### ENT.PossessionCrosshair
```lua
ENT.PossessionCrosshair = false
```
**Type:** boolean
**Default:** false
**Description:** Whether to show the crosshair during possession.

---

### ENT.PossessionMovement
```lua
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
```
**Type:** number
**Values:**
- `POSSESSION_MOVE_8DIR` - Standard 8-directional movement (WASD combinations)
- `POSSESSION_MOVE_4DIR` - 4-directional movement (only one direction at a time)
- `POSSESSION_MOVE_1DIR` - Free directional movement relative to camera
- `POSSESSION_MOVE_CUSTOM` - Custom movement (implement `PossessionControls` hook)

**Example:**
```lua
ENT.PossessionMovement = POSSESSION_MOVE_1DIR -- Camera-relative movement
```

---

### ENT.PossessionViews
```lua
ENT.PossessionViews = {
    {auto = true}, -- Auto-calculated third-person
    {eyepos = true, distance = 0}, -- First-person
    {offset = Vector(0, 0, 50), distance = 100}, -- Custom view
}
```
**Type:** table
**Description:** Array of camera view presets. Players can cycle through these with the view key.

**View Options:**
- `auto` (boolean) - Auto-calculate third-person view
- `eyepos` (boolean) - Use eye position
- `bone` (string) - Bone name to attach camera to
- `offset` (Vector) - Offset from origin point
- `distance` (number) - Distance from origin
- `center` (Vector) - Custom center point for calculations

**Example:**
```lua
ENT.PossessionViews = {
    -- Auto third-person
    {auto = true},

    -- First-person
    {eyepos = true, distance = 0},

    -- Over-the-shoulder
    {offset = Vector(-30, 50, 0), distance = 100},

    -- Top-down
    {offset = Vector(0, 0, 200), distance = 0},

    -- Bone-based (head camera)
    {bone = "ValveBiped.Bip01_Head1", offset = Vector(10, 0, 0), distance = 0}
}
```

---

### ENT.PossessionBinds
```lua
ENT.PossessionBinds = {
    [KEY_SPACE] = {
        {
            onkeypressed = function(self, ply)
                self:Jump()
            end,
            coroutine = true -- Runs in coroutine context
        }
    },
    [IN_ATTACK] = {
        {
            onkeydown = function(self, ply)
                self:Attack()
            end,
            coroutine = false -- Runs outside coroutine
        }
    }
}
```
**Type:** table
**Description:** Custom key bindings during possession.

**Bind Structure:**
```lua
ENT.PossessionBinds = {
    [key or convar] = {
        {
            -- Key events (for keys like KEY_SPACE)
            onkeypressed = function(self, ply) end, -- Fired once when pressed
            onkeydown = function(self, ply) end,    -- Fired every frame while held
            onkeyup = function(self, ply) end,      -- Fired every frame while not held
            onkeydownlast = function(self, ply) end,-- Fired last frame before release
            onkeyreleased = function(self, ply) end,-- Fired once when released

            -- Button events (for IN_* buttons)
            onbuttonpressed = function(self, ply) end,
            onbuttondown = function(self, ply) end,
            onbuttonup = function(self, ply) end,
            onbuttonreleased = function(self, ply) end,

            coroutine = true,  -- Run in behavior coroutine context
            client = false,    -- Run on client side
        }
    }
}
```

**Example:**
```lua
ENT.PossessionBinds = {
    -- Jump
    [KEY_SPACE] = {
        {
            onkeypressed = function(self, ply)
                if self:IsOnGround() then
                    self:Jump()
                end
            end,
            coroutine = true
        }
    },

    -- Primary attack
    [IN_ATTACK] = {
        {
            onkeydown = function(self, ply)
                self:PrimaryAttack()
            end,
            coroutine = false
        }
    },

    -- Secondary attack
    [IN_ATTACK2] = {
        {
            onbuttonpressed = function(self, ply)
                self:SecondaryAttack()
            end,
            coroutine = false
        }
    },

    -- Custom convar bind
    ["drgbase_possession_special"] = {
        {
            onkeypressed = function(self, ply)
                self:SpecialAbility()
            end,
            coroutine = true
        }
    }
}
```

---

## State Checking

### ENT:IsPossessionEnabled
```lua
boolean ENT:IsPossessionEnabled()
```
Checks if possession is enabled for this entity.

**Realm:** SHARED
**Returns:** (boolean) True if possession is enabled
**Example:**
```lua
if self:IsPossessionEnabled() then
    print("Can be possessed!")
end
```

---

### ENT:IsPossessed
```lua
boolean ENT:IsPossessed()
```
Checks if this entity is currently being possessed.

**Realm:** SHARED
**Returns:** (boolean) True if possessed
**Example:**
```lua
if self:IsPossessed() then
    -- Special behavior while possessed
end
```

---

### ENT:GetPossessor
```lua
Player ENT:GetPossessor()
```
Gets the player possessing this entity.

**Realm:** SHARED
**Returns:** (Player) Possessing player or NULL
**Example:**
```lua
local ply = self:GetPossessor()
if IsValid(ply) then
    print(ply:Nick() .. " is possessing this NPC")
end
```

---

### ENT:IsPossessor
```lua
boolean ENT:IsPossessor(Entity ent)
```
Checks if a specific entity is the possessor.

**Realm:** SHARED
**Parameters:**
- `ent` (Entity) - Entity to check
**Returns:** (boolean) True if ent is the possessor
**Example:**
```lua
if self:IsPossessor(ply) then
    print("This player is possessing the NPC")
end
```

---

### ENT:IsPossessedByLocalPlayer
```lua
boolean ENT:IsPossessedByLocalPlayer()
```
Checks if the local player is possessing this entity.

**Realm:** CLIENT
**Returns:** (boolean) True if local player is possessing
**Example:**
```lua
if CLIENT and self:IsPossessedByLocalPlayer() then
    -- Draw custom UI
end
```

---

## Possession Control (SERVER)

### ENT:Possess
```lua
string ENT:Possess(Player ply)
```
Starts possession of this entity by a player.

**Realm:** SERVER
**Parameters:**
- `ply` (Player) - Player to possess this entity
**Returns:** (string) Status:
- `"ok"` - Success
- `"disabled"` - Possession disabled
- `"already possessed"` - Already being possessed
- `"invalid"` - Invalid player
- `"not player"` - Entity is not a player
- `"not alive"` - Player is dead
- `"in vehicle"` - Player is in a vehicle
- `"already possessing"` - Player is already possessing something
- `"not allowed"` - Blocked by `CanPossess` hook

**Example:**
```lua
local result = npc:Possess(ply)
if result == "ok" then
    print("Possession successful!")
else
    print("Failed to possess: " .. result)
end
```

**Side Effects:**
- Hides player model
- Makes player invisible/invulnerable
- Disables flashlight
- Changes collision group to `COLLISION_GROUP_IN_VEHICLE`
- Stores player's pre-possession position/angles
- Triggers `OnPossessed` hook

---

### ENT:Dispossess
```lua
string ENT:Dispossess()
```
Ends possession of this entity.

**Realm:** SERVER
**Returns:** (string) Status:
- `"ok"` - Success
- `"not possessed"` - Not currently possessed
- `"not allowed"` - Blocked by `CanDispossess` hook

**Example:**
```lua
if self:IsPossessed() then
    self:Dispossess()
end
```

**Side Effects:**
- Restores player visibility
- Re-enables flashlight
- Restores normal collision
- Returns player to original position (unless teleport convar is set)
- Triggers `OnDispossessed` hook

---

### ENT:SetPossessionEnabled
```lua
ENT:SetPossessionEnabled(boolean enabled)
```
Enables or disables possession for this entity.

**Realm:** SERVER
**Parameters:**
- `enabled` (boolean) - Enable/disable possession
**Example:**
```lua
-- Disable possession
self:SetPossessionEnabled(false)

-- Enable possession
self:SetPossessionEnabled(true)
```

**Note:** Automatically dispossesses if disabled while possessed.

---

## Camera Views

### ENT:CurrentViewPreset
```lua
number, table ENT:CurrentViewPreset()
```
Gets the current camera view preset.

**Realm:** SHARED
**Returns:**
- (number) Current view index, or -1 if not possessed
- (table) Current view table

**Example:**
```lua
local index, view = self:CurrentViewPreset()
if index ~= -1 then
    print("Using view preset #" .. index)
    if view.auto then
        print("Auto third-person view")
    end
end
```

---

### ENT:CycleViewPresets
```lua
ENT:CycleViewPresets()
```
Cycles to the next camera view preset.

**Realm:** SHARED (call on either side)
**Example:**
```lua
-- SERVER
self:CycleViewPresets()

-- CLIENT (sends net message)
self:CycleViewPresets()
```

---

### ENT:PossessorView
```lua
Vector, Angle ENT:PossessorView()
```
Calculates the current camera position and angles based on the active view preset.

**Realm:** SHARED
**Returns:**
- (Vector) Camera origin
- (Angle) Camera angles

**Example:**
```lua
local origin, angles = self:PossessorView()
debugoverlay.Cross(origin, 10, 1, color_white, true)
```

---

## Lock-On System

### ENT:PossessionGetLockedOn
```lua
Entity ENT:PossessionGetLockedOn()
```
Gets the currently locked-on target.

**Realm:** SHARED
**Returns:** (Entity) Locked target or NULL
**Example:**
```lua
local target = self:PossessionGetLockedOn()
if IsValid(target) then
    -- Aim assist active
end
```

---

### ENT:PossessionLockOn
```lua
ENT:PossessionLockOn(Entity ent)
```
Sets the lock-on target.

**Realm:** SERVER
**Parameters:**
- `ent` (Entity) - Entity to lock onto, or NULL to clear
**Example:**
```lua
-- Lock onto enemy
local enemy = self:GetEnemy()
if IsValid(enemy) then
    self:PossessionLockOn(enemy)
end

-- Clear lock-on
self:PossessionLockOn(NULL)
```

---

### ENT:PossessionFetchLockOn
```lua
Entity ENT:PossessionFetchLockOn()
```
**Hook** to find a valid lock-on target.

**Realm:** SERVER
**Returns:** (Entity) Target entity or nil
**Example:**
```lua
function ENT:PossessionFetchLockOn()
    -- Find closest visible enemy within 1000 units
    local closest = self:GetClosestHostile()
    if IsValid(closest) and
       self:GetRangeTo(closest) < 1000 and
       self:Visible(closest) then
        return closest
    end
end
```

---

## Directional Vectors

### ENT:PossessorNormal
```lua
Vector ENT:PossessorNormal()
```
Gets the camera's forward vector (where possessor is looking).

**Realm:** SHARED
**Returns:** (Vector) Camera forward direction

---

### ENT:PossessorForward
```lua
Vector ENT:PossessorForward()
```
Gets the forward movement direction (camera forward projected onto ground, or direction to locked target).

**Realm:** SHARED
**Returns:** (Vector) Forward movement direction
**Note:** Returns direction to locked target if lock-on is active.

---

### ENT:PossessorRight
```lua
Vector ENT:PossessorRight()
```
Gets the right movement direction (perpendicular to forward).

**Realm:** SHARED
**Returns:** (Vector) Right movement direction

---

### ENT:PossessorUp
```lua
Vector ENT:PossessorUp()
```
Gets the up direction (always world up).

**Realm:** SHARED
**Returns:** (Vector) Up direction (always Vector(0,0,1))

---

### ENT:PossessorTrace
```lua
table ENT:PossessorTrace([table options])
```
Performs a trace from the camera position forward.

**Realm:** SHARED
**Parameters:**
- `options` (table) - Trace options (optional)
**Returns:** (table) Trace result
**Example:**
```lua
local tr = self:PossessorTrace()
if IsValid(tr.Entity) then
    print("Looking at: " .. tostring(tr.Entity))
end
```

---

## Movement Functions (SERVER)

### ENT:PossessionFaceForward
```lua
ENT:PossessionFaceForward()
```
Faces the entity toward the camera direction (or locked target).

**Realm:** SERVER

---

### ENT:PossessionMoveForward
```lua
ENT:PossessionMoveForward()
```
Moves the entity forward relative to camera/lock-on.

**Realm:** SERVER

---

### ENT:PossessionMoveBackward
```lua
ENT:PossessionMoveBackward()
```
Moves the entity backward relative to camera/lock-on.

**Realm:** SERVER

---

### ENT:PossessionMoveRight
```lua
ENT:PossessionMoveRight()
```
Moves the entity right relative to camera.

**Realm:** SERVER

---

### ENT:PossessionMoveLeft
```lua
ENT:PossessionMoveLeft()
```
Moves the entity left relative to camera.

**Realm:** SERVER

---

## Hooks

### ENT:CanPossess
```lua
boolean ENT:CanPossess(Player ply)
```
**Hook** to determine if a player can possess this entity.

**Realm:** SERVER
**Parameters:**
- `ply` (Player) - Player attempting possession
**Returns:** (boolean) True to allow, false to deny
**Example:**
```lua
function ENT:CanPossess(ply)
    -- Only allow admins
    if not ply:IsAdmin() then
        return false
    end

    -- Don't allow during combat
    if self:IsInCombat() then
        return false
    end

    return true
end
```

---

### ENT:CanDispossess
```lua
boolean ENT:CanDispossess(Player ply)
```
**Hook** to determine if possession can be ended.

**Realm:** SERVER
**Parameters:**
- `ply` (Player) - Possessing player
**Returns:** (boolean) True to allow, false to deny
**Example:**
```lua
function ENT:CanDispossess(ply)
    -- Can't dispossess during special animation
    if self:IsPlayingAnimation() then
        return false
    end
    return true
end
```

---

### ENT:OnPossessed
```lua
ENT:OnPossessed(Player ply)
```
**Hook** called when possession starts.

**Realm:** SHARED
**Parameters:**
- `ply` (Player) - Possessing player
**Example:**
```lua
function ENT:OnPossessed(ply)
    self:EmitSound("npc.possess_start")
    self:SetColor(Color(100, 100, 255))

    if SERVER then
        -- Clear current enemy
        self:SetEnemy(NULL)
    end
end
```

---

### ENT:OnDispossessed
```lua
ENT:OnDispossessed(Player ply)
```
**Hook** called when possession ends.

**Realm:** SHARED
**Parameters:**
- `ply` (Player) - Previously possessing player
**Example:**
```lua
function ENT:OnDispossessed(ply)
    self:EmitSound("npc.possess_end")
    self:SetColor(color_white)
end
```

---

### ENT:OnPossession
```lua
boolean ENT:OnPossession()
```
**Hook** called every frame during possession (before automatic movement handling).

**Realm:** SERVER
**Returns:** (boolean) Return true to prevent default movement handling
**Example:**
```lua
function ENT:OnPossession()
    -- Custom possession logic
    if self.customBehavior then
        self:HandleCustomBehavior()
        return true -- Skip default movement
    end
    return false -- Use default movement
end
```

---

### ENT:PossessionControls
```lua
ENT:PossessionControls(boolean forward, boolean backward, boolean right, boolean left)
```
**Hook** for custom movement control (when `PossessionMovement = POSSESSION_MOVE_CUSTOM`).

**Realm:** SERVER
**Parameters:**
- `forward` (boolean) - Forward key pressed
- `backward` (boolean) - Backward key pressed
- `right` (boolean) - Right key pressed
- `left` (boolean) - Left key pressed
**Example:**
```lua
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM

function ENT:PossessionControls(forward, backward, right, left)
    -- Tank controls
    if forward then
        self:MoveForward()
    end
    if backward then
        self:MoveBackward()
    end
    if right then
        self:TurnRight()
    end
    if left then
        self:TurnLeft()
    end
end
```

---

## Client Hooks

### ENT:PossessionHUD
```lua
boolean ENT:PossessionHUD()
```
**Hook** to draw custom HUD during possession.

**Realm:** CLIENT
**Returns:** (boolean) Return true to prevent default HUD
**Example:**
```lua
function ENT:PossessionHUD()
    local scrW, scrH = ScrW(), ScrH()

    -- Draw health bar
    local health = self:Health()
    local maxHealth = self:GetMaxHealth()
    local healthPercent = health / maxHealth

    surface.SetDrawColor(255, 0, 0)
    surface.DrawRect(50, scrH - 100, 300 * healthPercent, 30)

    -- Draw ammo if applicable
    if self.Weapon_Clip then
        draw.SimpleText(
            "Ammo: " .. self.Weapon_Clip .. " / " .. self.Weapon_ClipMax,
            "DermaDefault",
            scrW - 50, scrH - 50,
            color_white, TEXT_ALIGN_RIGHT
        )
    end

    -- Draw lock-on indicator
    local target = self:PossessionGetLockedOn()
    if IsValid(target) then
        draw.SimpleText(
            "LOCKED: " .. tostring(target),
            "DermaLarge",
            scrW / 2, 50,
            Color(255, 100, 100), TEXT_ALIGN_CENTER
        )
    end

    return true -- Prevent default HUD
end
```

---

### ENT:PossessionRender
```lua
ENT:PossessionRender()
```
**Hook** called during RenderScreenspaceEffects while possessing.

**Realm:** CLIENT
**Example:**
```lua
function ENT:PossessionRender()
    -- Night vision effect
    DrawColorModify({
        ["$pp_colour_addr"] = 0.1,
        ["$pp_colour_addg"] = 0.3,
        ["$pp_colour_addb"] = 0,
        ["$pp_colour_brightness"] = 0.2,
        ["$pp_colour_contrast"] = 1.2,
        ["$pp_colour_colour"] = 0.5,
        ["$pp_colour_mulr"] = 0,
        ["$pp_colour_mulg"] = 0,
        ["$pp_colour_mulb"] = 0
    })
end
```

---

### ENT:PossessionHalos
```lua
ENT:PossessionHalos()
```
**Hook** called during PreDrawHalos while possessing.

**Realm:** CLIENT
**Example:**
```lua
function ENT:PossessionHalos()
    -- Highlight enemies
    local enemies = {}
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 2000)) do
        if self:IsHostile(ent) then
            table.insert(enemies, ent)
        end
    end

    if #enemies > 0 then
        halo.Add(enemies, Color(255, 0, 0), 2, 2, 2, true, true)
    end

    -- Highlight locked target
    local target = self:PossessionGetLockedOn()
    if IsValid(target) then
        halo.Add({target}, Color(255, 255, 0), 5, 5, 3, true, true)
    end
end
```

---

## Player Helper Functions

### Player:DrG_IsPossessing
```lua
boolean Player:DrG_IsPossessing()
```
Checks if the player is possessing a nextbot.

**Realm:** SHARED
**Returns:** (boolean) True if possessing
**Example:**
```lua
if ply:DrG_IsPossessing() then
    print("Player is possessing an NPC")
end
```

---

### Player:DrG_Possessing / Player:DrG_GetPossessing
```lua
Entity Player:DrG_Possessing()
Entity Player:DrG_GetPossessing()
```
Gets the entity the player is possessing.

**Realm:** SHARED
**Returns:** (Entity) Possessed entity or NULL
**Example:**
```lua
local npc = ply:DrG_Possessing()
if IsValid(npc) then
    print("Possessing: " .. npc.PrintName)
end
```

---

## Complete Examples

### Basic Possession Setup
```lua
ENT.PossessionEnabled = true
ENT.PossessionPrompt = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR
ENT.PossessionCrosshair = true

ENT.PossessionViews = {
    {auto = true}, -- Third-person
    {eyepos = true, distance = 0}, -- First-person
}

ENT.PossessionBinds = {
    [KEY_SPACE] = {
        {
            onkeypressed = function(self, ply)
                if self:IsOnGround() then
                    self:Jump()
                end
            end,
            coroutine = true
        }
    },
    [IN_ATTACK] = {
        {
            onkeydown = function(self, ply)
                self:Attack()
            end,
            coroutine = false
        }
    }
}
```

---

### Advanced Camera System
```lua
ENT.PossessionViews = {
    -- Auto third-person
    {auto = true},

    -- First-person
    {eyepos = true, distance = 0},

    -- Over-the-shoulder
    {offset = Vector(-20, 40, 10), distance = 80},

    -- Top-down strategy view
    {offset = Vector(0, 0, 500), distance = 0},

    -- Head-mounted camera
    {bone = "ValveBiped.Bip01_Head1", offset = Vector(5, 0, 0), distance = 0},

    -- Weapon camera
    {bone = "ValveBiped.Anim_Attachment_RH", offset = Vector(10, 0, 0), distance = 20}
}
```

---

### Boss with Lock-On Combat
```lua
ENT.PossessionMovement = POSSESSION_MOVE_1DIR

function ENT:PossessionFetchLockOn()
    -- Find closest visible enemy within 2000 units
    local closest
    local closestDist = 2000

    for _, ent in ipairs(ents.GetAll()) do
        if not self:IsHostile(ent) then continue end
        if not IsValid(ent) then continue end
        if not ent:Health() or ent:Health() <= 0 then continue end

        local dist = self:GetRangeTo(ent)
        if dist > closestDist then continue end
        if not self:Visible(ent) then continue end

        closest = ent
        closestDist = dist
    end

    return closest
end

ENT.PossessionBinds = {
    [IN_ATTACK] = {
        {
            onbuttonpressed = function(self, ply)
                local target = self:PossessionGetLockedOn()
                if IsValid(target) then
                    self:FireProjectileAt(target)
                end
            end,
            coroutine = false
        }
    },
    [KEY_SPACE] = {
        {
            onkeypressed = function(self, ply)
                self:GroundSlam()
            end,
            coroutine = true
        }
    }
}

if CLIENT then
    function ENT:PossessionHUD()
        local target = self:PossessionGetLockedOn()
        if not IsValid(target) then return end

        local screenPos = target:WorldSpaceCenter():ToScreen()
        draw.SimpleText(
            "◄ " .. target:GetClass() .. " ►",
            "DermaLarge",
            screenPos.x, screenPos.y,
            Color(255, 100, 100, 255), TEXT_ALIGN_CENTER
        )

        return false -- Show default HUD too
    end

    function ENT:PossessionHalos()
        local target = self:PossessionGetLockedOn()
        if IsValid(target) then
            halo.Add({target}, Color(255, 0, 0), 5, 5, 2, true, true)
        end
    end
end
```

---

### Racing Game Controls
```lua
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM

function ENT:PossessionControls(forward, backward, right, left)
    local speed = forward and self.MaxSpeed or (backward and -self.MaxSpeed / 2 or 0)
    local turn = right and 1 or (left and -1 or 0)

    -- Apply acceleration
    self.currentSpeed = Lerp(0.1, self.currentSpeed or 0, speed)

    -- Apply steering (more responsive at lower speeds)
    local turnAmount = 2 * (1 - math.abs(self.currentSpeed) / self.MaxSpeed)

    -- Move and turn
    if self.currentSpeed ~= 0 then
        self:MoveForward(math.abs(self.currentSpeed))
        self:Turn(turn * turnAmount)
    end
end

ENT.PossessionBinds = {
    [KEY_SPACE] = {
        {
            onkeydown = function(self, ply)
                -- Handbrake
                self.currentSpeed = self.currentSpeed * 0.9
            end,
            coroutine = true
        }
    },
    [KEY_LSHIFT] = {
        {
            onkeydown = function(self, ply)
                -- Boost
                self.MaxSpeed = 500
            end,
            onkeyup = function(self, ply)
                self.MaxSpeed = 300
            end,
            coroutine = true
        }
    }
}
```

---

## Context Menu Integration

Possession is automatically integrated with the context menu when:
- `ENT.PossessionEnabled = true`
- `ENT.PossessionPrompt = true`
- `drgbase_possession_enable` is enabled

Players can right-click on the entity and select "Possess" to take control.

---

## See Also

- [AI System](../guides/ai.md) - AI behaviors affected by possession
- [Movement System](../guides/movement.md) - Movement functions used during possession
- [Input System](../guides/input.md) - Key binding and input handling
