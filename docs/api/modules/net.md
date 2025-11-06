# Network Module
**File:** `lua/drgbase/modules/net.lua`

Simplified networking API with automatic message packing and client-server callbacks.

## Overview

DrGBase provides a higher-level networking API that:
- Automatically serializes/deserializes multiple arguments
- Handles client-server communication without manual network string setup
- Provides callback system for request-response patterns
- Uses a single network string for all messages

## Message Functions

### net.DrG_Receive(name, callback)
Registers a handler for named network messages.

**Parameters:**
- `name` (string) - Message identifier
- `callback` (function) - Handler function
  - **Server:** `function(ply, ...)`  - Receives player and message arguments
  - **Client:** `function(...)` - Receives message arguments only
  - Pass `nil` to unregister

**Example:**
```lua
-- Server receives messages from clients
if SERVER then
    net.DrG_Receive("PlayerRequestHealth", function(ply, amount)
        ply:SetHealth(math.min(ply:Health() + amount, ply:GetMaxHealth()))
        print(ply:Nick(), "requested", amount, "health")
    end)
end

-- Client receives messages from server
if CLIENT then
    net.DrG_Receive("UpdateScore", function(score, highScore)
        print("Score updated:", score, "/", highScore)
        LocalPlayer().score = score
    end)
end

-- Unregister a handler
net.DrG_Receive("SomeMessage", nil)
```

### net.DrG_Send(name, ...)
Sends a named network message with arguments.

**Parameters:**
- `name` (string) - Message identifier
- `...` - Any number of networkable arguments

**Returns:**
- (boolean) - True if sent successfully, false if name is invalid

**Behavior:**
- **Server:** Broadcasts to all clients
- **Client:** Sends to server
- Automatically packs all arguments
- Supports any net-writable types

**Example:**
```lua
-- Client sends to server
if CLIENT then
    net.DrG_Send("RequestItem", "weapon_pistol", 5)
    net.DrG_Send("ChatMessage", "Hello world!")
end

-- Server sends to all clients
if SERVER then
    net.DrG_Send("GameStateUpdate", "round_end", winningTeam, totalScore)
    net.DrG_Send("ShowNotification", "Round starting in 10 seconds", 10)
end
```

## Callback Functions

Callbacks allow request-response communication patterns, similar to HTTP requests.

### net.DrG_DefineCallback(name, callback)
Defines a callback handler that can return values to the caller.

**Parameters:**
- `name` (string) - Callback identifier
- `callback` (function) - Handler function that returns response value(s)
  - Can return any networkable value(s)
  - Pass `nil` to unregister

**Example:**
```lua
-- Server defines a callback
if SERVER then
    net.DrG_DefineCallback("GetPlayerData", function(dataType)
        if dataType == "count" then
            return player.GetCount()
        elseif dataType == "names" then
            local names = {}
            for _, ply in ipairs(player.GetAll()) do
                table.insert(names, ply:Nick())
            end
            return names
        end
    end)
end

-- Client defines a callback
if CLIENT then
    net.DrG_DefineCallback("GetLocalSettings", function(settingName)
        return GetConVar(settingName):GetString()
    end)
end
```

### net.DrG_UseCallback(name, callback, ...)
Calls a remote callback and handles the response.

**Parameters:**
- `name` (string) - Callback identifier
- `callback` (function) - Response handler `function(...)`
  - Receives the return values from the remote callback
- `...` - Arguments to send
  - **Server:** First argument MUST be the target player

**Returns:**
- (boolean) - True if sent successfully, false otherwise

**Example:**
```lua
-- Client requests data from server
if CLIENT then
    net.DrG_UseCallback("GetPlayerData", function(count)
        print("Player count:", count)
    end, "count")

    net.DrG_UseCallback("GetPlayerData", function(names)
        print("Players:", table.concat(names, ", "))
    end, "names")
end

-- Server requests data from client
if SERVER then
    local ply = player.GetByID(1)
    net.DrG_UseCallback("GetLocalSettings", function(value)
        print(ply:Nick(), "has setting:", value)
    end, ply, "developer")  -- Note: ply is first argument
end
```

### net.DrG_RemoveCallback(name)
Removes a callback handler. Alias for `net.DrG_DefineCallback(name, nil)`.

**Parameters:**
- `name` (string) - Callback identifier

## Utility Functions

### net.DrG_WriteMessage(...)
Writes multiple arguments to the network stream.

**Internal function** - Used by DrG_Send and DrG_UseCallback.

- Automatically determines types
- Packs variable number of arguments
- Writes count header for unpacking

### net.DrG_ReadMessage()
Reads multiple arguments from the network stream.

**Internal function** - Used by message/callback receivers.

**Returns:**
- `args` (table) - Array of arguments
- `n` (number) - Number of arguments

## Complete Examples

### Simple Chat System
```lua
-- Server
if SERVER then
    util.AddNetworkString("DrGBaseNetMessage")

    net.DrG_Receive("ChatMessage", function(ply, message)
        -- Broadcast to all players
        net.DrG_Send("ChatMessage", ply:Nick(), message)
    end)
end

-- Client
if CLIENT then
    net.DrG_Receive("ChatMessage", function(name, message)
        chat.AddText(Color(255, 255, 255), name, ": ", message)
    end)

    -- Send message
    net.DrG_Send("ChatMessage", "Hello everyone!")
end
```

### Request-Response Pattern
```lua
-- Server provides data
if SERVER then
    net.DrG_DefineCallback("GetEntityInfo", function(entIndex)
        local ent = Entity(entIndex)
        if IsValid(ent) then
            return ent:GetClass(), ent:Health(), ent:GetPos()
        end
        return nil
    end)
end

-- Client requests data
if CLIENT then
    local ent = LocalPlayer():GetEyeTrace().Entity
    if IsValid(ent) then
        net.DrG_UseCallback("GetEntityInfo", function(class, health, pos)
            if class then
                print("Entity:", class)
                print("Health:", health)
                print("Position:", pos)
            else
                print("Invalid entity")
            end
        end, ent:EntIndex())
    end
end
```

### Async Data Loading
```lua
-- Server
if SERVER then
    net.DrG_DefineCallback("LoadProfile", function(steamID)
        local data = LoadPlayerProfile(steamID)  -- Expensive operation
        return data.level, data.experience, data.achievements
    end)
end

-- Client
if CLIENT then
    function LoadMyProfile()
        ShowLoadingScreen()

        net.DrG_UseCallback("LoadProfile", function(level, exp, achievements)
            HideLoadingScreen()
            UpdateUI(level, exp, achievements)
        end, LocalPlayer():SteamID())
    end
end
```

## Implementation Details

- Uses `"DrGBaseNetMessage"` for all regular messages
- Uses `"DrGBaseNetCallbackReq"` and `"DrGBaseNetCallbackRes"` for callbacks
- Callbacks use unique incremental IDs to match requests with responses
- Automatic cleanup of callback handlers after response
- Network strings are registered automatically on server

## Best Practices

1. **Use Messages for One-Way Communication**
   ```lua
   -- Good: Fire-and-forget updates
   net.DrG_Send("UpdatePosition", pos)
   ```

2. **Use Callbacks for Request-Response**
   ```lua
   -- Good: When you need a response
   net.DrG_UseCallback("ValidateAction", function(allowed)
       if allowed then DoAction() end
   end, actionType)
   ```

3. **Server-to-Client Callbacks: Always Pass Player First**
   ```lua
   -- Correct
   net.DrG_UseCallback("GetSetting", function(val) end, ply, "setting")

   -- Wrong
   net.DrG_UseCallback("GetSetting", function(val) end, "setting")
   ```

4. **Clean Up Handlers When Done**
   ```lua
   -- Remove when no longer needed
   net.DrG_Receive("TempMessage", nil)
   net.DrG_RemoveCallback("TempCallback")
   ```

## Performance Notes

- Minimal overhead compared to standard net library
- Single network string reduces registration overhead
- Callback system uses memory until response received
- All arguments are packed/unpacked automatically (slight CPU cost)
