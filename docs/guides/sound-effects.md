# Sound & Effects

This guide teaches you how to add sound effects to your DrGBase nextbots, including automatic sounds, custom sounds, and animation-synchronized audio.

## Table of Contents

1. [Introduction](#introduction)
2. [Configuring Automatic Sounds](#configuring-automatic-sounds)
3. [Playing Custom Sounds](#playing-custom-sounds)
4. [Sound Tables and Random Sounds](#sound-tables-and-random-sounds)
5. [Sound Events and Animation Events](#sound-events-and-animation-events)
6. [Footstep Sounds](#footstep-sounds)
7. [Volume and Pitch Control](#volume-and-pitch-control)
8. [Best Practices](#best-practices)

## Introduction

DrGBase provides a comprehensive sound system that makes it easy to add audio feedback to your NPCs. The system includes:

- **Automatic sounds** - Sounds that play automatically (idle, damage, death, etc.)
- **Slotted sounds** - Prevents sound spam with built-in delay management
- **Animation events** - Synchronize sounds with animations
- **Footstep system** - Automatic footstep sounds based on surface materials
- **Custom sound registration** - Define your own sound scripts

## Configuring Automatic Sounds

DrGBase nextbots support several automatic sound properties that play without any additional code:

### Basic Sound Properties

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Spawn sounds - Play once when the NPC spawns
ENT.OnSpawnSounds = {"npc/zombie/zombie_voice_idle1.wav"}

-- Idle sounds - Loop while NPC is alive
ENT.OnIdleSounds = {"NPC_HeadCrab.Idle"}
ENT.IdleSoundDelay = 2  -- Delay between idle sounds (in seconds)
ENT.ClientIdleSounds = false  -- If true, idle sounds play on client instead of server

-- Damage sounds - Play when NPC takes damage
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.DamageSoundDelay = 0.25  -- Minimum delay between damage sounds

-- Death sounds - Play when NPC dies
ENT.OnDeathSounds = {"Zombie.Die"}

-- Downed sounds - Play when NPC is downed but not dead
ENT.OnDownedSounds = {"npc/combine_soldier/pain1.wav"}
```

### Real-World Example: Zombie

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

ENT.PrintName = "Zombie"
ENT.Category = "My NPCs"
ENT.Models = {"models/Zombie/Classic.mdl"}

-- Sound configuration
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- These sounds play automatically!
-- - Zombie.Pain plays when damaged
-- - Zombie.Die plays when killed
```

## Playing Custom Sounds

For sounds that need to be played in specific situations, use DrGBase's sound functions.

### EmitSound - Basic Sound Playback

The standard `EmitSound()` function works on nextbots:

```lua
function ENT:OnMeleeAttack(enemy)
    -- Play attack sound
    self:EmitSound("Zombie.Attack")

    -- With volume and pitch
    self:EmitSound("Zombie.Attack", 75, 100, 1, CHAN_VOICE)

    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end
```

### EmitSlotSound - Prevent Sound Spam

`EmitSlotSound()` prevents sounds from playing too frequently by using delay slots:

```lua
-- EmitSlotSound(slot, duration, soundName, soundLevel, pitchPercent, volume, channel)

function ENT:CustomThink()
    -- Will only play if 7 seconds have passed since last "riseandshine" slot sound
    self:EmitSlotSound("riseandshine", 7, "DrGBase.RiseAndShine")

    -- Short delay slot for frequent sounds
    self:EmitSlotSound("breathing", 2, "npc/zombie/zombie_breathe1.wav")
end
```

**Parameters:**
- `slot` - Unique identifier for this sound slot (string)
- `duration` - How long to wait before allowing this slot to play again (seconds)
- `soundName` - Sound file path or registered sound name
- `soundLevel` - Volume level (optional)
- `pitchPercent` - Pitch percentage (optional, 100 = normal)
- `volume` - Volume multiplier (optional, 0-1)
- `channel` - Sound channel (optional, e.g., CHAN_VOICE, CHAN_WEAPON)

### Registering Custom Sounds

For complex sound scripts, use `sound.Add()`:

```lua
if SERVER then
    sound.Add({
        name = "DrGBase.RiseAndShine",
        sound = "vo/gman_misc/gman_riseshine.wav",
        channel = CHAN_VOICE,
        level = 60,
        pitch = {95, 105},  -- Random pitch range
        volume = 1.0
    })
end

-- Then use it
function ENT:OnRangeAttack(enemy)
    self:EmitSound("DrGBase.RiseAndShine")
end
```

## Sound Tables and Random Sounds

You can provide multiple sounds in a table, and DrGBase will randomly select one:

```lua
-- Multiple sound variations
ENT.OnDamageSounds = {
    "npc/zombie/zombie_pain1.wav",
    "npc/zombie/zombie_pain2.wav",
    "npc/zombie/zombie_pain3.wav",
    "npc/zombie/zombie_pain4.wav",
    "npc/zombie/zombie_pain5.wav",
    "npc/zombie/zombie_pain6.wav"
}

-- Using sound scripts
ENT.OnIdleSounds = {
    "NPC_Combine.Die1",
    "NPC_Combine.Die2",
    "NPC_Combine.Die3"
}
```

### Dynamic Sound Management

You can change sound tables at runtime:

```lua
function ENT:CustomInitialize()
    -- Start with normal idle sounds
    self.OnIdleSounds = {"NPC_HeadCrab.Idle"}
end

function ENT:HeadcrabLeap(pos)
    -- Disable idle sounds during leap
    self.OnIdleSounds = {}

    self:PlaySequence("jumpattack_broadcast")
    self:PauseCoroutine(0.5)
    self:EmitSound("NPC_Headcrab.Attack")
    self:Leap(pos, 400)

    -- Re-enable idle sounds
    self.OnIdleSounds = {"NPC_HeadCrab.Idle"}
end
```

## Sound Events and Animation Events

Synchronize sounds with specific animation frames or cycles.

### OnAnimEvent Hook

The `OnAnimEvent()` hook is called for animation events in your model:

```lua
function ENT:OnAnimEvent(event, eventTime, pos, ang)
    -- Called for model animation events
    -- event = event name/options from model
    -- eventTime = when the event occurs

    if self:IsAttacking() and self:GetCycle() > 0.3 then
        self:Attack({damage = 10}, function(self, hit)
            if #hit > 0 then
                self:EmitSound("Zombie.AttackHit")
            else
                self:EmitSound("Zombie.AttackMiss")
            end
        end)
    elseif math.random(2) == 1 then
        self:EmitSound("Zombie.FootstepLeft")
    else
        self:EmitSound("Zombie.FootstepRight")
    end
end
```

### SequenceEvent - Trigger Sounds at Specific Cycles

Use `SequenceEvent()` to run code at specific points in an animation:

```lua
function ENT:CustomInitialize()
    -- Play footstep sounds at 28% and 78% of walk/run animations
    for i, walk in ipairs({self.RunAnimation, self.WalkAnimation}) do
        self:SequenceEvent(
            self:SelectRandomSequence(walk),
            {0.28, 0.78},  -- Animation cycles (0 = start, 1 = end)
            function(self)
                self:EmitFootstep()
            end
        )
    end
end
```

### AddAnimEvent - Frame-Based Events

Add events at specific animation frames:

```lua
function ENT:CustomInitialize()
    -- Add footstep event at frame 10 and 25
    self:AddAnimEvent(
        "walk_all",  -- Sequence name
        {10, 25},    -- Frames
        "footstep"   -- Event name
    )
end

function ENT:OnAnimEvent(event, eventTime, pos, ang)
    if event == "footstep" then
        self:EmitFootstep()
    end
end
```

### Example: Combat Sounds

```lua
function ENT:OnMeleeAttack(enemy)
    -- Play attack animation
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)

    -- Attack sound plays at start
    self:EmitSound("Zombie.Attack")
end

function ENT:OnAnimEvent(event, eventTime, pos, ang)
    -- Hit sound plays when animation event fires
    if self:IsAttacking() and self:GetCycle() > 0.5 then
        self:Attack({
            damage = 20,
            type = DMG_SLASH
        }, function(self, hit)
            -- Callback fires after attack resolves
            if #hit > 0 then
                self:EmitSound("Zombie.AttackHit")
            else
                self:EmitSound("Zombie.AttackMiss")
            end
        end)
    end
end
```

## Footstep Sounds

DrGBase includes an automatic footstep system that plays sounds based on the surface material.

### EmitFootstep Function

```lua
-- Basic usage
self:EmitFootstep()

-- With parameters
self:EmitFootstep(soundLevel, pitchPercent, volume, channel)

-- Example
self:EmitFootstep(75, 100, 1, CHAN_BODY)
```

### Custom Footstep Sounds

Define custom footstep sounds per material type:

```lua
ENT.Footsteps = {
    [MAT_DEFAULT] = {
        "player/footsteps/concrete1.wav",
        "player/footsteps/concrete2.wav",
        "player/footsteps/concrete3.wav",
        "player/footsteps/concrete4.wav"
    },
    [MAT_METAL] = {
        "player/footsteps/metal1.wav",
        "player/footsteps/metal2.wav",
        "player/footsteps/metal3.wav",
        "player/footsteps/metal4.wav"
    },
    [MAT_DIRT] = {
        "player/footsteps/dirt1.wav",
        "player/footsteps/dirt2.wav",
        "player/footsteps/dirt3.wav",
        "player/footsteps/dirt4.wav"
    }
}
```

**Material Constants:**
- `MAT_DEFAULT` - Default material
- `MAT_METAL` - Metal surfaces
- `MAT_DIRT` - Dirt surfaces
- `MAT_CONCRETE` - Concrete surfaces
- `MAT_WOOD` - Wooden surfaces
- `MAT_GRASS` - Grass surfaces
- `MAT_SAND` - Sand surfaces
- And more (see Source Engine material types)

### Automatic Footsteps in Animations

```lua
function ENT:CustomInitialize()
    -- Emit footsteps at 28% and 78% of walk/run cycles
    for i, walk in ipairs({self.RunAnimation, self.WalkAnimation}) do
        self:SequenceEvent(
            self:SelectRandomSequence(walk),
            {0.28, 0.78},
            function(self)
                self:EmitFootstep()  -- Automatically picks correct material sound
            end
        )
    end
end
```

### Ladder Climbing Sounds

```lua
function ENT:OnClimbing(ladder, left, down)
    if IsValid(ladder) then
        -- Play ladder sound with 0.3 second delay slot
        self:EmitSlotSound(
            "DrGBaseLadderClimbing",
            0.3,
            "player/footsteps/ladder"..math.random(4)..".wav"
        )
    end
    return not down and left < 112.5
end
```

## Volume and Pitch Control

Control how sounds play with volume and pitch parameters.

### EmitSound Parameters

```lua
-- EmitSound(soundName, soundLevel, pitchPercent, volume, channel)

-- Normal sound
self:EmitSound("Zombie.Attack")

-- Louder sound (higher sound level)
self:EmitSound("Zombie.Attack", 90)

-- Higher pitch
self:EmitSound("Zombie.Attack", 75, 120)  -- 120% pitch

-- Quieter (volume multiplier)
self:EmitSound("Zombie.Attack", 75, 100, 0.5)  -- 50% volume

-- Specific channel
self:EmitSound("Zombie.Attack", 75, 100, 1, CHAN_WEAPON)
```

### Sound Channels

Use different channels for different types of sounds:

- `CHAN_AUTO` - Automatic channel selection
- `CHAN_WEAPON` - Weapon sounds
- `CHAN_VOICE` - Voice lines
- `CHAN_ITEM` - Item sounds
- `CHAN_BODY` - Body sounds (footsteps, movements)
- `CHAN_STREAM` - Streaming sounds
- `CHAN_STATIC` - Static sounds
- `CHAN_REPLACE` - Replace any sound on this entity

### Random Pitch Variation

```lua
-- Random pitch for variety
function ENT:OnMeleeAttack(enemy)
    local pitch = math.random(95, 105)
    self:EmitSound("Zombie.Attack", 75, pitch)
    self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
end

-- Or in sound.Add
sound.Add({
    name = "MyNPC.Attack",
    sound = "npc/attack.wav",
    channel = CHAN_WEAPON,
    pitch = {90, 110}  -- Random pitch between 90-110%
})
```

### Distance-Based Volume

```lua
function ENT:OnRangeAttack(enemy)
    local distance = self:GetRangeTo(enemy)

    -- Quieter sound for distant enemies
    if distance > 500 then
        self:EmitSound("Weapon.Shoot", 60, 100, 0.7)
    else
        self:EmitSound("Weapon.Shoot", 80, 100, 1.0)
    end
end
```

## Best Practices

### 1. Use Sound Scripts for Reusability

Instead of hardcoding file paths, register sound scripts:

```lua
-- Bad
self:EmitSound("npc/zombie/zombie_voice_idle1.wav")

-- Good
self:EmitSound("Zombie.Idle")  -- Uses existing sound script
```

### 2. Prevent Sound Spam with Slots

```lua
-- Bad - Will spam every think
function ENT:CustomThink()
    self:EmitSound("breathing.wav")
end

-- Good - Uses slot with delay
function ENT:CustomThink()
    self:EmitSlotSound("breathing", 2, "breathing.wav")
end
```

### 3. Use Multiple Sound Variations

```lua
-- Bad - Repetitive
ENT.OnDamageSounds = {"pain.wav"}

-- Good - Variety
ENT.OnDamageSounds = {
    "pain1.wav",
    "pain2.wav",
    "pain3.wav",
    "pain4.wav"
}
```

### 4. Match Sounds to Animations

```lua
function ENT:CustomInitialize()
    -- Sync footsteps with animation
    self:SequenceEvent(
        self:SelectRandomSequence(self.WalkAnimation),
        {0.25, 0.75},  -- When feet hit ground
        function(self)
            self:EmitFootstep()
        end
    )
end
```

### 5. Stop Sounds When Needed

```lua
function ENT:OnRemove()
    -- Stop looping sounds
    self:StopSound("DrGBase.RiseAndShine")
end

function ENT:OnDeath()
    -- Stop all sounds on death
    if isstring(self._DrGBaseIdleSound) then
        self:StopSound(self._DrGBaseIdleSound)
    end
end
```

### 6. Consider Sound Levels

```lua
-- Quiet sounds (whispers, small objects)
self:EmitSound("whisper.wav", 50)

-- Normal sounds (footsteps, speech)
self:EmitSound("speech.wav", 75)

-- Loud sounds (explosions, yells)
self:EmitSound("explosion.wav", 100)

-- Very loud (sirens, alarms)
self:EmitSound("alarm.wav", 140)
```

### 7. Context-Aware Sounds

```lua
function ENT:OnNewEnemy()
    -- Alert sound when spotting enemy
    self:EmitSound("Zombie.Alert")
end

function ENT:OnContact(ent)
    if ent == self:GetEnemy() then
        -- Bite sound on contact with enemy
        self:EmitSound("NPC_HeadCrab.Bite")
    end
end
```

## Complete Example

Here's a complete example combining all concepts:

```lua
if not DrGBase then return end
ENT.Base = "drgbase_nextbot"

-- Basic info
ENT.PrintName = "Sound Example NPC"
ENT.Category = "Examples"
ENT.Models = {"models/Zombie/Classic.mdl"}

-- Automatic sounds
ENT.OnSpawnSounds = {"npc/zombie/zombie_voice_idle1.wav"}
ENT.OnIdleSounds = {
    "npc/zombie/zombie_voice_idle1.wav",
    "npc/zombie/zombie_voice_idle2.wav",
    "npc/zombie/zombie_voice_idle3.wav"
}
ENT.IdleSoundDelay = 3
ENT.OnDamageSounds = {"Zombie.Pain"}
ENT.OnDeathSounds = {"Zombie.Die"}

-- Custom footsteps
ENT.Footsteps = {
    [MAT_DEFAULT] = {
        "npc/zombie/foot1.wav",
        "npc/zombie/foot2.wav",
        "npc/zombie/foot3.wav"
    }
}

if SERVER then
    -- Register custom sound
    sound.Add({
        name = "ExampleNPC.Roar",
        sound = "npc/zombie/zombie_alert1.wav",
        channel = CHAN_VOICE,
        level = 90,
        pitch = {80, 95}
    })

    function ENT:CustomInitialize()
        -- Set up footstep events
        self:SequenceEvent(
            self:SelectRandomSequence(self.WalkAnimation),
            {0.3, 0.8},
            function(self)
                self:EmitFootstep()
            end
        )
    end

    function ENT:OnNewEnemy()
        -- Roar when spotting enemy
        self:EmitSound("ExampleNPC.Roar")
    end

    function ENT:OnMeleeAttack(enemy)
        -- Attack sound
        self:EmitSound("Zombie.Attack", 75, math.random(95, 105))
        self:PlayActivityAndMove(ACT_MELEE_ATTACK1, 1, self.FaceEnemy)
    end

    function ENT:OnAnimEvent(event, eventTime, pos, ang)
        -- Attack hit/miss sounds
        if self:IsAttacking() and self:GetCycle() > 0.5 then
            self:Attack({damage = 20}, function(self, hit)
                if #hit > 0 then
                    self:EmitSound("Zombie.AttackHit")
                else
                    self:EmitSound("Zombie.AttackMiss")
                end
            end)
        end
    end

    function ENT:CustomThink()
        -- Breathing sound with slot delay
        if self:Health() < self:GetMaxHealth() * 0.3 then
            self:EmitSlotSound("breathing", 2, "npc/zombie/zombie_breathe1.wav", 60)
        end
    end

    function ENT:OnRemove()
        -- Clean up sounds
        self:StopSound("ExampleNPC.Roar")
    end
end

AddCSLuaFile()
DrGBase.AddNextbot(ENT)
```

## Summary

- Use **automatic sound properties** for spawn, idle, damage, and death sounds
- Use **EmitSound()** for immediate sound playback
- Use **EmitSlotSound()** to prevent sound spam with delay management
- Use **sound.Add()** to register custom sound scripts
- Use **SequenceEvent()** to sync sounds with animation cycles
- Use **EmitFootstep()** for automatic material-based footstep sounds
- Control volume with **sound level** and **volume parameters**
- Control pitch for **variation** and **effect**
- Always **stop looping sounds** in OnRemove

For more examples, check the NPCs in `lua/entities/npc_drg_*.lua`.
