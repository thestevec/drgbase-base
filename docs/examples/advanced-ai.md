# Advanced AI Behaviors Example

This example demonstrates advanced AI techniques including state machines, squad behavior, cover systems, and tactical decision-making.

## State Machine System

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Tactical Soldier"
ENT.Category = "Advanced NPCs"
ENT.Models = {"models/player/Group03/male_01.mdl"}
ENT.Factions = {FACTION_REBELS}

-- AI States
ENT.AI_STATE_PATROL = 1
ENT.AI_STATE_INVESTIGATE = 2
ENT.AI_STATE_COMBAT = 3
ENT.AI_STATE_COVER = 4
ENT.AI_STATE_RETREAT = 5

if SERVER then

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- AI state tracking
        self.AIState = self.AI_STATE_PATROL
        self.LastKnownEnemyPos = nil
        self.CoverPosition = nil
        self.AmmoPercent = 100
    end

    function ENT:CustomThink()
        -- State machine
        if self.AIState == self.AI_STATE_PATROL then
            self:StatePatrol()
        elseif self.AIState == self.AI_STATE_INVESTIGATE then
            self:StateInvestigate()
        elseif self.AIState == self.AI_STATE_COMBAT then
            self:StateCombat()
        elseif self.AIState == self.AI_STATE_COVER then
            self:StateCover()
        elseif self.AIState == self.AI_STATE_RETREAT then
            self:StateRetreat()
        end
    end

    -- State: Patrol
    function ENT:StatePatrol()
        local enemy = self:GetEnemy()
        if IsValid(enemy) then
            self:SetState(self.AI_STATE_COMBAT)
            return
        end

        -- Check for suspicious sounds
        local sound = self:GetLoudestSound()
        if sound and sound.distance < 500 then
            self.LastKnownEnemyPos = sound.position
            self:SetState(self.AI_STATE_INVESTIGATE)
        end
    end

    -- State: Investigate
    function ENT:StateInvestigate()
        local enemy = self:GetEnemy()
        if IsValid(enemy) then
            self:SetState(self.AI_STATE_COMBAT)
            return
        end

        if self.LastKnownEnemyPos then
            self:SetTarget(self.LastKnownEnemyPos)

            if self:GetPos():Distance(self.LastKnownEnemyPos) < 50 then
                -- Finished investigating
                self.LastKnownEnemyPos = nil
                self:SetState(self.AI_STATE_PATROL)
            end
        end
    end

    -- State: Combat
    function ENT:StateCombat()
        local enemy = self:GetEnemy()
        if not IsValid(enemy) then
            self:SetState(self.AI_STATE_INVESTIGATE)
            return
        end

        -- Low health? Retreat
        if self:Health() / self:GetMaxHealth() < 0.3 then
            self:SetState(self.AI_STATE_RETREAT)
            return
        end

        -- Low ammo? Take cover
        if self.AmmoPercent < 30 then
            self:SetState(self.AI_STATE_COVER)
            return
        end

        -- Engage enemy
        self:FaceTo(enemy:EyePos())
    end

    -- State: Cover
    function ENT:StateCover()
        if not self.CoverPosition then
            self.CoverPosition = self:FindCover()
        end

        if self.CoverPosition then
            self:SetTarget(self.CoverPosition)

            if self:GetPos():Distance(self.CoverPosition) < 100 then
                -- In cover, reload
                self:Wait(2)  -- Reload time
                self.AmmoPercent = 100
                self.CoverPosition = nil
                self:SetState(self.AI_STATE_COMBAT)
            end
        else
            -- No cover found, continue combat
            self:SetState(self.AI_STATE_COMBAT)
        end
    end

    -- State: Retreat
    function ENT:StateRetreat()
        local enemy = self:GetEnemy()
        if IsValid(enemy) then
            -- Run away from enemy
            local awayDir = (self:GetPos() - enemy:GetPos()):GetNormalized()
            local retreatPos = self:GetPos() + awayDir * 500
            self:SetTarget(retreatPos)

            -- Recovered health? Return to combat
            if self:Health() / self:GetMaxHealth() > 0.6 then
                self:SetState(self.AI_STATE_COMBAT)
            end
        else
            self:SetState(self.AI_STATE_PATROL)
        end
    end

    function ENT:SetState(newState)
        if self.AIState == newState then return end

        -- Exit current state
        if self.AIState == self.AI_STATE_COVER then
            self.CoverPosition = nil
        end

        -- Enter new state
        self.AIState = newState

        if newState == self.AI_STATE_RETREAT then
            self:EmitSound("vo/npc/male01/pain0"..math.random(1,9)..".wav")
        end
    end

    function ENT:FindCover()
        -- Simple cover finding (in production use navmesh)
        for i = 1, 8 do
            local angle = (360 / 8) * i
            local dir = Vector(math.cos(math.rad(angle)), math.sin(math.rad(angle)), 0)
            local testPos = self:GetPos() + dir * 300

            -- Check if position has cover
            local tr = util.TraceLine({
                start = testPos + Vector(0, 0, 50),
                endpos = testPos + Vector(0, 0, 50) - dir * 100,
                filter = self
            })

            if tr.Hit and tr.Entity:IsWorld() then
                return testPos
            end
        end

        return nil
    end

end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Squad Behavior

```lua
-- Squad coordination
function ENT:CustomInitialize()
    -- Join or create squad
    self.SquadID = "squad_" .. math.floor(self:GetPos():Length() / 1000)
    self:JoinSquad(self.SquadID)
end

function ENT:JoinSquad(squadID)
    _G.DrGSquads = _G.DrGSquads or {}
    _G.DrGSquads[squadID] = _G.DrGSquads[squadID] or {}
    table.insert(_G.DrGSquads[squadID], self)
    self.SquadID = squadID
end

function ENT:GetSquadMembers()
    if not self.SquadID or not _G.DrGSquads[self.SquadID] then return {} end
    return _G.DrGSquads[self.SquadID]
end

function ENT:OnNewEnemy(enemy)
    -- Alert squad
    for _, ally in ipairs(self:GetSquadMembers()) do
        if IsValid(ally) and ally ~= self then
            ally:AddTargetEntity(enemy)
            ally:EmitSound("npc/metropolice/vo/contactconfirmprosecuting.wav")
        end
    end
end

function ENT:OnRemove()
    -- Leave squad
    if self.SquadID and _G.DrGSquads[self.SquadID] then
        table.RemoveByValue(_G.DrGSquads[self.SquadID], self)
    end
end
```

## Flanking Behavior

```lua
function ENT:OnChaseEnemy(enemy)
    if not IsValid(enemy) then return end

    -- Try to flank instead of direct approach
    if CurTime() > (self.NextFlankCheck or 0) then
        self.NextFlankCheck = CurTime() + 3

        local toEnemy = (enemy:GetPos() - self:GetPos()):GetNormalized()
        local flankAngle = math.random(0, 1) == 0 and 90 or -90
        local flankDir = toEnemy:Angle()
        flankDir:RotateAroundAxis(Vector(0, 0, 1), flankAngle)

        local flankPos = enemy:GetPos() + flankDir:Forward() * 400
        self:AddPatrolPos(flankPos)
    end
end
```

## Suppression Fire

```lua
function ENT:StateCombat()
    local enemy = self:GetEnemy()
    if not IsValid(enemy) then return end

    -- Suppression fire - make enemy take cover
    if CurTime() > (self.NextSuppress or 0) then
        self.NextSuppress = CurTime() + 0.1

        -- Fire near enemy, not at them
        local nearTarget = enemy:GetPos() + VectorRand() * 50
        self:FaceTo(nearTarget)

        -- Apply suppression effect
        if enemy:IsPlayer() then
            enemy:SetNW2Float("Suppression", CurTime() + 1)
        end
    end
end
```

## Dynamic Difficulty

```lua
function ENT:CustomInitialize()
    -- Scale based on player skill
    local playerCount = #player.GetAll()
    local avgPlayerHealth = 0

    for _, ply in ipairs(player.GetAll()) do
        avgPlayerHealth = avgPlayerHealth + ply:Health()
    end
    avgPlayerHealth = avgPlayerHealth / math.max(playerCount, 1)

    -- Adjust NPC stats
    local difficultyMult = math.Clamp(avgPlayerHealth / 100, 0.5, 2.0)

    self:SetHealth(self.SpawnHealth * difficultyMult)
    self.WeaponAccuracy = self.WeaponAccuracy * difficultyMult
end
```

## Next Steps

- Apply to [Boss NPC Example](./boss-npc.md) for complex encounters
- Combine with [Custom Weapons](./custom-weapon.md) for tactical combat
- Use [Faction System](./faction-system.md) for team coordination
