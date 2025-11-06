# Common Pitfalls

Avoid these common mistakes when developing DrGBase NPCs.

## Initialization Issues

### Forgetting AddCSLuaFile()
```lua
-- Wrong: No AddCSLuaFile()
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"
-- ... code ...
DrGBase.AddNextbot(ENT)

-- Correct: Always include AddCSLuaFile()
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"
-- ... code ...
AddCSLuaFile()  // Must be before DrGBase.AddNextbot()
DrGBase.AddNextbot(ENT)
```

### Wrong Base Entity
```lua
-- Wrong: Using wrong base
ENT.Base = "base_nextbot"  // Wrong!

// Correct: Use DrGBase base
ENT.Base = "drgbase_nextbot"  // For regular NPCs
ENT.Base = "drgbase_nextbot_human"  // For weapon-using NPCs
ENT.Base = "drgbase_nextbot_sprite"  // For sprite NPCs
```

## AI Issues

### No Enemy Detection
```lua
// Problem: NPC doesn't detect enemies

// Common causes:
1. Factions not set correctly
   ENT.Factions = {FACTION_ZOMBIES}
   self:SetDefaultRelationship(D_HT)  // In CustomInitialize

2. Sight range too low
   ENT.SightRange = 2000  // Increase this

3. FOV too narrow
   ENT.SightFOV = 120  // Increase this

4. No relationship set
   // Call SetDefaultRelationship in CustomInitialize
```

### NPC Won't Attack
```lua
// Problem: NPC detects but doesn't attack

// Common causes:
1. Attack range set to 0
   ENT.MeleeAttackRange = 50  // Set appropriate range
   ENT.RangeAttackRange = 500

2. Missing attack function
   function ENT:OnMeleeAttack(enemy)  // Must implement this
       self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
   end

3. Missing damage in OnAnimEvent
   function ENT:OnAnimEvent()
       if self:IsAttacking() and self:GetCycle() > 0.4 then
           self:Attack({damage = 15, type = DMG_SLASH})
       end
   end
```

### NPC Doesn't Move
```lua
// Problem: NPC stands still

// Common causes:
1. Speed set to 0
   ENT.RunSpeed = 200  // Must be > 0
   ENT.WalkSpeed = 100

2. AI disabled globally
   // Check: drgbase_ai_enabled should be 1

3. Path invalid
   // NavMesh may not exist
   // Generate with: nav_generate (singleplayer only)

4. Stuck on geometry
   // Adjust collision bounds or spawn position
```

## Animation Issues

### Wrong Animation Constants
```lua
// Wrong: Using sequence names
ENT.RunAnimation = "run_all"  // Wrong!

// Correct: Use activity constants
ENT.RunAnimation = ACT_RUN
ENT.WalkAnimation = ACT_WALK
ENT.IdleAnimation = ACT_IDLE
```

### Model Doesn't Have Animation
```lua
// Problem: NPC slides without animating

// Solution: Check model has animation
ENT.UseWalkframes = true  // Use walk animation for running
ENT.RunAnimation = ACT_WALK  // Fallback to walk
```

## Damage Issues

### OnTakeDamage Blocking All Damage
```lua
// Wrong: Blocks all damage
function ENT:OnTakeDamage(dmg)
    return false  // Blocks ALL damage
end

// Correct: Return true or modifier
function ENT:OnTakeDamage(dmg)
    return true  // Allow damage
    // or
    return 0.5  // Take 50% damage
end
```

### Attack Not Dealing Damage
```lua
// Problem: Attack animation plays but no damage

// Cause: Missing Attack() call in OnAnimEvent
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({
            damage = 15,
            type = DMG_SLASH
        })  // This is REQUIRED for damage
    end
end
```

## Relationship Issues

### Faction Not Working
```lua
// Wrong: Just setting faction
ENT.Factions = {FACTION_REBELS}
// Not enough! Must also set default relationship:

// Correct: Set faction AND relationship
ENT.Factions = {FACTION_REBELS}

function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)  // Required!
end
```

### NPC Attacks Allies
```lua
// Problem: NPCs of same faction attack each other

// Cause: OnNewEnemy forcing hostility
// Wrong:
function ENT:OnNewEnemy(enemy)
    self:SetEntityRelationship(enemy, D_HT)  // Forces hostility!
end

// Correct: Check relationship first
function ENT:OnNewEnemy(enemy)
    // Don't override faction relationships
end
```

## Performance Pitfalls

### Expensive CustomThink
```lua
// Wrong: Expensive operations every tick
function ENT:CustomThink()
    local enemies = ents.FindInSphere(self:GetPos(), 5000)
    // Runs 66 times per second!
end

// Correct: Throttle expensive operations
function ENT:CustomThink()
    if CurTime() > (self.NextCheck or 0) then
        local enemies = ents.FindInSphere(self:GetPos(), 1000)
        self.NextCheck = CurTime() + 0.5
    end
end
```

### Timer Leaks
```lua
// Wrong: Timer not cleaned up
function ENT:CustomInitialize()
    timer.Create("MyTimer", 1, 0, function()
        self:DoSomething()  // Errors after NPC removed
    end)
end

// Correct: Use unique name and clean up
function ENT:CustomInitialize()
    timer.Create("MyTimer_"..self:EntIndex(), 1, 0, function()
        if not IsValid(self) then return end
        self:DoSomething()
    end)
end

function ENT:OnRemove()
    timer.Remove("MyTimer_"..self:EntIndex())
end
```

## Network Issues

### Constant Network Updates
```lua
// Wrong: Updates every tick
function ENT:CustomThink()
    self:SetNW2Int("Health", self:Health())  // 66 times per second!
end

// Correct: Only update on change
function ENT:OnTakeDamage(dmg)
    self:SetNW2Int("Health", self:Health())  // Only when health changes
    return true
end
```

## Debugging Tips

1. **Check console** - Most errors print to console
2. **Use developer 1** - Shows more debug info
3. **Test in singleplayer first** - Easier to debug
4. **Enable AI debug** - `drgbase_ai_debug 1`
5. **Use info tool** - Real-time NPC information
6. **Add print statements** - Debug complex logic

## Common Error Messages

### "attempt to call method 'Attack' (a nil value)"
- Calling Attack() outside OnAnimEvent or on non-DrGBase NPC

### "bad argument #1 to 'SetTarget'"
- Passing invalid vector/entity to SetTarget

### "Entity is not a nextbot"
- Using wrong base or DrGBase not installed

### "attempt to index field 'Factions' (a nil value)"
- Accessing Factions before it's defined

For troubleshooting help, see [Developer Tools](../tools/01-overview.md).
