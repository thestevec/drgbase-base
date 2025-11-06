# Animation Events

Animation event reference for DrGBase NPCs.

---

## Overview

Animation events are markers embedded in model animations that trigger at specific frames. They're used to synchronize gameplay actions (footsteps, attacks, sounds) with animations.

**Key Points:**
- Events are defined in the model, not in code
- `OnAnimEvent()` hook is called when events fire
- Commonly used for footsteps, attack damage, and sound effects

---

## The OnAnimEvent Hook

### Signature

**Server:**
```lua
function ENT:OnAnimEvent(event, options)
    -- Return true if event handled
    return false
end
```

**Client:**
```lua
function ENT:FireAnimationEvent(pos, angle, event, name)
    self:OnAnimEvent(name, event, pos, angle)
end
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `event` | number | Animation event ID (server) |
| `options` | string | Event options/data (server) |
| `name` | string | Event name (client) |
| `pos` | Vector | Event position (client) |
| `angle` | Angle | Event angle (client) |

### Return Value
- **Server:** Return `true` if event was handled, `false` otherwise
- **Client:** No return value

---

## Common Animation Event Types

### Source Engine Events

These are standard Source Engine animation events:

| Event Constant | Value | Description |
|----------------|-------|-------------|
| `AE_CL_PLAYSOUND` | 1 | Play sound effect |
| `AE_SV_PLAYSOUND` | 1 | Play sound (server) |
| `AE_CL_STOPSOUND` | 2 | Stop sound effect |
| `AE_NPC_WEAPON_FIRE` | 3 | Fire weapon |
| `AE_NPC_WEAPON_DROP` | 4 | Drop weapon |
| `AE_NPC_LEFTFOOT` | 5 | Left footstep |
| `AE_NPC_RIGHTFOOT` | 6 | Right footstep |
| `AE_CL_CREATE_PARTICLE_EFFECT` | 7 | Create particle effect |

**Note:** Event IDs may vary by model. The `options` parameter often contains additional data like sound file paths.

---

## Usage Patterns

### Basic Event Handling

```lua
function ENT:OnAnimEvent(event, options)
    if event == 1 then
        -- Footstep
        self:EmitSound("NPC.Footstep")
        return true
    end
    return false
end
```

### Zombie Example (from npc_drg_zombie.lua)

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        -- Deal damage during attack animation
        self:Attack({
            damage = 10,
            type = DMG_SLASH,
            viewpunch = Angle(20, math.random(-10, 10), 0)
        }, function(self, hit)
            if #hit > 0 then
                self:EmitSound("Zombie.AttackHit")
            else
                self:EmitSound("Zombie.AttackMiss")
            end
        end)
    elseif math.random(2) == 1 then
        -- Random footstep sounds
        self:EmitSound("Zombie.FootstepLeft")
    else
        self:EmitSound("Zombie.FootstepRight")
    end
end
```

### Antlion Example (from npc_drg_antlion.lua)

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() then
        if self:GetCycle() > 0.3 then
            -- Deal damage at right moment
            self:Attack({
                damage = 5,
                range = 50,
                type = DMG_SLASH,
                viewpunch = Angle(10, 0, 0)
            }, function(self, hit)
                if #hit > 0 then
                    self:EmitSound("NPC_Antlion.MeleeAttack")
                end
            end)
        else
            self:EmitSound("NPC_Antlion.MeleeAttackSingle")
        end
    elseif self:IsOnGround() then
        -- Footsteps
        self:EmitSound("NPC_Antlion.Footstep")
    end
end
```

### Human Example (from drgbase_nextbot_human/shared.lua)

```lua
function ENT:OnAnimEvent(event)
    -- Handle model-specific events
    -- The human base uses this for special animations
    return false -- Let base handle it
end
```

---

## Custom Animation Events

### Adding Events Manually

DrGBase allows you to add events to animations at specific frames:

```lua
function ENT:CustomInitialize()
    -- Add event at frame 10 of "attack1" sequence
    self:AddAnimEvent("attack1", 10, "damage")

    -- Add events at multiple frames
    self:AddAnimEvent("walk", {0, 12, 24}, "footstep")
end

function ENT:OnAnimEvent(event, options)
    if event == "damage" then
        self:Attack({damage = 25})
        return true
    elseif event == "footstep" then
        self:EmitSound("NPC.Footstep")
        return true
    end
    return false
end
```

### Sequence Events

Alternative to animation events, trigger callbacks at specific animation cycles:

```lua
function ENT:CustomInitialize()
    local attackSeq = self:LookupSequence("attack1")

    -- Trigger at 30% through animation
    self:SequenceEvent(attackSeq, 0.3, function(self)
        self:Attack({damage = 20})
    end)

    -- Trigger at multiple points
    self:SequenceEvent(attackSeq, {0.1, 0.5, 0.9}, function(self)
        self:EmitSound("NPC.Swing")
    end)
end
```

---

## Common Use Cases

### 1. Footsteps

```lua
function ENT:OnAnimEvent(event, options)
    -- Play footstep on any foot event
    if self:IsMoving() and self:IsOnGround() then
        self:EmitSound("NPC.Footstep")
        -- Optional: Create particle effect
        local effectdata = EffectData()
        effectdata:SetOrigin(self:GetPos())
        util.Effect("ManhackSparks", effectdata)
    end
    return false
end
```

### 2. Melee Attack Timing

```lua
function ENT:OnAnimEvent(event, options)
    if self:IsAttacking() then
        local cycle = self:GetCycle()
        -- Deal damage at peak of swing (30-50% through animation)
        if cycle > 0.3 and cycle < 0.5 then
            self:Attack({
                damage = 15,
                range = 60,
                type = DMG_SLASH
            })
            return true
        end
    end
    return false
end
```

### 3. Weapon Fire

```lua
function ENT:OnAnimEvent(event, options)
    if event == AE_NPC_WEAPON_FIRE then
        if IsValid(self:GetActiveWeapon()) then
            self:GetActiveWeapon():PrimaryAttack()
        end
        return true
    end
    return false
end
```

### 4. Sound Effects

```lua
function ENT:OnAnimEvent(event, options)
    if event == AE_CL_PLAYSOUND then
        -- options contains sound file path
        self:EmitSound(options)
        return true
    end
    return false
end
```

### 5. State-Based Events

```lua
function ENT:OnAnimEvent(event, options)
    local state = self:GetState()

    if state == "attacking" then
        self:HandleAttackEvent(event, options)
    elseif state == "moving" then
        self:HandleMovementEvent(event, options)
    end

    return false
end

function ENT:HandleAttackEvent(event, options)
    if self:GetCycle() > 0.4 then
        self:DealDamageInCone()
    end
end

function ENT:HandleMovementEvent(event, options)
    if math.random() < 0.5 then
        self:EmitSound("NPC.FootstepLeft")
    else
        self:EmitSound("NPC.FootstepRight")
    end
end
```

---

## Animation Cycle-Based Logic

Instead of relying on model events, you can use animation cycle:

### GetCycle()

Returns animation progress (0.0 to 1.0):

```lua
function ENT:OnAnimEvent(event, options)
    local cycle = self:GetCycle()

    if self:IsAttacking() then
        -- Early attack phase
        if cycle < 0.3 then
            -- Wind up
        -- Mid attack phase
        elseif cycle < 0.7 then
            self:Attack({damage = 20})
        -- End attack phase
        else
            -- Recovery
        end
    end

    return false
end
```

### Checking Attack State

```lua
function ENT:OnAnimEvent(event, options)
    if not self:IsAttacking() then return false end

    local cycle = self:GetCycle()

    -- Deal damage only once in middle of animation
    if cycle > 0.4 and cycle < 0.6 and not self._HasDealtDamage then
        self:Attack({damage = 25})
        self._HasDealtDamage = true
    end

    -- Reset flag at end
    if cycle > 0.95 then
        self._HasDealtDamage = false
    end

    return false
end
```

---

## Debugging Animation Events

### Enable Animation Debug

```
drgbase_debug_animations 1
```

### Print All Events

```lua
function ENT:OnAnimEvent(event, options)
    print("Animation Event:", event, options)
    print("Cycle:", self:GetCycle())
    print("Sequence:", self:GetSequenceName(self:GetSequence()))
    return false
end
```

### Visualize Event Timing

```lua
function ENT:OnAnimEvent(event, options)
    if self:IsAttacking() then
        -- Draw debug sphere when event fires
        debugoverlay.Sphere(self:GetPos(), 32, 0.5, Color(255, 0, 0))
    end
    return false
end
```

---

## Best Practices

### 1. Check State Before Acting
Always verify NPC state before handling events:
```lua
function ENT:OnAnimEvent(event, options)
    if not self:IsAttacking() then return false end
    -- Handle attack event
end
```

### 2. Return True When Handled
Prevent event from propagating if you handle it:
```lua
function ENT:OnAnimEvent(event, options)
    if event == AE_CL_PLAYSOUND then
        self:EmitSound(options)
        return true -- Event handled
    end
    return false -- Let other handlers process
end
```

### 3. Use Cycle for Timing
More reliable than counting events:
```lua
function ENT:OnAnimEvent(event, options)
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:DealDamage()
    end
    return false
end
```

### 4. Prevent Duplicate Actions
Use flags to prevent multiple triggers:
```lua
function ENT:OnAnimEvent(event, options)
    if self:IsAttacking() and not self._AttackExecuted then
        if self:GetCycle() > 0.4 then
            self:Attack({damage = 20})
            self._AttackExecuted = true
        end
    end
    return false
end
```

### 5. Clear Flags Properly
Reset state flags when animation ends:
```lua
function ENT:OnAnimationComplete(anim)
    self._AttackExecuted = false
    self._HasDealtDamage = false
end
```

---

## See Also

- [Animation System API](../api/nextbot/animations.md)
- [OnAnimEvent Hook](../api/nextbot/hooks.md#onannimevent)
- [Activity Reference](./activities.md)
- [Example NPCs](../examples/README.md)
