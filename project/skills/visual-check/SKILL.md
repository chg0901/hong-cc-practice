---
name: visual-check
description: Verify UI rendering quality using Playwright + Vision MCP. Use after any SVG/CSS/font/layout change to catch visual inconsistencies before committing.
---

# Visual Check Skill

## When to Use

Trigger this skill when:
- SVG scene rendering has been modified (fonts, icons, layout, colors)
- CSS changes affect card layouts or typography
- New UI components have been added to any page
- User reports visual inconsistencies
- Before committing frontend rendering changes

## Checklist

For each visual check, follow these steps:

### 1. Pre-check: Verify Server Running

```bash
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:5000/api/login
```

If not running, ask user to start: `conda activate ene && python smart_energy_platform.py`

### 2. Navigate & Login

```
browser_navigate → http://127.0.0.1:5000
browser_snapshot → find login button ref
browser_click → login button (auto-filled admin/admin123)
browser_handle_dialog → accept alert
browser_wait_for → 2s
```

### 3. Navigate to Target Page

Use `browser_run_code` for complex navigation (entering project, clicking tabs):

```javascript
async (page) => {
  // Enter first project
  const btn = document.querySelector('button'); // find "进入" button
  // ... click project enter button
  // ... click target tab link
}
```

### 4. Take Screenshots

**Full page:**
```javascript
await page.locator('svg.schematic-svg').first().screenshot({ 
  path: 'test_screenshots/{name}_full_{date}.png', type: 'png' 
});
```

**Zoomed area** (scroll to relevant section first):
```javascript
await page.evaluate(() => { 
  document.querySelector('main').scrollTop = scrollPosition; 
});
await svgEl.screenshot({ path: 'test_screenshots/{name}_zoomed_{date}.png', type: 'png' });
```

**Hover interaction** (use dispatchEvent, NOT Playwright hover):
```javascript
await page.evaluate(() => {
  const el = document.querySelector('svg .device-group[data-group-type="{type}"]');
  el.dispatchEvent(new MouseEvent('mouseenter', { bubbles: true }));
});
```

### 5. Analyze with Vision MCP

Use `mcp__MiniMax__understand_image` with specific prompts:

For **font consistency**: "Compare the font sizes of ALL card names. Are they EXACTLY the same? List each card name and its apparent size."

For **spacing consistency**: "Compare the gap between icon bottom and text top across ALL cards. Is it consistent?"

For **name uniqueness**: "Are all device names unique? List any duplicates."

For **layout quality**: "Are cards evenly spaced? Any overflow beyond dashed borders?"

### 6. Document Results

Record in work summary:

```markdown
### Vision MCP 验证结果

| 验证项 | 结果 | 说明 |
|--------|------|------|
| Font size consistency | PASS/FAIL | Detail |
| Icon-text spacing | PASS/FAIL | Detail |
| Card dimensions | PASS/FAIL | Detail |
| Name uniqueness | PASS/FAIL | Detail |
| Chinese rendering | PASS/FAIL | Detail |
```

### 7. Structured Visual Verdict（来自 OMC visual-verdict）

每次视觉检查后，输出结构化 JSON 便于后续自动化对比和回归检测：

```json
{
  "page": "cooling-dashboard",
  "timestamp": "2026-04-26T10:00",
  "checks": [
    {"item": "font_consistency", "status": "PASS", "detail": "All cards use 11px sans-serif"},
    {"item": "icon_rendering", "status": "FAIL", "detail": "chiller icon missing at row 3", "severity": "P1"},
    {"item": "layout_overflow", "status": "PASS", "detail": "No text overflow beyond card borders"},
    {"item": "chinese_rendering", "status": "PASS", "detail": "All CJK characters rendered correctly"},
    {"item": "color_consistency", "status": "PASS", "detail": "Status colors match design tokens"},
    {"item": "spacing_alignment", "status": "PASS", "detail": "Cards evenly spaced, no overlap"}
  ],
  "summary": {"pass": 5, "fail": 1, "total": 6},
  "screenshots": ["test_screenshots/cooling_dash_20260426.png"]
}
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| `page` | string | 页面标识（如 `cooling-dashboard`, `home`, `3d-scene`） |
| `timestamp` | ISO 8601 | 检查时间 |
| `checks[].item` | string | 检查项名称（snake_case） |
| `checks[].status` | PASS/FAIL | 检查结果 |
| `checks[].detail` | string | 具体描述 |
| `checks[].severity` | P0-P3 | 仅 FAIL 时需要，P0=阻断，P1=重要，P2=一般，P3=建议 |
| `summary` | object | 统计：pass/fail/total |
| `screenshots` | array | 关联的截图文件路径 |

**与 ZAI ui_diff_check 配合**：使用 `ui_diff_check` 工具对比基线截图和当前截图时，将差异结果合并到此 JSON 格式中。

### 7. Mermaid Visual Quality Check (when .mmd files are modified)

After rendering any mermaid PNG, MUST verify quality:

1. **Aspect ratio check**: `python -c "from PIL import Image; w,h=Image.open('png').size; r=w/h; print(f'{r:.2f}', 'OK' if 0.33<=r<=3.0 else 'FAIL')"`
2. **Vision MCP text quality**: Use ZAI `analyze_image` with prompt: "Check ALL nodes for: text overlap, text overflow, text truncation, node overlap, garbled Chinese. List each node status OK or FAIL."
3. **Fix if needed**: Shorten text, use `~~~` spacing, change TB↔LR, or split diagram. Re-render and re-check.

## Screenshot Naming

`{project}_{area}_{YYYYMMDD}.png`

- `qingdao_3d_scene_full_20260409.png` — Full SVG scene
- `qingdao_controller_area_20260409.png` — Controller section
- `qingdao_group_expanded_20260409.png` — Expanded device group

## Key Gotchas

1. **Playwright hover intercepted**: Module header `<div>` intercepts pointer events on SVG groups. Use `dispatchEvent(new MouseEvent('mouseenter'))` instead.
2. **SVG inside scrollable container**: Use `main.scrollTop` to scroll to controller area before zoomed screenshots.
3. **Console errors**: Check `browser_console_messages` after navigation — SVG generation errors may indicate data issues.
4. **MiniMax analysis precision**: Ask for pixel-level comparison with specific card names for better results.
5. **Mermaid diagrams**: If documenting architecture or flow in markdown, create mermaid text + render PNG per `.claude/rules/mermaid.md`. Must keep BOTH text version AND mermaid image. Preferred ratio: 4:3 > 16:9 > 3:4. After rendering, use ZAI `analyze_image` to verify no text overlap/overflow/truncation.

## Related Files

- Rules: [.claude/rules/visual-testing.md](../../.claude/rules/visual-testing.md)
- Rules: [.claude/rules/mermaid.md](../../.claude/rules/mermaid.md)
- Test script: [test_codes/test_visual_svg_cards.py](../../../test_codes/test_visual_svg_cards.py)
- Screenshots: [test_screenshots/](../../../test_screenshots/)
