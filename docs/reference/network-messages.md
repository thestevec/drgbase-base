# Network Messages

Network message reference for all DrGBase network strings.

---

## Overview

DrGBase uses Garry's Mod's networking system to synchronize data between server and client. All network strings are automatically registered by the framework.

**Note:** When using DrGBase's enhanced `net` module, you don't need to manually call `util.AddNetworkString()` - it's handled automatically.

---

## Complete Network Message List

### Core System Messages

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseChatPrint` | Server → Client | Send chat messages to specific players | autorun/drgbase.lua |

### Player Systems

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseSelectEntity` | Server → Client | Notify player of entity selection | meta/player.lua |
| `DrGBaseDeselectEntity` | Server → Client | Notify player of entity deselection | meta/player.lua |
| `DrGBasePlayerLuminosity` | Server → Client | Update player luminosity for NPC vision | meta/player.lua |

### NPC Awareness & Relationships

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseNextbotPlayerAwareness` | Server → Client | Sync NPC awareness of player | nextbot/awareness.lua |
| `DrGBaseNextbotPlayerRelationship` | Server → Client | Sync NPC relationship with player | nextbot/relationships.lua |

### Possession System

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseNextbotCanPossess` | Server → Client | Notify player can possess NPC | possession.lua |
| `DrGBaseNextbotCantPossess` | Server → Client | Notify player cannot possess NPC | possession.lua |
| `DrGBasePossessionCycleViewPresets` | Client → Server | Cycle through possession camera presets | nextbot/possession.lua |

### Enhanced Networking

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseNetMessage` | Bidirectional | Generic network message wrapper | modules/net.lua |
| `DrGBaseNetCallbackReq` | Bidirectional | Network callback request | modules/net.lua |
| `DrGBaseNetCallbackRes` | Bidirectional | Network callback response | modules/net.lua |

### Navigation & Nodegraph

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseNodegraph` | Server → Client | Send nodegraph data for visualization | nodegraph.lua |

### Projectiles & Effects

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseFlashGrenade` | Server → Client | Flashbang screen effect | proj_drg_flashbang.lua |

### Tools

| Message Name | Direction | Purpose | File |
|--------------|-----------|---------|------|
| `DrGBaseFactionTool` | Server → Client | Update faction tool UI | stools/drgbase_tool_faction.lua |

---

## Network Message Details

### Chat & Communication

#### DrGBaseChatPrint
**Purpose:** Send formatted chat messages to specific players

**Usage:**
```lua
-- Server-side
net.Start("DrGBaseChatPrint")
    net.WriteTable({
        Color(255, 100, 100),
        "[DrGBase] ",
        Color(255, 255, 255),
        "NPC spawned"
    })
net.Send(player)
```

### Player Interaction

#### DrGBaseSelectEntity / DrGBaseDeselectEntity
**Purpose:** Notify player when they select/deselect an entity with possession system

**Used by:** Possession lock-on system

#### DrGBasePlayerLuminosity
**Purpose:** Updates player's luminosity value for NPC vision calculations

**Controlled by:** `drgbase_update_luminosity` convar

### NPC Awareness

#### DrGBaseNextbotPlayerAwareness
**Purpose:** Synchronizes NPC's awareness level of a player to the client

**Values:**
- 0 = Unaware
- 1 = Suspicious
- 2 = Aware/Alert

#### DrGBaseNextbotPlayerRelationship
**Purpose:** Synchronizes NPC's relationship disposition toward player

**Values:** D_HT, D_LI, D_FR, D_NU (see Enumerations)

### Possession System

#### DrGBaseNextbotCanPossess / DrGBaseNextbotCantPossess
**Purpose:** Shows/hides UI indicators when aiming at possessable NPCs

**Triggered by:** Lock-on system when looking at NPCs

#### DrGBasePossessionCycleViewPresets
**Purpose:** Client requests to cycle through possession camera view presets

**Direction:** Client → Server

**Triggered by:** Player input while possessing

### Enhanced Networking Module

#### DrGBaseNetMessage
**Purpose:** Generic wrapper for enhanced network messages

**Features:**
- Automatic string registration
- Type inference
- Simplified sending/receiving

**Example:**
```lua
-- Using DrGBase's enhanced net module
net.DrG_Send("MessageName", {
    data1 = "value",
    data2 = 123
}, player)
```

#### DrGBaseNetCallbackReq / DrGBaseNetCallbackRes
**Purpose:** Implements network callbacks with return values

**Features:**
- Request-response pattern
- Automatic callback handling
- Async network communication

### Navigation

#### DrGBaseNodegraph
**Purpose:** Sends nodegraph data to client for debug visualization

**Controlled by:**
- `drgbase_nodegraph_display`
- `drgbase_nodegraph_distance`
- `drgbase_nodegraph_type`
- `drgbase_nodegraph_transparent`

### Visual Effects

#### DrGBaseFlashGrenade
**Purpose:** Creates flashbang screen effect on client

**Parameters:** Position, intensity, duration

**Used by:** proj_drg_flashbang entity

---

## Using Network Messages

### Basic Networking

```lua
-- Server → Client
net.Start("DrGBaseChatPrint")
    net.WriteString("Hello")
net.Send(player)

-- Client receives
net.Receive("DrGBaseChatPrint", function()
    local message = net.ReadString()
    print(message)
end)
```

### Enhanced Networking (Recommended)

DrGBase provides enhanced networking that handles registration automatically:

```lua
-- Server → Client (simplified)
net.DrG_Send("MyMessage", {
    player = ply,
    damage = 50,
    position = Vector(0, 0, 0)
}, targetPlayer)

-- Client (simplified)
net.DrG_Receive("MyMessage", function(data)
    print(data.player, data.damage, data.position)
end)
```

**Benefits:**
- No manual `util.AddNetworkString()` needed
- Automatic type handling
- Cleaner syntax
- Built-in error handling

---

## Important Notes

1. **Automatic Registration:** DrGBase network strings are registered automatically during initialization

2. **Timing:** Network strings must be registered before use. DrGBase handles this in `autorun/drgbase.lua`

3. **Enhanced Module:** Use `net.DrG_Send()` and `net.DrG_Receive()` for simplified networking

4. **Direction:** Pay attention to message direction - sending from wrong realm will cause errors

5. **Recipients:** Always validate player entities before sending

---

## See Also

- [Network Module API](../api/modules/net.md)
- [Client-Server Architecture](../architecture/05-client-server.md)
- [Possession System](../systems/possession/README.md)
