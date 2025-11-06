# Networking Best Practices

Networking in Garry's Mod follows a client-server architecture where the server is authoritative. Understanding how to network data efficiently is crucial for creating DrGBase addons that work smoothly in multiplayer environments. This guide covers essential networking best practices.

## Best Practice 1: Understand Server-Client Authority

The server is always authoritative. Clients receive updates from the server but should never directly control gameplay logic.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- Server controls the authoritative state
        self._customMode = "idle"
        self:SetNW2String("CustomMode", "idle")
    end

    function ENT:SetCustomMode(mode)
        -- Server validates and updates
        if mode ~= "idle" and mode ~= "combat" and mode ~= "patrol" then
            return false  -- Invalid mode
        end

        self._customMode = mode
        self:SetNW2String("CustomMode", mode)  -- Network to clients
        return true
    end

    function ENT:OnMeleeAttack(enemy)
        -- Server handles gameplay logic
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
        self:SetCustomMode("combat")
    end
end

if CLIENT then
    function ENT:CustomDraw()
        -- Client reads networked state for display only
        local mode = self:GetNW2String("CustomMode", "idle")

        if mode == "combat" then
            -- Draw combat indicator
            local pos = self:GetPos() + Vector(0, 0, 90)
            local ang = LocalPlayer():EyeAngles()
            ang:RotateAroundAxis(ang:Forward(), 90)
            ang:RotateAroundAxis(ang:Right(), 90)

            cam.Start3D2D(pos, ang, 0.1)
                draw.SimpleText("!", "DermaLarge", 0, 0, Color(255, 0, 0), TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
            cam.End3D2D()
        end
    end
end
```

### ❌ DON'T

```lua
if CLIENT then
    function ENT:OnMeleeAttack(enemy)
        -- BAD: Client trying to control gameplay logic
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
        self._customMode = "combat"  -- Won't affect server!

        -- BAD: Client setting networked vars
        self:SetNW2String("CustomMode", "combat")  -- Only server can set NW2 vars
    end
end

if SERVER then
    function ENT:CustomThink()
        -- BAD: Reading client input directly without validation
        if self:GetNW2Bool("ClientWantsToAttack") then
            self:Attack()  -- Security risk!
        end

        return 0.5
    end
end
```

### Why This Matters

Clients can be manipulated by malicious players. Never trust client input for gameplay decisions. The server must always validate and execute logic, then inform clients of the results.

## Best Practice 2: Use NW2 Vars Efficiently

NW2 (Networked Var 2) variables automatically sync from server to clients. Use them sparingly for client-visible state only.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- Only network what clients need to see
        self:SetNW2Int("DisplayPhase", 1)
        self:SetNW2Bool("IsEnraged", false)

        -- Keep server-only data private
        self._damageHistory = {}
        self._targetPriority = {}
        self._internalCooldowns = {}
    end

    function ENT:SetPhase(phase)
        local currentPhase = self:GetNW2Int("DisplayPhase", 1)

        -- Only network when value actually changes
        if currentPhase ~= phase then
            self:SetNW2Int("DisplayPhase", phase)
            self:OnPhaseChanged(phase)
        end
    end

    function ENT:OnTakeDamage(dmg)
        local health = self:Health()

        -- Check for enrage threshold
        if health <= self:GetMaxHealth() * 0.3 and not self:GetNW2Bool("IsEnraged") then
            self:SetNW2Bool("IsEnraged", true)
            self:EmitSound("boss/enrage.wav")
        end
    end
end

if CLIENT then
    function ENT:CustomDraw()
        local phase = self:GetNW2Int("DisplayPhase", 1)
        local enraged = self:GetNW2Bool("IsEnraged", false)

        -- Use networked data for visuals
        if enraged then
            local color = Color(255, 100, 100)
            render.SetColorModulation(color.r / 255, color.g / 255, color.b / 255)
            self:DrawModel()
            render.SetColorModulation(1, 1, 1)
        end
    end
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- BAD: Networking everything unnecessarily
        self:SetNW2Table("DamageHistory", {})  -- Large data structure
        self:SetNW2Table("TargetPriority", {})
        self:SetNW2Table("AllCooldowns", {})
        self:SetNW2Vector("InternalPathPos", Vector())
        self:SetNW2Float("PathProgress", 0)
    end

    function ENT:CustomThink()
        -- BAD: Updating networked vars every think
        self:SetNW2Int("DisplayPhase", self:CalculatePhase())  -- Even if unchanged
        self:SetNW2Vector("InternalPathPos", self:GetPathPos())  -- Server-only data
        self:SetNW2Float("PathProgress", self:GetPathProgress())  -- Not needed by clients

        -- BAD: Networking large tables
        table.insert(self._damageHistory, {time = CurTime(), damage = 10})
        self:SetNW2String("DamageHistoryJSON", util.TableToJSON(self._damageHistory))

        return 0.1
    end
end
```

### Why This Matters

Every NW2 variable update sends a network message to all clients. Updating 10 variables every 0.1 seconds for 20 NPCs means 2000 messages per second, causing significant lag and bandwidth waste.

## Best Practice 3: Use DrGBase's Net Library for Custom Messages

For complex data that needs to be sent once or to specific players, use DrGBase's net.DrG_* functions.

### ✅ DO

```lua
if SERVER then
    function ENT:AwardPlayerKill(ply, xp)
        -- Send targeted message to specific player
        net.Start("MyModNPCKillReward")
        net.WriteEntity(self)
        net.WriteUInt(xp, 16)
        net.Send(ply)
    end

    function ENT:OnDeath(dmg)
        local attacker = dmg:GetAttacker()

        if IsValid(attacker) and attacker:IsPlayer() then
            local xp = self:CalculateXPReward(attacker)
            self:AwardPlayerKill(attacker, xp)
        end
    end

    -- Register network string once in shared file or autorun
    util.AddNetworkString("MyModNPCKillReward")
end

if CLIENT then
    -- Receive targeted message
    net.Receive("MyModNPCKillReward", function()
        local npc = net.ReadEntity()
        local xp = net.ReadUInt(16)

        -- Display reward notification
        chat.AddText(Color(255, 255, 0), "You gained ", Color(0, 255, 0), tostring(xp), Color(255, 255, 0), " XP!")
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:OnDeath(dmg)
        local attacker = dmg:GetAttacker()

        if IsValid(attacker) and attacker:IsPlayer() then
            -- BAD: Using NW2 vars for one-time messages
            self:SetNW2Entity("LastKiller", attacker)
            self:SetNW2Int("KillXP", self:CalculateXPReward(attacker))

            -- This sends to ALL clients, not just the killer
            -- Also wastes network updates for temporary data
        end
    end
end

if CLIENT then
    function ENT:OnRemove()
        -- BAD: Trying to read one-time data after entity removed
        local killer = self:GetNW2Entity("LastKiller")
        local xp = self:GetNW2Int("KillXP")
        -- Entity might already be removed, data might be lost
    end
end
```

### Why This Matters

NW2 vars broadcast to all clients and persist until changed. For one-time events or player-specific messages, direct net messages are more efficient and appropriate.

## Best Practice 4: Batch Network Messages When Possible

Group related data into single messages instead of sending many small messages.

### ✅ DO

```lua
if SERVER then
    function ENT:SyncCompleteState(ply)
        -- Send all relevant state in one message
        net.Start("MyModNPCSyncState")
        net.WriteEntity(self)
        net.WriteUInt(self:Health(), 16)
        net.WriteUInt(self:GetMaxHealth(), 16)
        net.WriteUInt(self:GetPhase(), 8)
        net.WriteBool(self:IsEnraged())
        net.WriteVector(self:GetTarget())
        net.WriteTable(self:GetActiveBuffs())  -- Table sent once
        net.Send(ply)
    end

    function ENT:OnPlayerRequestState(ply)
        -- One comprehensive message instead of multiple small ones
        self:SyncCompleteState(ply)
    end
end

if CLIENT then
    net.Receive("MyModNPCSyncState", function()
        local npc = net.ReadEntity()
        if not IsValid(npc) then return end

        -- Read all data from single message
        local health = net.ReadUInt(16)
        local maxHealth = net.ReadUInt(16)
        local phase = net.ReadUInt(8)
        local enraged = net.ReadBool()
        local target = net.ReadVector()
        local buffs = net.ReadTable()

        -- Update client-side display
        npc._clientHealth = health
        npc._clientMaxHealth = maxHealth
        npc._clientPhase = phase
        npc._clientEnraged = enraged
        npc._clientTarget = target
        npc._clientBuffs = buffs
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:OnPlayerRequestState(ply)
        -- BAD: Sending many separate messages
        net.Start("MyModNPCHealth")
        net.WriteEntity(self)
        net.WriteUInt(self:Health(), 16)
        net.Send(ply)

        net.Start("MyModNPCMaxHealth")
        net.WriteEntity(self)
        net.WriteUInt(self:GetMaxHealth(), 16)
        net.Send(ply)

        net.Start("MyModNPCPhase")
        net.WriteEntity(self)
        net.WriteUInt(self:GetPhase(), 8)
        net.Send(ply)

        net.Start("MyModNPCEnraged")
        net.WriteEntity(self)
        net.WriteBool(self:IsEnraged())
        net.Send(ply)

        -- 4 messages instead of 1!
    end
end
```

### Why This Matters

Each net message has overhead. Sending 4 messages wastes bandwidth and CPU cycles. Batching related data reduces network traffic significantly.

## Best Practice 5: Validate All Client Input

Never trust data from clients. Always validate before processing.

### ✅ DO

```lua
if SERVER then
    util.AddNetworkString("MyModNPCCommand")

    function ENT:ProcessClientCommand(ply, command, arg)
        -- Validate player
        if not IsValid(ply) or not ply:IsPlayer() then
            return false
        end

        -- Validate relationship
        if self:GetRelationship(ply) ~= D_LI then
            return false  -- Not an ally
        end

        -- Validate distance
        if self:GetRangeTo(ply) > 500 then
            return false  -- Too far away
        end

        -- Validate command
        local validCommands = {
            ["follow"] = true,
            ["stay"] = true,
            ["attack"] = true
        }

        if not validCommands[command] then
            DrGBase.Print("Invalid command from player: " .. command)
            return false
        end

        -- Execute validated command
        if command == "follow" then
            self:SetFollowTarget(ply)
        elseif command == "stay" then
            self:SetStayPosition(self:GetPos())
        elseif command == "attack" then
            if IsValid(arg) and arg:IsNPC() or arg:IsPlayer() then
                self:SetEnemy(arg)
            end
        end

        return true
    end

    net.Receive("MyModNPCCommand", function(len, ply)
        local npc = net.ReadEntity()
        local command = net.ReadString()
        local arg = net.ReadEntity()

        if not IsValid(npc) or not npc.IsDrGNextbot then
            return  -- Invalid entity
        end

        npc:ProcessClientCommand(ply, command, arg)
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
        local arg = net.ReadEntity()

        -- BAD: No validation at all
        if command == "follow" then
            npc:SetFollowTarget(ply)
        elseif command == "stay" then
            npc:SetStayPosition(npc:GetPos())
        elseif command == "attack" then
            npc:SetEnemy(arg)
        elseif command == "teleport" then
            -- BAD: Accepting dangerous commands
            npc:SetPos(arg:GetPos())
        elseif command == "kill" then
            -- BAD: Client can kill any NPC
            npc:TakeDamage(9999)
        elseif command == "remove" then
            -- BAD: Client can remove entities
            npc:Remove()
        end
    end)
end
```

### Why This Matters

Malicious clients can send arbitrary data. Without validation, they can exploit your addon to cheat, grief, or crash the server. Always validate sender, target, distance, and command validity.

## Best Practice 6: Use Prediction Carefully

Client-side prediction can make interactions feel more responsive, but it must match server logic exactly.

### ✅ DO

```lua
-- Shared prediction for simple, safe operations
function ENT:CanInteract(ply)
    if not IsValid(ply) or not ply:IsPlayer() then
        return false
    end

    if self:GetRangeTo(ply) > 100 then
        return false
    end

    if self:GetRelationship(ply) ~= D_LI then
        return false
    end

    return true
end

if SERVER then
    function ENT:OnUse(ply)
        -- Server validates then executes
        if not self:CanInteract(ply) then
            return
        end

        -- Server authoritative action
        self:SetFollowTarget(ply)
        self:EmitSound("npc/scanner/scanner_talk1.wav")

        -- Notify client of success
        net.Start("MyModNPCInteract")
        net.WriteEntity(self)
        net.WriteBool(true)
        net.Send(ply)
    end

    util.AddNetworkString("MyModNPCInteract")
end

if CLIENT then
    function ENT:DrawHint()
        local ply = LocalPlayer()

        -- Client predicts interaction availability
        if self:CanInteract(ply) then
            -- Draw "Press E to command" hint
            local pos = self:GetPos() + Vector(0, 0, 80)
            local ang = ply:EyeAngles()
            ang:RotateAroundAxis(ang:Forward(), 90)
            ang:RotateAroundAxis(ang:Right(), 90)

            cam.Start3D2D(pos, ang, 0.1)
                draw.SimpleText("Press E to command", "DermaDefault", 0, 0, Color(255, 255, 255), TEXT_ALIGN_CENTER)
            cam.End3D2D()
        end
    end
end
```

### ❌ DON'T

```lua
if CLIENT then
    function ENT:OnUse(ply)
        -- BAD: Client trying to execute gameplay logic
        if self:CanInteract(ply) then
            self:SetFollowTarget(ply)  -- Only changes client-side!
            self:EmitSound("npc/scanner/scanner_talk1.wav")  -- Out of sync with server

            -- State diverges from server
        end
    end
end

if SERVER then
    function ENT:OnUse(ply)
        -- BAD: Different logic than client
        if self:GetRangeTo(ply) > 200 then  -- Different range than client check
            return
        end

        self:SetFollowTarget(ply)
        -- Client and server are now out of sync
    end
end
```

### Why This Matters

When client and server prediction logic differs, players see one thing happen locally but then it gets "corrected" by the server, causing jarring visual glitches. Keep prediction logic identical and limited to safe operations.

## Best Practice 7: Handle Late-Joining Players

Players who join mid-game need to receive the current state of existing entities.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        self._customPhase = 1
        self._isEnraged = false

        -- Set initial networked state
        self:SetNW2Int("CustomPhase", 1)
        self:SetNW2Bool("IsEnraged", false)

        -- NW2 vars automatically sync to late-joiners
    end

    -- For complex state that can't use NW2, use PlayerInitialSpawn hook
    hook.Add("PlayerInitialSpawn", "MyModSyncNPCState", function(ply)
        timer.Simple(2, function()  -- Small delay for client to be ready
            if not IsValid(ply) then return end

            -- Send current state of all custom NPCs
            for _, npc in ipairs(ents.FindByClass("npc_mymod_*")) do
                if npc.IsDrGNextbot then
                    net.Start("MyModNPCSyncState")
                    net.WriteEntity(npc)
                    net.WriteUInt(npc._customPhase or 1, 8)
                    net.WriteBool(npc._isEnraged or false)
                    net.Send(ply)
                end
            end
        end)
    end)

    util.AddNetworkString("MyModNPCSyncState")
end

if CLIENT then
    net.Receive("MyModNPCSyncState", function()
        local npc = net.ReadEntity()
        if not IsValid(npc) then return end

        npc._clientPhase = net.ReadUInt(8)
        npc._clientEnraged = net.ReadBool()
    end)
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- BAD: Not networking initial state
        self._customPhase = 1
        self._isEnraged = false

        -- Late-joiners won't know this state
    end

    function ENT:SetPhase(phase)
        self._customPhase = phase

        -- BAD: Only sending to current players
        for _, ply in ipairs(player.GetAll()) do
            net.Start("MyModSetPhase")
            net.WriteEntity(self)
            net.WriteUInt(phase, 8)
            net.Send(ply)
        end

        -- Player who joins 5 seconds later won't receive this
    end
end
```

### Why This Matters

Players who join after NPCs spawn won't see effects or behavior correctly if they don't receive the current state. NW2 vars automatically handle this, but custom net messages need explicit handling.

## Best Practice 8: Clean Up Network Receivers

Remove network receivers when entities are removed to prevent errors and memory leaks.

### ✅ DO

```lua
if SERVER then
    function ENT:CustomInitialize()
        local netName = "MyModNPCCommand_" .. self:EntIndex()

        util.AddNetworkString(netName)

        -- Store net receiver identifier
        self._netReceiverName = netName

        -- Register receiver
        net.Receive(netName, function(len, ply)
            if not IsValid(self) then return end
            self:ProcessCommand(ply, net.ReadString())
        end)

        -- Note: In practice, use a global receiver and entity lookup
        -- This is just for demonstration
    end
end

-- Better approach: Use one global receiver
if SERVER then
    util.AddNetworkString("MyModNPCCommand")

    net.Receive("MyModNPCCommand", function(len, ply)
        local npc = net.ReadEntity()
        local command = net.ReadString()

        -- Validate entity still exists
        if not IsValid(npc) or not npc.IsDrGNextbot then
            return
        end

        npc:ProcessCommand(ply, command)
    end)

    -- Single receiver, no cleanup needed per-entity
end
```

### ❌ DON'T

```lua
if SERVER then
    function ENT:CustomInitialize()
        -- BAD: Creating network strings per entity
        util.AddNetworkString("MyModNPCCommand_" .. self:EntIndex())

        net.Receive("MyModNPCCommand_" .. self:EntIndex(), function(len, ply)
            -- BAD: No IsValid check
            self:ProcessCommand(ply, net.ReadString())
            -- Will error if self is removed
        end)

        -- No cleanup - receiver stays registered forever
    end
end
```

### Why This Matters

Network receivers persist after entities are removed unless explicitly cleaned up. Use global receivers with entity lookup instead of per-entity receivers to avoid accumulation and memory leaks.

## Real-World Example: Boss NPC with Proper Networking

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Networked Boss"
ENT.SpawnHealth = 5000

-- Only network essential client-visible state
if SERVER then
    util.AddNetworkString("BossPhaseChange")
    util.AddNetworkString("BossAbilityUsed")
    util.AddNetworkString("BossPlayerDamaged")
end

if SERVER then
    function ENT:CustomInitialize()
        -- Server-only data (not networked)
        self._abilityCooldowns = {}
        self._damageHistory = {}
        self._targetPriorities = {}

        -- Client-visible state (networked via NW2)
        self:SetNW2Int("BossPhase", 1)
        self:SetNW2Bool("BossEnraged", false)
    end

    function ENT:SetPhase(phase)
        local currentPhase = self:GetNW2Int("BossPhase", 1)

        if currentPhase ~= phase then
            self:SetNW2Int("BossPhase", phase)

            -- Send one-time event to all clients
            net.Start("BossPhaseChange")
            net.WriteEntity(self)
            net.WriteUInt(phase, 8)
            net.WriteUInt(currentPhase, 8)
            net.Broadcast()

            self:OnPhaseChanged(phase)
        end
    end

    function ENT:UseAbility(name, target)
        -- Server validates and executes
        if not self:CanUseAbility(name) then
            return false
        end

        -- Execute ability (server authoritative)
        local success = self:ExecuteAbility(name, target)

        if success then
            -- Notify all clients for visual effects
            net.Start("BossAbilityUsed")
            net.WriteEntity(self)
            net.WriteString(name)
            net.WriteEntity(target or NULL)
            net.Broadcast()
        end

        return success
    end

    function ENT:OnTakeDamage(dmg)
        local attacker = dmg:GetAttacker()

        if IsValid(attacker) and attacker:IsPlayer() then
            -- Send targeted message to damaging player
            net.Start("BossPlayerDamaged")
            net.WriteEntity(self)
            net.WriteFloat(dmg:GetDamage())
            net.Send(attacker)
        end

        -- Check enrage threshold
        if self:Health() <= self:GetMaxHealth() * 0.3 then
            if not self:GetNW2Bool("BossEnraged") then
                self:SetNW2Bool("BossEnraged", true)
                self:EmitSound("boss/enrage.wav")
            end
        end
    end
end

if CLIENT then
    -- Receive phase change event
    net.Receive("BossPhaseChange", function()
        local boss = net.ReadEntity()
        local newPhase = net.ReadUInt(8)
        local oldPhase = net.ReadUInt(8)

        if not IsValid(boss) then return end

        -- Create visual effect for phase change
        local effectData = EffectData()
        effectData:SetOrigin(boss:GetPos())
        effectData:SetEntity(boss)
        util.Effect("boss_phase_change", effectData)

        -- Screen shake for nearby players
        if LocalPlayer():GetPos():Distance(boss:GetPos()) < 1000 then
            util.ScreenShake(boss:GetPos(), 10, 5, 2, 1000)
        end
    end)

    -- Receive ability used event
    net.Receive("BossAbilityUsed", function()
        local boss = net.ReadEntity()
        local abilityName = net.ReadString()
        local target = net.ReadEntity()

        if not IsValid(boss) then return end

        -- Play client-side visual effects
        boss:CreateAbilityEffect(abilityName, target)
    end)

    -- Receive damage feedback (player-specific)
    net.Receive("BossPlayerDamaged", function()
        local boss = net.ReadEntity()
        local damage = net.ReadFloat()

        if not IsValid(boss) then return end

        -- Show damage number
        boss:ShowDamageNumber(damage)
    end)

    function ENT:CustomDraw()
        -- Read networked state
        local phase = self:GetNW2Int("BossPhase", 1)
        local enraged = self:GetNW2Bool("BossEnraged", false)

        -- Apply visual effects based on state
        if enraged then
            render.SetColorModulation(1, 0.5, 0.5)
            self:DrawModel()
            render.SetColorModulation(1, 1, 1)
        else
            self:DrawModel()
        end

        -- Draw phase indicator
        self:DrawPhaseIndicator(phase)
    end

    function ENT:CreateAbilityEffect(name, target)
        -- Client-side effect creation
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        effectData:SetEntity(self)

        if IsValid(target) then
            effectData:SetStart(target:GetPos())
        end

        util.Effect("boss_ability_" .. name, effectData)
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

This boss NPC demonstrates proper networking: NW2 vars for persistent state, net messages for one-time events, server authority for all logic, and efficient targeted messages for player-specific feedback.

## Tips and Recommendations

1. **Minimize NW2 Usage**: Only network what clients need to see. Keep gameplay logic server-side.

2. **Use Net Messages for Events**: One-time events (ability used, phase changed) should use net messages, not NW2 vars.

3. **Validate Everything**: Never trust client input. Always validate sender, target, and data.

4. **Batch When Possible**: Group related data into single messages to reduce overhead.

5. **Test in Multiplayer**: Always test with multiple clients to catch networking issues early.

6. **Monitor Network Usage**: Use `net_graph 3` to monitor network traffic and identify issues.

7. **Document Your Protocol**: Comment which messages are sent when and what data they contain.

8. **Consider Lag**: Design your networking to handle 50-100ms latency gracefully.
