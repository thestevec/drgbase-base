# Networking System

Client-server communication.

**File:** `modules/net.lua`

## Overview

DrGBase extends Garry's Mod's networking library with enhanced message passing and client-server callbacks. This system allows easy communication between server and client with automatic data serialization.

**Key Features:**
- **Net Messages** - Simple message passing with automatic serialization
- **Net Callbacks** - Request-response pattern for client-server communication
- **Auto-Serialization** - Automatically packs/unpacks multiple arguments
- **Bidirectional** - Works server-to-client and client-to-server

## Net Messages

**Define Handler:**
```lua
net.DrG_Receive("MyMessage", function(...)
    print("Received:", ...)
end)
```

**Send Message:**
```lua
net.DrG_Send("MyMessage", arg1, arg2, arg3)
```

## Net Callbacks

**Define Callback:**
```lua
net.DrG_DefineCallback("GetData", function(ply)
    return ply:Health(), ply:GetMaxHealth()
end)
```

**Use Callback:**
```lua
net.DrG_UseCallback("GetData", function(health, maxHealth)
    print("Health: " .. health .. "/" .. maxHealth)
end, ply)
```
