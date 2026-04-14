# TODO Tracking Rules

## Per-Section TODO Subsections (mandatory)

Every task section (一、二、三...) that generates pending work must include a `### 待办事项` subsection **within the section itself**, immediately after the `### 小结`. Place this before any section-closing `---` separator.

Rules:
- Automated tests that **passed** this session: record under `### 自动化测试汇总（已完成）` — not a TODO item
- Manual UI tests generated in this session: list as `- [ ]` items with `#### UI-N. 标题` format
- Future feature work triggered by this session: list as `- [ ]` items under a brief header
- Sections with **no pending work** (all auto-verified, no UI changes): **omit** `### 待办事项`

Example:
```markdown
### 待办事项

> 前置操作：重启后端 ...

#### UI-1. 功能名称（新）
- [ ] 操作步骤 → 预期结果

#### 功能待办
- [ ] 方案B: Token认证...
```

## Final Consolidated Chapter (mandatory)

At the end of every work summary (after all task sections, before `## ChangeLogs`), add:

```markdown
## 九、待办事项汇总与手动测试
```

(Number as 九/十/十一/... depending on how many task sections exist)

This chapter must contain:
1. **A. 遗留待办** — carried-forward items from previous sessions, with date attribution
2. **B. 本日新功能 UI 测试索引** — cross-reference table pointing to per-section 待办事项 subsections
3. **C. 功能待办** — all functional TODOs from all sections
4. **D. 自动化测试汇总（最终）** — comprehensive table of ALL test files with counts and results

Do NOT duplicate the individual UI test items in the final chapter — use the cross-reference table to keep them DRY. Only the functional TODOs (方案B, enhancement ideas, etc.) should be listed in full in the final chapter.

## Work Summary Structure

Every work summary file (`docs/work_summary_YYYYMMDD.md`) must end with a consolidated "待办事项汇总" section containing:

1. **遗留文档待办** - Carried forward from previous summaries (with source date noted)
2. **需人工验证的UI测试项** - Manual browser tests (see manual-testing.md)
3. **功能增强建议** - Future enhancement ideas (prioritized)
4. **自动化测试汇总** - Table of all test files, counts, and results

## Cross-Session TODO Consolidation

When starting a new work summary or updating an existing one:

1. Read ALL previous work summaries (`docs/work_summary_*.md`) for unchecked `- [ ]` items
2. Carry forward any still-pending items to the current summary's consolidated section
3. Mark items as completed `- [x]` in the original file when done
4. Remove duplicate items (check for identical or near-identical entries)
5. Add source attribution: `（来自 MMDD）` for carried-forward items

## TODO Lifecycle

```
New task identified → Add as `- [ ]` in current work_summary
  ↓
Task completed → Mark `- [x]` in work_summary, record result
  ↓
Next session → Scan for remaining `- [ ]`, carry forward to new summary
```

## When to Update TODOs

- After completing each major task (not just at session end)
- When discovering new issues during testing or implementation
- When user reports bugs or requests features
- Before committing: ensure TODO list is current

## Format Rules

- Use `- [ ]` for pending, `- [x]` for completed
- Group by category (文档/UI测试/功能增强/自动测试)
- For UI test items, use the `#### UI-N. 标题` format with checkbox sub-items
- Include "前置操作" instructions before manual test sections
