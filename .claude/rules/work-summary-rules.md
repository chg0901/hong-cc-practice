# Work Summary & Documentation Rules

## Trigger

Documentation updates are **automatic** -- happen immediately after each task, no user prompt needed:
- Code change done -> write docs -> commit + push (all in one flow)
- Do not wait for user to say "write docs" or "commit"
- Do not ask permission before committing unless changes are destructive

## Files That Must Be Updated

| File | What to Update | When |
|------|---------------|------|
| `docs/work_summary_YYYYMMDD.md` | New numbered section + modified files table | Every task |
| `CLAUDE.md` | Database tables, module descriptions, latest Change Log entry (1 entry only) | New tables/modules |
| `docs/changelog.md` | Full change log history (all entries) | Every task |
| `.claude/rules/api-endpoints.md` | API endpoint reference | New/modified endpoints |
| `README.md` | Update log section | Significant features |

## Document-Level Structure

Each `docs/work_summary_YYYYMMDD.md` must follow this top-level order:

1. **## Insights** -- All `★ Insight` blocks from the session (FIRST section)
2. **## 一、二、三...** -- Individual task sections (middle)
3. **## 待办事项汇总** -- Consolidated TODOs
4. **## ChangeLogs** -- Date-topic entries for today (LAST section)

## Task Section Structure

Each numbered section must contain these subsections in order:

### 1. 现状 (Current State)
What is broken or missing, which files/modules are affected, user-facing symptoms.

### 2. 解决方案 (Solution)
Architecture/design decisions, why this approach over alternatives, key trade-offs.

### 3. 具体实施内容 (Implementation Details)
Files modified/created, API changes, database schema changes, frontend UI changes, code snippets.

### 4. 技术要点 / Insights
- Every `★ Insight` block from the conversation must be copied here **verbatim**
- Additional why/trade-off/gotcha notes specific to this codebase
- Format: `### 技术要点` -> `- **[主题]**：[说明 why/trade-off/gotcha]`
- Do NOT record general programming concepts -- only codebase-specific insights

### 5. 涉及文件 (Modified Files Table)

```markdown
| 文件 | 变更类型 | 说明 |
|------|----------|------|
| [filename.py](path/to/file.py) | 修改/新建 | 简要说明 |
```

File paths MUST use markdown hyperlinks `[filename](relative/path)`. Mermaid figures: `[figure.png](mermaid/figure/figure.png)` + source link.

### 6. 小结 (Section Summary)
2-3 sentences summarizing what was accomplished and any remaining items.

## CLAUDE.md Change Log Format

```markdown
### YYYY-MM-DD -- 简短主题
- [HH:MM:SS] Added/Fixed/Enhanced [description] ([file.py](file.py))
```

- Reverse chronological order (newest first)
- Same-day: `### YYYY-MM-DD -- 主题描述` (NO numeric suffixes)
- **每个条目必须包含时间戳** `[HH:MM:SS]`
- Link to relevant files using markdown `[name](path)` format

## Insight Recording Rule

At conversation end, collect ALL `★ Insight` blocks:
- Write to **Insights** section at the top of work summary
- Each insight: bullet point with **bold topic** + explanation
- **Insights 必须使用中文**：即使原文是英文，也需提供中文翻译

## Hyperlinks and Internal Navigation

### Within same file (anchor links)
- Chinese/CJK characters kept as-is, ASCII letters/numbers kept, spaces -> hyphens
- Punctuation stripped: `、（）【】《》""''。，！？：；/\---...` etc.
- `#` prefix always lowercase

```markdown
## 二十、功能增强实现
-> anchor: #二十功能增强实现
[见第二十节](#二十功能增强实现)
```

### Across files (relative path links)
```markdown
[见 work_summary_20260331.md](../docs/work_summary_20260331.md#十xxx)
[test_xxx.py](../test_codes/test_xxx.py)
```

### When to add hyperlinks
- 待办清单 `见第XX节` / `来自MMDD` -> 必须加锚链接
- 涉及文件表格 -> 相对路径链接
- 章节引用 -> 加锚链接
- 跨文档引用 -> 完整相对路径链接

## Sections to Check

When adding new features:
- `CLAUDE.md ## Database Structure` -- New tables listed
- `CLAUDE.md ## Frontend Module System` -- New modules/tabs described
- `CLAUDE.md ## Change Log` -- Latest entry only; full entry goes to `docs/changelog.md`
- `.claude/rules/api-endpoints.md` -- New endpoints with correct HTTP methods
- `.claude/rules/data-scripts.md` -- LSTM config or data processing changes

## Automated Test Documentation

When adding test files:
1. `testing.md`: Add to "Existing Test Files" table
2. Work summary: Add test results table with pass/fail counts
3. Link test files: `[test_xxx.py](../test_codes/test_xxx.py)`

## ChangeLogs

- [2026-04-13 20:56:00] Initial: merged from changelog.md + doc-structure.md