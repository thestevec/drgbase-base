# Network Messages Reference

Complete reference of DrGBase network messages for client-server communication.

## Core Network Messages

### DrGBaseNetMessage
General-purpose network message system for DrGBase communication.

**Direction**: Bidirectional (Server ↔ Client)

```lua
-- Server to clients
net.Start("DrGBaseNetMessage")
net.WriteString("MessageType")
net.WriteTable(data)
net.Broadcast()

-- Client receives
net.Receive("DrGBaseNetMessage", function()
    local msgType = net.ReadString()
    local data = net.ReadTable()
end)
```

### DrGBaseNetCallbackReq
Network callback request - client requests server response.

**Direction**: Client → Server

```lua
-- Client sends request
net.Start("DrGBaseNetCallbackReq")
net.WriteString("RequestID")
net.WriteEntity(npc)
net.SendToServer()
```

### DrGBaseNetCallbackRes
Network callback response - server responds to client request.

**Direction**: Server → Client

```lua
-- Server responds
net.Start("DrGBaseNetCallbackRes")
net.WriteString("RequestID")
net.WriteTable(response)
net.Send(ply)
```

## Entity Selection Messages

### DrGBaseSelectEntity
Notifies client when entity is selected by tools.

**Direction**: Server → Client

```lua
net.Start("DrGBaseSelectEntity")
net.WriteEntity(entity)
net.Send(ply)
```

### DrGBaseDeselectEntity
Notifies client when entity is deselected.

**Direction**: Server → Client

```lua
net.Start("DrGBaseDeselectEntity")
net.WriteEntity(entity)
net.Send(ply)
```

## NPC Awareness Messages

### DrGBaseNextbotPlayerAwareness
Updates client about NPC awareness of player.

**Direction**: Server → Client

**Usage**: Used for stealth mechanics and UI indicators.

```lua
-- Server notifies client
net.Start("DrGBaseNextbotPlayerAwareness")
net.WriteEntity(npc)
net.WriteFloat(awarenessLevel)  // 0.0 to 1.0
net.Send(ply)
```

### DrGBaseNextbotPlayerRelationship
Synchronizes NPC-player relationship status.

**Direction**: Server → Client

```lua
net.Start("DrGBaseNextbotPlayerRelationship")
net.WriteEntity(npc)
net.WriteEntity(player)
net.WriteInt(disposition, 4)  // D_HT, D_LI, D_NU, D_FR
net.Broadcast()
```

## Tool Messages

### DrGBaseFactionTool
Faction tool communication for faction management UI.

**Direction**: Bidirectional

```lua
-- Client requests faction change
net.Start("DrGBaseFactionTool")
net.WriteEntity(npc)
net.WriteString("FACTION_REBELS")
net.SendToServer()

-- Server confirms
net.Start("DrGBaseFactionTool")
net.WriteEntity(npc)
net.WriteString(util.TableToJSON(factions))
net.Send(ply)
```

## Possession Messages

### DrGBaseNextbotCanPossess
Server informs client possession is allowed.

**Direction**: Server → Client

```lua
net.Start("DrGBaseNextbotCanPossess")
net.WriteEntity(npc)
net.Send(ply)
```

### DrGBaseNextbotCantPossess
Server informs client possession is denied.

**Direction**: Server → Client

```lua
net.Start("DrGBaseNextbotCantPossess")
net.WriteEntity(npc)
net.WriteString("Reason for denial")
net.Send(ply)
```

### DrGBasePossessionCycleViewPresets
Cycles through possession camera view presets.

**Direction**: Client → Server

```lua
-- Client requests view change
net.Start("DrGBasePossessionCycleViewPresets")
net.SendToServer()
```

## Projectile Messages

### DrGBaseFlashGrenade
Handles flashbang grenade effects on clients.

**Direction**: Server → Clients

```lua
net.Start("DrGBaseFlashGrenade")
net.WriteVector(position)
net.WriteFloat(intensity)
net.Broadcast()
```

## Developer Messages

### DrGBaseNodegraph
Transmits nodegraph data for visualization.

**Direction**: Server → Client

**Usage**: Debug visualization of navigation nodes.

```lua
net.Start("DrGBaseNodegraph")
net.WriteTable(nodes)
net.Send(ply)
```

### DrGBaseChatPrint
Sends formatted chat messages to client.

**Direction**: Server → Client

```lua
net.Start("DrGBaseChatPrint")
net.WriteString("Message text")
net.WriteColor(Color(255, 255, 255))
net.Send(ply)
```

## Player State Messages

### DrGBasePlayerLuminosity
Updates player luminosity level for stealth detection.

**Direction**: Server → Client

**Usage**: NPCs detect players based on light level.

```lua
net.Start("DrGBasePlayerLuminosity")
net.WriteFloat(luminosity)  // 0.0 (dark) to 1.0 (bright)
net.Send(ply)
```

## Usage Best Practices

### Minimize Network Traffic
```lua
-- Good: Only send when changed
if self.LastHealth ~= self:Health() then
    net.Start("DrGBaseNetMessage")
    net.WriteInt(self:Health(), 10)
    net.Broadcast()
    self.LastHealth = self:Health()
end

-- Bad: Constant updates
function ENT:Think()
    net.Start("DrGBaseNetMessage")
    net.WriteInt(self:Health(), 10)
    net.Broadcast()
end
```

### Validate All Network Data
```lua
net.Receive("DrGBaseNetMessage", function(len, ply)
    local entity = net.ReadEntity()
    
    -- Always validate
    if not IsValid(entity) then return end
    if not entity.Base == "drgbase_nextbot" then return end
    if not ply:IsAdmin() then return end  // If admin-only
    
    -- Process message
end)
```

### Use Appropriate Data Types
```lua
-- Efficient: Use smallest type needed
net.WriteUInt(health, 10)  // 0-1023 (uses 10 bits)
net.WriteBool(isAlive)     // Uses 1 bit
net.WriteInt(damage, 8)    // -128 to 127 (uses 8 bits)

-- Inefficient: Using larger types
net.WriteInt(health, 32)   // Uses 32 bits for same data
net.WriteString("true")    // Uses many bits for boolean
```

For networking best practices, see [Networking Best Practices](../best-practices/03-networking.md).
