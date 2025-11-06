# DrGBase Documentation

Welcome to the comprehensive DrGBase documentation! This documentation covers everything you need to create custom Nextbot NPCs for Garry's Mod.

## 📚 Documentation Structure

### 🎓 [Examples](examples/) - Learn by Doing
Complete tutorials and working examples:
- [Getting Started](examples/01-getting-started.md) - Your first NPC
- [Ranged NPCs](examples/02-ranged-npc.md) - Projectile attacks
- [Melee Combat](examples/03-melee-combat.md) - Advanced melee
- [Humanoid Weapons](examples/04-humanoid-weapons.md) - Weapon-wielding NPCs
- [Sprite NPCs](examples/05-sprite-npcs.md) - 2D character creation
- [Custom AI](examples/06-custom-ai.md) - Advanced AI patterns
- [Relationships](examples/07-relationships.md) - Factions & dispositions
- [Possession](examples/08-possession.md) - Player control system
- [Animations](examples/09-animations.md) - Animation system

### 🛠️ [Tools](tools/) - Debug & Test
Developer tools and debugging:
- [Overview](tools/overview.md) - All available tools
- [Info Tool](tools/info-tool.md) - Real-time NPC diagnostics
- [Faction Tool](tools/faction-tool.md) - Faction management
- [Debug Tools](tools/debug-tools.md) - Testing utilities
- [Console Commands](tools/console-commands.md) - Command reference

### 📖 [Guides](guides/) - Best Practices
In-depth guides for quality development:
- [Performance](guides/performance.md) - Optimization techniques
- [Debugging](guides/debugging.md) - Finding and fixing issues
- [Common Pitfalls](guides/common-pitfalls.md) - Avoid mistakes
- [Code Organization](guides/code-organization.md) - Structure your code
- [Testing](guides/testing.md) - QA strategies

### 📋 [Reference](reference/) - Quick Lookup
API reference and quick guides:
- [Quick Reference](reference/quick-reference.md) - All APIs at a glance

## 🚀 Quick Start

### 1. Your First NPC (5 minutes)

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

### 2. Test Your NPC

1. Generate navmesh: `nav_generate` in console
2. Open spawn menu (Q)
3. Go to NPCs → My NPCs
4. Spawn "My Zombie"

### 3. Next Steps

- Read [Getting Started Tutorial](examples/01-getting-started.md)
- Explore [Code Examples](examples/)
- Use [Info Tool](tools/info-tool.md) to debug
- Check [Common Pitfalls](guides/common-pitfalls.md)

## 📊 Documentation Packages

### Low Priority Documentation (Completed)

**Package 16: Code Examples** ✅
- 9 comprehensive tutorials
- Working code examples
- Step-by-step guides

**Package 17: Developer Tools** ✅
- 5 tool documentation files
- Debugging workflows
- Console command reference

**Package 18: Best Practices** ✅
- 5 comprehensive guides
- Performance optimization
- Testing strategies

**Package 19: Reference** ✅
- Quick reference guide
- Complete API overview

## 🔍 Finding What You Need

### I want to...

**Learn the basics**
→ Start with [Getting Started](examples/01-getting-started.md)

**Create a specific NPC type**
→ Check [Examples](examples/) for your use case

**Debug an issue**
→ Use [Info Tool](tools/info-tool.md) + [Debugging Guide](guides/debugging.md)

**Optimize performance**
→ Read [Performance Guide](guides/performance.md)

**Look up an API**
→ Check [Quick Reference](reference/quick-reference.md)

**Avoid common mistakes**
→ Review [Common Pitfalls](guides/common-pitfalls.md)

## 🎯 By Experience Level

### Beginner
1. [Getting Started](examples/01-getting-started.md)
2. [Melee Combat](examples/03-melee-combat.md)
3. [Info Tool](tools/info-tool.md)
4. [Common Pitfalls](guides/common-pitfalls.md)

### Intermediate
1. [Custom AI](examples/06-custom-ai.md)
2. [Relationships](examples/07-relationships.md)
3. [Performance Guide](guides/performance.md)
4. [Code Organization](guides/code-organization.md)

### Advanced
1. [Sprite NPCs](examples/05-sprite-npcs.md)
2. [Possession System](examples/08-possession.md)
3. [Debugging Guide](guides/debugging.md)
4. [Testing Guide](guides/testing.md)

## 📦 Documentation Statistics

- **Total Files**: 28+ documentation files
- **Code Examples**: 9 complete tutorials
- **Tool Docs**: 5 tool guides
- **Best Practices**: 5 comprehensive guides
- **Estimated Reading Time**: 6-8 hours for complete documentation
- **Estimated Implementation Time**: 60-75 hours for all examples

## 🔗 Quick Links

### Essential Documentation
- [Quick Reference](reference/quick-reference.md) - Bookmark this!
- [Getting Started](examples/01-getting-started.md) - Start here
- [Common Pitfalls](guides/common-pitfalls.md) - Save yourself time

### Most Popular Topics
- [Melee Combat](examples/03-melee-combat.md)
- [Custom AI](examples/06-custom-ai.md)
- [Relationships & Factions](examples/07-relationships.md)
- [Performance Optimization](guides/performance.md)

### Troubleshooting
- [Debugging Guide](guides/debugging.md)
- [Info Tool](tools/info-tool.md)
- [Console Commands](tools/console-commands.md)

## 📝 Contributing

Found an error? Have a suggestion? Create an issue on GitHub!

## 📜 License

This documentation is provided as-is for use with DrGBase framework.

## 🙏 Credits

- **DrGBase Framework**: DrVrej
- **Documentation**: Comprehensive Low Priority Documentation Package
- **Community**: All contributors and testers

---

**Happy NPC Creating! 🎮**

For questions, check existing documentation or community resources.
