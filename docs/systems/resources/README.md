# Resource Management

Model and sound precaching.

**File:** `resources.lua`

## Overview

The resource management system handles precaching and downloading of custom content (models, materials, sounds) so that clients can see and hear DrGBase content.

**Key Features:**
- **Automatic Precaching** - Model/sound precaching for Source engine
- **Download Management** - FastDL support for custom content
- **Icon Precaching** - DrGBase UI icon automatically added

## Adding Resources

**Precache Models:**
```lua
function ENT:Initialize()
    util.PrecacheModel("models/mymodel.mdl")
    self:SetModel("models/mymodel.mdl")
end
```

**Add to Download:**
```lua
if SERVER then
    resource.AddFile("models/mymodel.mdl")
    resource.AddFile("materials/models/mymodel/texture.vmt")
    resource.AddFile("materials/models/mymodel/texture.vtf")
    resource.AddFile("sound/mynpc/attack.wav")
end
```
