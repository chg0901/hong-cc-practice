# Manual UI Testing Rules

## When to Create Manual Test Items

Every new frontend feature (new tab, modal, CRUD UI, interactive component) must have corresponding manual test items in the work summary. These cover interactions that cannot be verified via API tests alone:

- Modal open/close, form field rendering
- Cascading dropdown behavior (company -> project)
- Tab switching and content toggle
- Table data display after CRUD operations
- Animation and visual feedback (color changes, folding)
- Browser-specific rendering issues

## Manual Test Item Format

```markdown
#### UI-N. 功能名称（新/已有）
- [ ] 操作步骤1 → 预期结果1
- [ ] 操作步骤2 → 预期结果2
```

Rules:
- Each item describes ONE user action and its expected outcome
- Use arrow `→` to separate action from expected result
- Mark with `（新）` if this is a newly added feature
- Include prerequisite steps if needed (e.g., "先新增一条记录")

## Prerequisites Section

Always include this before the UI test items:

```markdown
> 前置操作：重启后端 `conda activate ene && python smart_energy_platform.py`，浏览器访问 `http://127.0.0.1:5000`，登录 admin/admin123
```

## After Manual Testing

- Mark each item as `- [x]` when verified in browser
- If a test fails, document the issue inline: `- [ ] 操作 → 预期 **[FAIL: 实际现象描述]**`
- Create a new numbered section in work_summary to describe the fix
- Re-test and mark as passed after fix

## Relationship to Automated Tests

- Automated API tests (in `test_codes/`) cover backend CRUD correctness
- Manual UI tests cover frontend rendering, interaction, and UX
- Both must pass before a feature is considered complete
- Record automated test results in work_summary alongside manual test checklist

## Visual Verification for Manual Test Items

Items involving visual rendering should be verified with Vision MCP. The PostToolUse hook auto-triggers `/visual-check` for SVG/CSS changes. See [visual-testing.md](visual-testing.md) for the full verification workflow.
