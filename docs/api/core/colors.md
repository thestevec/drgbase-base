# Colors

Color definitions for UI and display.

**File:** `lua/drgbase/colors.lua`

## Color Constants

DrGBase provides predefined color constants for consistent UI and display rendering. Each color comes in both opaque and transparent (alpha = 0) versions.

### Opaque Colors

| Constant | RGB Values | Description |
|----------|------------|-------------|
| `DrGBase.CLR_WHITE` | `Color(255, 255, 255)` | Pure white |
| `DrGBase.CLR_GREEN` | `Color(150, 255, 40)` | Bright lime green |
| `DrGBase.CLR_RED` | `Color(255, 50, 50)` | Bright red |
| `DrGBase.CLR_CYAN` | `Color(0, 200, 200)` | Cyan/aqua |
| `DrGBase.CLR_PURPLE` | `Color(220, 40, 115)` | Pink-purple |
| `DrGBase.CLR_BLUE` | `Color(50, 100, 255)` | Bright blue |
| `DrGBase.CLR_ORANGE` | `Color(255, 150, 30)` | Bright orange |
| `DrGBase.CLR_DARKGRAY` | `Color(20, 20, 20)` | Nearly black dark gray |
| `DrGBase.CLR_LIGHTGRAY` | `Color(200, 200, 200)` | Light gray |

### Transparent Colors

Transparent versions (alpha = 0) for use with color transitions and lerping:

| Constant | RGB Values | Description |
|----------|------------|-------------|
| `DrGBase.CLR_WHITE_TR` | `Color(255, 255, 255, 0)` | Transparent white |
| `DrGBase.CLR_GREEN_TR` | `Color(150, 255, 40, 0)` | Transparent green |
| `DrGBase.CLR_RED_TR` | `Color(255, 50, 50, 0)` | Transparent red |
| `DrGBase.CLR_CYAN_TR` | `Color(0, 200, 200, 0)` | Transparent cyan |
| `DrGBase.CLR_PURPLE_TR` | `Color(220, 40, 115, 0)` | Transparent purple |
| `DrGBase.CLR_BLUE_TR` | `Color(50, 100, 255, 0)` | Transparent blue |
| `DrGBase.CLR_ORANGE_TR` | `Color(255, 150, 30, 0)` | Transparent orange |
| `DrGBase.CLR_DARKGRAY_TR` | `Color(20, 20, 20, 0)` | Transparent dark gray |
| `DrGBase.CLR_LIGHTGRAY_TR` | `Color(200, 200, 200, 0)` | Transparent light gray |

## Usage Examples

```lua
-- Using in surface drawing
surface.SetDrawColor(DrGBase.CLR_RED)
surface.DrawRect(x, y, w, h)

-- Using for entity colors
ent:SetColor(DrGBase.CLR_CYAN)

-- Using for debug overlays
debugoverlay.Text(pos, "Debug Info", 1, DrGBase.CLR_GREEN)

-- Color transitions (lerp from opaque to transparent)
local alpha = math.sin(CurTime()) * 0.5 + 0.5
local color = LerpVector(alpha, DrGBase.CLR_RED, DrGBase.CLR_RED_TR)
```

---

## See Also

- [UI Panels](./dpanels.md)
