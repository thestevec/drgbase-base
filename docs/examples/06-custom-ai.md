# Custom AI Behavior

## Overview

DrGBase provides a flexible AI system that can be customized using behavior coroutines, state machines, and custom logic. This guide covers advanced AI programming techniques.

## AI Behavior Modes

DrGBase has built-in AI behaviors, but you can override them with custom logic:

### Default AI Behavior

```lua
-- Default behavior (automatic):
-- 1. Detect enemies
-- 2. Chase enemies
-- 3. Attack when in range
-- 4. Patrol when idle
```

### Custom AI Mode

```lua
function ENT:AIBehaviour()
    -- This function replaces ALL default AI
    -- Runs as a coroutine (can use self:Wait())

    while true do
        -- Your custom AI logic here
        self:MyCustomBehavior()
        self:Wait(1)
    end
end
```

**Important:** When `AIBehaviour()` exists, default AI is disabled!

## Simple Custom AI Examples

### 1. Patrol-Only AI

```lua
function ENT:AIBehaviour()
    while true do
        -- Add random patrol point
        local pos = self:RandomPos(2000)
        self:AddPatrolPos(pos)

        -- Walk to patrol point
        self:FollowPath(pos)

        -- Wait at patrol point
        self:Wait(math.random(3, 7))
    end
end
```

### 2. Flee from Players

```lua
function ENT:AIBehaviour()
    while true do
        local nearbyPlayers = self:FindPlayersInRange(500)

        if #nearbyPlayers > 0 then
            -- Run away from closest player
            local closest = nearbyPlayers[1]
            local fleePos = self:GetPos() + (self:GetPos() - closest:GetPos()):GetNormalized() * 500

            self:FollowPath(fleePos)
        else
            -- Idle behavior
            self:AddPatrolPos(self:RandomPos(1000))
        end

        self:Wait(1)
    end
end

function ENT:FindPlayersInRange(range)
    local players = {}
    for _, ply in ipairs(player.GetAll()) do
        if self:GetPos():Distance(ply:GetPos()) <= range then
            table.insert(players, ply)
        end
    end
    return players
end
```

### 3. Guard Position

```lua
function ENT:CustomInitialize()
    self.GuardPosition = self:GetPos()
    self.GuardRadius = 300
end

function ENT:AIBehaviour()
    while true do
        local enemy = self:GetEnemy()

        if IsValid(enemy) then
            local distFromGuard = self.GuardPosition:Distance(self:GetPos())

            if distFromGuard < self.GuardRadius then
                -- Attack if within guard radius
                self:AttackEntity(enemy)
            else
                -- Return to guard position
                self:FollowPath(self.GuardPosition)
            end
        else
            -- No enemy, stay at guard position
            if self.GuardPosition:Distance(self:GetPos()) > 50 then
                self:FollowPath(self.GuardPosition)
            end
        end

        self:Wait(0.5)
    end
end
```

### 4. Follow Leader

```lua
function ENT:AIBehaviour()
    while true do
        local leader = self:FindLeader()

        if IsValid(leader) then
            local distance = self:GetPos():Distance(leader:GetPos())

            if distance > 200 then
                -- Follow leader
                self:FollowPath(leader:GetPos())
            elseif distance < 100 then
                -- Too close, back off
                local backPos = self:GetPos() + (self:GetPos() - leader:GetPos()):GetNormalized() * 50
                self:FollowPath(backPos)
            end
            -- Otherwise stay in formation
        else
            -- No leader, wander
            self:AddPatrolPos(self:RandomPos(1000))
        end

        self:Wait(0.5)
    end
end

function ENT:FindLeader()
    -- Find nearest friendly NPC
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 1000)) do
        if ent:GetClass() == "npc_drg_squad_leader" then
            return ent
        end
    end
    return nil
end
```

## Advanced AI Patterns

### State Machine

```lua
function ENT:CustomInitialize()
    self.AIState = "idle"
    self.StateTimer = 0
end

function ENT:AIBehaviour()
    while true do
        if self.AIState == "idle" then
            self:State_Idle()
        elseif self.AIState == "patrol" then
            self:State_Patrol()
        elseif self.AIState == "alert" then
            self:State_Alert()
        elseif self.AIState == "chase" then
            self:State_Chase()
        elseif self.AIState == "attack" then
            self:State_Attack()
        end

        self:Wait(0.1)
    end
end

function ENT:State_Idle()
    self.StateTimer = self.StateTimer + 0.1

    if self.StateTimer > 5 then
        self:ChangeState("patrol")
    end

    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        self:ChangeState("alert")
    end
end

function ENT:State_Patrol()
    if not self:HasPatrolPos() then
        self:AddPatrolPos(self:RandomPos(1500))
    end

    local enemy = self:GetEnemy()
    if IsValid(enemy) then
        self:ChangeState("alert")
    end

    if self:ReachedPatrol() then
        self:ChangeState("idle")
    end
end

function ENT:State_Alert()
    self:EmitSound("Alert.Sound")
    local enemy = self:GetEnemy()

    if not IsValid(enemy) then
        self:ChangeState("idle")
        return
    end

    self:FaceTowards(enemy)
    self:Wait(1) -- Look at enemy for 1 second

    self:ChangeState("chase")
end

function ENT:State_Chase()
    local enemy = self:GetEnemy()

    if not IsValid(enemy) then
        self:ChangeState("idle")
        return
    end

    self:FollowPath(enemy:GetPos())

    if self:IsInRange(enemy:GetPos(), self.MeleeAttackRange) then
        self:ChangeState("attack")
    end
end

function ENT:State_Attack()
    local enemy = self:GetEnemy()

    if not IsValid(enemy) then
        self:ChangeState("idle")
        return
    end

    if not self:IsInRange(enemy:GetPos(), self.MeleeAttackRange + 50) then
        self:ChangeState("chase")
        return
    end

    -- Perform attack
    self:OnMeleeAttack(enemy)
    self:Wait(1.5) -- Attack cooldown
end

function ENT:ChangeState(newState)
    self.AIState = newState
    self.StateTimer = 0
    print("State changed to: " .. newState)
end
```

### Behavior Tree

```lua
function ENT:AIBehaviour()
    while true do
        self:BehaviorTree()
        self:Wait(0.1)
    end
end

function ENT:BehaviorTree()
    -- Priority-based behavior selection
    if self:Behavior_HandleCombat() then return end
    if self:Behavior_Investigate() then return end
    if self:Behavior_Patrol() then return end
    self:Behavior_Idle()
end

function ENT:Behavior_HandleCombat()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return false end

    local distance = self:GetPos():Distance(enemy:GetPos())

    if distance > 1000 then
        -- Too far, disengage
        return false
    elseif distance > self.RangeAttackRange then
        -- Chase
        self:FollowPath(enemy:GetPos())
    elseif distance > self.MeleeAttackRange then
        -- In range attack range
        if self.RangeAttackRange > 0 then
            self:OnRangeAttack(enemy)
        else
            self:FollowPath(enemy:GetPos())
        end
    else
        -- Melee range
        self:OnMeleeAttack(enemy)
    end

    return true -- Behavior handled
end

function ENT:Behavior_Investigate()
    if not self.InvestigatePos then return false end

    if self:IsInRange(self.InvestigatePos, 50) then
        -- Reached investigation point
        self:FaceTowards(self.InvestigatePos)
        self:Wait(2)
        self.InvestigatePos = nil
        return false
    end

    self:FollowPath(self.InvestigatePos)
    return true
end

function ENT:Behavior_Patrol()
    if not self:HasPatrolPos() then return false end

    if self:ReachedPatrol() then
        self:Wait(math.random(2, 5))
        return true
    end

    return true
end

function ENT:Behavior_Idle()
    self:AddPatrolPos(self:RandomPos(1500))
    self:Wait(1)
end

-- Trigger investigation
function ENT:OnHear(sound)
    if not IsValid(self:GetEnemy()) then
        self.InvestigatePos = sound:GetSoundPosition()
    end
end
```

### Squad AI

```lua
ENT.IsSquadLeader = false
ENT.SquadMembers = {}

function ENT:CustomInitialize()
    if math.random(4) == 1 then
        self.IsSquadLeader = true
        self:RecruitSquad()
    else
        self:FindSquadLeader()
    end
end

function ENT:AIBehaviour()
    while true do
        if self.IsSquadLeader then
            self:LeaderAI()
        else
            self:MemberAI()
        end
        self:Wait(0.5)
    end
end

function ENT:LeaderAI()
    local enemy = self:GetEnemy()

    if IsValid(enemy) then
        -- Command squad to attack
        for _, member in ipairs(self.SquadMembers) do
            if IsValid(member) then
                member:SetEnemy(enemy)
            end
        end

        -- Leader attacks
        self:AttackEntity(enemy)
    else
        -- Lead patrol
        if not self:HasPatrolPos() then
            self:AddPatrolPos(self:RandomPos(2000))
        end
    end

    -- Clean up dead members
    for i = #self.SquadMembers, 1, -1 do
        if not IsValid(self.SquadMembers[i]) then
            table.remove(self.SquadMembers, i)
        end
    end
end

function ENT:MemberAI()
    if not IsValid(self.SquadLeader) then
        -- Leader died, become independent
        self.IsSquadLeader = true
        return
    end

    local enemy = self:GetEnemy()

    if IsValid(enemy) then
        -- Attack enemy
        self:AttackEntity(enemy)
    else
        -- Follow leader
        local distance = self:GetPos():Distance(self.SquadLeader:GetPos())
        if distance > 200 then
            self:FollowPath(self.SquadLeader:GetPos())
        end
    end
end

function ENT:RecruitSquad()
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
        if ent:GetClass() == self:GetClass() and ent != self then
            if not ent.IsSquadLeader and not IsValid(ent.SquadLeader) then
                ent.SquadLeader = self
                table.insert(self.SquadMembers, ent)
            end
        end
    end
end

function ENT:FindSquadLeader()
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
        if ent:GetClass() == self:GetClass() and ent.IsSquadLeader then
            self.SquadLeader = ent
            table.insert(ent.SquadMembers, self)
            return
        end
    end
end
```

## Mixing Default AI with Custom Logic

If you want to use default AI but add custom behavior:

### Override Specific Hooks

```lua
-- Keep default AI, but customize enemy handling
function ENT:HandleEnemy(enemy)
    if not IsValid(enemy) then return end

    local distance = self:GetPos():Distance(enemy:GetPos())

    if distance < 200 then
        -- Custom close-range behavior
        self:FleeFromEnemy(enemy)
    else
        -- Use default chase behavior
        self:ChaseEnemy(enemy)
    end
end

function ENT:FleeFromEnemy(enemy)
    local fleePos = self:GetPos() + (self:GetPos() - enemy:GetPos()):GetNormalized() * 500
    self:FollowPath(fleePos)
end
```

### Custom Think Logic

```lua
function ENT:CustomThink()
    -- Runs every tick, doesn't override AI

    -- Check for nearby explosions
    for _, ent in ipairs(ents.FindInSphere(self:GetPos(), 500)) do
        if ent:GetClass() == "env_explosion" then
            -- React to explosion
            self:Flinch()
            self:EmitSound("Fear.Sound")
        end
    end

    -- Regenerate health over time
    if self:Health() < self:GetMaxHealth() then
        self:SetHealth(self:Health() + 0.1)
    end
end
```

## AI Utility Functions

### Path Finding

```lua
-- Follow path to position
local result = self:FollowPath(pos)
-- Returns: "complete", "failed", "stuck", "timeout", "invalid"

-- Check if path exists
if self:ComputePath(pos) then
    self:FollowPath(pos)
else
    print("No path available")
end

-- Refresh current path
self:RefreshPath()

-- Get current path
local path = self:GetPath()
```

### Detection

```lua
-- Check if can see entity
if self:Visible(entity) then
    print("Can see entity")
end

-- Find enemies in range
local enemies = self:FindEnemiesInRange(500)

-- Check if in range
if self:IsInRange(pos, 100) then
    print("In range")
end
```

### Utility

```lua
-- Wait (in coroutine)
self:Wait(2) -- Wait 2 seconds

-- Timer (one-shot)
self:Timer(2, function(self)
    print("Timer complete")
end)

-- Loop timer (repeating)
self:LoopTimer(1, function(self)
    print("Every second")
end)

-- Random position
local pos = self:RandomPos(500) -- Within 500 units
```

## Debugging AI

### Print Statements

```lua
function ENT:AIBehaviour()
    while true do
        print("AI State:", self.AIState)
        print("Enemy:", tostring(self:GetEnemy()))
        print("Position:", self:GetPos())

        self:MyAILogic()
        self:Wait(1)
    end
end
```

### Visual Debug

```lua
function ENT:CustomThink()
    if GetConVar("drgbase_debug"):GetBool() then
        -- Draw target position
        if self.TargetPos then
            debugoverlay.Cross(self.TargetPos, 20, 0.1, Color(255, 0, 0), true)
            debugoverlay.Line(self:GetPos(), self.TargetPos, 0.1, Color(0, 255, 0), true)
        end

        -- Draw detection range
        debugoverlay.Sphere(self:GetPos(), self.SightRange, 0.1, Color(0, 0, 255, 10), true)
    end
end
```

## Source Reference

Base AI: `lua/entities/drgbase_nextbot/ai.lua`
Behaviors: `lua/entities/drgbase_nextbot/behaviors.lua`

## Related Documentation

- [Pathfinding System](../reference/pathfinding.md)
- [Detection System](../reference/detection.md)
- [Hooks Reference](../reference/hooks.md)
