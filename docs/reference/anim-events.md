# Animation Events Reference

Guide to using animation events for precise timing in DrGBase NPCs.

## What Are Animation Events?

Animation events allow you to trigger code at specific points during an animation, such as when a punch connects or when a foot hits the ground.

## OnAnimEvent Hook

The `OnAnimEvent` hook is called every frame during animations. Use it to check animation progress and trigger actions.

```lua
function ENT:OnAnimEvent()
    // Called every frame while animation plays
end
```

## Timing Methods

### Using GetCycle()

`GetCycle()` returns animation progress from 0.0 (start) to 1.0 (end).

```lua
function ENT:OnAnimEvent()
    local cycle = self:GetCycle()
    
    if cycle > 0.4 and cycle < 0.6 then
        // Execute during middle 20% of animation
    end
end
```

### Using IsAttacking()

`IsAttacking()` returns true if the NPC is currently performing an attack animation.

```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() then
        // NPC is in attack animation
    end
end
```

## Attack Damage Timing

The most common use of animation events is dealing damage at the right moment during attack animations.

### Melee Attack Example
```lua
function ENT:OnMeleeAttack(enemy)
    self:EmitSound("Zombie.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({
            damage = 15,
            type = DMG_SLASH,
            viewpunch = Angle(10, math.random(-5, 5), 0)
        })
    end
end
```

**Why cycle > 0.4?** The attack connects 40% through the animation, when the arm swings forward.

### Ranged Attack Example
```lua
function ENT:OnRangeAttack(enemy)
    self:PlayActivity(ACT_RANGE_ATTACK1)
    self:PauseCoroutine(1.0)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        // Spawn projectile at 30% through animation
        local proj = ents.Create("proj_drg_plasma")
        proj:SetPos(self:GetPos() + Vector(0, 0, 40))
        proj:Spawn()
        proj:SetOwner(self)
        
        local phys = proj:GetPhysicsObject()
        if IsValid(phys) then
            local dir = (self:GetEnemy():GetPos() - proj:GetPos()):GetNormalized()
            phys:SetVelocity(dir * 800)
        end
    end
end
```

## Multiple Attack Variations

Handle different attack animations with different timings.

```lua
function ENT:OnMeleeAttack(enemy)
    self.AttackType = math.random(1, 3)
    
    if self.AttackType == 1 then
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    elseif self.AttackType == 2 then
        self:PlayActivityAndMove(ACT_MELEE_ATTACK2, 1, self.FaceEnemy)
    else
        self:PlayActivityAndMove(ACT_RANGE_ATTACK1, 1, self.FaceEnemy)
    end
end

function ENT:OnAnimEvent()
    if not self:IsAttacking() then return end
    
    if self.AttackType == 1 and self:GetCycle() > 0.4 then
        // Quick jab - damage at 40%
        self:Attack({damage = 10, type = DMG_CLUB})
    elseif self.AttackType == 2 and self:GetCycle() > 0.6 then
        // Heavy swing - damage at 60%
        self:Attack({damage = 25, type = DMG_SLASH})
    elseif self.AttackType == 3 and self:GetCycle() > 0.3 then
        // Projectile attack at 30%
        self:SpawnProjectile()
    end
end
```

## Preventing Multiple Triggers

Animation events are called every frame, so you need to prevent triggering multiple times.

### Method 1: Using Flag
```lua
function ENT:OnMeleeAttack(enemy)
    self.HasAttacked = false
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if self:IsAttacking() and not self.HasAttacked and self:GetCycle() > 0.4 then
        self:Attack({damage = 15, type = DMG_SLASH})
        self.HasAttacked = true
    end
end
```

### Method 2: Using Cycle Range
```lua
function ENT:OnAnimEvent()
    local cycle = self:GetCycle()
    
    // Only trigger between 0.4 and 0.45 (very narrow window)
    if self:IsAttacking() and cycle > 0.4 and cycle < 0.45 then
        self:Attack({damage = 15, type = DMG_SLASH})
    end
end
```

### Method 3: DrGBase Built-in (Recommended)
```lua
function ENT:OnAnimEvent()
    // DrGBase's Attack() automatically prevents double-triggering
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({damage = 15, type = DMG_SLASH})
    end
end
```

## Sound and Effect Timing

Use animation events for perfectly timed sounds and effects.

```lua
function ENT:OnAnimEvent()
    local cycle = self:GetCycle()
    
    // Footstep at 30%
    if cycle > 0.3 and cycle < 0.35 then
        self:EmitSound("NPC.Footstep")
        
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        util.Effect("dust", effectData)
    end
    
    // Second footstep at 80%
    if cycle > 0.8 and cycle < 0.85 then
        self:EmitSound("NPC.Footstep")
    end
    
    // Attack connects at 50%
    if self:IsAttacking() and cycle > 0.5 and cycle < 0.55 then
        self:Attack({damage = 15, type = DMG_SLASH})
        self:EmitSound("Zombie.AttackHit")
    end
end
```

## Multi-Hit Combos

Create combo attacks with multiple damage events.

```lua
function ENT:OnMeleeAttack(enemy)
    self.ComboHits = {false, false, false}  // Track each hit
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent()
    if not self:IsAttacking() then return end
    
    local cycle = self:GetCycle()
    
    // First hit at 25%
    if cycle > 0.25 and not self.ComboHits[1] then
        self:Attack({damage = 5, type = DMG_CLUB})
        self.ComboHits[1] = true
    end
    
    // Second hit at 50%
    if cycle > 0.50 and not self.ComboHits[2] then
        self:Attack({damage = 5, type = DMG_CLUB})
        self.ComboHits[2] = true
    end
    
    // Final hit at 75%
    if cycle > 0.75 and not self.ComboHits[3] then
        self:Attack({damage = 10, type = DMG_SLASH})
        self.ComboHits[3] = true
    end
end
```

## Determining Correct Timing

How to find the right cycle value for your animations:

### Method 1: Visual Inspection
1. Enable developer mode: `developer 1`
2. Spawn your NPC
3. Add debug print to OnAnimEvent:
```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() then
        print("Cycle:", self:GetCycle())
    end
end
```
4. Watch console and attack animation
5. Note the cycle when attack should connect

### Method 2: Model Viewer
1. Open model in HLMV (Half-Life Model Viewer)
2. Play attack animation
3. Note frame when attack connects
4. Convert frame to cycle: `cycle = frame / totalFrames`

### Method 3: Trial and Error
```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() then
        local cycle = self:GetCycle()
        
        // Draw effect at current cycle to visualize
        debugoverlay.Cross(self:GetPos() + self:GetForward() * 50, 10, 0.1, Color(255, 0, 0), true)
        
        if cycle > 0.5 then
            self:Attack({damage = 15, type = DMG_SLASH})
        end
    end
end
```

## Common Timing Values

Typical cycle values for different attack types:

- **Quick jab**: 0.3 - 0.4
- **Heavy swing**: 0.5 - 0.7
- **Overhead smash**: 0.6 - 0.8
- **Projectile spawn**: 0.2 - 0.4
- **Footsteps**: 0.3 and 0.8
- **Reload**: 0.6 - 0.7

## Animation Finished Detection

Detect when animation completes:

```lua
function ENT:OnAnimEvent()
    if self:GetCycle() >= 0.99 then
        // Animation almost finished
        self:OnAnimationFinished()
    end
end

function ENT:OnAnimationFinished()
    // Animation completed
end
```

## Advanced: Sequence Events

For models with built-in animation events:

```lua
function ENT:OnAnimEvent(event, options)
    // event is the event ID
    // options contains event data
    
    if event == AE_CL_PLAYSOUND then
        // Play custom sound instead
        self:EmitSound("Custom.Sound")
    end
end
```

## Common Issues

### Damage Triggers Multiple Times
```lua
// Problem: Attack() called every frame
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({damage = 15})  // Called many times!
    end
end

// Solution: Narrow the cycle window
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 and self:GetCycle() < 0.45 then
        self:Attack({damage = 15})  // Only called once
    end
end
```

### Attack Happens Too Early/Late
```lua
// Adjust cycle value up or down
if self:GetCycle() > 0.5 then  // Try 0.4, 0.45, 0.55, 0.6 until it looks right
    self:Attack({damage = 15})
end
```

### No Damage Dealt
```lua
// Ensure Attack() is inside IsAttacking() check
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({damage = 15, type = DMG_SLASH})  // REQUIRED
    end
end
```

For complete examples, see [Simple Melee NPC](../examples/simple-melee-npc.md) and [Ranged NPC](../examples/ranged-npc.md).
