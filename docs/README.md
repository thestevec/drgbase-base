# DrGBase Documentation

Welcome to the comprehensive documentation for **DrGBase** - a powerful Garry's Mod nextbot framework for creating sophisticated AI-driven NPCs with advanced behaviors, weapon systems, and possession mechanics.

## 🚀 Quick Start

### Your First NPC (5 minutes)

Create `lua/entities/npc_mypack_zombie.lua`:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "My Zombie"
ENT.Category = "My NPCs"
ENT.Models = {"models/Zombie/Classic.mdl"}
ENT.SpawnHealth = 100
ENT.MeleeAttackRange = 30
ENT.Factions = {FACTION_ZOMBIES}

if SERVER then
    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)
    end

    function ENT:OnMeleeAttack(enemy)
        self:EmitSound("Zombie.Attack")
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnAnimEvent()
        if self:IsAttacking() and self:GetCycle() > 0.3 then
            self:Attack({damage = 10, type = DMG_SLASH})
        end
    end

    function ENT:OnIdle()
        self:AddPatrolPos(self:RandomPos(1500))
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

**Test your NPC:**
1. Generate navmesh: `nav_generate` in console
2. Open spawn menu (Q)
3. Go to NPCs → My NPCs
4. Spawn "My Zombie"

**Next Steps:**
- Read [Getting Started Tutorial](./getting-started/README.md)
- Explore [Code Examples](./examples/README.md)
- Use [Info Tool](./tools/02-info-tool.md) to debug
- Check [Common Pitfalls](./best-practices/06-common-pitfalls.md)

## 📚 Documentation Structure

### 🎓 [Getting Started](./getting-started/README.md)
- [Introduction](./getting-started/01-introduction.md) - What is DrGBase?
- [Installation](./getting-started/02-installation.md) - Setup guide
- [Quick Start Guide](./getting-started/03-quick-start.md) - First steps
- [Your First NPC](./getting-started/04-first-npc.md) - Detailed tutorial
- [Configuration & ConVars](./getting-started/05-configuration.md) - Settings reference

### 🏗️ [Architecture](./architecture/README.md)
- [Overview](./architecture/01-overview.md) - System design
- [File Structure](./architecture/02-file-structure.md) - Project layout
- [Initialization System](./architecture/03-initialization.md) - Startup process
- [Module System](./architecture/04-module-system.md) - Code organization
- [Client-Server Architecture](./architecture/05-client-server.md) - Networking
- [Design Patterns](./architecture/06-design-patterns.md) - Best practices

### 📚 [Core Systems](./systems/README.md)
- [AI System](./systems/ai/README.md) - Behavior and decision making
- [Movement & Pathfinding](./systems/movement/README.md) - Navigation
- [Combat & Weapons](./systems/combat/README.md) - Attack systems
- [Animation System](./systems/animation/README.md) - Animations
- [Relationship & Factions](./systems/relationships/README.md) - Team dynamics
- [Possession System](./systems/possession/README.md) - Player control
- [Status & Health](./systems/status/README.md) - Conditions
- [Networking](./systems/networking/README.md) - Multiplayer
- [Spawner System](./systems/spawners/README.md) - Spawn management
- [Resource Management](./systems/resources/README.md) - Optimization

### 📖 [API Reference](./api/README.md)

#### Core Framework
- [DrGBase Global](./api/core/drgbase.md) - Main framework
- [Entity Helpers](./api/core/entity-helpers.md) - Utility functions
- [Enumerations](./api/core/enumerations.md) - Constants
- [Colors](./api/core/colors.md) - Color utilities

#### Nextbot Base
- [Base Configuration](./api/nextbot/base-config.md) - Properties
- [AI Functions](./api/nextbot/ai.md) - Intelligence
- [Movement Functions](./api/nextbot/movement.md) - Locomotion
- [Animation Functions](./api/nextbot/animation.md) - Animations
- [Weapon Functions](./api/nextbot/weapons.md) - Combat
- [Relationship Functions](./api/nextbot/relationships.md) - Factions
- [Detection Functions](./api/nextbot/detection.md) - Perception
- [Awareness Functions](./api/nextbot/awareness.md) - Senses
- [Patrol Functions](./api/nextbot/patrol.md) - Wandering
- [Path Functions](./api/nextbot/path.md) - Pathfinding
- [Status Functions](./api/nextbot/status.md) - Conditions
- [Possession Functions](./api/nextbot/possession.md) - Control
- [Hooks](./api/nextbot/hooks.md) - Events

#### Weapon Base
- [Base Configuration](./api/weapon/base-config.md) - Weapon properties
- [Primary Attack](./api/weapon/primary.md) - Main fire mode
- [Secondary Attack](./api/weapon/secondary.md) - Alt fire mode
- [Weapon Functions](./api/weapon/functions.md) - Weapon API

#### Projectile Base
- [Base Configuration](./api/projectile/base-config.md) - Projectile properties
- [Projectile Functions](./api/projectile/functions.md) - Projectile API

#### Metatable Extensions
- [Entity](./api/meta/entity.md) - Entity extensions
- [NPC](./api/meta/npc.md) - NPC extensions
- [Player](./api/meta/player.md) - Player extensions
- [PhysObj](./api/meta/physobj.md) - Physics extensions
- [Vector](./api/meta/vector.md) - Vector extensions

#### Utility Modules
- [Coroutine](./api/modules/coroutine.md) - Async operations
- [Debug Overlay](./api/modules/debugoverlay.md) - Visual debugging
- [Math](./api/modules/math.md) - Math helpers
- [NavMesh](./api/modules/navmesh.md) - Navigation
- [Net](./api/modules/net.md) - Networking
- [Render](./api/modules/render.md) - Drawing
- [String](./api/modules/string.md) - String utilities
- [Table](./api/modules/table.md) - Table utilities
- [Timer](./api/modules/timer.md) - Delayed execution
- [Util](./api/modules/util.md) - Miscellaneous

### 📝 [Guides](./guides/README.md)
- [Creating Custom NPCs](./guides/creating-npcs.md) - NPC development
- [Creating Custom Weapons](./guides/creating-weapons.md) - Weapon systems
- [Creating Projectiles](./guides/creating-projectiles.md) - Projectile mechanics
- [Setting Up Factions](./guides/factions.md) - Team systems
- [Implementing Possession](./guides/possession.md) - Player control
- [Animation Setup](./guides/animations.md) - Animation system
- [Pathfinding & Navigation](./guides/pathfinding.md) - Movement AI
- [Sound & Effects](./guides/sound-effects.md) - Audio integration
- [Testing & Debugging](./guides/debugging.md) - Troubleshooting
- [Performance Optimization](./guides/optimization.md) - Efficiency
- [Spawner Configuration](./guides/spawners.md) - Spawn systems

### 💡 [Examples](./examples/README.md)
- [Simple Melee NPC](./examples/simple-melee-npc.md) - Basic combat
- [Ranged Combat NPC](./examples/ranged-npc.md) - Projectile attacks
- [Flying NPC](./examples/flying-npc.md) - Air movement
- [Sprite-Based NPC](./examples/sprite-npc.md) - 2D characters
- [Boss NPC](./examples/boss-npc.md) - Complex enemies
- [Custom Weapon](./examples/custom-weapon.md) - Weapon creation
- [Custom Projectile](./examples/custom-projectile.md) - Projectile types
- [Advanced AI Behaviors](./examples/advanced-ai.md) - Complex AI
- [Faction System](./examples/faction-system.md) - Team mechanics

### 🛠️ [Developer Tools](./tools/README.md)
- [Tool Overview](./tools/01-overview.md) - Available tools
- [Info Tool](./tools/02-info-tool.md) - NPC diagnostics
- [Damage Tool](./tools/03-damage-tool.md) - Damage testing
- [Faction Tool](./tools/04-faction-tool.md) - Faction management
- [Relationship Tool](./tools/05-relationship-tool.md) - Relationship testing
- [AI Control Tools](./tools/06-ai-tools.md) - AI debugging
- [Entity Manipulation Tools](./tools/07-entity-tools.md) - Entity control

### ✅ [Best Practices](./best-practices/README.md)
- [Code Organization](./best-practices/01-code-organization.md) - Structure
- [Performance Guidelines](./best-practices/02-performance.md) - Optimization
- [Networking Best Practices](./best-practices/03-networking.md) - Multiplayer
- [Security Considerations](./best-practices/04-security.md) - Safety
- [Testing Strategies](./best-practices/05-testing.md) - QA
- [Common Pitfalls](./best-practices/06-common-pitfalls.md) - Avoid mistakes

### 🔧 [Reference](./reference/README.md)
- [ConVar Reference](./reference/convars.md) - Console variables
- [Hook Reference](./reference/hooks.md) - Event hooks
- [Network Messages](./reference/network-messages.md) - Net messages
- [Enumerations Reference](./reference/enums.md) - Constants
- [Activity IDs](./reference/activities.md) - Animations
- [Animation Events](./reference/anim-events.md) - Animation hooks

## 🔍 Finding What You Need

### I want to...

**Learn the basics**
→ Start with [Getting Started](./getting-started/README.md)

**Create a specific NPC type**
→ Check [Examples](./examples/README.md) for your use case

**Debug an issue**
→ Use [Info Tool](./tools/02-info-tool.md) + [Debugging Guide](./guides/debugging.md)

**Optimize performance**
→ Read [Performance Guide](./guides/optimization.md)

**Look up an API**
→ Check [API Reference](./api/README.md)

**Avoid common mistakes**
→ Review [Common Pitfalls](./best-practices/06-common-pitfalls.md)

## 🎯 By Experience Level

### Beginner
1. [Getting Started](./getting-started/README.md)
2. [Simple Melee NPC](./examples/simple-melee-npc.md)
3. [Info Tool](./tools/02-info-tool.md)
4. [Common Pitfalls](./best-practices/06-common-pitfalls.md)

### Intermediate
1. [Advanced AI Behaviors](./examples/advanced-ai.md)
2. [Faction System](./examples/faction-system.md)
3. [Performance Guide](./guides/optimization.md)
4. [Code Organization](./best-practices/01-code-organization.md)

### Advanced
1. [Sprite NPCs](./examples/sprite-npc.md)
2. [Possession System](./guides/possession.md)
3. [Debugging Guide](./guides/debugging.md)
4. [Architecture Overview](./architecture/README.md)

## 📦 Documentation Statistics

- **Total Files**: 100+ documentation files
- **Code Examples**: 9 complete tutorials
- **Tool Docs**: 7 tool guides
- **Best Practices**: 6 comprehensive guides
- **Estimated Reading Time**: 10-15 hours for complete documentation
- **Estimated Implementation Time**: 60-75 hours for all examples

## 🔗 Quick Links

### Essential Documentation
- [API Reference](./api/README.md) - Bookmark this!
- [Getting Started](./getting-started/README.md) - Start here
- [Common Pitfalls](./best-practices/06-common-pitfalls.md) - Save yourself time

### Most Popular Topics
- [Creating NPCs](./guides/creating-npcs.md)
- [Advanced AI](./examples/advanced-ai.md)
- [Factions & Relationships](./guides/factions.md)
- [Performance Optimization](./guides/optimization.md)

### Troubleshooting
- [Debugging Guide](./guides/debugging.md)
- [Info Tool](./tools/02-info-tool.md)
- [ConVar Reference](./reference/convars.md)

## 📝 Contributing to Documentation

This documentation is open for contributions. If you find errors or want to improve any section, please submit a pull request.

## 📜 Version

This documentation is for DrGBase version corresponding to commit `aa96812`.

**Last Updated:** 2025-11-06

---

**Happy NPC Creating! 🎮**

For questions, check existing documentation or community resources.
