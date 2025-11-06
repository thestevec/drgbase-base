# Design Patterns

## Inheritance Pattern

DrGBase uses prototype-based inheritance from Garry's Mod's entity system.

### Entity Inheritance

```lua
ENT.Base = "drgbase_nextbot"
```

**How inheritance works:**
1. Set `ENT.Base` to parent class name
2. Child inherits all properties and methods from parent
3. Child can override any parent method
4. Properties are shallow-copied at registration

**Method override:**
```lua
-- Parent defines
function ENT:Initialize()
    -- Parent logic
end

-- Child overrides
function ENT:Initialize()
    -- Child replaces parent completely
end
```

**Calling parent methods:**
Garry's Mod doesn't provide `super` or `base`, so call by name:
```lua
function ENT:CustomInitialize()
    -- Call specific parent implementation if needed
    -- (Not common in DrGBase due to hook pattern)
    self:SetColor(Color(255, 0, 0))
end
```

DrGBase uses hooks instead of requiring parent calls (see Hook System Pattern).

### Multi-Level Inheritance

```
drgbase_nextbot
    ↓
drgbase_nextbot_human
    ↓
Your Custom Human NPC
```

## Hook System Pattern

DrGBase's primary customization mechanism is hooks - predefined extension points.

### Framework Hooks

Instead of overriding base methods, you implement hook functions:

```lua
function ENT:CustomInitialize()
    -- Called automatically by Initialize()
    -- No need to call parent
    self:SetColor(Color(255, 0, 0))
end

function ENT:CustomThink()
    -- Called every tick by Think()
    -- Add custom behavior here
end

function ENT:OnTakeDamage(dmg)
    -- Called when damaged
    -- Return false to block damage
    return true  -- Allow damage
end

function ENT:OnDeath(dmg, delay, hitgroup)
    -- Called on death
    -- Custom death behavior
end
```

**Available hook categories:**
- **Lifecycle:** `CustomInitialize()`, `CustomPostInitialize()`
- **Think:** `CustomThink()`, `CustomThinkAI()`
- **Damage:** `OnTakeDamage()`, `OnDeath()`, `OnDamageEntity()`
- **Combat:** `OnMeleeAttack()`, `OnRangeAttack()`, `OnReload()`
- **AI:** `OnNewEnemy()`, `OnLostEnemy()`, `OnIdle()`
- **Movement:** `OnNavAreaChanged()`, `OnLeaveGround()`, `OnLandOnGround()`
- **Animation:** `OnAnimEvent()`, `OnSequenceFinished()`

See [Hooks API](../api/nextbot/hooks.md) for complete list.

### Hook Flow

```
Engine Event → Base Method → Custom Hook → Return to Engine
    ↓              ↓               ↓
  Damage      OnTakeDamage()  Your Code    → Allow/Block
```

**Example:**
```
Player shoots NPC
  ↓
Engine calls ENT:OnTakeDamage(dmg)
  ↓
Base method calls your hook: self:OnTakeDamage(dmg)
  ↓
Your code: return false  -- Block damage
  ↓
Base method respects return value
  ↓
No damage applied
```

## Registry Pattern

DrGBase uses a registry pattern to track and manage entities centrally.

### Nextbot Registry

```lua
DrGBase.AddNextbot(ENT)
```

**How entities register:**
1. Entity file defines `ENT.PrintName` and `ENT.Category`
2. Calls `DrGBase.AddNextbot(ENT)` at end of file
3. Function validates and registers entity
4. Precaches resources
5. Adds to spawn menu

**What registration does:**
- Validates entity has required fields
- Precaches models and sounds
- Registers language strings
- Sets up killicons
- Adds hook mixins (intercepts engine hooks)
- Adds to DrGBase entity list
- Integrates with spawn menu

### Weapon Registry

```lua
DrGBase.AddWeapon(SWEP)
```

Same pattern as nextbots:
- Validates weapon
- Registers in spawn menu
- Sets up resources
- Adds to weapon list

### Spawner Registry

```lua
DrGBase.AddSpawner(ENT)
```

Registers spawner entities for spawn menu integration.

**Benefits of Registry Pattern:**
- Centralized entity management
- Consistent initialization
- Easy enumeration of all entities
- Automatic resource handling

## Factory Pattern

Entity creation uses the factory pattern through `ents.Create()`.

### Entity Creation

```lua
-- Factory method
local npc = ents.Create("npc_drg_zombie")
if IsValid(npc) then
    npc:SetPos(pos)
    npc:SetAngles(ang)
    npc:Spawn()
end
```

**How it works:**
1. `ents.Create(class)` looks up class in registry
2. Creates new entity instance
3. Copies properties from class definition
4. Returns entity reference
5. You call `:Spawn()` to initialize

**Benefits:**
- Don't need to know entity internals
- Consistent creation interface
- Automatic property copying
- Easy to create many instances

### Projectile Creation

DrGBase provides helper factories:

```lua
-- Simplified projectile factory
self:FireProjectile("proj_drg_grenade", pos, ang)
```

Internally:
```lua
function ENT:FireProjectile(class, pos, ang)
    local proj = ents.Create(class)  -- Factory
    proj:SetPos(pos)
    proj:SetAngles(ang)
    proj:SetOwner(self)
    proj:Spawn()
    return proj
end
```

Higher-level factories hide complexity.

## Observer Pattern

DrGBase uses observer pattern for event notifications through hooks.

### Event Notifications

Entities are "observers" that get notified of events:

```lua
function ENT:OnNewEnemy(ent)
    -- Automatically called when new enemy detected
    print("New enemy:", ent)
end

function ENT:OnLostEnemy(ent)
    -- Automatically called when enemy lost
    print("Lost enemy:", ent)
end

function ENT:OnTakeDamage(dmg)
    -- Notified of damage event
    print("Took damage:", dmg:GetDamage())
end
```

**How observers are notified:**

```lua
-- In base AI code (pseudo-code)
function ENT:UpdateEnemies()
    local newEnemy = self:FindNewEnemy()
    if newEnemy then
        -- Notify observers (your hook)
        if isfunction(self.OnNewEnemy) then
            self:OnNewEnemy(newEnemy)
        end
    end
end
```

**Benefits:**
- Loose coupling (base code doesn't know about custom behavior)
- Easy to extend with new observers
- Multiple entities can observe same events
- Events propagate automatically

**Real-world usage:**
```lua
-- Observer gets notified of death
function ENT:OnDeath(dmg)
    -- Drop items
    self:DropLoot()

    -- Notify other NPCs
    for _, ally in ipairs(self:GetAllies()) do
        ally:OnAllyDied(self)
    end
end
```

## State Pattern

DrGBase uses state pattern for managing NPC behavior modes.

### Behavior States

NPCs track state to determine behavior:

```lua
-- States defined as properties
ENT.State = STATE_IDLE  -- Current state

-- State changes trigger different behaviors
if self.State == STATE_IDLE then
    self:IdleBehavior()
elseif self.State == STATE_ALERT then
    self:AlertBehavior()
elseif self.State == STATE_COMBAT then
    self:CombatBehavior()
end
```

**How states are managed:**
- State stored as entity property
- Think loop checks current state
- Different logic executes per state
- State transitions handled in AI code

**State transitions:**
```lua
-- Detect enemy → Change state
if self:GetEnemy() then
    self.State = STATE_COMBAT
else
    self.State = STATE_IDLE
end
```

### Movement States

Movement has separate state:

```lua
-- Movement type property
ENT.MoveType = MOVE_WALK  -- Walking
ENT.MoveType = MOVE_RUN   -- Running
ENT.MoveType = MOVE_CLIMB -- Climbing

-- Affects speed and animation
function ENT:GetMoveSpeed()
    if self.MoveType == MOVE_RUN then
        return self.RunSpeed
    else
        return self.WalkSpeed
    end
end
```

**Benefits:**
- Clear behavior organization
- Easy to add new states
- State-specific logic isolated
- Predictable state transitions

## Strategy Pattern

Different behaviors are implemented as interchangeable strategies.

### Movement Strategies

NPCs can use different movement strategies:

```lua
-- Ground movement strategy
ENT.UseWalkframes = true
function ENT:GroundMove(pos)
    self.loco:SetDesiredSpeed(self.RunSpeed)
    -- Use pathfinding
end

-- Flying movement strategy
ENT.Flying = true
function ENT:FlyMove(pos)
    local dir = (pos - self:GetPos()):GetNormalized()
    self:SetVelocity(dir * self.FlySpeed)
    -- Direct movement, no pathfinding
end
```

The movement strategy is swapped based on entity type.

### Attack Strategies

Different attack strategies for different situations:

```lua
function ENT:OnMeleeAttack(enemy)
    -- Melee strategy: close range, high damage
    if self:GetRangeTo(enemy) < self.MeleeAttackRange then
        self:DealDamage(enemy, 50)
    end
end

function ENT:OnRangeAttack(enemy)
    -- Range strategy: fire projectile
    local proj = self:FireProjectile("proj_drg_bullet")
    proj:SetVelocity((enemy:GetPos() - self:GetPos()):GetNormalized() * 1000)
end
```

AI chooses strategy based on context:
```lua
if self:GetRangeTo(enemy) < 100 then
    self:MeleeAttack(enemy)  -- Use melee strategy
else
    self:RangeAttack(enemy)  -- Use range strategy
end
```

**Benefits:**
- Swap strategies without changing AI code
- Easy to add new strategies
- Strategies independent of each other
- Clear separation of concerns

## Singleton Pattern

DrGBase uses the singleton pattern for global framework access.

### Global Framework Table

```lua
DrGBase = DrGBase or {}
```

**How it works:**
- `DrGBase` is the singleton instance
- Only one exists per realm (server/client)
- `or {}` prevents recreation if already exists
- Global access from anywhere

**Why singleton:**
- Framework doesn't need multiple instances
- Provides namespace for all framework functions
- Easy access: just use `DrGBase.Function()`
- Prevents naming conflicts

**Usage:**
```lua
-- Anywhere in code
DrGBase.Print("Hello")  -- Access singleton
DrGBase.AddNextbot(ENT)

-- Add to singleton
DrGBase.MyNewFunction = function()
    -- Extends the singleton
end
```

**Benefits:**
- Single point of access
- Global state management
- Namespace pollution prevention
- Simple to use

## Template Method Pattern

Base class defines algorithm structure; subclasses fill in specific steps.

### Think Template

```lua
function ENT:Think()
    -- Template method (in base class)
    self.loco:SetDesiredSpeed(self:GetIdealSpeed())

    -- Update systems
    self:ThinkAI()
    self:ThinkMovement()
    self:ThinkWeapons()

    -- Extension point - subclass customization
    if isfunction(self.CustomThink) then
        self:CustomThink()
    end

    self:NextThink(CurTime())
    return true
end
```

**Structure:**
1. Base defines overall algorithm
2. Calls subclass hooks at specific points
3. Subclass implements hooks, not whole method

### Initialize Template

```lua
function ENT:Initialize()
    -- Template method
    -- Step 1: Setup physics
    self:SetSolid(SOLID_BBOX)

    -- Step 2: Setup model
    self:SetModel(self:GetNextbotModel())

    -- Step 3: Setup systems
    self:InitializeAI()
    self:InitializeMovement()

    -- Step 4: Extension point
    if isfunction(self.CustomInitialize) then
        self:CustomInitialize()
    end

    -- Step 5: Post-init
    if isfunction(self.CustomPostInitialize) then
        self:CustomPostInitialize()
    end
end
```

**Benefits:**
- Framework controls the "when"
- Developer controls the "what"
- Consistent initialization order
- No need to remember to call parent
- Can't forget critical steps

## Decorator Pattern

Add functionality to objects without modifying their structure.

### Metatable Extensions

DrGBase decorates engine types with additional methods:

```lua
-- Get Entity metatable
local ENT = FindMetaTable("Entity")

-- Decorate with new method
function ENT:FindInCone(origin, direction, range, degrees, filter)
    -- New functionality added to ALL entities
    local entities = {}
    for _, ent in ipairs(ents.FindInSphere(origin, range)) do
        if self:InCone(ent, origin, direction, degrees) then
            table.insert(entities, ent)
        end
    end
    return entities
end
```

**How it works:**
- Get metatable for base type (Entity, Player, etc.)
- Add new methods to metatable
- All instances of that type gain the method
- Doesn't modify engine code

**Usage:**
```lua
-- Any entity can now use decorated method
local inCone = self:FindInCone(pos, dir, 1000, 45)
```

**Benefits:**
- Extend engine functionality
- No need to modify base classes
- All instances get new methods
- Transparent to users

**Other decorators in DrGBase:**
```lua
-- Vector decoration
local VEC = FindMetaTable("Vector")
function VEC:DrG_Length2D()
    return math.sqrt(self.x^2 + self.y^2)
end

-- Player decoration
local PLY = FindMetaTable("Player")
function PLY:DrG_IsPossessing()
    return self:GetPossessedNextbot() ~= nil
end
```

## Facade Pattern

Provide a simple interface to complex subsystems.

### Simplified Interfaces

DrGBase wraps complex Source engine APIs:

```lua
-- Simple facade
ENT:MoveToPos(pos)
```

**Hides complexity:**
```lua
-- What MoveToPos actually does (simplified):
function ENT:MoveToPos(pos)
    -- Complex pathfinding setup
    self.Path = Path("Follow")
    self.Path:SetMinLookAheadDistance(300)
    self.Path:SetGoalTolerance(20)
    self.Path:Compute(self, pos)

    -- Complex movement logic
    if self.Path:IsValid() then
        self.loco:FaceTowards(self.Path:GetCurrentGoal().pos)
        self.loco:Approach(self.Path:GetCurrentGoal().pos, 1)
    end
end
```

**More facades:**
```lua
-- Simple damage facade
ENT:TakeDamage(50)
-- Instead of creating CTakeDamageInfo, setting attacker, etc.

-- Simple projectile facade
self:FireProjectile("proj_drg_bullet")
-- Instead of ents.Create, SetPos, SetAngles, SetOwner, SetVelocity, Spawn

-- Simple trace facade
self:TraceLine(Vector(0, 0, 0), Vector(100, 0, 0))
-- Instead of setting up trace data table with all parameters
```

**Benefits:**
- Easy to use for beginners
- Hides implementation complexity
- Consistent API across framework
- Can change implementation without breaking user code

## Module Pattern

Organize code into modules that encapsulate functionality.

### Encapsulation

Modules hide implementation details and expose only public API:

```lua
-- Module file: drgbase/mymodule.lua

-- Private (local) variables and functions
local privateData = {}
local function privateHelper()
    -- Internal implementation
    return privateData
end

-- Public exports to DrGBase table
DrGBase.PublicFunction = function(arg)
    -- Uses private helpers
    local data = privateHelper()
    return data[arg]
end

DrGBase.AnotherPublicFunction = function()
    -- Another public method
end

-- Only DrGBase.* functions are accessible outside
-- privateHelper() cannot be called from other files
```

**Benefits:**
- Information hiding
- Clear public API
- Private implementation can change
- Prevents naming conflicts

**Example - Networking Module:**
```lua
-- drgbase/modules/net.lua

-- Private callback storage
local NET_MESSAGES = {}

-- Public: Register receiver
function net.DrG_Receive(name, callback)
    NET_MESSAGES[name] = callback  -- Stores in private table
end

-- Public: Send message
function net.DrG_Send(name, ...)
    net.Start("DrGBaseNetMessage")
    net.WriteString(name)
    -- Implementation details
end

-- Users only see public functions
-- Internal message storage is hidden
```

## Best Practices

Guidelines for applying design patterns effectively in DrGBase.

### When to Use Inheritance

**Use inheritance when:**
- Creating a new NPC that shares most behavior with existing one
- Extending a base weapon class
- Creating variants of existing entities

```lua
-- Good: Specialized zombie
ENT.Base = "npc_drg_zombie"
ENT.PrintName = "Fast Zombie"
ENT.RunSpeed = 400  -- Override property
```

**Don't use inheritance when:**
- Just sharing utility functions (use helpers instead)
- Need multiple "parent" behaviors (use composition)
- Creating completely different entity (use base directly)

### When to Use Hooks

**Use hooks when:**
- Customizing standard entity behavior
- Responding to events
- Adding logic at specific points in lifecycle

```lua
-- Good: Hook for custom behavior
function ENT:OnTakeDamage(dmg)
    if dmg:GetDamage() > 50 then
        self:Flee()  -- Custom flee behavior
    end
    return true
end
```

**Don't use hooks when:**
- Need to completely replace core functionality (override method instead)
- Want to run code independent of entity lifecycle (use timers)

### When to Use Composition

**Use composition when:**
- Need to mix functionality from multiple sources
- Want to add optional features
- Building modular systems

```lua
-- Good: Compose with modules
function ENT:Initialize()
    -- Add inventory system
    self.Inventory = {}

    -- Add quest system
    self.Quests = QuestManager:Create()

    -- Add dialogue system
    self.Dialogue = DialogueTree:Load("zombie_talk")
end
```

**Composition over inheritance:**
```lua
-- Instead of complex inheritance:
-- npc_base → npc_armed → npc_armed_flying → npc_armed_flying_boss

-- Use composition:
ENT.Base = "drgbase_nextbot"
ENT.HasWeapon = true
ENT.CanFly = true
ENT.IsBoss = true
```

### General Guidelines

1. **Prefer hooks over method overrides** - Easier to maintain
2. **Use facades for complex operations** - Improves usability
3. **Keep modules focused** - Single responsibility
4. **Document patterns used** - Help future developers
5. **Test pattern implementations** - Ensure they work as intended

---

**Previous:** [Client-Server Architecture](./05-client-server.md) | **Next:** [Core Systems](../../systems/README.md)
