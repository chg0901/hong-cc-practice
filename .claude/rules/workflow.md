# Project Workflow Rules

## Documentation Tracking
- Update `README.md` changelog after each significant modification
- Write plans and context contraction summaries to `My_Plans.md`
- When implementing new features, fixing bugs, or modifying code, update all related README/docs (e.g. `CLAUDE.md`, `docs/readme_*.md`, module-specific readmes) in the same commit
- If unsure which docs are affected, grep for references to the changed file/module across `*.md` files

## Work Summary Update

See [work-summary-rules.md](work-summary-rules.md) for full rules: trigger, document structure, section format, ChangeLogs format, hyperlinks, insight recording.

## File Move Safety

**Lesson learned**: Moving `db_manager.py` to `scripts/tools/` broke `smart_energy_platform.py` at runtime (import `from db_manager import get_db_manager` failed). Always grep before moving.
- Before moving/archiving any file, grep for all imports and references to it across the project
- If any active code imports the file, either keep it in place or update all import paths
- Run `python -c "import module_name"` to verify imports still work after moving files
- Pay special attention to files imported by the main application (smart_energy_platform.py)

## Testing & Validation
- Test modified or newly added code before committing
- Validate LSTM training with python scripts; visualize training logs and add result images to reports
- Add usage/reproduction commands alongside results in reports

## Report Figures
- When adding figures to markdown reports, include explanations and findings
- Image links must not contain line ranges (no `:1400-1450` in image URLs)
- Show figure in report, add superlinks for source scripts and output files

## Git Proxy Bypass

See [proxy-rules.md](proxy-rules.md) for all NO_PROXY patterns and environment variables.

## Daily Branch Workflow (mandatory)

**Why**: Keeps main clean, isolates daily work for easy review/rollback, makes commit history legible by date.

Use a **new git branch per day** as the default development pattern:

```bash
# Session start — create today's branch
git checkout main
git pull origin main
git checkout -b 2026-04-04          # use today's date

# Session end — merge back and clean up
git checkout main
git merge --no-ff 2026-04-04 -m "feat/fix/docs: summary of today's work"
git push origin main
git branch -d 2026-04-04
```

**Branch naming**: `YYYY-MM-DD` or `YYYY-MM-DD-<topic>` for multi-day features

**Rules**:
- At every new session, check current branch — if on main, create today's branch immediately
- Do NOT accumulate multiple days on one branch
- Merge commit message summarizes all tasks done that day
- After merge, always delete the day branch locally

## Function/Code Deletion Rules

### 删除前检查清单

删除任何函数或大段代码前，**必须**完成以下检查：

1. **Grep 全局引用**：在整个项目中搜索函数名，确认无其他调用者
   ```bash
   grep -rn "function_name" --include="*.py" --include="*.js" --include="*.html" .
   ```
2. **备份文件**：`cp target_file.py target_file.py.bak`
3. **备份数据库**（若涉及数据变更）：`shutil.copy2('smart_energy.db', 'smart_energy_backup_before_xxx.db')`
4. **识别关联数据**：确认删除不会影响关键功能（admin 用户、模板库、权限、表结构等）

### 删除后验证

1. **语法检查**：`python -c "import py_compile; py_compile.compile('file.py', doraise=True)"`
2. **导入测试**：`python -c "import module_name"`
3. **功能验证**：启动后端确认无报错，检查关键数据（项目数、设备数、admin 用户）
4. **数据一致性**：确认数据库中无孤立数据

### 记录要求

删除操作必须在 work_summary 中记录：
- 删除的函数名和行数
- 影响分析（哪些功能不受影响）
- 验证结果（语法/导入/启动/数据）
- 备份文件路径
