# Network Module
**File:** `lua/drgbase/modules/net.lua`

The network module provides a high-level networking API that simplifies client-server communication in Garry's Mod. It offers named message systems and callback-based request-response patterns.

## Overview

DrGBase's networking system abstracts away the complexity of Garry's Mod's net library by providing:

1. **Named Messages**: Send and receive messages by name without managing network strings manually
2. **Automatic Serialization**: Send multiple arguments of any type without manual packing
3. **Callback System**: Request-response pattern for client-server communication with callbacks
4. **Type Safety**: Automatic type detection and serialization using `net.WriteType/ReadType`

All network strings are automatically registered by the framework, so you don't need to call `util.AddNetworkString()`.

---

## Functions

### net.DrG_WriteMessage(...)

**Realm:** 🟣 SHARED

**Parameters:**
- `...` (any) - Variable number of arguments of any networkable type

**Returns:**
- Nothing

**Description:**

Writes multiple values to the current network message in a way that they can be read back with `net.DrG_ReadMessage()`. This function automatically handles type detection and serialization for all standard Garry's Mod types (numbers, strings, vectors, entities, etc.).

Internally, it first writes the number of arguments, then writes each argument using `net.WriteType()` which automatically determines the correct write function based on the argument type.

This is a low-level function typically used internally by other DrGBase networking functions. Most users should use `net.DrG_Send()` instead.

**Example:**
```lua
-- Manual usage (usually you'd use net.DrG_Send instead)
net.Start("MyNetworkString")
net.DrG_WriteMessage("Hello", 42, Vector(0, 0, 0), player)
net.Broadcast()
```

**Notes:**
- Automatically counts and writes the number of arguments
- Uses `net.WriteType()` for automatic type serialization
- Should be paired with `net.DrG_ReadMessage()` on the receiving end
- Part of the internal implementation for higher-level networking functions

**See Also:**
- [net.DrG_ReadMessage](#netDrG_ReadMessage)
- [net.DrG_Send](#netDrG_Sendname-)

---

### net.DrG_ReadMessage()

**Realm:** 🟣 SHARED

**Parameters:**
- None

**Returns:**
- `table` - Array of received arguments
- `number` - Count of arguments received

**Description:**

Reads multiple values from the current network message that were written with `net.DrG_WriteMessage()`. Returns both the arguments as a table and the count of arguments.

This function reads the argument count first, then reads each argument using `net.ReadType()` which automatically determines the correct read function based on the type information in the stream.

This is a low-level function typically used internally by other DrGBase networking functions. Most users should use `net.DrG_Receive()` instead.

**Example:**
```lua
-- Manual usage (usually you'd use net.DrG_Receive instead)
net.Receive("MyNetworkString", function()
    local args, count = net.DrG_ReadMessage()
    print("Received", count, "arguments:", unpack(args))
end)
```

**Notes:**
- Must be paired with `net.DrG_WriteMessage()` on the sending end
- Returns unpacked arguments as a table and count as separate values
- Use with `table.DrG_Unpack()` to convert back to multiple return values
- Part of the internal implementation for higher-level networking functions

**See Also:**
- [net.DrG_WriteMessage](#netDrG_WriteMessage)
- [net.DrG_Receive](#netDrG_Receivename-callback)

---

### net.DrG_Receive(name, callback)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Unique name for this message handler
- `callback` (function or nil) - Function to call when message is received. On SERVER, receives `(ply, ...)` where `ply` is the sender. On CLIENT, receives `(...)` the message arguments directly. Set to `nil` to unregister.

**Returns:**
- Nothing

**Description:**

Registers a named network message receiver. When a message with the matching name is sent via `net.DrG_Send()`, this callback will be invoked with the message arguments.

On the server, the callback receives the sending player as the first argument, followed by the message data. On the client, the callback only receives the message data.

This function provides a simpler alternative to Garry's Mod's `net.Receive()` by handling message names and argument unpacking automatically. You don't need to manually call `util.AddNetworkString()` as the framework handles this automatically.

To unregister a message handler, call this function again with the same name and `nil` as the callback.

**Example:**
```lua
-- SERVER: Register a handler for player chat messages
net.DrG_Receive("PlayerCustomChat", function(ply, message, color)
    print(ply:Nick(), "sent:", message)
    -- Broadcast to all players
    net.DrG_Send("BroadcastChat", ply:Nick(), message, color)
end)

-- CLIENT: Register a handler for broadcast messages
net.DrG_Receive("BroadcastChat", function(nick, message, color)
    chat.AddText(color, nick, Color(255, 255, 255), ": ", message)
end)

-- CLIENT: Send a chat message to server
net.DrG_Send("PlayerCustomChat", "Hello world!", Color(255, 100, 100))

-- Unregister a handler
net.DrG_Receive("PlayerCustomChat", nil)
```

**Notes:**
- Message names must be unique and match between sender and receiver
- Server callbacks receive the sending player as the first parameter
- Client callbacks do not receive a player parameter
- Network strings are automatically registered by the framework
- Setting callback to `nil` removes the handler
- Can handle any number and type of arguments automatically

**See Also:**
- [net.DrG_Send](#netDrG_Sendname-)

---

### net.DrG_Send(name, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Name of the message to send
- `...` (any) - Variable number of arguments to send with the message

**Returns:**
- `boolean` - `true` if message was sent successfully, `false` if name was invalid

**Description:**

Sends a named network message with any number of arguments. On the server, broadcasts to all clients. On the client, sends to the server.

This is the primary function for sending network messages in DrGBase. It handles all the complexity of message packing, type serialization, and network string management automatically. Simply provide a message name and any arguments you want to send.

The receiving end should use `net.DrG_Receive()` with the same message name to handle the message.

**Example:**
```lua
-- SERVER: Send game state update to all clients
net.DrG_Send("GameStateUpdate", "round_started", 300, true)

-- CLIENT: Send player action to server
net.DrG_Send("PlayerAction", "jump", Vector(0, 0, 100))

-- SERVER: Send entity data
net.DrG_Send("EntityData", myEntity, myEntity:Health(), myEntity:GetPos())

-- CLIENT: Send complex data structures
local data = {
    name = "Player",
    score = 1000,
    position = LocalPlayer():GetPos()
}
net.DrG_Send("PlayerData", data.name, data.score, data.position)
```

**Notes:**
- Automatically broadcasts on SERVER, sends to server on CLIENT
- Use `net.DrG_SendPly()` for sending to specific players (if available)
- No need to manually call `util.AddNetworkString()` or manage network strings
- Can send any number of arguments of any networkable type
- Returns `false` if the name parameter is not a string

**See Also:**
- [net.DrG_Receive](#netDrG_Receivename-callback)
- [net.DrG_UseCallback](#netDrG_UseCallbackname-callback-)

---

### net.DrG_DefineCallback(name, callback)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Unique name for this callback handler
- `callback` (function or nil) - Function to call when callback is requested. Should return values to send back. Set to `nil` to unregister.

**Returns:**
- Nothing

**Description:**

Defines a network callback that can be invoked remotely and return data to the caller. This provides a request-response pattern for network communication, similar to remote procedure calls (RPC).

When another realm uses `net.DrG_UseCallback()` with this callback name, this function will be executed and its return values will be sent back to the caller's callback function.

This is extremely useful for requesting data from the server or performing operations that need confirmation or results.

**Example:**
```lua
-- SERVER: Define a callback that returns player health
net.DrG_DefineCallback("GetPlayerHealth", function(targetName)
    for _, ply in ipairs(player.GetAll()) do
        if ply:Nick() == targetName then
            return ply:Health(), ply:GetMaxHealth()
        end
    end
    return 0, 100 -- Default if not found
end)

-- SERVER: Define a callback for purchasing items
net.DrG_DefineCallback("PurchaseItem", function(itemID, quantity)
    local cost = GetItemCost(itemID) * quantity
    local hasEnough = GetPlayerMoney() >= cost
    if hasEnough then
        RemovePlayerMoney(cost)
        GivePlayerItem(itemID, quantity)
        return true, "Purchase successful"
    else
        return false, "Insufficient funds"
    end
end)

-- CLIENT: Define a callback for UI data requests
net.DrG_DefineCallback("GetInventoryDisplay", function()
    local inventory = LocalPlayer().Inventory or {}
    return inventory, #inventory
end)

-- Unregister a callback
net.DrG_DefineCallback("GetPlayerHealth", nil)
```

**Notes:**
- Callback names must be unique and match between definer and user
- The callback function can return any number of values
- Return values are automatically serialized and sent back
- Setting callback to `nil` removes the handler
- Network strings are automatically registered by the framework
- Different from `net.DrG_Receive()` - this supports returning data to the caller

**See Also:**
- [net.DrG_UseCallback](#netDrG_UseCallbackname-callback-)
- [net.DrG_RemoveCallback](#netDrG_RemoveCallbackname)

---

### net.DrG_RemoveCallback(name)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Name of the callback to remove

**Returns:**
- Nothing

**Description:**

Removes a previously defined network callback. This is a convenience function that calls `net.DrG_DefineCallback(name, nil)`.

**Example:**
```lua
-- Define a callback
net.DrG_DefineCallback("MyCallback", function() return true end)

-- Remove it later
net.DrG_RemoveCallback("MyCallback")
```

**See Also:**
- [net.DrG_DefineCallback](#netDrG_DefineCallbackname-callback)

---

### net.DrG_UseCallback(name, callback, ...)

**Realm:** 🟣 SHARED

**Parameters:**
- `name` (string) - Name of the callback to invoke (must be defined on the other realm)
- `callback` (function) - Function to call with the response values
- `...` (any) - Arguments to pass to the remote callback. On SERVER, first argument must be the target player.

**Returns:**
- `boolean` - `true` if request was sent successfully, `false` if parameters were invalid

**Description:**

Invokes a remote network callback and handles the response. This provides a request-response pattern where you can call a function on another realm and receive its return values asynchronously.

On the server, the first argument after the callback must be a player entity to specify which client to query. On the client, arguments are sent directly to the server.

When the remote callback completes, your callback function will be invoked with the return values from the remote callback.

**Example:**
```lua
-- CLIENT: Request health info from server
net.DrG_UseCallback("GetPlayerHealth", function(health, maxHealth)
    print("Player health:", health, "/", maxHealth)
    -- Update UI with health info
    UpdateHealthDisplay(health, maxHealth)
end, LocalPlayer():Nick())

-- CLIENT: Purchase an item and handle response
net.DrG_UseCallback("PurchaseItem", function(success, message)
    if success then
        chat.AddText(Color(0, 255, 0), message)
    else
        chat.AddText(Color(255, 0, 0), message)
    end
end, "weapon_pistol", 1)

-- SERVER: Request inventory data from a specific client
net.DrG_UseCallback("GetInventoryDisplay", function(inventory, count)
    print("Player has", count, "items")
    for i, item in ipairs(inventory) do
        print("-", item.name)
    end
end, targetPlayer)

-- SERVER: Query multiple clients
for _, ply in ipairs(player.GetAll()) do
    net.DrG_UseCallback("GetPing", function(ping)
        print(ply:Nick(), "reports ping:", ping)
    end, ply)
end
```

**Notes:**
- The remote callback must be defined with `net.DrG_DefineCallback()` first
- On SERVER: First argument after callback MUST be a valid player entity
- On CLIENT: Arguments are sent directly to server
- Callback is invoked asynchronously when response is received
- Returns `false` if name is invalid or (on SERVER) if first argument isn't a player
- Each request gets a unique ID for matching responses
- Callbacks are automatically cleaned up after being invoked once

**See Also:**
- [net.DrG_DefineCallback](#netDrG_DefineCallbackname-callback)
- [net.DrG_Send](#netDrG_Sendname-)
