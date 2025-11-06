# Hook Reference

Quick reference for all available DrGBase nextbot hooks.

For detailed documentation, see [Nextbot Hooks API](../api/nextbot/hooks.md).

---

## Quick Reference Table

### Initialization & Lifecycle

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `CustomInitialize()` | None | None | SERVER | After NPC spawns, base initialization complete |
| `CustomThink()` | None | None | SERVER | Every server tick |
| `OnRemove()` | None | None | SERVER | When entity is removed |

### Combat Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnMeleeAttack(enemy)` | enemy (Entity) | None | SERVER | When performing melee attack |
| `OnRangeAttack(enemy)` | enemy (Entity) | None | SERVER | When performing ranged attack |
| `OnTakeDamage(dmg)` | dmg (CTakeDamageInfo) | boolean | SERVER | When NPC takes damage (return false to prevent) |
| `OnDeath(dmg, delay, hitgroup)` | dmg, delay, hitgroup | None | SERVER | When NPC dies |
| `OnKilled(attacker, inflictor)` | attacker, inflictor | None | SERVER | When NPC is killed by attacker |

### AI Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnNewEnemy(enemy)` | enemy (Entity) | None | SERVER | When new enemy acquired |
| `OnLostEnemy(enemy)` | enemy (Entity) | None | SERVER | When enemy lost |
| `OnEnemyVisible(enemy)` | enemy (Entity) | None | SERVER | When enemy becomes visible |
| `OnEnemyInvisible(enemy)` | enemy (Entity) | None | SERVER | When enemy becomes invisible |
| `OnHearSound(sound)` | sound (table) | None | SERVER | When NPC hears a sound |

### Movement Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnMovementComplete()` | None | None | SERVER | When pathfinding movement completes |
| `OnMovementFailed()` | None | None | SERVER | When pathfinding fails |
| `OnLanded()` | None | None | SERVER | When NPC lands after jump/fall |
| `OnClimbStart()` | None | None | SERVER | When climbing starts |
| `OnClimbEnd()` | None | None | SERVER | When climbing ends |

### Animation Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnAnimationStart(anim)` | anim (number) | None | SHARED | When animation starts playing |
| `OnAnimationComplete(anim)` | anim (number) | None | SHARED | When animation finishes |
| `OnAnimEvent(event, options)` | event, options | boolean | SERVER | For animation events (return true if handled) |

### Relationship Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnRelationshipChanged(entity, oldDisp, newDisp)` | entity, oldDisp, newDisp | None | SERVER | When relationship changes |
| `OnFactionChanged(oldFaction, newFaction)` | oldFaction, newFaction | None | SERVER | When faction changes |

### Possession Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnPossessed(player)` | player (Player) | None | SERVER | When player possesses NPC |
| `OnUnpossessed(player)` | player (Player) | None | SERVER | When possession ends |
| `OnPossessionThink()` | None | None | SERVER | Every tick while possessed |

### Weapon Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnWeaponEquipped(weapon)` | weapon (Entity) | None | SERVER | When weapon equipped |
| `OnWeaponRemoved(weapon)` | weapon (Entity) | None | SERVER | When weapon removed |

### Collision Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnTouch(entity)` | entity (Entity) | None | SERVER | When NPC touches entity |

### Utility Hooks

| Hook | Parameters | Returns | Realm | When Called |
|------|------------|---------|-------|-------------|
| `OnSpotCreated(spot)` | spot (Vector) | None | SERVER | When NPC finds spot (cover/patrol/etc) |

---

## Common Usage Patterns

### Basic NPC Setup

```lua
function ENT:CustomInitialize()
    self:SetHealth(200)
    self:GiveWeapon("weapon_ar2")
    self:SetFaction(FACTION_COMBINE)
    self:SetDefaultRelationship(D_HT)
end
```

### Combat Implementation

```lua
function ENT:OnMeleeAttack(enemy)
    self:EmitSound("NPC.Attack")
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

function ENT:OnAnimEvent(event, options)
    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({
            damage = 20,
            type = DMG_SLASH,
            viewpunch = Angle(10, 0, 0)
        })
        return true
    end
    return false
end
```

### AI Awareness

```lua
function ENT:OnNewEnemy(enemy)
    self:EmitSound("NPC.Alert")
    self:PlayAnimation(ACT_SIGNAL_GROUP)
end

function ENT:OnHearSound(sound)
    if sound.type == "danger" then
        self:FaceTowards(sound.pos)
        self:AddPatrolPos(sound.pos)
    end
end
```

### Damage Handling

```lua
function ENT:OnTakeDamage(dmg)
    -- Reduce bullet damage
    if dmg:IsBulletDamage() then
        dmg:ScaleDamage(0.5)
    end

    -- Play pain sound occasionally
    if math.random() < 0.3 then
        self:EmitSound("NPC.Pain")
    end

    return true -- Allow damage
end

function ENT:OnDeath(dmg, delay, hitgroup)
    self:EmitSound("NPC.Death")

    -- Drop weapon
    if IsValid(self:GetActiveWeapon()) then
        self:GetActiveWeapon():Drop()
    end
end
```

---

## Hook Priority Order

Some hooks are called in specific orders:

1. **Damage Flow:** `OnTakeDamage` → `OnDeath` → `OnKilled`
2. **Animation Flow:** `OnAnimationStart` → `OnAnimEvent` (during) → `OnAnimationComplete`
3. **Enemy Flow:** `OnNewEnemy` → `OnEnemyVisible`/`OnEnemyInvisible` → `OnLostEnemy`
4. **Movement Flow:** (path start) → `OnMovementComplete` OR `OnMovementFailed`

---

## Important Notes

1. **Return Values:** Only some hooks expect return values:
   - `OnTakeDamage(dmg)` - Return false to prevent damage
   - `OnAnimEvent(event, options)` - Return true if event handled

2. **Always Check Validity:** Use `IsValid()` on entity parameters

3. **Avoid Blocking:** Don't run long operations in hooks, use timers instead

4. **Coroutines:** Most AI hooks run in coroutines, so you can use `self:Wait()`, `self:PauseCoroutine()`, etc.

5. **Call Parent:** If extending behavior, call parent hooks:
   ```lua
   function ENT:CustomInitialize()
       if self.BaseClass.CustomInitialize then
           self.BaseClass.CustomInitialize(self)
       end
       -- Your custom code
   end
   ```

---

## See Also

- [Nextbot Hooks API (Detailed)](../api/nextbot/hooks.md)
- [Base Configuration](../api/nextbot/base-config.md)
- [Creating NPCs Guide](../guides/creating-npcs.md)
- [Examples](../examples/README.md)
