# Player Possession System

## Overview

DrGBase allows players to take control of NPCs directly, creating unique gameplay experiences. Players can possess NPCs with custom controls, views, and abilities.

## Basic Possession Setup

### Enable Possession

```lua
ENT.PossessionEnabled = true
```

### Basic Configuration

```lua
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR  -- Movement type
ENT.PossessionCrosshair = true                -- Show crosshair
ENT.PossessionPrompt = "Press E to possess"   -- Interaction prompt

ENT.PossessionViews = {
    {
        offset = Vector(0, 30, 20),    -- Camera offset from NPC
        distance = 100                  -- Camera distance
    }
}
```

## Movement Types

```lua
POSSESSION_MOVE_CUSTOM  -- Custom movement (manual)
POSSESSION_MOVE_8DIR    -- 8-directional (W/A/S/D + diagonals)
POSSESSION_MOVE_4DIR    -- 4-directional (W/A/S/D only)
POSSESSION_MOVE_ANALOG  -- Analog (smooth any direction)
POSSESSION_MOVE_CTRL    -- Full control override
```

## Camera Views

### Multiple Views

```lua
ENT.PossessionViews = {
    -- Third person view
    {
        offset = Vector(0, 30, 20),     -- Behind and above
        distance = 100,
        fov = 90
    },
    -- First person view
    {
        offset = Vector(7.5, 0, 0),     -- Eye position offset
        distance = 0,                   -- 0 = first person
        eyepos = true                   -- Use NPC eye position
    },
    -- Top-down view
    {
        offset = Vector(0, 0, 300),     -- Directly above
        distance = 0,
        angles = Angle(90, 0, 0)        -- Look down
    },
    -- Side view
    {
        offset = Vector(0, 100, 0),
        distance = 0,
        fov = 70
    }
}
```

Players cycle through views with their "Use" key (E by default).

### Dynamic Camera

```lua
function ENT:PossessionThink(possessor)
    -- Adjust camera based on velocity
    local speed = self:GetVelocity():Length()

    if speed > 200 then
        -- Pull camera back when moving fast
        self.PossessionViews[1].distance = 150
    else
        self.PossessionViews[1].distance = 100
    end
end
```

## Custom Controls

### Key Bindings

```lua
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            -- Called when attack key pressed
            self:EmitSound("Attack")
            self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.PossessionFaceForward)
        end
    }},

    [IN_ATTACK2] = {{
        coroutine = false,
        onkeydown = function(self)
            -- Secondary attack
            self:SpecialAbility()
        end,
        onkeyup = function(self)
            -- Called when key released
            print("Released attack2")
        end
    }},

    [IN_RELOAD] = {{
        coroutine = true,
        onkeypressed = function(self)
            -- Called repeatedly while held
            self:ChargeAttack()
        end
    }},

    [IN_JUMP] = {{
        coroutine = false,
        onkeypressed = function(self)
            if not self:IsOnGround() then return end
            self:EmitFootstep()
            self:Jump()
        end
    }},

    [IN_USE] = {{
        coroutine = false,
        onkeydown = function(self)
            self:InteractWithObject()
        end
    }},

    [IN_DUCK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:Crouch()
        end,
        onkeyup = function(self)
            self:UnCrouch()
        end
    }}
}
```

### Available Keys

```lua
IN_ATTACK       -- Left mouse / Primary attack
IN_ATTACK2      -- Right mouse / Secondary attack
IN_JUMP         -- Space
IN_DUCK         -- Ctrl
IN_FORWARD      -- W
IN_BACK         -- S
IN_MOVELEFT     -- A
IN_MOVERIGHT    -- D
IN_USE          -- E
IN_RELOAD       -- R
IN_SPEED        -- Shift (sprint)
IN_WALK         -- Alt (walk)
IN_ZOOM         -- Mouse wheel (zoom)
IN_WEAPON1      -- 1 key
IN_WEAPON2      -- 2 key
IN_BULLRUSH     -- (custom)
```

### Coroutine vs. Non-Coroutine

```lua
-- Coroutine (blocks execution, can use self:Wait())
coroutine = true,
onkeydown = function(self)
    self:DoSomething()
    self:Wait(1)  -- Wait works here
    self:DoSomethingElse()
end

-- Non-coroutine (instant, no blocking)
coroutine = false,
onkeydown = function(self)
    self:DoSomething()
    -- Can't use self:Wait() here!
end
```

## Complete Possession Examples

### Headcrab (Leap Attack)

```lua
ENT.PossessionEnabled = true
ENT.PossessionCrosshair = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR

ENT.PossessionViews = {
    {
        offset = Vector(0, 10, 10),
        distance = 50
    },
    {
        offset = Vector(7.5, 0, 0),
        distance = 0,
        eyepos = true
    }
}

ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            local tr = self:PossessorTrace() -- Trace from camera
            self:HeadcrabLeap(tr.HitPos)
        end
    }}
}

function ENT:HeadcrabLeap(pos)
    self:FaceTo(pos)
    self:PlaySequence("jumpattack_broadcast")
    self:PauseCoroutine(0.5)
    self.CanBite = true
    self:EmitSound("NPC_Headcrab.Attack")
    self:Leap(pos, 400)
    self.CanBite = false
end
```

### Zombie (Melee Attack)

```lua
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR

ENT.PossessionViews = {
    {
        offset = Vector(0, 30, 20),
        distance = 100
    },
    {
        offset = Vector(7.5, 0, 0),
        distance = 0,
        eyepos = true
    }
}

ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            self:EmitSound("Zombie.Attack")
            self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.PossessionFaceForward)
        end
    }}
}
```

### Advanced: Sprite with Jump

```lua
ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_8DIR

ENT.PossessionViews = {
    {
        offset = Vector(0, 30, 20),
        distance = 100
    },
    {
        offset = Vector(5, 0, 0),
        distance = 0,
        eyepos = true
    }
}

ENT.PossessionBinds = {
    [IN_JUMP] = {{
        coroutine = false,
        onkeypressed = function(self)
            if not self:IsOnGround() then return end
            self:EmitFootstep()
            self:Jump()
        end
    }}
}
```

## Advanced Possession Features

### Custom Movement

```lua
ENT.PossessionMovement = POSSESSION_MOVE_CUSTOM

function ENT:PossessionThink(possessor)
    local move = Vector(0, 0, 0)

    -- Custom movement logic
    if possessor:KeyDown(IN_FORWARD) then
        move = move + self:GetForward() * 200
    end
    if possessor:KeyDown(IN_BACK) then
        move = move - self:GetForward() * 200
    end
    if possessor:KeyDown(IN_MOVELEFT) then
        move = move - self:GetRight() * 200
    end
    if possessor:KeyDown(IN_MOVERIGHT) then
        move = move + self:GetRight() * 200
    end

    -- Apply movement
    if move:Length() > 0 then
        self:SetVelocity(move * FrameTime())
    end

    -- Custom camera rotation
    if possessor:KeyDown(IN_ATTACK2) then
        self:RotateCamera(possessor)
    end
end
```

### Possession UI

```lua
if CLIENT then
    function ENT:PossessionDraw()
        local possessor = self:GetPossessor()
        if not IsValid(possessor) or possessor != LocalPlayer() then return end

        -- Draw custom HUD
        local scrW, scrH = ScrW(), ScrH()

        -- Health bar
        draw.RoundedBox(4, scrW/2 - 100, scrH - 50, 200, 20, Color(40, 40, 40, 200))
        draw.RoundedBox(4, scrW/2 - 100, scrH - 50, 200 * (self:Health() / self:GetMaxHealth()), 20, Color(255, 0, 0, 200))

        -- Ability cooldowns
        draw.SimpleText("Attack: " .. math.Round(self:GetCooldown("attack"), 1) .. "s",
            "DermaDefault", scrW/2, scrH - 80, Color(255, 255, 255), TEXT_ALIGN_CENTER)

        -- Crosshair
        local tr = possessor:GetEyeTrace()
        local pos = tr.HitPos:ToScreen()
        draw.Circle(pos.x, pos.y, 4, Color(255, 255, 255, 200))

        -- Instructions
        draw.SimpleText("Attack: LMB | Jump: SPACE | Exit: USE",
            "DermaDefault", scrW/2, scrH - 110, Color(255, 255, 255), TEXT_ALIGN_CENTER)
    end
end
```

### Ability System

```lua
ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            if self:GetCooldown("attack") > 0 then
                return -- Still on cooldown
            end

            self:BasicAttack()
            self:SetCooldown("attack", 1.5) -- 1.5 second cooldown
        end
    }},

    [IN_ATTACK2] = {{
        coroutine = true,
        onkeydown = function(self)
            if self:GetCooldown("special") > 0 then
                return
            end

            self:SpecialAbility()
            self:SetCooldown("special", 5) -- 5 second cooldown
        end
    }},

    [IN_RELOAD] = {{
        coroutine = true,
        onkeydown = function(self)
            if self:GetCooldown("ultimate") > 0 then
                return
            end

            self:UltimateAbility()
            self:SetCooldown("ultimate", 30) -- 30 second cooldown
        end
    }}
}
```

### Possession Restrictions

```lua
function ENT:CanBePossessed(player)
    -- Check if player is allowed to possess
    if player:IsAdmin() then
        return true -- Admins can always possess
    end

    if self:Health() < self:GetMaxHealth() * 0.5 then
        return false, "NPC is too damaged to possess"
    end

    if self:IsInCombat() then
        return false, "Cannot possess during combat"
    end

    return true
end

function ENT:OnPossess(player)
    -- Called when possession starts
    print(player:Nick() .. " possessed " .. self:GetClass())
    self:EmitSound("Possess.Start")
end

function ENT:OnDispossess(player)
    -- Called when possession ends
    print(player:Nick() .. " left " .. self:GetClass())
    self:EmitSound("Possess.End")
end
```

### Exit Possession

```lua
-- Player can exit by pressing USE (E) by default

-- Or force exit from code:
self:Dispossess()

-- Check if possessed:
if self:IsPossessed() then
    local player = self:GetPossessor()
    print("Possessed by " .. player:Nick())
end
```

## Utility Functions

### Possession Trace

```lua
-- Get trace from player camera
local tr = self:PossessorTrace()
print(tr.HitPos)
print(tr.Entity)

-- Get camera normal
local normal = self:PossessorNormal()
```

### Face Camera Direction

```lua
-- Face where player is looking
self.PossessionFaceForward = function(self)
    local possessor = self:GetPossessor()
    if not IsValid(possessor) then return end

    local angles = possessor:EyeAngles()
    angles.p = 0
    angles.r = 0
    self:SetAngles(angles)
end
```

## Complete Example: Boss NPC

```lua
ENT.Base = "drgbase_nextbot"
ENT.PrintName = "Possessable Boss"
ENT.SpawnHealth = 500

ENT.PossessionEnabled = true
ENT.PossessionMovement = POSSESSION_MOVE_ANALOG
ENT.PossessionCrosshair = true

ENT.PossessionViews = {
    {offset = Vector(0, 50, 30), distance = 150, fov = 90},
    {offset = Vector(10, 0, 0), distance = 0, eyepos = true, fov = 100}
}

ENT.PossessionBinds = {
    [IN_ATTACK] = {{
        coroutine = true,
        onkeydown = function(self)
            if self:GetCooldown("melee") > 0 then return end

            self:EmitSound("Boss.Melee")
            self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1.5, self.PossessionFaceForward)
            self:SetCooldown("melee", 2)
        end
    }},

    [IN_ATTACK2] = {{
        coroutine = true,
        onkeydown = function(self)
            if self:GetCooldown("ranged") > 0 then return end

            local tr = self:PossessorTrace()
            self:FireProjectile(tr.HitPos)
            self:SetCooldown("ranged", 3)
        end
    }},

    [IN_RELOAD] = {{
        coroutine = true,
        onkeydown = function(self)
            if self:GetCooldown("ultimate") > 0 then return end

            self:UltimateAttack()
            self:SetCooldown("ultimate", 20)
        end
    }},

    [IN_JUMP] = {{
        coroutine = false,
        onkeypressed = function(self)
            if self:IsOnGround() then
                self:Jump()
            end
        end
    }}
}

if SERVER then
    function ENT:FireProjectile(target)
        -- Implementation here
    end

    function ENT:UltimateAttack()
        -- Massive AOE attack
        util.BlastDamage(self, self, self:GetPos(), 500, 100)
    end
end

if CLIENT then
    function ENT:PossessionDraw()
        if self:GetPossessor() != LocalPlayer() then return end

        local scrW, scrH = ScrW(), ScrH()

        -- Draw cooldowns
        draw.SimpleText("Melee: " .. math.max(0, math.Round(self:GetCooldown("melee"), 1)),
            "DermaLarge", 50, scrH - 100, Color(255, 255, 255))
        draw.SimpleText("Ranged: " .. math.max(0, math.Round(self:GetCooldown("ranged"), 1)),
            "DermaLarge", 50, scrH - 70, Color(255, 255, 255))
        draw.SimpleText("Ultimate: " .. math.max(0, math.Round(self:GetCooldown("ultimate"), 1)),
            "DermaLarge", 50, scrH - 40, Color(255, 0, 0))
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Troubleshooting

### Can't possess NPC
- Ensure `PossessionEnabled = true`
- Check if NPC is possessed by someone else
- Verify player is close enough and looking at NPC
- Check console for errors

### Controls don't work
- Verify key bindings in `PossessionBinds`
- Check `coroutine` flag matches function needs
- Use `print()` to debug if function is called

### Camera is weird
- Check `PossessionViews` offset/distance values
- Ensure at least one view is defined
- Try first person view (distance = 0)

### Player stuck in possession
- Press USE (E) to exit
- Call `self:Dispossess()` in console
- Check `OnDispossess` isn't preventing exit

## Source Reference

Possession: `lua/entities/drgbase_nextbot/possession.lua` (420 lines)

## Related Documentation

- [Input System](../reference/input.md)
- [Camera System](../reference/camera.md)
- [Animation System](09-animations.md)
