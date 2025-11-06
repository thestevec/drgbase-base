# Security Considerations

Security is paramount in multiplayer Garry's Mod addons. Malicious players can exploit poorly secured code to cheat, grief other players, crash servers, or gain unauthorized access. This guide covers essential security best practices for DrGBase development.

## Best Practice 1: Never Trust Client Input

All data from clients must be validated before processing. Clients can send arbitrary values and should never be trusted.

### ✅ DO

```lua
if SERVER then
    util.AddNetworkString("MyModSetNPCTarget")

    net.Receive("MyModSetNPCTarget", function(len, ply)
        local npc = net.ReadEntity()
        local target = net.ReadEntity()

        -- Validate NPC exists and is correct type
        if not IsValid(npc) or not npc.IsDrGNextbot then
            DrGBase.Print("Invalid NPC from player " .. ply:Nick())
            return
        end

        -- Validate player owns/created this NPC
        if npc:GetCreator() ~= ply then
            DrGBase.Print("Player " .. ply:Nick() .. " tried to control NPC they don't own")
            return
        end

        -- Validate target exists and is valid
        if not IsValid(target) then
            return
        end

        -- Validate target is attackable
        if not (target:IsNPC() or target:IsPlayer()) then
            return
        end

        -- Validate distance (prevent remote control)
        if npc:GetPos():Distance(ply:GetPos()) > 500 then
            return
        end

        -- Validate relationship (only command friendly NPCs)
        if npc:GetRelationship(ply) ~= D_LI then
            return
        end

        -- All validation passed, execute command
        npc:SetEnemy(target)
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModSetNPCTarget")

    net.Receive("MyModSetNPCTarget", function(len, ply)
        local npc = net.ReadEntity()
        local target = net.ReadEntity()

        -- BAD: No validation at all
        npc:SetEnemy(target)

        -- Player can:
        -- - Control any NPC (even other players' NPCs)
        -- - Set invalid targets
        -- - Crash server with NULL entities
        -- - Control NPCs from across the map
    end)
end
```

### Why This Matters

Without validation, malicious players can manipulate game state, control entities they shouldn't, crash the server, or disrupt other players' experiences. Every net message is a potential exploit if not properly validated.

## Best Practice 2: Validate Ownership and Permissions

Always check if a player has permission to perform an action on an entity.

### ✅ DO

```lua
if SERVER then
    function ENT:CanPlayerControl(ply)
        -- Check if player is valid
        if not IsValid(ply) or not ply:IsPlayer() then
            return false
        end

        -- Check ownership
        if self:GetCreator() ~= ply then
            return false
        end

        -- Check relationship
        if self:GetRelationship(ply) ~= D_LI then
            return false
        end

        -- Check distance
        if self:GetRangeTo(ply) > 500 then
            return false
        end

        -- Check player is alive
        if not ply:Alive() then
            return false
        end

        return true
    end

    util.AddNetworkString("MyModNPCCommand")

    net.Receive("MyModNPCCommand", function(len, ply)
        local npc = net.ReadEntity()
        local command = net.ReadString()

        if not IsValid(npc) or not npc.IsDrGNextbot then
            return
        end

        -- Centralized permission check
        if not npc:CanPlayerControl(ply) then
            DrGBase.Print("Player " .. ply:Nick() .. " denied control of NPC")
            return
        end

        -- Whitelist valid commands
        local validCommands = {
            follow = true,
            stay = true,
            attack = true,
            defend = true
        }

        if not validCommands[command] then
            DrGBase.Print("Invalid command '" .. command .. "' from " .. ply:Nick())
            return
        end

        -- Execute safe command
        npc:ExecuteCommand(command, ply)
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModNPCCommand")

    net.Receive("MyModNPCCommand", function(len, ply)
        local npc = net.ReadEntity()
        local command = net.ReadString()

        -- BAD: No ownership check
        -- BAD: No permission check
        -- BAD: No command validation
        -- BAD: Executing arbitrary commands

        if command == "follow" then
            npc:SetFollowTarget(ply)
        elseif command == "remove" then
            -- BAD: Player can remove any NPC!
            npc:Remove()
        elseif command == "teleport" then
            -- BAD: Player can teleport any NPC!
            npc:SetPos(ply:GetPos())
        elseif command == "damage" then
            -- BAD: Player can damage any NPC!
            npc:TakeDamage(999)
        end
    end)
end
```

### Why This Matters

Without ownership and permission checks, any player can control, modify, or remove entities they shouldn't have access to. This enables griefing and disrupts gameplay for legitimate players.

## Best Practice 3: Sanitize String Inputs

Never use string inputs directly in commands, SQL queries, or file operations without sanitization.

### ✅ DO

```lua
if SERVER then
    util.AddNetworkString("MyModSetNPCName")

    net.Receive("MyModSetNPCName", function(len, ply)
        local npc = net.ReadEntity()
        local name = net.ReadString()

        if not IsValid(npc) or not npc:CanPlayerControl(ply) then
            return
        end

        -- Sanitize string length
        if #name > 32 then
            name = string.sub(name, 1, 32)
        end

        if #name < 1 then
            return
        end

        -- Remove dangerous characters
        name = string.gsub(name, "[^%w%s%-_]", "")

        -- Prevent newlines and special characters
        name = string.Trim(name)

        -- Validate result
        if #name < 1 then
            return
        end

        -- Safe to use
        npc:SetNW2String("CustomName", name)
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModSetNPCName")

    net.Receive("MyModSetNPCName", function(len, ply)
        local npc = net.ReadEntity()
        local name = net.ReadString()

        -- BAD: No sanitization
        -- BAD: No length check
        -- BAD: Allows any characters

        npc:SetNW2String("CustomName", name)

        -- Player can send:
        -- - Extremely long strings (lag/crash)
        -- - Special characters (breaking UI)
        -- - Newlines (breaking chat/logs)
        -- - Escape sequences (potential exploits)
    end)
end
```

### Why This Matters

Unsanitized strings can contain malicious content that breaks UIs, crashes clients, exploits parsing vulnerabilities, or enables code injection in certain contexts.

## Best Practice 4: Rate Limit Network Messages

Prevent spam and denial-of-service attacks by rate limiting incoming messages.

### ✅ DO

```lua
if SERVER then
    -- Track last message time per player
    local playerMessageTimes = {}

    util.AddNetworkString("MyModNPCCommand")

    net.Receive("MyModNPCCommand", function(len, ply)
        local steamID = ply:SteamID()
        local now = CurTime()

        -- Initialize timing table
        if not playerMessageTimes[steamID] then
            playerMessageTimes[steamID] = {}
        end

        local lastTime = playerMessageTimes[steamID].command or 0

        -- Rate limit: 1 command per 0.5 seconds
        if now - lastTime < 0.5 then
            -- Too fast, ignore request
            return
        end

        -- Update last command time
        playerMessageTimes[steamID].command = now

        -- Process command
        local npc = net.ReadEntity()
        local command = net.ReadString()

        if not IsValid(npc) or not npc:CanPlayerControl(ply) then
            return
        end

        npc:ExecuteCommand(command, ply)
    end)

    -- Clean up disconnected players
    hook.Add("PlayerDisconnected", "MyModCleanupRateLimit", function(ply)
        playerMessageTimes[ply:SteamID()] = nil
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModNPCCommand")

    net.Receive("MyModNPCCommand", function(len, ply)
        -- BAD: No rate limiting
        -- Malicious player can spam thousands of messages per second

        local npc = net.ReadEntity()
        local command = net.ReadString()

        npc:ExecuteCommand(command, ply)

        -- Server can be overwhelmed with network spam
        -- Causes lag for all players
    end)
end
```

### Why This Matters

Without rate limiting, malicious players can flood the server with network messages, causing lag, overwhelming the server, and potentially crashing it. Rate limiting prevents these denial-of-service attacks.

## Best Practice 5: Validate Numerical Ranges

Always validate that numbers are within expected ranges to prevent exploits.

### ✅ DO

```lua
if SERVER then
    util.AddNetworkString("MyModSetNPCScale")

    net.Receive("MyModSetNPCScale", function(len, ply)
        local npc = net.ReadEntity()
        local scale = net.ReadFloat()

        if not IsValid(npc) or not npc:CanPlayerControl(ply) then
            return
        end

        -- Validate range
        if scale < 0.5 then
            scale = 0.5
        elseif scale > 3.0 then
            scale = 3.0
        end

        -- Validate is a number (not NaN or Inf)
        if scale ~= scale then  -- NaN check
            return
        end

        if math.abs(scale) == math.huge then  -- Infinity check
            return
        end

        -- Safe to apply
        npc:SetModelScale(scale)
    end)

    util.AddNetworkString("MyModSetAbilityCooldown")

    net.Receive("MyModSetAbilityCooldown", function(len, ply)
        local npc = net.ReadEntity()
        local cooldown = net.ReadUInt(16)

        if not IsValid(npc) or not npc:CanPlayerControl(ply) then
            return
        end

        -- Validate range (1-60 seconds)
        cooldown = math.Clamp(cooldown, 1, 60)

        npc:SetAbilityCooldown(cooldown)
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModSetNPCScale")

    net.Receive("MyModSetNPCScale", function(len, ply)
        local npc = net.ReadEntity()
        local scale = net.ReadFloat()

        -- BAD: No validation
        npc:SetModelScale(scale)

        -- Player can send:
        -- - Negative values (inverted models)
        -- - Zero (invisible models)
        -- - Huge values (performance/visual issues)
        -- - NaN or Inf (crashes/undefined behavior)
    end)

    util.AddNetworkString("MyModSetAbilityCooldown")

    net.Receive("MyModSetAbilityCooldown", function(len, ply)
        local npc = net.ReadEntity()
        local cooldown = net.ReadUInt(16)

        -- BAD: No range check
        npc:SetAbilityCooldown(cooldown)

        -- Player can set:
        -- - Zero cooldown (spam abilities)
        -- - Huge cooldown (break the NPC)
    end)
end
```

### Why This Matters

Invalid numerical values can cause crashes, undefined behavior, visual glitches, or enable cheating. Always validate that numbers are within expected, safe ranges.

## Best Practice 6: Protect Server Console Commands

ConCommands that affect gameplay must validate permissions.

### ✅ DO

```lua
if SERVER then
    -- Admin-only command
    concommand.Add("mymod_spawn_boss", function(ply, cmd, args)
        -- Server console always allowed
        if not IsValid(ply) then
            SpawnBossNPC()
            return
        end

        -- Validate player is admin
        if not ply:IsAdmin() then
            DrGBase.Print("Access denied: Admin only", {player = ply, chat = true, color = DrGBase.CLR_RED})
            return
        end

        -- Validate cooldown
        if ply._lastBossSpawn and CurTime() - ply._lastBossSpawn < 300 then
            DrGBase.Print("Boss can only be spawned once per 5 minutes", {player = ply, chat = true, color = DrGBase.CLR_RED})
            return
        end

        ply._lastBossSpawn = CurTime()
        SpawnBossNPC()
        DrGBase.Print("Boss spawned by " .. ply:Nick(), {chat = true})
    end)

    -- Regular player command with restrictions
    concommand.Add("mymod_heal_npcs", function(ply, cmd, args)
        if not IsValid(ply) then return end

        -- Validate cooldown
        if ply._lastNPCHeal and CurTime() - ply._lastNPCHeal < 60 then
            return
        end

        ply._lastNPCHeal = CurTime()

        -- Only heal NPCs the player owns
        local healed = 0
        for _, npc in ipairs(ents.FindInSphere(ply:GetPos(), 500)) do
            if npc.IsDrGNextbot and npc:GetCreator() == ply then
                npc:SetHealth(math.min(npc:Health() + 50, npc:GetMaxHealth()))
                healed = healed + 1

                if healed >= 5 then
                    break  -- Limit how many can be healed at once
                end
            end
        end
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    -- BAD: No permission check
    concommand.Add("mymod_spawn_boss", function(ply, cmd, args)
        SpawnBossNPC()
        -- Any player can spawn boss NPCs!
    end)

    -- BAD: No cooldown or limits
    concommand.Add("mymod_heal_npcs", function(ply, cmd, args)
        -- Heal ALL NPCs in the game
        for _, npc in ipairs(ents.GetAll()) do
            if npc.IsDrGNextbot then
                npc:SetHealth(npc:GetMaxHealth())
            end
        end
        -- Player can spam this to make NPCs invincible
    end)

    -- BAD: Dangerous admin command without protection
    concommand.Add("mymod_remove_all_npcs", function(ply, cmd, args)
        for _, npc in ipairs(ents.GetAll()) do
            if npc.IsDrGNextbot then
                npc:Remove()
            end
        end
        -- Any player can remove all NPCs!
    end)
end
```

### Why This Matters

Console commands without permission checks enable any player to execute powerful actions, grief others, spawn entities, or manipulate game state. Always validate permissions and add cooldowns.

## Best Practice 7: Prevent Resource Exhaustion

Limit resource-intensive operations to prevent server crashes or lag.

### ✅ DO

```lua
if SERVER then
    util.AddNetworkString("MyModSpawnMinions")

    -- Track spawns per player
    local playerSpawnCounts = {}

    net.Receive("MyModSpawnMinions", function(len, ply)
        local npc = net.ReadEntity()
        local count = net.ReadUInt(8)

        if not IsValid(npc) or not npc:CanPlayerControl(ply) then
            return
        end

        -- Limit count
        count = math.Clamp(count, 1, 3)

        -- Track total spawns
        local steamID = ply:SteamID()
        playerSpawnCounts[steamID] = (playerSpawnCounts[steamID] or 0)

        -- Enforce per-player entity limit
        local currentCount = 0
        for _, ent in ipairs(ents.GetAll()) do
            if ent.IsDrGNextbot and ent:GetCreator() == ply then
                currentCount = currentCount + 1
            end
        end

        if currentCount >= 20 then
            DrGBase.Print("Entity limit reached (20 per player)", {player = ply, chat = true, color = DrGBase.CLR_RED})
            return
        end

        -- Prevent too many spawns in short time
        if playerSpawnCounts[steamID] > 10 then
            return
        end

        playerSpawnCounts[steamID] = playerSpawnCounts[steamID] + count

        -- Reset counter after delay
        timer.Simple(60, function()
            if not playerSpawnCounts[steamID] then return end
            playerSpawnCounts[steamID] = math.max(0, playerSpawnCounts[steamID] - count)
        end)

        -- Spawn with limits
        for i = 1, count do
            local minion = ents.Create("npc_mymod_minion")
            minion:SetPos(npc:GetPos() + Vector(math.random(-100, 100), math.random(-100, 100), 0))
            minion:SetCreator(ply)
            minion:Spawn()
        end
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModSpawnMinions")

    net.Receive("MyModSpawnMinions", function(len, ply)
        local npc = net.ReadEntity()
        local count = net.ReadUInt(8)

        -- BAD: No validation or limits
        for i = 1, count do
            local minion = ents.Create("npc_mymod_minion")
            minion:SetPos(npc:GetPos())
            minion:Spawn()
        end

        -- Player can:
        -- - Send count = 255 (spawn 255 entities)
        -- - Spam this message (thousands of entities)
        -- - Crash server with entity overflow
    end)
end
```

### Why This Matters

Without resource limits, malicious players can spawn unlimited entities, causing severe lag or crashing the server. Always enforce reasonable limits on resource-intensive operations.

## Best Practice 8: Secure File Operations

Never allow players to directly control file paths or operations.

### ✅ DO

```lua
if SERVER then
    -- Predefined, safe configuration files
    local ALLOWED_CONFIGS = {
        ["default"] = "data/mymod/config_default.txt",
        ["hardcore"] = "data/mymod/config_hardcore.txt",
        ["easy"] = "data/mymod/config_easy.txt"
    }

    util.AddNetworkString("MyModLoadConfig")

    net.Receive("MyModLoadConfig", function(len, ply)
        if not ply:IsAdmin() then
            return
        end

        local configName = net.ReadString()

        -- Validate against whitelist
        if not ALLOWED_CONFIGS[configName] then
            DrGBase.Print("Invalid config name", {player = ply, chat = true, color = DrGBase.CLR_RED})
            return
        end

        -- Use safe, predefined path
        local path = ALLOWED_CONFIGS[configName]

        if file.Exists(path, "DATA") then
            local data = file.Read(path, "DATA")
            -- Process configuration safely
            LoadConfiguration(data)
            DrGBase.Print("Configuration loaded: " .. configName, {player = ply, chat = true})
        end
    end)

    function SaveNPCPreset(npc, presetName)
        -- Sanitize preset name
        presetName = string.gsub(presetName, "[^%w%-_]", "")
        presetName = string.sub(presetName, 1, 32)

        if #presetName < 1 then
            return false
        end

        -- Use safe directory
        local path = "mymod/presets/" .. presetName .. ".txt"

        -- Save data
        local data = util.TableToJSON(npc:GetPresetData())
        file.Write(path, data)

        return true
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    util.AddNetworkString("MyModLoadConfig")

    net.Receive("MyModLoadConfig", function(len, ply)
        local filePath = net.ReadString()

        -- BAD: Using client-provided path directly
        if file.Exists(filePath, "DATA") then
            local data = file.Read(filePath, "DATA")
            LoadConfiguration(data)
        end

        -- Player can:
        -- - Read any file on the server
        -- - Access sensitive data
        -- - Use path traversal (../../passwords.txt)
    end)

    util.AddNetworkString("MyModSavePreset")

    net.Receive("MyModSavePreset", function(len, ply)
        local fileName = net.ReadString()
        local data = net.ReadString()

        -- BAD: No sanitization
        file.Write(fileName, data)

        -- Player can:
        -- - Overwrite important files
        -- - Write to arbitrary locations
        -- - Inject malicious data
    end)
end
```

### Why This Matters

Unsecured file operations enable path traversal attacks, allow reading sensitive files, overwriting important data, or injecting malicious content. Always use whitelists and sanitize file paths.

## Real-World Example: Secure Boss NPC

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Secure Boss"
ENT.SpawnHealth = 5000

if SERVER then
    -- Secure network strings
    util.AddNetworkString("BossCommand")
    util.AddNetworkString("BossRequestSync")

    -- Rate limiting
    local playerCommandTimes = {}
    local playerSyncRequests = {}

    function ENT:CanPlayerCommand(ply)
        -- Validate player
        if not IsValid(ply) or not ply:IsPlayer() or not ply:Alive() then
            return false
        end

        -- Validate relationship
        if self:GetRelationship(ply) ~= D_LI then
            return false
        end

        -- Validate distance
        if self:GetRangeTo(ply) > 500 then
            return false
        end

        -- Validate not in combat
        if self:HasEnemy() then
            return false
        end

        return true
    end

    function ENT:ExecuteSafeCommand(ply, command, arg)
        -- Whitelist commands
        local validCommands = {
            follow = function(self, ply, arg)
                self:SetFollowTarget(ply)
            end,
            stay = function(self, ply, arg)
                self:SetStayPosition(self:GetPos())
            end,
            defend = function(self, ply, arg)
                self:SetDefendTarget(ply)
            end
        }

        local commandFunc = validCommands[command]
        if not commandFunc then
            return false
        end

        commandFunc(self, ply, arg)
        return true
    end

    -- Secure command receiver
    net.Receive("BossCommand", function(len, ply)
        local boss = net.ReadEntity()
        local command = net.ReadString()
        local arg = net.ReadEntity()

        -- Validate entity
        if not IsValid(boss) or boss:GetClass() ~= "npc_mymod_boss" then
            return
        end

        -- Rate limit: 1 command per second
        local steamID = ply:SteamID()
        local now = CurTime()

        if not playerCommandTimes[steamID] then
            playerCommandTimes[steamID] = 0
        end

        if now - playerCommandTimes[steamID] < 1.0 then
            return
        end

        playerCommandTimes[steamID] = now

        -- Validate permissions
        if not boss:CanPlayerCommand(ply) then
            return
        end

        -- Sanitize command
        command = string.lower(string.Trim(command))
        if #command > 20 then
            return
        end

        -- Execute safely
        boss:ExecuteSafeCommand(ply, command, arg)
    end)

    -- Secure sync receiver with rate limiting
    net.Receive("BossRequestSync", function(len, ply)
        local boss = net.ReadEntity()

        if not IsValid(boss) or boss:GetClass() ~= "npc_mymod_boss" then
            return
        end

        -- Rate limit: 1 sync request per 5 seconds
        local steamID = ply:SteamID()
        local now = CurTime()

        if not playerSyncRequests[steamID] then
            playerSyncRequests[steamID] = 0
        end

        if now - playerSyncRequests[steamID] < 5.0 then
            return
        end

        playerSyncRequests[steamID] = now

        -- Send safe, read-only data
        net.Start("BossSyncData")
        net.WriteEntity(boss)
        net.WriteUInt(math.Clamp(boss:GetNW2Int("BossPhase", 1), 1, 5), 8)
        net.WriteBool(boss:GetNW2Bool("BossEnraged", false))
        net.Send(ply)
    end)

    -- Cleanup on disconnect
    hook.Add("PlayerDisconnected", "BossSecurityCleanup", function(ply)
        local steamID = ply:SteamID()
        playerCommandTimes[steamID] = nil
        playerSyncRequests[steamID] = nil
    end)
end

if CLIENT then
    -- Client can only request, not command directly
    function ENT:RequestCommand(command)
        net.Start("BossCommand")
        net.WriteEntity(self)
        net.WriteString(command)
        net.WriteEntity(NULL)
        net.SendToServer()
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

This secure boss demonstrates proper validation, rate limiting, whitelisted commands, permission checking, and safe data handling.

## Tips and Recommendations

1. **Assume Hostility**: Treat all client input as potentially malicious until validated.

2. **Whitelist, Don't Blacklist**: Define what's allowed rather than trying to block everything that's forbidden.

3. **Use Least Privilege**: Only give players the minimum permissions they need.

4. **Log Suspicious Activity**: Track and log unusual patterns or repeated violations.

5. **Test with Malicious Intent**: Try to break your own code by sending invalid data.

6. **Keep Validation Simple**: Complex validation logic is more likely to have holes.

7. **Fail Securely**: When validation fails, deny access rather than allowing it.

8. **Update Regularly**: Keep track of discovered exploits in the GMod community and patch accordingly.

9. **Rate Limit Everything**: Any network message from clients should be rate-limited.

10. **Document Security Decisions**: Comment why certain validations exist so they're not removed later.
