---
name: interaction-check
description: Verify UI interaction behavior using Playwright. Test clicks, hovers, form submissions, modals, dropdowns, and navigation flows. Use after any frontend JS/HTML change.
---

# Interaction Check Skill

## When to Use

Trigger this skill when:
- Frontend JavaScript has been modified (event handlers, AJAX calls, animations)
- New interactive components added (buttons, modals, dropdowns, tabs)
- User reports interaction bugs (clicks not working, hover not expanding)
- After modifying SVG interaction code (zoom, pan, hover groups)
- Before committing frontend interaction changes

## Checklist

### 1. Structural Verification (browser_snapshot)

Verify DOM elements exist and have correct attributes:

```
browser_navigate → target URL
browser_snapshot → check element refs
```

Assertion patterns:
- Modal exists after trigger click: `heading "Modal Title" exists`
- Dropdown populated: `combobox has >1 option`
- Button enabled/disabled: `button [disabled]` or `button [cursor=pointer]`
- Table rows match expected count
- Alert dialog appears after action

### 2. Click Interaction

```
browser_click → button/modal trigger
browser_wait_for → 1s
browser_snapshot → verify result
```

Common patterns:
- **CRUD operations**: Click "新建" → modal opens → fill form → submit → table row added
- **Tab switching**: Click sidebar link → content area updates
- **Delete confirmation**: Click "删除" → confirm dialog → item removed
- **Form submission**: Fill fields → submit → success/error message

### 3. Hover Interaction (SVG)

For SVG hover interactions, Playwright hover often fails due to overlapping elements. Use dispatchEvent:

```javascript
async (page) => {
  // Trigger hover on SVG group element
  await page.evaluate(() => {
    const el = document.querySelector('svg .device-group[data-group-type="{type}"]');
    if (el) {
      el.dispatchEvent(new MouseEvent('mouseenter', { bubbles: true }));
    }
  });
  await page.waitForTimeout(500);
}
```

Verify:
- Expanded content appears (previously hidden elements visible)
- Z-order change (hovered element moves to front)
- Count badge shows correct number
- Leave event restores original state

### 4. Form Interaction

```
browser_snapshot → get field refs
browser_fill_form → fill multiple fields
browser_select_option → dropdown selection
browser_click → submit button
browser_handle_dialog → accept/dismiss alert
```

### 5. Navigation Flow

```
browser_navigate → home page
browser_click → project "进入" button
browser_wait_for → 1s
browser_click → sidebar menu item
browser_wait_for → 1s
browser_snapshot → verify content loaded
```

### 6. Console Error Check

After interaction, check for JavaScript errors:

```
browser_console_messages → level: "error"
```

Common errors to look for:
- `TypeError: Cannot read properties of undefined` — missing data
- `ReferenceError: xxx is not defined` — missing function
- `SyntaxError` — malformed JS
- Network errors — API endpoint not found

## Interaction Test Documentation

Record in work summary:

```markdown
### Interaction Test Results

| Test | Action | Expected | Actual | Result |
|------|--------|----------|--------|--------|
| I-1 | Click 3D tab | SVG scene loads | Scene visible | PASS |
| I-2 | Hover device group | Cards expand | 3 cards shown | PASS |
| I-3 | Click zoom + | Scene zooms in | Zoomed view | PASS |
```

## Key Gotchas

1. **Login required**: Most pages redirect to login. Always login first.
2. **Alert dialogs**: Many actions trigger `alert()`. Must call `browser_handle_dialog` before proceeding.
3. **AJAX delays**: Content loads async. Use `browser_wait_for` 1-3s after navigation.
4. **SVG hover workaround**: Module headers intercept Playwright hover. Use `dispatchEvent`.
5. **Project context**: Must "进入" a project before accessing sub-pages (3D, control, etc.).
6. **Session expiry**: Long test sessions may timeout. Re-login if page shows login form.
7. **Mermaid diagrams**: If documenting interaction flows in markdown, create mermaid text + render PNG per `.claude/rules/mermaid.md`. Must keep BOTH text version AND mermaid image. Preferred ratio: 4:3 > 16:9 > 3:4. After rendering, use ZAI `analyze_image` to verify no text overlap/overflow/truncation.

## Related Files

- Rules: [.claude/rules/visual-testing.md](.claude/rules/visual-testing.md)
- Rules: [.claude/rules/manual-testing.md](.claude/rules/manual-testing.md)
- Visual check skill: [.claude/skills/visual-check/SKILL.md](../visual-check/SKILL.md)
