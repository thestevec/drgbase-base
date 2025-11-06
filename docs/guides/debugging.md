# Testing & Debugging

Learn how to debug and troubleshoot DrGBase nextbots using the built-in tools and techniques.

## Table of Contents
1. [Introduction to Debugging Tools](#introduction-to-debugging-tools)
2. [Debug Console Variables (ConVars)](#debug-console-variables-convars)
3. [Console Commands](#console-commands)
4. [Common Debugging Techniques](#common-debugging-techniques)
5. [Performance Debugging](#performance-debugging)
6. [Common Error Messages](#common-error-messages)
7. [Visual Debugging Overlays](#visual-debugging-overlays)
8. [Best Practices](#best-practices)

## Introduction to Debugging Tools

DrGBase provides several debugging tools to help you identify and fix issues with your nextbots:

- **Debug ConVars**: Console variables that enable visualization and logging
- **Visual Overlays**: In-game visual indicators for traces, trajectories, and relationships
- **Print Functions**: Console logging functions for tracking NPC state
- **Developer Mode**: Enhanced console output for detailed debugging

### Setting Up for Debugging

Enable developer mode in the console for detailed output:

```
developer 1
```

This enables additional console messages that help track what your NPCs are doing. Set back to `0` to disable.

## Debug Console Variables (ConVars)

DrGBase includes several debug convars that visualize NPC behavior in real-time.

### drgbase_debug_traces

Visualizes trace lines used for line-of-sight checks and detection.

**Syntax:**
```
drgbase_debug_traces <duration>
```

**Parameters:**
- `0` - Disabled (default)
- `>0` - Duration in seconds to display each trace line

**Usage:**
```
drgbase_debug_traces 5
```

**What it shows:**
- **Green lines**: Successful traces (no obstruction)
- **Red lines**: Failed traces (hit something)
- **White lines**: Extension to the end position

**Use cases:**
- Debugging why NPCs can't see targets
- Troubleshooting line-of-sight issues
- Understanding obstacle detection
- Verifying collision detection

**Example scenario:**
```lua
-- In your NPC code
function ENT:CustomThink()
    -- Enable debug traces first: drgbase_debug_traces 3
    local tr = util.DrG_TraceLine({
        start = self:GetPos() + Vector(0, 0, 50),
        endpos = self:GetEnemy():GetPos() + Vector(0, 0, 50),
        filter = self
    })
    -- Green line = can see enemy
    -- Red line = something blocking view
end
```

### drgbase_debug_relationships

Prints detailed relationship and faction information to console when NPCs evaluate targets.

**Syntax:**
```
drgbase_debug_relationships <0|1>
```

**Parameters:**
- `0` - Disabled (default)
- `1` - Enabled

**Usage:**
```
drgbase_debug_relationships 1
```

**What it shows:**
Console output like:
```
[DrGBase] Entity(1)[npc_my_custom_zombie]: 'Player[1][PlayerName]' D_NU => D_HT.
```

This shows:
- The NPC making the relationship change
- The target entity
- Old relationship (D_NU = neutral)
- New relationship (D_HT = hate/enemy)

**Relationship codes:**
- `D_LI` - Like/Ally
- `D_HT` - Hate/Enemy
- `D_FR` - Fear/Afraid
- `D_NU` - Neutral
- `D_ER` - Error

**Use cases:**
- Debugging faction systems
- Understanding why NPCs target or ignore certain entities
- Troubleshooting relationship priorities
- Verifying CustomRelationship() logic

### drgbase_debug_trajectories

Visualizes projectile trajectory predictions and physics calculations.

**Syntax:**
```
drgbase_debug_trajectories <duration>
```

**Parameters:**
- `0` - Disabled (default)
- `>0` - Duration in seconds to display trajectories

**Usage:**
```
drgbase_debug_trajectories 5
```

**What it shows:**
- **Green lines**: Initial velocity vector
- **White lines**: Predicted trajectory path
- **Red lines**: Trajectory after predicted duration
- **Vertical line**: Height indicator for ballistic trajectories

**Use cases:**
- Debugging projectile weapons
- Troubleshooting aiming calculations
- Verifying ballistic physics
- Understanding DrG_ThrowAt() and DrG_AimAt() behavior

**Example:**
```lua
-- Enable trajectory debug first: drgbase_debug_trajectories 3
function ENT:ThrowProjectileAtTarget(target)
    local proj = ents.Create("prop_physics")
    proj:SetPos(self:GetPos() + Vector(0, 0, 50))
    proj:SetModel("models/props_junk/watermelon01.mdl")
    proj:Spawn()

    local phys = proj:GetPhysicsObject()
    if IsValid(phys) then
        -- This will draw the predicted trajectory
        phys:DrG_ThrowAt(target:GetPos(), {magnitude = 500})
    end
end
```

### drgbase_debug_animations

Prints animation state changes and sequence information to console.

**Syntax:**
```
drgbase_debug_animations <0|1>
```

**Parameters:**
- `0` - Disabled (default)
- `1` - Enabled (also freezes animation when NPC AI is disabled)

**Usage:**
```
drgbase_debug_animations 1
```

**What it shows:**
- Animation sequence changes
- Activity transitions
- Gesture layers being played
- Pose parameter updates

**Use cases:**
- Debugging animation issues
- Understanding why wrong animations play
- Troubleshooting animation transitions
- Verifying PlaySequenceAndWait() behavior

**Note:** When enabled, NPCs with disabled AI (via AI Disable tool) will freeze their animations, making it easier to inspect specific frames.

### Nodegraph Debug ConVars

These convars help debug the navigation system.

#### drgbase_nodegraph_display

Displays the navigation nodegraph as wireframe boxes in the world.

**Syntax:**
```
developer 1
drgbase_nodegraph_display <0|1>
```

**Parameters:**
- `0` - Disabled (default)
- `1` - Enabled (requires `developer 1`)

**What it shows:**
Colored boxes representing navigation nodes:
- **Green**: Ground nodes
- **Cyan**: Air nodes
- **Purple**: Climb nodes
- **Blue**: Water nodes

#### drgbase_nodegraph_distance

Maximum distance from crosshair to display nodes.

**Syntax:**
```
drgbase_nodegraph_distance <units>
```

**Default:** `1500`

#### drgbase_nodegraph_type

Which type of nodes to display.

**Syntax:**
```
drgbase_nodegraph_type <type>
```

**Values:**
- `0` - Ground nodes (green)
- `1` - Air nodes (cyan)
- `2` - Climb nodes (purple) - **Default**
- `3` - Water nodes (blue)

#### drgbase_nodegraph_transparent

Renders nodegraph with transparency.

**Syntax:**
```
drgbase_nodegraph_transparent <0|1>
```

**Example nodegraph debugging session:**
```
developer 1
drgbase_nodegraph_display 1
drgbase_nodegraph_type 0
drgbase_nodegraph_distance 3000
```

## Console Commands

### Built-in Garry's Mod Commands

These standard GMod commands are useful for debugging:

#### developer

Enables detailed console output.

```
developer 1  // Enable
developer 0  // Disable
```

When enabled, shows:
- Entity spawn messages
- Network string registrations
- File includes
- DrGBase initialization messages

#### nav_generate

Generates a navigation mesh for pathfinding (singleplayer only).

```
nav_generate
```

**Important:** This command only works in singleplayer. For multiplayer servers, you must:
1. Load the map in singleplayer
2. Run `nav_generate`
3. Wait for completion (can take several minutes)
4. The .nav file is saved automatically
5. Upload the .nav file to your server

Without a navmesh, NPCs will have poor pathfinding performance.

#### nav_edit

Toggles navigation mesh editing mode (singleplayer only).

```
nav_edit 1  // Enable editing
nav_edit 0  // Disable editing
```

Allows you to manually edit navigation areas.

#### ent_fire

Execute commands on specific entities.

```
ent_fire <targetname> <command> [value]
```

**Example:**
```
ent_fire !picker setrelationship "player D_LI 99"
```

Point at an NPC and this sets all players as allies.

#### ai_show_connect

Visualizes NPC connections and relationships.

```
ai_show_connect 1  // Enable
ai_show_connect 0  // Disable
```

Shows lines between NPCs and their targets/allies.

#### ai_debug_los

Shows line-of-sight debug information.

```
ai_debug_los 1  // Enable
ai_debug_los 0  // Disable
```

### Useful Lua Console Commands

You can execute Lua code directly in console with `lua_run` (singleplayer) or `lua_run_cl` (client-side):

#### Spawn Test NPC
```lua
lua_run Entity(1):Give("weapon_drgbase_possessor")
```

#### Get All DrGBase NPCs
```lua
lua_run PrintTable(DrGBase.GetNextbots())
```

#### Force NPC to Target Player
```lua
lua_run Entity(2):SetEntityRelationship(Entity(1), D_HT)
```

#### Check NPC Relationships
```lua
lua_run print(Entity(2):GetRelationship(Entity(1)))
```

#### Teleport NPC to Crosshair
```lua
lua_run Entity(2):SetPos(Entity(1):GetEyeTrace().HitPos)
```

## Common Debugging Techniques

### Technique 1: Print Debugging

Use DrGBase's print functions to track NPC state:

```lua
function ENT:CustomThink()
    -- Basic print
    DrGBase.Print("Current state: " .. tostring(self:GetState()))

    -- Info print (green header)
    DrGBase.Info("Found enemy: " .. tostring(self:GetEnemy()))

    -- Error print (red text)
    DrGBase.Error("Invalid enemy target!")

    -- Print to chat (visible to players)
    DrGBase.Print("NPC spawned!", {chat = true})
end
```

**Print function options:**

```lua
DrGBase.Print(message, {
    chat = false,        -- Print to chat instead of console
    color = Color(255, 255, 255),  -- Text color
    title = Color(0, 255, 0),      -- Title/header color
    player = ply         -- Specific player (for chat messages)
})
```

### Technique 2: Visual Debugging with Debug Overlays

Use Source engine debug overlays for in-game visualization:

```lua
function ENT:CustomThink()
    -- Draw a line
    debugoverlay.Line(
        self:GetPos(),
        self:GetEnemy():GetPos(),
        5,  -- Duration in seconds
        Color(255, 0, 0),
        false  -- Don't ignore Z
    )

    -- Draw a box
    debugoverlay.Box(
        self:GetPos(),
        Vector(-10, -10, 0),  -- Mins
        Vector(10, 10, 50),    -- Maxs
        5,
        Color(0, 255, 0, 50)
    )

    -- Draw text
    debugoverlay.Text(
        self:GetPos() + Vector(0, 0, 80),
        "HP: " .. self:Health(),
        5
    )

    -- Draw sphere
    debugoverlay.Sphere(
        self:GetPos(),
        100,  -- Radius
        5,
        Color(0, 0, 255, 50)
    )
end
```

### Technique 3: State Tracking

Track NPC states to understand behavior:

```lua
function ENT:CustomInitialize()
    self._lastState = nil
end

function ENT:CustomThink()
    local state = self:GetState()

    -- Print when state changes
    if state ~= self._lastState then
        DrGBase.Print("State changed: " .. tostring(self._lastState) .. " => " .. tostring(state))
        self._lastState = state
    end
end
```

### Technique 4: Conditional Breakpoints

Add conditional debugging that only triggers in specific scenarios:

```lua
function ENT:OnTakeDamage(dmg)
    -- Only debug when taking unusual damage
    if dmg:GetDamage() > 100 then
        DrGBase.Error("Taking massive damage: " .. dmg:GetDamage())
        DrGBase.Print("Attacker: " .. tostring(dmg:GetAttacker()))
        DrGBase.Print("Damage type: " .. dmg:GetDamageType())

        -- Visual indicator
        debugoverlay.Sphere(self:GetPos(), 200, 5, Color(255, 0, 0))
    end
end
```

### Technique 5: Enemy Tracking

Debug targeting and enemy selection:

```lua
function ENT:CustomThink()
    local enemy = self:GetEnemy()

    if IsValid(enemy) then
        -- Show range
        local dist = self:GetRangeTo(enemy)
        debugoverlay.Text(
            self:GetPos() + Vector(0, 0, 100),
            string.format("Enemy: %s (%.0f units)", tostring(enemy), dist),
            0.1
        )

        -- Show line to enemy
        local canSee = self:CanSee(enemy)
        debugoverlay.Line(
            self:GetPos() + Vector(0, 0, 50),
            enemy:GetPos() + Vector(0, 0, 50),
            0.1,
            canSee and Color(0, 255, 0) or Color(255, 0, 0)
        )
    else
        debugoverlay.Text(
            self:GetPos() + Vector(0, 0, 100),
            "No enemy",
            0.1
        )
    end
end
```

### Technique 6: Hook Debugging

Override hooks to see when they're called:

```lua
function ENT:OnSpotted(ent)
    DrGBase.Info("Spotted entity: " .. tostring(ent))
    debugoverlay.Line(self:GetPos(), ent:GetPos(), 3, Color(255, 255, 0))
end

function ENT:OnLostSight(ent)
    DrGBase.Info("Lost sight of: " .. tostring(ent))
end

function ENT:OnContact(ent)
    DrGBase.Print("Touched: " .. tostring(ent))
    debugoverlay.Sphere(self:GetPos(), 50, 2, Color(255, 0, 255))
end

function ENT:OnNewEnemy(enemy, old)
    DrGBase.Error("NEW ENEMY: " .. tostring(enemy) .. " (was: " .. tostring(old) .. ")")
end
```

## Performance Debugging

### Identifying Performance Issues

Common signs of performance problems:
- Server lag or stuttering
- FPS drops when NPCs spawn
- Delayed NPC reactions
- Pathfinding taking too long

### Performance Debug ConVars

```
developer 1              // See timing information
ai_show_think_tolerance 1  // Show NPCs thinking too long
```

### Check NPC Count

Too many NPCs can cause lag:

```lua
lua_run print("DrGBase NPCs: " .. #DrGBase.GetNextbots())
lua_run print("All NPCs: " .. #ents.FindByClass("npc_*"))
```

**Recommended limits:**
- **Singleplayer:** 50-100 NPCs
- **Multiplayer (dedicated server):** 20-50 NPCs
- **Multiplayer (listen server):** 10-30 NPCs

### Profiling Think Functions

Measure how long your Think functions take:

```lua
function ENT:CustomThink()
    local startTime = SysTime()

    -- Your think logic here
    self:DoComplexCalculations()

    local endTime = SysTime()
    local duration = (endTime - startTime) * 1000  -- Convert to milliseconds

    if duration > 1 then  -- Warn if think takes > 1ms
        DrGBase.Error(string.format("Think took %.2fms!", duration))
    end
end
```

### Performance Optimization Checklist

1. **Reduce sight range:**
```lua
ENT.SightRange = 2000  -- Don't use excessive values like 10000
```

2. **Increase think delay:**
```lua
ENT.ThinkDelay = 0.1  -- Default is 0.1, can increase to 0.2-0.5 for simple NPCs
```

3. **Limit pathfinding:**
```
drgbase_compute_delay 0.2  // Increase path computation delay
```

4. **Use navmesh:**
```
nav_generate  // In singleplayer, then upload .nav file to server
```

5. **Reduce expensive operations:**
```lua
-- BAD: Every frame
function ENT:CustomThink()
    local enemies = ents.FindInSphere(self:GetPos(), 5000)
end

-- GOOD: Cached, only update occasionally
function ENT:CustomInitialize()
    self._nearbyEnemies = {}
    self._nextEnemyScan = 0
end

function ENT:CustomThink()
    if CurTime() > self._nextEnemyScan then
        self._nearbyEnemies = ents.FindInSphere(self:GetPos(), 2000)
        self._nextEnemyScan = CurTime() + 1  -- Scan every 1 second
    end
end
```

6. **Disable unnecessary features:**
```lua
ENT.UseInertia = false      -- If you don't need smooth acceleration
ENT.CollisionBounds = false  -- If default bounds work fine
```

### Server Performance Commands

Check server performance:

```
net_graph 1  // Show network performance
stats        // Show server statistics
```

## Common Error Messages

### Lua Errors

#### Error: `attempt to call method 'MethodName' (a nil value)`

**Cause:** Trying to call a function that doesn't exist.

**Common reasons:**
1. Typo in function name
2. Using client-side function on server (or vice versa)
3. DrGBase not fully loaded

**Fix:**
```lua
-- BAD: Typo
self:GetEnmy()

-- GOOD:
self:GetEnemy()

-- BAD: Client function on server
if SERVER then
    self:GetLocalPlayerRelationship()  -- CLIENT ONLY!
end

-- GOOD: Check realm
if CLIENT then
    self:GetLocalPlayerRelationship()
end
```

#### Error: `'}' expected near 'end'`

**Cause:** Mismatched braces or end keywords.

**Fix:** Count your pairs:
- Every `function` needs an `end`
- Every `if` needs an `end`
- Every `{` needs a `}`
- Every `then` is part of an `if`

```lua
-- BAD: Missing end
function ENT:CustomThink()
    if self:GetEnemy() then
        self:Attack()
    -- Missing end here!
}  -- Wrong bracket type

-- GOOD:
function ENT:CustomThink()
    if self:GetEnemy() then
        self:Attack()
    end  -- Closes if
end  -- Closes function
```

#### Error: `attempt to index field 'PropertyName' (a nil value)`

**Cause:** Using a property before it's defined.

**Fix:** Define all ENT properties at the top of the file:

```lua
-- BAD:
function ENT:CustomInitialize()
    self:SetHealth(self.MaxHealth)  -- MaxHealth not defined yet!
end

ENT.MaxHealth = 100  -- Defined too late

-- GOOD:
ENT.MaxHealth = 100  -- Define first

function ENT:CustomInitialize()
    self:SetHealth(self.MaxHealth)  -- Now it exists
end
```

#### Error: `bad argument #1 to 'FunctionName'`

**Cause:** Passing wrong type of argument to a function.

**Common examples:**

```lua
-- BAD: Passing nil
self:SetEnemy(nil)  -- Can't set enemy to nil directly

-- GOOD:
self:ResetEnemy()  -- Use the proper function

-- BAD: Wrong type
self:PlaySequenceAndWait(true)  -- Expects string or number

-- GOOD:
self:PlaySequenceAndWait("idle01")  -- String sequence name
self:PlaySequenceAndWait(ACT_IDLE)  -- Or activity number
```

#### Error: `BaseClass is nil`

**Cause:** ENT.Base is incorrect or DrGBase not loaded.

**Fix:**
```lua
ENT.Base = "drgbase_nextbot"  -- Must be exactly this

-- Also verify DrGBase is installed and loaded:
-- Check console for: [DrGBase] Hi! :)
```

### Animation Errors

#### NPC plays wrong animations

**Diagnosis steps:**

1. **Enable developer mode:**
```
developer 1
```

2. **Check what activities are available:**
```lua
lua_run for k, v in pairs(Entity(2):GetSequenceList()) do print(k, v, Entity(2):GetSequenceActivityName(k)) end
```

3. **Verify animation properties:**
```lua
-- Make sure you're using activities (ACT_*), not sequence names
ENT.WalkAnimation = ACT_WALK  -- GOOD
ENT.WalkAnimation = "walk_all"  -- BAD (this is a sequence name)
```

4. **Test with known-good models:**
Use `models/vortigaunt.mdl` or `models/zombie/classic.mdl` which have standard animations.

#### NPC animation frozen

**Possible causes:**
1. `drgbase_debug_animations` is enabled with AI disabled
2. PlaySequenceAndWait() stuck in a loop
3. Animation rate is 0

**Fix:**
```
drgbase_debug_animations 0  // Disable debug mode
```

```lua
-- Check animation rate
ENT.WalkAnimRate = 1  // Make sure it's not 0
```

### Relationship Errors

#### NPC not attacking players

**Debug steps:**

1. **Check relationship:**
```
drgbase_debug_relationships 1
```

2. **Verify in console:**
```lua
lua_run print(Entity(2):GetRelationship(Entity(1)))  -- Should print D_HT
```

3. **Common causes:**
- Same faction: `self:JoinFaction(FACTION_PLAYERS)` makes NPCs friendly to players
- Custom relationship overrides: Check `CustomRelationship` hook
- Same team: `self:SetTeam(1)` and player on team 1

**Fix:**
```lua
function ENT:CustomInitialize()
    -- Make hostile to players
    self:SetPlayersRelationship(D_HT)

    -- Or set default to hostile
    self:SetDefaultRelationship(D_HT)
end
```

#### NPC ignoring certain entities

**Enable debug:**
```
drgbase_debug_relationships 1
drgbase_debug_traces 5
```

**Check if entity is being ignored:**
```lua
lua_run print(Entity(2):IsIgnored(Entity(3)))
```

**Common causes:**
- Entity has FL_NOTARGET flag
- Entity is dead (health <= 0)
- `ShouldIgnore` hook returns true
- `ai_ignoreplayers` is enabled

### Pathfinding Errors

#### NPC not moving/stuck

**Check navmesh:**
```
nav_edit 1  // Must be in singleplayer
```

If no blue grid appears, you need to generate a navmesh:
```
nav_generate
```

**Check if NPC is trying to move:**
```lua
lua_run print("IsMoving:", Entity(2):IsMoving())
lua_run print("HasPath:", Entity(2):HasPath())
```

**Common causes:**
1. No navmesh
2. Collision bounds too large
3. MovementSpeed is 0

**Fix:**
```lua
ENT.MovementSpeed = 150  -- Make sure it's not 0

-- Check collision bounds
ENT.CollisionBounds = Vector(13, 13, 72)  -- Should fit through doors
```

### Sound Errors

#### Error: `bad argument #1 to 'EmitSound'`

**Cause:** Invalid sound path.

**Test sound:**
```lua
lua_run Entity(1):EmitSound("NPC_Dog.Angry")  // Known-good sound
```

**Find valid sounds:**
1. Browse `garrysmod/sound/` folder
2. Use absolute path from sound folder
3. Don't include "sound/" in the path

```lua
-- BAD:
self:EmitSound("sound/npc/zombie/zombie_voice_idle1.wav")

-- GOOD:
self:EmitSound("npc/zombie/zombie_voice_idle1.wav")

-- Also works (table format):
self:EmitSound("NPC_Zombie.Idle")
```

## Visual Debugging Overlays

### Built-in Debug Overlays

DrGBase trace functions automatically use debug overlays when enabled:

```lua
-- Enable trace visualization
-- drgbase_debug_traces 5

-- These will show visual lines:
util.DrG_TraceLine({
    start = self:GetPos(),
    endpos = targetPos,
    filter = self
})

util.DrG_TraceHull({
    start = self:GetPos(),
    endpos = targetPos,
    mins = Vector(-10, -10, 0),
    maxs = Vector(10, 10, 50),
    filter = self
})
```

### Custom Debug Overlays

Create your own debug visualizations:

```lua
function ENT:CustomThink()
    -- Draw sight range
    debugoverlay.Sphere(
        self:GetPos(),
        self.SightRange,
        0.1,
        Color(0, 255, 0, 10),
        false
    )

    -- Draw melee attack range
    debugoverlay.Sphere(
        self:GetPos(),
        self.MeleeAttackRange,
        0.1,
        Color(255, 0, 0, 20),
        false
    )

    -- Draw state text
    debugoverlay.Text(
        self:GetPos() + Vector(0, 0, 80),
        "State: " .. tostring(self:GetState()) .. "\nHP: " .. self:Health(),
        0.1,
        false
    )

    -- Draw path (if moving)
    if self:HasPath() then
        local path = self:GetPath()
        for i = 1, path:GetLength() - 1 do
            local seg1 = path:GetSegment(i)
            local seg2 = path:GetSegment(i + 1)
            if seg1 and seg2 then
                debugoverlay.Line(seg1.pos, seg2.pos, 0.1, Color(255, 255, 0))
            end
        end
    end
end
```

### Trajectory Visualization

Use the built-in trajectory overlay:

```lua
-- Enable trajectory debug
-- drgbase_debug_trajectories 5

function ENT:ThrowProjectile()
    local proj = ents.Create("prop_physics")
    proj:SetPos(self:GetPos() + self:GetForward() * 50)
    proj:SetModel("models/props_junk/watermelon01.mdl")
    proj:Spawn()

    local phys = proj:GetPhysicsObject()
    if IsValid(phys) then
        -- This automatically shows trajectory when debug is enabled
        phys:DrG_ThrowAt(self:GetEnemy():GetPos(), {
            magnitude = 800,
            recursive = true
        })
    end
end
```

## Best Practices

### 1. Use Debug Convars During Development

Enable debug features while developing:

```
developer 1
drgbase_debug_traces 3
drgbase_debug_relationships 1
```

Disable before release for performance:

```
developer 0
drgbase_debug_traces 0
drgbase_debug_relationships 0
```

### 2. Add Debug Modes to Your NPCs

Include a debug flag in your NPC:

```lua
ENT.DebugMode = false  -- Set to true during development

function ENT:CustomThink()
    if self.DebugMode then
        -- Debug visualization
        debugoverlay.Text(
            self:GetPos() + Vector(0, 0, 100),
            string.format("State: %s\nEnemy: %s\nHP: %d",
                tostring(self:GetState()),
                tostring(self:GetEnemy()),
                self:Health()
            ),
            0.1
        )

        -- Draw sight range
        debugoverlay.Sphere(self:GetPos(), self.SightRange, 0.1, Color(0, 255, 0, 5))
    end
end
```

### 3. Use Meaningful Print Messages

Make debugging easier with descriptive messages:

```lua
-- BAD:
DrGBase.Print("1")

-- GOOD:
DrGBase.Print("CustomThink: No valid enemy found, entering idle state")
```

### 4. Check for Nil Values

Always validate entities before using them:

```lua
-- BAD:
self:SetEnemy(self:GetClosestEnemy())
self:GetEnemy():SetHealth(0)  -- Can error if no enemy!

-- GOOD:
local enemy = self:GetClosestEnemy()
if IsValid(enemy) then
    self:SetEnemy(enemy)
    enemy:SetHealth(0)
else
    DrGBase.Print("Warning: No valid enemy found")
end
```

### 5. Use Try-Catch for Risky Code

Protect against errors in production:

```lua
function ENT:CustomThink()
    local success, err = pcall(function()
        -- Risky code here
        self:DoComplexOperation()
    end)

    if not success then
        DrGBase.Error("CustomThink error: " .. tostring(err))
    end
end
```

### 6. Log Important State Changes

Track when critical events occur:

```lua
function ENT:OnTakeDamage(dmg)
    if self:Health() < self:GetMaxHealth() * 0.25 then
        DrGBase.Print(string.format("%s entering low health state (HP: %d/%d)",
            tostring(self),
            self:Health(),
            self:GetMaxHealth()
        ))
    end
end

function ENT:OnNewEnemy(enemy, old)
    DrGBase.Info(string.format("%s switching target: %s => %s",
        tostring(self),
        tostring(old),
        tostring(enemy)
    ))
end
```

### 7. Document Weird Behaviors

Add comments explaining non-obvious code:

```lua
function ENT:CustomThink()
    -- HACK: Wait 1 second before attacking to avoid animation overlap bug
    if self._justSpawned and CurTime() < self._spawnTime + 1 then
        return
    end

    -- TODO: This is a temporary workaround for pathfinding issue #42
    if not self:HasPath() and IsValid(self:GetEnemy()) then
        self:ChaseEnemy()  -- Force path recalculation
    end
end
```

### 8. Test in Both Singleplayer and Multiplayer

Some issues only appear in specific environments:

- **Singleplayer:** Test basic functionality
- **Listen server:** Test as host
- **Dedicated server:** Test with real network latency
- **Multiple players:** Test with 2+ players

### 9. Profile Before Optimizing

Measure performance before and after optimizations:

```lua
-- Before optimization
function ENT:CustomThink()
    local startTime = SysTime()

    -- Original code
    for i = 1, 1000 do
        self:ExpensiveOperation()
    end

    DrGBase.Print(string.format("Original: %.2fms", (SysTime() - startTime) * 1000))
end

-- After optimization
function ENT:CustomThink()
    local startTime = SysTime()

    -- Optimized code
    self:CachedExpensiveOperation()

    DrGBase.Print(string.format("Optimized: %.2fms", (SysTime() - startTime) * 1000))
end
```

### 10. Keep a Debug Command List

Create a text file with useful debug commands:

```
// debug_commands.txt

// Enable all debugging
developer 1
drgbase_debug_traces 5
drgbase_debug_relationships 1
drgbase_debug_animations 1
drgbase_nodegraph_display 1

// Spawn test NPC
ent_create npc_my_custom_npc

// Give possession tool
lua_run Entity(1):Give("weapon_drgbase_possessor")

// Set NPC as enemy
lua_run Entity(2):SetEntityRelationship(Entity(1), D_HT)

// Teleport to crosshair
lua_run Entity(2):SetPos(Entity(1):GetEyeTrace().HitPos)

// Performance check
lua_run print("NPCs:", #DrGBase.GetNextbots())
```

### 11. Use Version Control

Commit working versions before major changes:

```bash
git add .
git commit -m "Working version before pathfinding rewrite"
```

This lets you easily revert if debugging reveals your changes broke something.

### 12. Ask for Help with Logs

When asking for help, include:
1. Full error message from console
2. Relevant code sections
3. Steps to reproduce
4. What you've already tried
5. DrGBase version and Garry's Mod version

## Debugging Checklist

When your NPC isn't working:

- [ ] Check console for red error messages
- [ ] Enable `developer 1`
- [ ] Verify `ENT.Base = "drgbase_nextbot"`
- [ ] Confirm DrGBase is installed (look for "[DrGBase] Hi! :)" on load)
- [ ] Check all functions have matching `end` keywords
- [ ] Verify animation activities exist for the model
- [ ] Enable `drgbase_debug_relationships 1` for targeting issues
- [ ] Enable `drgbase_debug_traces 5` for line-of-sight issues
- [ ] Check if navmesh exists (`nav_edit 1` in singleplayer)
- [ ] Verify ENT properties are defined before functions
- [ ] Test with a known-good model like `models/vortigaunt.mdl`
- [ ] Check realm (SERVER vs CLIENT) for functions
- [ ] Verify IsValid() checks for all entities
- [ ] Look for nil values in function arguments

---

**Previous:** [Animations Guide](./animations.md) | **Next:** [Optimization Guide](./optimization.md)
