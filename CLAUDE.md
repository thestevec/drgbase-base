# DrGBase - AI Assistant Documentation

This document provides AI assistants (like Claude) with structured guidance for understanding, maintaining, and extending the DrGBase framework for Garry's Mod.

## What is DrGBase?

**DrGBase** is a comprehensive Garry's Mod nextbot framework for creating sophisticated AI-driven NPCs with advanced behaviors, weapon systems, and possession mechanics. It provides a robust foundation for NPC development with built-in systems for AI, movement, combat, animations, relationships, and more.

## 📋 Quick Reference

### Core Philosophy

DrGBase follows these key principles:

1. **Hook-Based Customization** - Extend behavior through hooks, not method overrides
2. **Property-Driven Configuration** - Configure NPCs via ENT properties, not runtime code
3. **Built-in First** - Always use built-in DrGBase functionality before creating custom solutions
4. **Realm Separation** - Clear SERVER/CLIENT separation for proper multiplayer support
5. **Registry Pattern** - All entities register via `DrGBase.AddNextbot(ENT)`, `DrGBase.AddWeapon(SWEP)`, etc.

### Essential Documentation Routes

For comprehensive information, **always refer to**:

- **[Complete Documentation Index](docs/README.md)** - Master documentation hub
- **[Architecture Overview](docs/architecture/README.md)** - System design and patterns
- **[Design Patterns](docs/architecture/06-design-patterns.md)** - **CRITICAL**: Understanding DrGBase patterns
- **[Common Pitfalls](docs/best-practices/06-common-pitfalls.md)** - **CRITICAL**: Avoid common mistakes
- **[API Reference](docs/api/README.md)** - Complete API documentation

## 🏗️ Architecture & Core Concepts

### Initialization Flow

```
1. lua/autorun/drgbase.lua          - Entry point
2. lua/drgbase/*.lua                - Core framework functions
3. lua/drgbase/meta/*.lua           - Metatable extensions (Entity, NPC, Player, etc.)
4. lua/drgbase/modules/*.lua        - Utility modules (net, timer, coroutine, etc.)
5. lua/entities/drgbase_nextbot/    - Base nextbot implementation
6. lua/weapons/drgbase_weapon/      - Base weapon implementation
```

**Key Files to Understand:**

- `lua/autorun/drgbase.lua` - Framework initialization
- `lua/entities/drgbase_nextbot/shared.lua` - Base nextbot properties
- `lua/entities/drgbase_nextbot/ai.lua` - AI decision-making
- `lua/entities/drgbase_nextbot/hooks.lua` - Hook system implementation
- `lua/entities/drgbase_nextbot/movements.lua` - Movement and pathfinding
- `lua/entities/drgbase_nextbot/weapons.lua` - Weapon handling

### Design Patterns (CRITICAL TO UNDERSTAND)

DrGBase uses these patterns extensively - **read [Design Patterns](docs/architecture/06-design-patterns.md)** for details:

1. **Inheritance Pattern** - `ENT.Base = "drgbase_nextbot"`
2. **Hook System** - CustomInitialize, OnMeleeAttack, OnDeath, etc.
3. **Registry Pattern** - `DrGBase.AddNextbot(ENT)`
4. **Factory Pattern** - `ents.Create()` and helper functions
5. **Observer Pattern** - Event notifications via hooks
6. **State Pattern** - Behavior states (idle, alert, combat)
7. **Template Method** - Think(), Initialize() with extension points
8. **Decorator Pattern** - Metatable extensions
9. **Facade Pattern** - Simplified interfaces for complex operations

## 🔧 Working with DrGBase Code

### Creating a New NPC - Required Pattern

**ALWAYS follow this pattern** (details in [docs/guides/creating-npcs.md](docs/guides/creating-npcs.md)):

```lua
-- 1. ALWAYS check DrGBase exists
if not DrGBase then return end

-- 2. Set base class
ENT.Base = "drgbase_nextbot"

-- 3. Configure at CLASS LEVEL (not in functions!)
ENT.PrintName = "My NPC"
ENT.Category = "My NPCs"
ENT.Models = {"models/player.mdl"}
ENT.SpawnHealth = 100
ENT.WalkSpeed = 100
ENT.RunSpeed = 200

-- 4. Use SERVER/CLIENT guards for realm-specific code
if SERVER then
    -- Server-only initialization
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    -- Server-only hooks
    function ENT:OnMeleeAttack(enemy)
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnAnimEvent()
        if self:IsAttacking() and self:GetCycle() > 0.3 then
            self:Attack({damage = 10, type = DMG_SLASH})
        end
    end
end

if CLIENT then
    -- Client-only rendering
    function ENT:CustomDraw()
        self:DrawModel()
    end
end

-- 5. ALWAYS include at end of file
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

### Critical Requirements for NPC Development

**Configuration**
- ✅ Set properties at **class level** (outside functions)
- ❌ Do NOT set core properties in CustomInitialize() - it's too late!
- ✅ Use CustomInitialize() for runtime/conditional setup only

**Hooks (Use instead of overriding methods)**
- Use `CustomInitialize()` not `Initialize()`
- Use `CustomThink()` not `Think()`
- Use `OnMeleeAttack()`, `OnRangeAttack()`, `OnDeath()`, etc.
- See [docs/api/nextbot/hooks.md](docs/api/nextbot/hooks.md) for complete list

**Realm Separation**
- Wrap server-only code in `if SERVER then ... end`
- Wrap client-only code in `if CLIENT then ... end`
- Keep shared code outside guards

**Required Boilerplate**
- Start with `if not DrGBase then return end`
- End with `AddCSLuaFile()` and `DrGBase.AddNextbot(ENT)`

## 📚 API Reference Quick Links

### Nextbot Base

Configuration and core functionality:
- **[Base Configuration Properties](docs/api/nextbot/base-config.md)** - All ENT properties
- **[Hooks Reference](docs/api/nextbot/hooks.md)** - All available hooks
- **[AI Functions](docs/api/nextbot/ai.md)** - Enemy detection, targeting
- **[Movement Functions](docs/api/nextbot/movement.md)** - Pathfinding, locomotion
- **[Animation Functions](docs/api/nextbot/animation.md)** - Animation control
- **[Weapon Functions](docs/api/nextbot/weapons.md)** - Weapon handling
- **[Combat Functions](docs/api/combat-system.md)** - Damage, attacks
- **[Relationship Functions](docs/api/nextbot/relationships.md)** - Factions, allies

### Systems

Deep dives into specific systems:
- **[AI System](docs/api/ai-system.md)** - How AI decision-making works
- **[Movement System](docs/api/movement-system.md)** - Pathfinding and navigation
- **[Combat System](docs/api/combat-system.md)** - Damage and attacks

### Metatable Extensions

DrGBase extends engine metatables:
- **[Entity Extensions](docs/api/meta/entity.md)** - Methods added to all entities
- **[NPC Extensions](docs/api/meta/npc.md)** - Methods added to NPCs
- **[Player Extensions](docs/api/meta/player.md)** - Methods added to players
- **[Vector Extensions](docs/api/meta/vector.md)** - Vector helpers

## 🎯 Common Tasks & Where to Look

### Task: Understanding How Feature X Works

1. Check **[API Reference](docs/api/README.md)** for function documentation
2. Read **[Architecture docs](docs/architecture/README.md)** for system understanding
3. Look at **[Examples](docs/examples/README.md)** for real implementations
4. Review source: `lua/entities/drgbase_nextbot/[system].lua`

### Task: Creating a New NPC Type

1. **FIRST**: Read **[Creating NPCs Guide](docs/guides/creating-npcs.md)**
2. **THEN**: Review **[Common Pitfalls](docs/best-practices/06-common-pitfalls.md)**
3. Study similar examples in **[Examples](docs/examples/README.md)**:
   - Simple melee: [docs/examples/simple-melee-npc.md](docs/examples/simple-melee-npc.md)
   - Ranged combat: [docs/examples/ranged-npc.md](docs/examples/ranged-npc.md)
   - Flying NPC: [docs/examples/flying-npc.md](docs/examples/flying-npc.md)
   - Boss NPC: [docs/examples/boss-npc.md](docs/examples/boss-npc.md)

### Task: Adding Weapons/Projectiles

1. **Weapons**: Read **[Creating Weapons Guide](docs/guides/creating-weapons.md)**
2. **Projectiles**: Read **[Creating Projectiles Guide](docs/guides/creating-projectiles.md)**
3. Review API:
   - [Weapon Base Config](docs/api/weapon/base-config.md)
   - [Projectile Base Config](docs/api/projectile/base-config.md)

### Task: Debugging Issues

1. **FIRST**: Check **[Common Pitfalls](docs/best-practices/06-common-pitfalls.md)**
2. Enable developer mode: `developer 1` in console
3. Use Info Tool: See [docs/tools/02-info-tool.md](docs/tools/02-info-tool.md)
4. Review [Debugging Guide](docs/guides/debugging.md)
5. Check navmesh loaded: `navmesh.IsLoaded()`

### Task: Performance Optimization

1. Read **[Performance Guidelines](docs/best-practices/02-performance.md)**
2. Read **[Optimization Guide](docs/guides/optimization.md)**
3. Avoid common mistakes:
   - Don't network excessive data
   - Don't use loops in Think functions
   - Clean up timers properly
   - Check navmesh existence

## ⚠️ Critical Compliance Rules

**ALWAYS follow these rules when working with DrGBase:**

### 1. Properties Must Be Class-Level
```lua
✅ CORRECT:
ENT.SpawnHealth = 100  -- At class level

❌ WRONG:
function ENT:CustomInitialize()
    self.SpawnHealth = 100  -- Too late! Already initialized
end
```

### 2. Use Hooks, Not Overrides
```lua
✅ CORRECT:
function ENT:CustomThink()
    -- Your code
end

❌ WRONG:
function ENT:Think()
    -- Don't override base methods!
end
```

### 3. Check DrGBase Exists
```lua
✅ CORRECT:
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

❌ WRONG:
ENT.Base = "drgbase_nextbot"  -- Crashes if DrGBase missing!
```

### 4. Include Required Boilerplate
```lua
✅ CORRECT:
AddCSLuaFile()
DrGBase.AddNextbot(ENT)

❌ WRONG:
-- Missing! Won't work properly in multiplayer
```

### 5. Realm Guards for Multiplayer
```lua
✅ CORRECT:
if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end
end

❌ WRONG:
function ENT:CustomInitialize()
    self:SetDefaultRelationship(D_HT)  -- Errors on client!
end
```

### 6. Check Navmesh Before Pathfinding
```lua
✅ CORRECT:
if navmesh.IsLoaded() then
    self:FollowPath(pos)
end

❌ WRONG:
self:FollowPath(pos)  -- Crashes on maps without navmesh!
```

### 7. Use Built-in Timer System
```lua
✅ CORRECT:
self:Timer(5, function()
    self:DoSomething()
end, true)  -- Auto-cleanup

❌ WRONG:
timer.Create("MyTimer", 5, 0, function()
    self:DoSomething()  -- Errors after entity removed!
end)
```

## 🔍 Code Location Reference

### Understanding Core Systems

**AI Decision Making**
- Primary: `lua/entities/drgbase_nextbot/ai.lua`
- Docs: [docs/api/ai-system.md](docs/api/ai-system.md)

**Movement & Pathfinding**
- Primary: `lua/entities/drgbase_nextbot/movements.lua`
- Locomotion: `lua/entities/drgbase_nextbot/locomotion.lua`
- Path: `lua/entities/drgbase_nextbot/path.lua`
- Docs: [docs/api/movement-system.md](docs/api/movement-system.md)

**Combat & Weapons**
- Weapons: `lua/entities/drgbase_nextbot/weapons.lua`
- Base Weapon: `lua/weapons/drgbase_weapon/`
- Projectiles: `lua/entities/proj_drg_default/`
- Docs: [docs/api/combat-system.md](docs/api/combat-system.md)

**Animations**
- Primary: `lua/entities/drgbase_nextbot/animations.lua`
- Docs: [docs/api/nextbot/animation.md](docs/api/nextbot/animation.md)

**Relationships & Factions**
- Primary: `lua/entities/drgbase_nextbot/relationships.lua`
- Docs: [docs/api/nextbot/relationships.md](docs/api/nextbot/relationships.md)

**Hooks System**
- Primary: `lua/entities/drgbase_nextbot/hooks.lua`
- Docs: [docs/api/nextbot/hooks.md](docs/api/nextbot/hooks.md)

**Possession System**
- Primary: `lua/entities/drgbase_nextbot/possession.lua`
- Docs: [docs/api/nextbot/possession.md](docs/api/nextbot/possession.md)

## 📖 Examples for Learning

Study these complete, working examples:

- **[Simple Melee NPC](docs/examples/simple-melee-npc.md)** - Start here for basics
- **[Ranged Combat NPC](docs/examples/ranged-npc.md)** - Projectile attacks
- **[Flying NPC](docs/examples/flying-npc.md)** - Custom movement
- **[Sprite NPC](docs/examples/sprite-npc.md)** - 2D characters
- **[Boss NPC](docs/examples/boss-npc.md)** - Complex multi-phase enemy
- **[Advanced AI](docs/examples/advanced-ai.md)** - Complex behaviors
- **[Faction System](docs/examples/faction-system.md)** - Team dynamics

## 🎓 Learning Path

**For New Development:**
1. Read [Getting Started](docs/getting-started/README.md)
2. Study [Design Patterns](docs/architecture/06-design-patterns.md) - **CRITICAL**
3. Review [Common Pitfalls](docs/best-practices/06-common-pitfalls.md) - **CRITICAL**
4. Create [Simple Melee NPC](docs/examples/simple-melee-npc.md)
5. Explore [API Reference](docs/api/README.md) as needed

**For Understanding Existing Code:**
1. Read [Architecture Overview](docs/architecture/README.md)
2. Study [Design Patterns](docs/architecture/06-design-patterns.md)
3. Review specific system docs in [API Reference](docs/api/README.md)
4. Examine source in `lua/entities/drgbase_nextbot/`

**For Debugging:**
1. Check [Common Pitfalls](docs/best-practices/06-common-pitfalls.md) first
2. Review [Debugging Guide](docs/guides/debugging.md)
3. Use [Info Tool](docs/tools/02-info-tool.md)
4. Check console with `developer 1`

## 🚀 Best Practices

### Code Organization
- Read: [docs/best-practices/01-code-organization.md](docs/best-practices/01-code-organization.md)
- Keep configuration separate from implementation
- Use tables for abilities/config data
- Comment complex logic

### Performance
- Read: [docs/best-practices/02-performance.md](docs/best-practices/02-performance.md)
- Don't use loops in Think functions
- Network only when values change
- Clean up timers and hooks
- Check navmesh before pathfinding

### Networking
- Read: [docs/best-practices/03-networking.md](docs/best-practices/03-networking.md)
- Only network what clients need to see
- Use net messages for one-time events
- Don't use SetNW2* in Think loops
- Test in multiplayer!

### Security
- Read: [docs/best-practices/04-security.md](docs/best-practices/04-security.md)
- Validate user input
- Use SERVER realm for damage/logic
- Don't trust client data

## 📝 Quick Checklist for Code Review

When reviewing or creating DrGBase code, verify:

- [ ] Starts with `if not DrGBase then return end`
- [ ] Properties set at class level, not in CustomInitialize
- [ ] Uses hooks (CustomInitialize, CustomThink, etc.) not overrides
- [ ] SERVER/CLIENT guards in correct places
- [ ] Ends with `AddCSLuaFile()` and `DrGBase.AddNextbot(ENT)`
- [ ] Checks `navmesh.IsLoaded()` before pathfinding
- [ ] Uses `self:Timer()` or cleans up manual timers
- [ ] No loops in Think functions
- [ ] Networks only changed values
- [ ] Tested in multiplayer if networking used

## 🔗 External Resources

- **[Complete Documentation](docs/README.md)** - Start here for comprehensive docs
- **[GitHub Repository](https://github.com/Cpt-Hazama/DrGBase)** - Source code
- **[Workshop Page](https://steamcommunity.com/sharedfiles/filedetails/?id=1815683702)** - Steam Workshop

## 📊 Documentation Statistics

- **100+** documentation files
- **9** complete example NPCs
- **7** developer tools
- **6** best practice guides
- **Full API coverage** of all systems

---

**When in doubt:**
1. Check [docs/README.md](docs/README.md) first
2. Review [Common Pitfalls](docs/best-practices/06-common-pitfalls.md)
3. Study [Design Patterns](docs/architecture/06-design-patterns.md)
4. Look at [Examples](docs/examples/README.md)

**Remember:** DrGBase provides built-in solutions for most NPC needs. Always check if functionality already exists before implementing custom solutions!
