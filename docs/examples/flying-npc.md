# Flying NPC Example

This example demonstrates creating a flying/hovering NPC that navigates in 3D space. Perfect for aerial enemies, drones, or flying creatures.

## Overview

Flying NPCs in DrGBase use special locomotion properties to move in 3D space without gravity constraints.

## Complete Code

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Misc --
ENT.PrintName = "Flying Drone"
ENT.Category = "Custom NPCs"
ENT.Models = {"models/Combine_Helicopter/helicopter_bomb01.mdl"}
ENT.CollisionBounds = Vector(15, 15, 15)
ENT.BloodColor = BLOOD_COLOR_MECH

-- Stats --
ENT.SpawnHealth = 70

-- Flying --
ENT.Fly = true  -- Enable flying
ENT.FlyHeight = 150  -- Preferred hover height
ENT.FlyClimbHeight = 100  -- How high it can climb

-- AI --
ENT.RangeAttackRange = 1000
ENT.MeleeAttackRange = 0
ENT.ReachEnemyRange = 600
ENT.AvoidEnemyRange = 300

-- Relationships --
ENT.Factions = {FACTION_COMBINE}

-- Movements --
ENT.RunSpeed = 200
ENT.WalkSpeed = 100
ENT.Acceleration = 300

-- Detection --
ENT.SightRange = 2000
ENT.SightFOV = 180

-- Sounds --
ENT.LoopSounds = {"npc/scanner/scanner_scan_loop2.wav"}

if SERVER then

    function ENT:CustomInitialize()
        self:SetDefaultRelationship(D_HT)

        -- Add glow effect
        self:CreateLight(Color(255, 50, 50), 150)

        -- Start slightly randomized height
        local pos = self:GetPos()
        pos.z = pos.z + math.random(-20, 20)
        self:SetPos(pos)
    end

    function ENT:CreateLight(color, brightness)
        local light = ents.Create("light_dynamic")
        if IsValid(light) then
            light:SetKeyValue("brightness", brightness)
            light:SetKeyValue("distance", 200)
            light:SetPos(self:GetPos())
            light:SetParent(self)
            light:SetColor(color)
            light:Spawn()
            light:Activate()
            light:Fire("TurnOn", "", 0)
            self:DeleteOnRemove(light)
        end
    end

    -- Custom flight behavior --

    function ENT:CustomThink()
        -- Bob up and down slightly
        local time = CurTime()
        local bobAmount = math.sin(time * 2) * 5
        local targetHeight = self.FlyHeight + bobAmount

        -- Adjust height smoothly
        local pos = self:GetPos()
        local tr = util.TraceLine({
            start = pos,
            endpos = pos - Vector(0, 0, 10000),
            filter = self
        })

        if tr.Hit then
            local currentHeight = pos.z - tr.HitPos.z
            if currentHeight < targetHeight then
                self:SetVelocity(self:GetVelocity() + Vector(0, 0, 2))
            elseif currentHeight > targetHeight then
                self:SetVelocity(self:GetVelocity() + Vector(0, 0, -2))
            end
        end
    end

    -- AI --

    function ENT:OnRangeAttack(enemy)
        self:FaceTo(enemy:EyePos())
        self:PauseCoroutine(0.3)

        self:EmitSound("weapons/ar2/fire1.wav")

        -- Fire projectile
        local proj = ents.Create("proj_drg_plasma")
        if IsValid(proj) then
            local startPos = self:GetPos()
            local dir = (enemy:EyePos() - startPos):GetNormalized()

            proj:SetPos(startPos)
            proj:SetAngles(dir:Angle())
            proj:Spawn()
            proj:SetOwner(self)

            local phys = proj:GetPhysicsObject()
            if IsValid(phys) then
                phys:SetVelocity(dir * 700)
            end
        end

        self:PauseCoroutine(1.0)
    end

    function ENT:OnIdle()
        -- Fly to random position at preferred height
        local pos = self:RandomPos(1500)
        pos.z = pos.z + self.FlyHeight
        self:AddPatrolPos(pos)
    end

    function ENT:OnReachedPatrol()
        self:Wait(math.random(2, 4))
    end

    function ENT:OnNewEnemy(enemy)
        self:EmitSound("npc/scanner/scanner_alert1.wav")
    end

    function ENT:OnDeath(dmg, hitgroup)
        -- Explosion
        local effectData = EffectData()
        effectData:SetOrigin(self:GetPos())
        util.Effect("Explosion", effectData)

        self:EmitSound("npc/scanner/scanner_explode_crash2.wav")
    end

end

-- Register --
AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Key Flying Properties

```lua
ENT.Fly = true  -- Enable 3D flying
ENT.FlyHeight = 150  -- Preferred hover height above ground
ENT.FlyClimbHeight = 100  -- Max climb/descent per path segment
```

## Customization Examples

### Aggressive Diving Attack

```lua
function ENT:OnMeleeAttack(enemy)
    -- Dive at enemy
    local targetPos = enemy:GetPos()
    self:SetTarget(targetPos)

    -- Wait until close
    while self:GetPos():Distance(enemy:GetPos()) > 100 do
        self:PauseCoroutine(0.1)
    end

    -- Deal damage
    self:Attack({damage = 30, type = DMG_CRUSH})

    -- Fly back up
    local pos = self:GetPos()
    pos.z = pos.z + self.FlyHeight
    self:SetTarget(pos)

    self:PauseCoroutine(2.0)
end
```

### Circling Behavior

```lua
function ENT:OnChaseEnemy(enemy)
    -- Circle around enemy at distance
    local dist = self.ReachEnemyRange
    local angle = CurTime() * 50  -- Rotation speed
    local offset = Vector(
        math.cos(math.rad(angle)) * dist,
        math.sin(math.rad(angle)) * dist,
        self.FlyHeight
    )

    local targetPos = enemy:GetPos() + offset
    self:SetTarget(targetPos)
end
```

### Landing Behavior

```lua
ENT.CanLand = true
ENT.LandTime = 5  -- Land for 5 seconds

function ENT:OnIdle()
    if self.CanLand and math.random(100) < 20 then  -- 20% chance
        self:Land()
    else
        -- Normal patrol
        local pos = self:RandomPos(1000)
        pos.z = pos.z + self.FlyHeight
        self:AddPatrolPos(pos)
    end
end

function ENT:Land()
    -- Find ground
    local tr = util.TraceLine({
        start = self:GetPos(),
        endpos = self:GetPos() - Vector(0, 0, 10000),
        filter = self
    })

    if tr.Hit then
        self:SetTarget(tr.HitPos + Vector(0, 0, 20))
        self:Wait(self.LandTime)

        -- Take off
        local pos = self:GetPos()
        pos.z = pos.z + self.FlyHeight
        self:SetTarget(pos)
    end
end
```

## Common Issues and Solutions

### Issue: Flying NPC Falls to Ground

**Solutions:**
1. Make sure `ENT.Fly = true` is set
2. Check that `FlyHeight` is greater than 0
3. Verify no errors in console

### Issue: NPC Flies Too High/Low

**Solutions:**
```lua
ENT.FlyHeight = 200  -- Adjust this value
ENT.FlyClimbHeight = 150  -- Adjust maximum climb rate
```

### Issue: Erratic Movement

**Solutions:**
1. Increase `Acceleration` for smoother movement
2. Adjust path node generation
3. Use `CustomThink` for custom height control

## Performance Tips

1. **Limit sight range for flying NPCs** - They can see far, use conservatively
2. **Don't spawn too many** - Flying AI is more expensive than ground AI
3. **Use LoopSounds sparingly** - Flying sounds can overlap and become loud

## Next Steps

- Try [Boss NPC Example](./boss-npc.md) for complex multi-phase combat
- Learn [Advanced AI](./advanced-ai.md) for sophisticated behaviors
- Explore [Custom Projectiles](./custom-projectile.md) for aerial weapons
