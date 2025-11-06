# Networking Best Practices

Efficient client-server communication for DrGBase NPCs.

## Network Messages

### Use SetNW2 Over SetNW
```lua
-- Good: SetNW2 is more efficient
self:SetNW2Int("Health", self:Health())
self:SetNW2Bool("IsAngry", true)
self:SetNW2Float("Speed", self.RunSpeed)

-- Avoid: Old SetNW (less efficient)
self:SetNWInt("Health", self:Health())
```

### Minimize Network Updates
```lua
-- Good: Only update on change
function ENT:SetCustomState(state)
    if self:GetNW2String("State") == state then return end
    self:SetNW2String("State", state)
end

-- Bad: Constant updates
function ENT:CustomThink()
    self:SetNW2String("State", self.CurrentState)  -- Every tick!
end
```

## Net Messages

### Define Once, Use Everywhere
```lua
-- lua/autorun/server/sv_netmessages.lua
if SERVER then
    util.AddNetworkString("NPCDied")
    util.AddNetworkString("NPCSpawned")
end

-- Use in entities
function ENT:OnDeath()
    net.Start("NPCDied")
    net.WriteEntity(self)
    net.Broadcast()
end
```

### Compress Data
```lua
-- Good: Send only necessary data
net.Start("NPCUpdate")
net.WriteUInt(self:EntIndex(), 13)  -- 13 bits for entity index
net.WriteUInt(self:Health(), 10)     -- 10 bits for health (0-1023)
net.Broadcast()

-- Bad: Send everything
net.Start("NPCUpdate")
net.WriteEntity(self)
net.WriteTable({health = self:Health(), maxhealth = self:GetMaxHealth()})
net.Broadcast()
```

## Client-Side Prediction

### Don't Sync Everything
```lua
-- Server-only data
if SERVER then
    self.AIState = "combat"  -- Not networked
    self.PathTarget = Vector(0,0,0)  -- Not networked
end

-- Networked only when needed
self:SetNW2Bool("IsAggro", true)  -- Clients need to know
```

## Performance Tips

1. **Batch network updates** - Send multiple values in one message
2. **Use appropriate data types** - UInt instead of Int when possible
3. **Limit broadcast frequency** - Not every tick
4. **Remove obsolete network vars** - Clean up when no longer needed

For detailed networking examples, see Garry's Mod wiki on net library.
