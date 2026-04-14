# Visual & Interaction Verification Rules

## MANDATORY: Auto-Trigger Skills After Frontend Changes

**Before committing ANY frontend file change, invoke the matching skill:**

| Modified file | Invoke skill | MCP tools used |
|---------------|-------------|----------------|
| `static/svg/schematic_generator.js` | `/visual-check` | Playwright + MiniMax |
| `static/svg/device_icons.js` | `/visual-check` | Playwright + MiniMax |
| `js/modules/3d.js` | `/visual-check` + `/interaction-check` | Playwright + MiniMax |
| `js/modules/control.js` | `/interaction-check` | Playwright |
| `js/modules/meteorology.js` | `/interaction-check` | Playwright |
| `js/modules/*.js` (event logic) | `/interaction-check` | Playwright |
| Any `.css` / layout HTML | `/visual-check` | Playwright + MiniMax |

Note: The PostToolUse Edit|Write hook in settings.json auto-triggers these skills for SVG/CSS/JS/mermaid files. This table documents the mapping; the hook handles enforcement.

For mermaid rendering and quality rules, see [mermaid.md](mermaid.md).

**Rule: SVG/JS/CSS rendering code changed = you MUST run `/visual-check` before commit. No exceptions.**

## Three-Layer Test Stack

| Layer | Tools | When to use |
|-------|-------|-------------|
| API tests | `requests` (Python) | Backend CRUD, always run first |
| Structural tests | Playwright `browser_snapshot` | DOM existence, text content |
| Visual tests | Playwright screenshot + Vision MCP | Layout, icons, fonts, charts, colors |

**Order**: API tests -> Structural snapshot -> Visual screenshot -> Vision analysis

## Vision MCP Tools Quick Reference

### MiniMax (`mcp__MiniMax__understand_image`) — PRIMARY visual tool
- Full-page UI analysis, layout, alignment, icon rendering, font consistency
- Input: local file path or URL + analysis prompt
- Use as first-pass for all visual verification

### ZAI (`mcp__zai-mcp-server__*`) — SPECIALIZED tools
- `extract_text_from_screenshot` — OCR for Chinese + numbers
- `ui_diff_check` — Compare baseline vs current screenshot (regression)
- `analyze_data_visualization` — Charts, graphs, dashboards data correctness
- `diagnose_error_screenshot` — Error message diagnosis
- `analyze_image` — General image analysis (cross-validation fallback)

### GLM/智谱 (`mcp__4_5v_mcp__analyze_image`) — CROSS-VALIDATION
- Remote URL image analysis, supports detailed prompts
- Use for cross-validation when MiniMax and ZAI disagree on a visual finding
- Input: remote URL + prompt (does NOT support local file paths)

## Cross-Validation Strategy

When visual findings are critical (font consistency, subtle alignment issues), use multiple MCPs:

| Scenario | Primary | Cross-validator | When to cross-validate |
|----------|---------|-----------------|----------------------|
| Font/spacing consistency | MiniMax | ZAI `analyze_image` | Always for SVG card changes |
| Text OCR correctness | ZAI `extract_text` | MiniMax | When Chinese text may be garbled |
| Regression comparison | ZAI `ui_diff_check` | MiniMax | After refactoring existing UI |
| Chart data accuracy | ZAI `analyze_data_viz` | MiniMax | For EDA/weather dashboards |
| Layout overflow | MiniMax | GLM `analyze_image` | When MiniMax uncertain |

**Cross-validation rule**: If two MCPs disagree, take a third screenshot with different zoom level and re-analyze.

### Playwright (`mcp__plugin_playwright_playwright__*`) — BROWSER automation
- `browser_navigate` / `browser_snapshot` / `browser_take_screenshot`
- `browser_click` / `browser_fill_form` / `browser_select_option`
- `browser_handle_dialog` — Accept/dismiss alert dialogs
- `browser_run_code` — Custom JS (hover workaround, scroll, etc.)
- `browser_console_messages` — Check for JS errors

## Quick Workflow: Login + Navigate + Screenshot

```
1. browser_navigate -> http://127.0.0.1:5000
2. browser_snapshot -> find login button ref
3. browser_click -> login (auto-filled admin/admin123)
4. browser_handle_dialog -> accept "登录成功" alert
5. browser_wait_for -> 2s
6. browser_run_code -> click "进入" + target tab
7. browser_wait_for -> 2s
8. browser_take_screenshot -> test_screenshots/{name}_{date}.png
9. mcp__MiniMax__understand_image -> analyze screenshot
```

## SVG Hover Workaround

Playwright `.hover()` is intercepted by `.module-header-with-back` div overlay. Use:

```javascript
await page.evaluate(() => {
  const el = document.querySelector('svg .device-group[data-group-type="{type}"]');
  el.dispatchEvent(new MouseEvent('mouseenter', { bubbles: true }));
});
```

## EDA Data Verification (matplotlib)

For data quality checks (weather, sensor, energy):

```python
df = pd.read_sql_query('SELECT city, temperature FROM weather_data', conn)
df.boxplot(column='temperature', by='city')
plt.savefig(f'visualizations/eda_{timestamp}.png', dpi=150)
```

Then analyze with MiniMax or `analyze_data_visualization`.

## Screenshot Naming

`test_screenshots/{project}_{area}_{YYYYMMDD}.png`

- `qingdao_3d_scene_full_20260409.png` — Full SVG scene
- `qingdao_controller_area_20260409.png` — Zoomed controller section
- `qingdao_group_expanded_20260409.png` — Expanded device group

## Vision MCP Result Documentation

Record in work summary under the task section:

```markdown
### Vision MCP 验证结果
| 验证项 | 结果 | 说明 |
|--------|------|------|
| Font size consistency | PASS | MiniMax confirmed all cards identical |
```

## Related Skills
- `/visual-check` — Full visual verification workflow (6-step checklist)
- `/interaction-check` — Interaction behavior testing (6 categories)

## Related Test Files
- [test_visual_svg_cards.py](../../test_codes/test_visual_svg_cards.py) — API + Vision MCP results (11 tests)
- [static/test_device_icons.html](../../static/test_device_icons.html) — Browser-side JS unit tests

## Mermaid Diagram Requirement

Any architecture diagram, flow chart, or layered structure described in visual testing documentation **must** be rendered as mermaid diagram + PNG alongside the text version. See `.claude/rules/mermaid.md` for rendering commands and color conventions.

## ChangeLogs
- [2026-04-13 20:56:00] Deduplicated: removed mermaid rendering rules (pointer to mermaid.md), simplified trigger table (PostToolUse hook is authoritative)
- [2026-04-09 — Consolidated visual-testing.md + visual-testing-workflow.md + visual-verification.md](changes/2026-04-09)
- [2026-04-08 — Initial: Three-layer test stack, Vision MCP tools, TODO mapping](changes/2026-04-08)
