# Spawner Configuration

DrGBase includes a powerful spawner system that allows you to create entities that automatically spawn and manage NPCs. Spawners are ideal for creating enemy waves, ambient NPC populations, or dynamic encounters in your maps.

## Table of Contents

1. [Introduction](#introduction)
2. [Creating a Basic Spawner](#creating-a-basic-spawner)
3. [Configuring Spawn Settings](#configuring-spawn-settings)
4. [Spawning Different NPC Types](#spawning-different-npc-types)
5. [Spawn Conditions and Triggers](#spawn-conditions-and-triggers)
6. [Wave-Based Spawning](#wave-based-spawning)
7. [Programmatic Spawner Creation](#programmatic-spawner-creation)
8. [Best Practices](#best-practices)

## Introduction

The DrGBase spawner system provides:

- **Automatic spawning**: Spawners continuously maintain a set quantity of NPCs
- **Spawn management**: Automatically respawns NPCs when they die
- **Flexible configuration**: Control spawn radius, quantity, delays, and NPC types
- **Weighted spawning**: Set spawn probabilities for different NPC types
- **Custom hooks**: Override behavior with BeforeSpawn and AfterSpawn hooks
- **Coroutine-based**: Efficient spawning that doesn't block the server

## Creating a Basic Spawner

To create a custom spawner, create a new file in `lua/entities/` that inherits from `spwn_drg_default`:

```lua
-- lua/entities/spwn_zombie_basic.lua
ENT.Base = "spwn_drg_default"
ENT.Type = "point"

-- Display information
ENT.PrintName = "Zombie Spawner"
ENT.Category = "My NPCs"

-- Spawner configuration
ENT.ToSpawn = {"npc_drg_zombie"}  -- List of NPC classes to spawn
ENT.Radius = 200                   -- Spawn radius in units
ENT.Quantity = 5                   -- Maximum number of NPCs to maintain
ENT.Delay = 2                      -- Delay in seconds between spawns
ENT.AutoRemove = true              -- Remove NPCs when spawner is removed
```

This creates a spawner that will maintain 5 zombies within a 200-unit radius, spawning a new one every 2 seconds when one dies.

## Configuring Spawn Settings

### Spawn Radius

The `Radius` property controls how far from the spawner NPCs can appear:

```lua
ENT.Radius = 500  -- NPCs spawn within 500 units of the spawner
```

To change the radius at runtime:

```lua
spawner:SetRadius(1000)  -- Change to 1000 units
local radius = spawner:GetRadius()  -- Get current radius
```

### Spawn Quantity

The `Quantity` property controls how many NPCs the spawner maintains:

```lua
ENT.Quantity = 10  -- Maintain 10 NPCs at a time
```

To change the quantity at runtime:

```lua
spawner:SetQuantity(15)  -- Increase to 15 NPCs
local quantity = spawner:GetQuantity()  -- Get current quantity
```

### Spawn Delay

The `Delay` property controls the time between spawns (in seconds):

```lua
ENT.Delay = 5  -- Wait 5 seconds between each spawn
```

To change the delay at runtime:

```lua
spawner:SetDelay(1)  -- Reduce to 1 second
local delay = spawner:GetDelay()  -- Get current delay
```

### Auto-Remove

The `AutoRemove` property determines if spawned NPCs are removed when the spawner is:

```lua
ENT.AutoRemove = true   -- NPCs removed with spawner (default)
ENT.AutoRemove = false  -- NPCs persist after spawner removal
```

To change auto-remove at runtime:

```lua
spawner:EnableAutoRemove(false)  -- Disable auto-remove
local enabled = spawner:EnableAutoRemove()  -- Check current setting
```

## Spawning Different NPC Types

### Single NPC Type

The simplest configuration spawns one type of NPC:

```lua
ENT.ToSpawn = {"npc_drg_zombie"}
```

### Multiple NPC Types (Equal Weight)

To spawn multiple NPC types with equal probability:

```lua
ENT.ToSpawn = {
    "npc_drg_zombie",
    "npc_drg_skeleton",
    "npc_drg_ghost"
}
```

Each type has an equal 33% chance of being spawned.

### Weighted Spawning

To control spawn probability, use a table with NPC classes as keys and weights as values:

```lua
ENT.ToSpawn = {
    ["npc_drg_zombie"] = 5,      -- 62.5% chance (5 out of 8)
    ["npc_drg_skeleton"] = 2,    -- 25% chance (2 out of 8)
    ["npc_drg_boss"] = 1         -- 12.5% chance (1 out of 8)
}
```

Higher numbers mean the NPC spawns more frequently.

### Dynamic Spawn Lists

You can modify the spawn list at runtime:

```lua
function ENT:CustomInitialize()
    -- Add a new NPC type
    self:AddToSpawn("npc_drg_spider", 3)  -- Add with weight of 3

    -- Remove an NPC type
    self:RemoveToSpawn("npc_drg_zombie")

    -- Add multiple types at once
    self:AddToSpawn({"npc_drg_bat", "npc_drg_rat"}, 2)
end
```

### Getting Spawned Entities

Track which NPCs the spawner has created:

```lua
function ENT:CustomThink()
    local spawnedNPCs = self:GetSpawned()
    print("Currently managing " .. #spawnedNPCs .. " NPCs")

    -- Iterate through spawned NPCs
    for i, npc in ipairs(spawnedNPCs) do
        if IsValid(npc) then
            -- Do something with the NPC
        end
    end
end
```

## Spawn Conditions and Triggers

### BeforeSpawn Hook

The `BeforeSpawn` hook is called before an NPC is created. Return `false` to prevent spawning:

```lua
function ENT:BeforeSpawn(class)
    -- Only spawn during night time
    local tod = GetGlobalFloat("TimeOfDay", 12)
    if tod > 6 and tod < 18 then
        return false  -- Don't spawn during day (6 AM to 6 PM)
    end

    -- Only spawn if players are nearby
    local nearbyPlayers = ents.FindInSphere(self:GetPos(), 1000)
    local hasPlayers = false
    for _, ent in ipairs(nearbyPlayers) do
        if ent:IsPlayer() then
            hasPlayers = true
            break
        end
    end

    if not hasPlayers then
        return false  -- No players nearby
    end

    -- Allow spawn
    return true
end
```

### AfterSpawn Hook

The `AfterSpawn` hook is called after an NPC is created but before it's tracked. Return `false` to cancel tracking and remove the NPC:

```lua
function ENT:AfterSpawn(npc)
    -- Give spawned NPCs special weapons
    if npc:GetClass() == "npc_drg_soldier" then
        npc:Give("weapon_ar2")
    end

    -- Set custom health based on difficulty
    local difficulty = GetConVar("gamemode_difficulty"):GetInt()
    npc:SetHealth(npc:Health() * difficulty)

    -- Set the NPC's target to nearest player
    local target = self:FindNearestPlayer()
    if IsValid(target) then
        npc:SetEnemy(target)
    end

    -- Optional: Return false to remove the NPC and not track it
    if npc:Health() <= 0 then
        return false
    end
end

function ENT:FindNearestPlayer()
    local nearestPlayer = nil
    local nearestDist = math.huge

    for _, ply in ipairs(player.GetAll()) do
        local dist = self:GetPos():Distance(ply:GetPos())
        if dist < nearestDist then
            nearestDist = dist
            nearestPlayer = ply
        end
    end

    return nearestPlayer
end
```

### Conditional Spawning Example

Create a spawner that only activates when a player enters a trigger:

```lua
ENT.Base = "spwn_drg_default"
ENT.Type = "point"

ENT.PrintName = "Triggered Spawner"
ENT.Category = "My NPCs"

ENT.ToSpawn = {"npc_drg_zombie"}
ENT.Radius = 300
ENT.Quantity = 0  -- Start disabled
ENT.Delay = 1

function ENT:CustomInitialize()
    self.Activated = false
end

function ENT:BeforeSpawn(class)
    -- Only spawn if activated
    return self.Activated
end

-- Call this function to activate the spawner
function ENT:Activate()
    if not self.Activated then
        self.Activated = true
        self:SetQuantity(10)  -- Start spawning 10 NPCs
        print("Spawner activated!")
    end
end

-- Call this function to deactivate
function ENT:Deactivate()
    self.Activated = false
    self:SetQuantity(0)  -- Stop spawning
end
```

## Wave-Based Spawning

Create a spawner that spawns NPCs in waves with breaks between waves:

```lua
ENT.Base = "spwn_drg_default"
ENT.Type = "point"

ENT.PrintName = "Wave Spawner"
ENT.Category = "My NPCs"

ENT.ToSpawn = {"npc_drg_zombie", "npc_drg_skeleton"}
ENT.Radius = 400
ENT.Quantity = 0  -- Controlled by wave system
ENT.Delay = 0.5

function ENT:CustomInitialize()
    self.CurrentWave = 0
    self.WaveInProgress = false
    self.WaveSize = 5
    self.TimeBetweenWaves = 15  -- 15 seconds between waves
    self.NextWaveTime = CurTime() + 5  -- First wave in 5 seconds
end

function ENT:CustomThink()
    -- Check if it's time to start a new wave
    if not self.WaveInProgress and CurTime() >= self.NextWaveTime then
        self:StartWave()
    end

    -- Check if wave is complete
    if self.WaveInProgress then
        local spawnedNPCs = self:GetSpawned()
        local aliveCount = 0

        for _, npc in ipairs(spawnedNPCs) do
            if IsValid(npc) and npc:Health() > 0 then
                aliveCount = aliveCount + 1
            end
        end

        -- Wave complete when all NPCs are dead
        if aliveCount == 0 and self:GetQuantity() == 0 then
            self:CompleteWave()
        end
    end

    self:NextThink(CurTime() + 0.1)
    return true
end

function ENT:StartWave()
    self.CurrentWave = self.CurrentWave + 1
    self.WaveInProgress = true

    -- Increase difficulty each wave
    self.WaveSize = 5 + (self.CurrentWave * 2)
    self:SetQuantity(self.WaveSize)

    -- Announce wave
    PrintMessage(HUD_PRINTTALK, "Wave " .. self.CurrentWave .. " starting! " .. self.WaveSize .. " enemies incoming!")
end

function ENT:CompleteWave()
    self.WaveInProgress = false
    self:SetQuantity(0)  -- Stop spawning
    self.NextWaveTime = CurTime() + self.TimeBetweenWaves

    -- Announce completion
    PrintMessage(HUD_PRINTTALK, "Wave " .. self.CurrentWave .. " complete! Next wave in " .. self.TimeBetweenWaves .. " seconds.")
end

function ENT:AfterSpawn(npc)
    -- Once we've spawned enough for this wave, stop spawning
    local spawnedCount = #self:GetSpawned()
    if spawnedCount >= self.WaveSize then
        self:SetQuantity(0)  -- Prevent more spawns
    end
end
```

### Progressive Difficulty Waves

Create waves that get progressively harder:

```lua
function ENT:StartWave()
    self.CurrentWave = self.CurrentWave + 1
    self.WaveInProgress = true

    -- Progressive difficulty
    if self.CurrentWave <= 3 then
        -- Early waves: only basic enemies
        self:RemoveToSpawn({"npc_drg_skeleton", "npc_drg_boss"})
        self:AddToSpawn("npc_drg_zombie", 1)
        self.WaveSize = 5 + (self.CurrentWave * 2)

    elseif self.CurrentWave <= 6 then
        -- Mid waves: mix of enemies
        self:RemoveToSpawn("npc_drg_boss")
        self:AddToSpawn("npc_drg_zombie", 2)
        self:AddToSpawn("npc_drg_skeleton", 1)
        self.WaveSize = 10 + (self.CurrentWave * 2)

    else
        -- Late waves: all enemies including bosses
        self:AddToSpawn("npc_drg_zombie", 3)
        self:AddToSpawn("npc_drg_skeleton", 2)
        self:AddToSpawn("npc_drg_boss", 1)
        self.WaveSize = 15 + (self.CurrentWave * 3)
    end

    self:SetQuantity(self.WaveSize)
    PrintMessage(HUD_PRINTTALK, "Wave " .. self.CurrentWave .. " - " .. self.WaveSize .. " enemies!")
end
```

## Programmatic Spawner Creation

Create spawners dynamically through code instead of placing them in the map:

```lua
-- Basic spawner creation
local spawner = DrGBase.CreateSpawner(
    Vector(0, 0, 0),           -- Position
    "npc_drg_zombie",          -- NPC class to spawn
    200,                       -- Radius
    5                          -- Quantity
)

-- Create with multiple NPC types
local spawner = DrGBase.CreateSpawner(
    Vector(0, 0, 0),
    {
        ["npc_drg_zombie"] = 3,
        ["npc_drg_skeleton"] = 1
    },
    300,
    10
)

-- Create with custom spawner class
local spawner = DrGBase.CreateSpawner(
    Vector(0, 0, 0),
    "npc_drg_zombie",
    200,
    5,
    "spwn_my_custom_spawner"   -- Custom spawner entity class
)

-- Configure after creation
spawner:SetDelay(2)
spawner:EnableAutoRemove(false)
```

### Dynamic Spawner Example

Create spawners based on player count:

```lua
hook.Add("PlayerInitialSpawn", "CreateSpawnersForPlayers", function(ply)
    local playerCount = #player.GetAll()
    local spawnPoints = ents.FindByClass("info_player_start")

    -- Create one spawner per spawn point, scaled by player count
    for _, spawnPoint in ipairs(spawnPoints) do
        local spawner = DrGBase.CreateSpawner(
            spawnPoint:GetPos() + Vector(0, 0, 50),
            {"npc_drg_zombie", "npc_drg_skeleton"},
            500,
            3 * playerCount  -- Scale with player count
        )

        spawner:SetDelay(3)
    end
end)
```

## Best Practices

### 1. Use Appropriate Spawn Delays

Don't spawn too quickly, as it can overwhelm players and hurt performance:

```lua
-- Good: Reasonable delay
ENT.Delay = 2

-- Bad: No delay, can spawn hundreds per second
ENT.Delay = 0
```

### 2. Set Reasonable Quantities

Limit the number of NPCs to maintain good performance:

```lua
-- Good: Manageable number
ENT.Quantity = 10

-- Bad: Too many NPCs
ENT.Quantity = 1000
```

### 3. Use Weighted Spawning for Balance

Make bosses and special enemies rare:

```lua
ENT.ToSpawn = {
    ["npc_drg_zombie"] = 10,      -- Common
    ["npc_drg_skeleton"] = 5,     -- Uncommon
    ["npc_drg_boss"] = 1          -- Rare
}
```

### 4. Implement Spawn Conditions

Use `BeforeSpawn` to avoid spawning in inappropriate situations:

```lua
function ENT:BeforeSpawn(class)
    -- Don't spawn if there are too many NPCs globally
    local allNPCs = ents.FindByClass("npc_*")
    if #allNPCs > 100 then
        return false
    end

    return true
end
```

### 5. Clean Up When Appropriate

Use `AutoRemove` wisely:

```lua
-- For temporary encounters: enable auto-remove
ENT.AutoRemove = true

-- For persistent population: disable auto-remove
ENT.AutoRemove = false
```

### 6. Test Spawn Radius

Visualize your spawn radius during development:

```lua
function ENT:CustomThink()
    if developer:GetBool() then
        debugoverlay.Sphere(self:GetPos(), self:GetRadius(), 0.1, Color(255, 0, 0, 50))
    end

    self:NextThink(CurTime() + 0.1)
    return true
end
```

### 7. Provide Player Feedback

Let players know when spawners activate:

```lua
function ENT:Activate()
    self.Activated = true
    self:SetQuantity(10)

    -- Visual effect
    local effectdata = EffectData()
    effectdata:SetOrigin(self:GetPos())
    util.Effect("Explosion", effectdata)

    -- Sound
    self:EmitSound("ambient/explosions/explode_4.wav")

    -- Message
    PrintMessage(HUD_PRINTTALK, "Enemy spawner activated!")
end
```

### 8. Scale Difficulty

Adjust spawner difficulty based on player count or skill:

```lua
function ENT:CustomInitialize()
    local playerCount = #player.GetAll()

    -- Scale quantity with players
    self:SetQuantity(self.Quantity * playerCount)

    -- Reduce delay with more players
    self:SetDelay(math.max(0.5, self.Delay / playerCount))
end
```

### 9. Use Categories

Organize your spawners with appropriate categories:

```lua
ENT.Category = "My Game - Spawners"  -- Good: Specific category
ENT.Category = "Spawners"            -- Bad: Too generic
```

### 10. Document Your Spawners

Add comments to explain your spawner's purpose:

```lua
-- This spawner creates a challenging encounter with mixed enemy types.
-- Designed for 2-4 players in the warehouse area.
-- Wave size scales with player count.
ENT.PrintName = "Warehouse Defense Spawner"
ENT.Category = "My Game - Defense Spawners"
```

## Advanced Example: Boss Fight Spawner

Here's a complete example of a boss fight spawner with minions:

```lua
ENT.Base = "spwn_drg_default"
ENT.Type = "point"

ENT.PrintName = "Boss Fight Spawner"
ENT.Category = "My Game - Spawners"

ENT.ToSpawn = {"npc_drg_zombie"}  -- Minions
ENT.Radius = 400
ENT.Quantity = 0
ENT.Delay = 2
ENT.AutoRemove = true

function ENT:CustomInitialize()
    self.BossSpawned = false
    self.MinionWaveSize = 5
    self.CurrentMinions = 0
    self.Phase = 0
    self.MaxPhases = 3
end

function ENT:BeforeSpawn(class)
    -- Only spawn minions if boss is alive
    if not self.BossSpawned then
        return false
    end

    if not IsValid(self.Boss) or self.Boss:Health() <= 0 then
        return false
    end

    return true
end

function ENT:AfterSpawn(npc)
    -- Make minions follow the boss
    if IsValid(self.Boss) then
        npc:SetEnemy(self.Boss:GetEnemy())
    end
end

function ENT:SpawnBoss()
    self.Boss = ents.Create("npc_drg_boss")
    if not IsValid(self.Boss) then return end

    self.Boss:SetPos(self:GetPos())
    self.Boss:Spawn()
    self.BossSpawned = true
    self.Phase = 1

    -- Boss callback when it takes damage
    self.Boss:AddCallback("OnTakeDamage", function(boss, dmg)
        self:CheckPhaseTransition()
    end)

    PrintMessage(HUD_PRINTTALK, "Boss spawned!")
end

function ENT:CheckPhaseTransition()
    if not IsValid(self.Boss) then return end

    local healthPercent = self.Boss:Health() / self.Boss:GetMaxHealth()
    local requiredPhase = math.ceil((1 - healthPercent) * self.MaxPhases)

    if requiredPhase > self.Phase and self.Phase < self.MaxPhases then
        self.Phase = requiredPhase
        self:StartMinionWave()
        PrintMessage(HUD_PRINTTALK, "Boss phase " .. self.Phase .. "!")
    end
end

function ENT:StartMinionWave()
    self.MinionWaveSize = 5 + (self.Phase * 3)
    self:SetQuantity(self.MinionWaveSize)

    -- Add variety in later phases
    if self.Phase >= 2 then
        self:AddToSpawn("npc_drg_skeleton", 1)
    end
end

function ENT:CustomThink()
    -- Check if boss is dead
    if self.BossSpawned and (not IsValid(self.Boss) or self.Boss:Health() <= 0) then
        self:SetQuantity(0)  -- Stop spawning minions
        PrintMessage(HUD_PRINTTALK, "Boss defeated!")

        -- Remove after delay
        timer.Simple(5, function()
            if IsValid(self) then
                self:Remove()
            end
        end)
    end

    self:NextThink(CurTime() + 0.5)
    return true
end

-- Start the fight when called
function ENT:StartFight()
    self:SpawnBoss()
end
```

This guide covers the essential aspects of the DrGBase spawner system. Experiment with different configurations to create dynamic and engaging NPC encounters in your maps and game modes.
