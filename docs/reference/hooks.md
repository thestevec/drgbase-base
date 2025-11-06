# DrGBase Hooks Reference

Complete reference of all available DrGBase hooks for NPCs.

## Initialization Hooks

### CustomInitialize()
Called once when NPC spawns.
```lua
function ENT:CustomInitialize()
    // Set up initial state
    self:SetDefaultRelationship(D_HT)
    self:SetBodygroup(1, math.random(0, 2))
end
```

### CustomThink()
Called every AI think (default: 30 times per second).
```lua
function ENT:CustomThink()
    // Custom AI logic
    // Called continuously while NPC exists
end
```

## Combat Hooks

### OnMeleeAttack(enemy)
Called when NPC performs melee attack.
```lua
function ENT:OnMeleeAttack(enemy)
    self:EmitSound("Zombie.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### OnRangeAttack(enemy)
Called when NPC performs ranged attack.
```lua
function ENT:OnRangeAttack(enemy)
    self:FaceTo(enemy:EyePos())
    // Spawn projectile here
    self:PauseCoroutine(1.0)
end
```

### OnAnimEvent()
Called during animations for precise timing.
```lua
function ENT:OnAnimEvent()
    if self:IsAttacking() and self:GetCycle() > 0.4 then
        self:Attack({damage = 15, type = DMG_SLASH})
    end
end
```

## Damage Hooks

### OnTakeDamage(dmginfo)
Called when NPC takes damage. Return value modifies damage.
```lua
function ENT:OnTakeDamage(dmginfo)
    return true  // Allow damage
    // return false  // Block damage
    // return 0.5  // Take 50% damage
    // return 2.0  // Take 200% damage
end
```

### OnDeath(dmginfo, hitgroup)
Called when NPC dies.
```lua
function ENT:OnDeath(dmginfo, hitgroup)
    self:EmitSound("Zombie.Die")
    // Spawn items, effects, etc.
end
```

## Enemy Detection Hooks

### OnNewEnemy(enemy)
Called when new enemy is detected.
```lua
function ENT:OnNewEnemy(enemy)
    self:EmitSound("Zombie.Alert")
    // Alert allies, change behavior, etc.
end
```

### OnLostEnemy(enemy)
Called when enemy is lost.
```lua
function ENT:OnLostEnemy(enemy)
    self.LastKnownEnemyPos = enemy:GetPos()
end
```

### OnChaseEnemy(enemy)
Called while chasing enemy.
```lua
function ENT:OnChaseEnemy(enemy)
    // Custom chase behavior
    // Called continuously while chasing
end
```

## Movement Hooks

### OnIdle()
Called when NPC has no enemy and finished patrol.
```lua
function ENT:OnIdle()
    self:AddPatrolPos(self:RandomPos(1500))
end
```

### OnReachedPatrol(pos)
Called when NPC reaches patrol destination.
```lua
function ENT:OnReachedPatrol(pos)
    self:Wait(math.random(3, 7))
end
```

### OnLandOnGround()
Called when NPC lands after being airborne.
```lua
function ENT:OnLandOnGround()
    self:EmitFootstep()
end
```

### OnContact(entity)
Called when NPC touches another entity.
```lua
function ENT:OnContact(entity)
    if entity:IsPlayer() then
        // Do something
    end
end
```

## Animation Hooks

### OnAnimationFinished()
Called when current animation finishes.
```lua
function ENT:OnAnimationFinished()
    // Animation completed
end
```

## Relationship Hooks

### OnNewAlly(ally)
Called when new ally is detected.
```lua
function ENT:OnNewAlly(ally)
    // Coordinate with ally
end
```

## Weapon Hooks (Human NPCs only)

### OnWeaponFired(weapon)
Called when NPC fires weapon.
```lua
function ENT:OnWeaponFired(weapon)
    // Custom behavior after firing
end
```

### OnReload()
Called when NPC reloads weapon.
```lua
function ENT:OnReload()
    // Take cover while reloading
end
```

## Sound Hooks

### OnFootstep(pos, foot)
Called for each footstep.
```lua
function ENT:OnFootstep(pos, foot)
    // Custom footstep handling
    self:EmitFootstep()
end
```

## Removal Hooks

### OnRemove()
Called when NPC is removed from world.
```lua
function ENT:OnRemove()
    // Cleanup timers, effects, etc.
    timer.Remove("MyTimer_"..self:EntIndex())
end
```

For usage examples, see [Examples documentation](../examples/).
