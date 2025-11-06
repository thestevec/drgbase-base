# Security Considerations

Protecting your DrGBase addon from exploits and abuse.

## Input Validation

### Validate All Inputs
```lua
-- Good: Validate before use
function ENT:SetCustomDamage(damage)
    damage = tonumber(damage) or 10
    damage = math.Clamp(damage, 1, 1000)
    self.CustomDamage = damage
end

-- Bad: No validation
function ENT:SetCustomDamage(damage)
    self.CustomDamage = damage  -- Could be anything!
end
```

## Console Command Safety

### Admin-Only Commands
```lua
-- Good: Check permissions
concommand.Add("yourmod_spawn_boss", function(ply, cmd, args)
    if not ply:IsAdmin() then
        ply:ChatPrint("Admin only!")
        return
    end

    -- Spawn boss
end)

-- Bad: No permission check
concommand.Add("yourmod_spawn_boss", function(ply, cmd, args)
    -- Anyone can spawn boss
end)
```

## Network Message Security

### Validate Networked Data
```lua
-- Server receives net message
net.Receive("SetNPCTarget", function(len, ply)
    if not ply:IsAdmin() then return end  -- Permission check

    local npc = net.ReadEntity()
    local target = net.ReadVector()

    -- Validate
    if not IsValid(npc) then return end
    if not npc.Base == "drgbase_nextbot" then return end

    npc:SetTarget(target)
end)
```

## SQL Injection Prevention

### Use Prepared Statements
```lua
-- Good: Parameterized queries
sql.Query("INSERT INTO npc_stats VALUES (" .. sql.SQLStr(npcClass) .. ", " .. tonumber(kills) .. ")")

-- Better: Use proper escaping
local escapedClass = sql.SQLStr(npcClass)
sql.Query("INSERT INTO npc_stats VALUES (" .. escapedClass .. ", " .. kills .. ")")
```

## File Access

### Restrict File Operations
```lua
-- Good: Validate paths
function LoadNPCConfig(filename)
    -- Only allow specific files
    if not string.match(filename, "^[a-z0-9_]+%.txt$") then
        return false
    end

    local path = "yourmod/configs/" .. filename
    return file.Read(path, "DATA")
end
```

## Rate Limiting

### Prevent Spam
```lua
-- Limit command usage
local lastUse = {}

concommand.Add("yourmod_command", function(ply, cmd, args)
    local now = CurTime()
    if (lastUse[ply] or 0) > now - 1 then  -- 1 second cooldown
        ply:ChatPrint("Wait before using again")
        return
    end

    lastUse[ply] = now
    -- Command logic
end)
```

## Security Checklist

- [ ] All console commands check permissions
- [ ] Network messages validate sender
- [ ] User input is sanitized
- [ ] File paths are validated
- [ ] Rate limiting on spam-able operations
- [ ] SQL queries use proper escaping
- [ ] Admin tools are admin-only
