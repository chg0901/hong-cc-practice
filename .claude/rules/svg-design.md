# SVG Scene Design Rules

## Responsive SVG Layout

All SVG scene/schematic views MUST be responsive:

1. **SVG sizing**: `width: 100%; height: auto` — viewBox scales with container width
2. **NO fixed max-height** on SVG element — let content determine height
3. **Container**: `max-height: 80vh` with `overflow: auto` for scrollable large diagrams
4. **Dynamic viewBox height**: Calculate from device count before closing `</svg>`
5. **Breakpoints**: No fixed pixel widths — use percentage-based layouts

## Design System — Unified Across All Projects

### Color Tokens (Semantic)

| Token | Value | Usage |
|-------|-------|-------|
| `bg-dark` | `#1a1a2e` | SVG scene background base |
| `card-bg` | `#ffffff` | Device card background |
| `card-bg-inactive` | `#f5f5f5` | Offline/disabled card |
| `accent-sensor` | `#3498db` | Sensor icons, connections |
| `accent-controller` | `#e74c3c` | Controller icons, connections |
| `accent-subsystem-irrigation` | `#4caf50` | Irrigation subsystem |
| `accent-subsystem-climate` | `#ff9800` | Climate control subsystem |
| `accent-subsystem-cooling` | `#2196f3` | Cooling/refrigeration subsystem |
| `status-on` | `#4caf50` | Active/running status |
| `status-off` | `#9e9e9e` | Inactive/offline status |
| `status-fault` | `#f44336` | Fault/error status |
| `status-standby` | `#2196f3` | Standby mode |
| `text-primary` | `#ecf0f1` | Scene titles on dark bg |
| `text-secondary` | `#b0bec5` | Subtitles, labels on dark bg |
| `text-card` | `#424242` | Device names on card bg |
| `group-badge` | `#e53935` | Device group count badge |
| `group-border` | `#42a5f5` | Device group expanded border |

### City/Theme Accent Colors

Defined in `SchematicGenerator.projectProfiles`:
- Coastal (Qingdao): `#00bcd4`
- Tech (Beijing): `#7c4dff`
- Tropical (Guangzhou): `#ff6d00`
- Watertown (Hangzhou): `#26a69a`
- Basin (Chengdu): `#ff5722`
- Government (Nanjing): `#1565c0`
- Industrial (Qingdao cooling): `#546e7a`
- Commercial (Beijing cooling): `#ffa000`
- Skyscraper (Shanghai): `#90a4ae`

### Typography

| Role | Size | Weight | Color | Font |
|------|------|--------|-------|------|
| Scene title | 16px | bold | `text-primary` | Microsoft YaHei |
| Subtitle | 11px | normal | `text-secondary` | Microsoft YaHei |
| Zone label | 12px | bold | `text-primary` | Microsoft YaHei |
| Subsystem label | 10px | bold | subsystem accent | Microsoft YaHei |
| Device name | 11px | normal | `text-card` | sans-serif |
| Status text | 9px | bold | status color | sans-serif |
| Badge count | 9-10px | bold | `#ffffff` | sans-serif |

### Card Dimensions

| Element | Width | Height | Icon Size |
|---------|-------|--------|-----------|
| Sensor card (zone) | 85px | 100px | 42px |
| Controller card | 90px | 105px | 44px |
| Equipment room card | 80px | 95px | 38px |
| Terminal (floor) | 78px | 90px | 38px |
| Group card (collapsed) | 95px | 110px | 48px |
| Pump station | 68px | 85px | 34px |

### Subsystem Grouping Rules

Controllers are classified into subsystems:

| Subsystem | Device Types | Label | Color |
|-----------|-------------|-------|-------|
| Irrigation | valve, irrigation_valve, pump, pump_chilled, wet_pad_pump, sprinkler, fertigation | Irrigation | `#4caf50` |
| Climate Control | fan, light, light_controller, shade_controller, heater_controller, co2_generator | Climate | `#ff9800` |
| Cooling System | chiller, cooling_tower, heat_pump_cooling, heat_pump_heating, pump_heating, boiler, heat_exchanger | Cooling | `#2196f3` |

### Layout Rules

1. **Max columns per row**: `Math.floor((areaWidth + gap) / (cardWidth + gap))`
2. **Zone width**: `(svgWidth - 80) / 3 - 10` (3 zones in agriculture)
3. **Auto-rows**: Devices wrap to next row when exceeding columns
4. **SVG height**: Dynamically calculated from total content height
5. **Subsystem sections**: Dashed border with label, colored by subsystem type
6. **Connection lines**: Animated dashed lines from control center to each subsystem section

### Interaction Rules

1. **Zoom**: Buttons (reset/+/) + mouse wheel (centered on cursor) + drag to pan
2. **Device groups**: Collapsed by default (1 icon + count badge), CSS hover to expand
3. **Z-order**: Hovered group moved to SVG root (absolute coords), restored on mouseleave
4. **No hover-only interactions**: All hover actions have click alternatives where applicable

## Verification After SVG Changes

After modifying SVG scene code, the PostToolUse hook auto-triggers `/visual-check`. See [visual-testing.md](visual-testing.md) for the full verification workflow (Playwright + Vision MCP).

## SVG Rendering Pitfalls

### pointer-events 阻塞

不可见的 `<rect>` 背景元素会阻塞子元素的鼠标事件。始终对纯装饰性背景矩形添加 `pointer-events="none"`。

### CJK 感知文本截断

固定字符数（如 7 字符）对中文文本无效（每个 CJK 字符 = 2 宽度单位）。使用宽度感知截断：

```javascript
const cjkLen = (s) => [...s].reduce((a, c) => a + (/[\u4e00-\u9fff]/.test(c) ? 2 : 1), 0);
```

### Z-Order 提升模式

展开的分组需要临时提升到 SVG 根节点：
1. 计算绝对坐标（累加父级 translate）
2. `appendChild` 移到 SVG 根节点
3. `mouseleave` 时恢复原始父级和位置

## ChangeLogs

- [2026-04-13 20:56:00] Simplified verification section: PostToolUse hook now auto-triggers /visual-check
- [2026-04-09 — Added mandatory visual verification after SVG changes](changes/2026-04-09)
- [2026-04-08 — Initial: responsive SVG rules, design tokens, subsystem grouping, typography](changes/2026-04-08)
