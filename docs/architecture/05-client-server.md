# Client-Server Architecture

## Realm Separation

Source engine uses a client-server architecture where game logic is separated into two realms:

**SERVER realm:**
- Runs game simulation
- Authoritative for all game state
- Handles AI, physics, damage
- Only one instance (the server)

**CLIENT realm:**
- Receives state from server
- Renders graphics and UI
- Handles input
- Multiple instances (one per player)

**In Garry's Mod Lua:**
- `SERVER` global is true on server
- `CLIENT` global is true on client
- Code can check realm with `if SERVER then ... end`
- Shared code runs on both realms

This separation ensures:
- Server has authority (prevents cheating)
- Clients only need rendering code
- Network bandwidth is optimized

## Server-Side (SERVER realm)

The server is responsible for all authoritative game simulation.

### Server Responsibilities
- AI logic and decision making
- Pathfinding calculations
- Damage calculations
- Health management
- Spawn control
- Physics simulation authority

### Server-Only Files

Files prefixed with `sv_` only run on the server and are never sent to clients:
- `sv_admin.lua` - Admin-only code
- `sv_database.lua` - Database operations
- Any sensitive or server-only logic

DrGBase modules that are server-only:
- `nodegraph.lua` - AI navigation (pathfinding is server-side)
- Parts of `navmesh.lua` - Navigation mesh queries

## Client-Side (CLIENT realm)

The client handles all presentation and user interaction.

### Client Responsibilities
- Rendering and visual effects
- UI and HUD
- Sound playback (client-side)
- Animations (prediction)
- Input handling
- Camera control

### Client-Only Files

Files prefixed with `cl_` only run on the client:
- `cl_hud.lua` - HUD rendering
- `cl_effects.lua` - Visual effects
- Any rendering or UI code

DrGBase modules that are client-only:
- `spawnmenu.lua` - Spawn menu integration
- `dpanels.lua` - UI panel utilities
- Parts of `render.lua` - Rendering helpers

## Shared Code (Both realms)

Shared code runs on both server and client realms.

### Shared Responsibilities
- Entity configuration
- Constants and enumerations
- Utility functions
- Structure definitions

## Network Communication

Client and server communicate through two primary mechanisms:

### Network Variables (NW2)

```lua
-- Server sets
self:SetNW2Int("Health", 100)

-- Client reads
local health = self:GetNW2Int("Health")
```

**DrGBase Network Variables:**
DrGBase entities use NW2 variables for automatic state synchronization:
- Health, position, angles (automatic by engine)
- Custom entity state
- Possession status
- AI state (when needed for client prediction)

**Advantages:**
- Automatic synchronization
- Bandwidth efficient (only changed values sent)
- Simple to use

### Network Messages

```lua
-- Server sends
net.Start("DrG_ChatMessage")
    net.WriteString("Message")
net.Send(ply)

-- Client receives
net.Receive("DrG_ChatMessage", function()
    local msg = net.ReadString()
end)
```

**DrGBase Network Messages:**
- `DrGBaseChatPrint` - Server to client chat messages
- `DrGBaseNetMessage` - General purpose message system
- `DrGBaseNetCallbackReq` - Callback request
- `DrGBaseNetCallbackRes` - Callback response

**Usage Pattern:**
```lua
-- Server → Client
if SERVER then
    net.Start("MyMessage")
    net.WriteString("Hello")
    net.Send(player)
end

-- Client receives
if CLIENT then
    net.Receive("MyMessage", function()
        local msg = net.ReadString()
        print(msg)
    end)
end
```

### DrGBase Network System

```lua
-- Custom network functions
net.DrG_Send(name, ...)
net.DrG_Receive(name, callback)
```

## State Synchronization

Keeping client and server state synchronized is critical for multiplayer gameplay.

### Automatic Sync

**Network Variables (NW2):**
- Set on server: `self:SetNW2Int("Health", 100)`
- Automatically sent to all clients
- Read on client: `self:GetNW2Int("Health")`
- Updated automatically when value changes

**Entity Properties:**
- Position, angles, velocity sync automatically
- Model, skin, bodygroups sync automatically
- No manual code needed

### Manual Sync

**Network Messages:**
For events that need manual control:
```lua
-- Server sends event
net.Start("NPCDied")
net.WriteEntity(npc)
net.Broadcast()

-- Client receives and handles
net.Receive("NPCDied", function()
    local npc = net.ReadEntity()
    -- Play death effect
end)
```

**When to use:**
- One-time events (death, spawn)
- Player commands
- Custom client effects
- Anything not suitable for NW2 vars

### Prediction

**Client-Side Prediction:**
- Animations predicted client-side
- Movement interpolated between updates
- Reduces perceived lag
- Server is still authoritative

DrGBase uses prediction for:
- Animation playback
- Movement smoothing
- Visual effects timing

## Performance Considerations

Network performance is crucial for smooth multiplayer gameplay.

### Minimize Network Traffic

**Best practices:**

1. **Use NW2 variables instead of frequent net messages**
   ```lua
   -- Bad: Send every tick
   timer.Create("SendHealth", 0, 0, function()
       net.Start("UpdateHealth")
       net.WriteInt(health, 32)
       net.Broadcast()
   end)

   -- Good: Use NW2 (auto-sends only on change)
   self:SetNW2Int("Health", health)
   ```

2. **Send only necessary data**
   ```lua
   -- Bad: Send entire table
   net.WriteTable(hugeTable)

   -- Good: Send only what changed
   net.WriteInt(changedValue, 32)
   ```

3. **Batch updates when possible**
   ```lua
   -- Send multiple values in one message
   net.Start("StateUpdate")
   net.WriteInt(health, 32)
   net.WriteInt(armor, 32)
   net.WriteVector(velocity)
   net.Broadcast()
   ```

4. **Use appropriate data types**
   - `WriteUInt(val, 8)` for 0-255 (1 byte)
   - `WriteUInt(val, 16)` for 0-65535 (2 bytes)
   - Don't use `WriteInt(val, 32)` when smaller types work

### Update Frequency

**Every Tick (Server → All Clients):**
- Entity positions (automatic)
- NW2 variable changes (automatic, only if changed)
- Critical state updates

**Less Frequent:**
- Health updates (only when damaged)
- AI state (only when state changes)
- Player stats (throttled updates)

**Event-Based Only:**
- Death notifications
- Spawn events
- One-time effects

## Debugging

Debugging client-server issues requires understanding which realm code runs on.

### Console Commands
```
sv_showimpacts 1
cl_showpos 1
net_graph 1
```

### Common Issues

**1. Variable is nil on client**
- **Cause:** NW2 variable not set on server, or entity not valid yet
- **Fix:** Check `IsValid(ent)` before accessing, ensure server sets value

**2. Network message not received**
- **Cause:** Network string not registered, or receiver not added
- **Fix:** Ensure `util.AddNetworkString()` on server, `net.Receive()` on correct realm

**3. Desynced state**
- **Cause:** Client modifying authoritative state, or prediction issues
- **Fix:** Only modify state on server, clients should be read-only

**4. "Tried to use unregistered net message"**
- **Cause:** Network string not registered before use
- **Fix:** Add `util.AddNetworkString()` in `autorun/`, not entity `Initialize()`

**5. Client code running on server (or vice versa)**
- **Cause:** Missing `if SERVER` or `if CLIENT` check
- **Fix:** Properly guard realm-specific code:
  ```lua
  if SERVER then
      -- Server-only code
  end

  if CLIENT then
      -- Client-only code
  end
  ```

**6. High network usage**
- **Cause:** Sending too much data or too frequently
- **Fix:** Use NW2 variables, throttle updates, send only changes

**Debugging Tools:**
```
// Console commands
net_graph 1           // Show network stats
cl_showpos 1          // Show position and velocity
developer 1           // Show developer messages
net_start            // Log network messages
```

---

**Previous:** [Module System](./04-module-system.md) | **Next:** [Design Patterns](./06-design-patterns.md)
