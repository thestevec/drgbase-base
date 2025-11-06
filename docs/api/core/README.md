# Core Framework API

Global DrGBase functions and core utilities.

## Files

- **[drgbase.md](./drgbase.md)** - DrGBase global table and main functions
- **[entity-helpers.md](./entity-helpers.md)** - Entity utility functions
- **[enumerations.md](./enumerations.md)** - Constants and enumerations
- **[colors.md](./colors.md)** - Color definitions

## DrGBase Global Functions

### Registration

Register DrGBase entities and weapons with the framework. These functions handle clientside/serverside inclusion and register entities in the spawn menu.

- `DrGBase.AddNextbot(ENT)` - Register a nextbot entity (call at end of entity file)
- `DrGBase.AddWeapon(SWEP)` - Register a weapon (call at end of weapon file)
- `DrGBase.AddSpawner(ENT)` - Register a spawner entity

### File Inclusion

Helper functions for including Lua files with proper realm handling (AddCSLuaFile, include).

- `DrGBase.IncludeFile(fileName)` - Include a single file (handles CLIENT/SERVER automatically)
- `DrGBase.IncludeFolder(folder)` - Include all files in folder (non-recursive)
- `DrGBase.RecursiveInclude(folder)` - Recursively include folder and subfolders

### Output

Formatted console output functions with DrGBase branding and color coding.

- `DrGBase.Print(msg, options)` - Print formatted message with custom colors
- `DrGBase.Info(...)` - Print info message (white text)
- `DrGBase.Error(...)` - Print error message (red text)
- `DrGBase.ErrorInfo(...)` - Print error with additional info

---

See individual files for detailed documentation.
