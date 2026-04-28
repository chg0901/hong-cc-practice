# Automated Testing Rules

## Mandatory Testing

Every new feature or bug fix MUST include automated tests before committing:

1. **API endpoint tests**: For every new/modified API endpoint, write a test that:
   - Creates test data via the API
   - Verifies the response code and data structure
   - Tests the full lifecycle (create -> read -> update -> delete -> restore if soft-delete)
   - Cleans up all test data in a `finally` block

2. **Soft-delete tests**: For any module with CRUD, test the complete cycle:
   - Create -> Delete -> Verify in deleted list -> Restore -> Verify restored

3. **Edge case tests**: Test error paths (missing params, invalid IDs, permission checks)

## Test File Conventions

- Test files go in `test_codes/` directory
- Naming: `test_{feature_name}.py` (e.g., `test_project_crud_and_soft_delete.py`)
- Each test file must be runnable standalone: `python test_codes/test_xxx.py`
- Tests must check server availability before running
- Tests must clean up all created data (use try/finally)
- Print `[PASS]` or `[FAIL]` per test, summary at end with total/passed/failed counts
- Exit code 0 = all pass, 1 = failures

## Test Structure Template

```python
BASE = 'http://127.0.0.1:5000'
results = []

def test(group, name, ok, detail=''):
    results.append((group, name, ok, detail))
    print(f'  [{"PASS" if ok else "FAIL"}] {name}' + (f' -- {detail}' if detail else ''))

# ... test functions ...

# Summary
passed = sum(1 for _, _, ok, _ in results if ok)
failed = len(results) - passed
print(f'Total: {len(results)} | Passed: {passed} | Failed: {failed}')
```

## Running Tests on Windows

See [proxy-rules.md](proxy-rules.md) for NO_PROXY requirements. Test command format:

```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe test_codes/test_xxx.py
```

## When to Run Tests

- After implementing any backend API changes: run the relevant test file
- Before committing: run all related tests, note results in commit message or work_summary
- Test results must be recorded in `docs/work_summary_YYYYMMDD.md`

## Existing Test Files

Full catalog (23 test files with coverage descriptions and run commands):
→ See [memory/reference_test_catalog.md](memory/reference_test_catalog.md)

## Visual UI Testing (Playwright + Vision MCP)

See [visual-testing.md](visual-testing.md) for full rules on the three-layer test stack:
- **Playwright `browser_snapshot`**: DOM structure, element existence, text content
- **Playwright `browser_take_screenshot`** + **MiniMax/ZAI Vision MCP**: Layout, icons, charts, colors
- Screenshots saved to `test_screenshots/` (gitignored, ephemeral artifacts)

## Test Data Cleanup Rules

### 原则

测试数据**必须在测试完成后立即清理**，不得残留到下一次测试或影响生产界面。

### 测试隔离：Pre-cleanup 原则

**规则**：永远不要假设数据库为空。每个测试组在开始前必须执行 pre-cleanup。

**为什么**：远期日期（2099）只避免日期冲突，不能防止 `< ?` 查询匹配到历史数据。软删除的测试数据仍会干扰消费量计算。

```python
# 正确的测试隔离模式
def test_something():
    try:
        # Pre-cleanup: 清理同 project+type 的已有记录
        existing = get_records(project_id=test_pid, meter_type='water')
        for rec in existing:
            delete_record(rec['id'])

        # Run test
        result = create_record(...)
        assert result['code'] == 200
    finally:
        # Post-cleanup: 删除本测试创建的记录
        delete_record(result['data']['id'])
```

**要点**：
- 使用当天日期（不用 2099），测试数据更真实
- `reporter` 字段标记为 `test_xxx_{timestamp}`，便于识别
- Pre-cleanup + post-cleanup 双重保障

### 清理方式

1. **自动化测试（test_codes/）**：每个测试文件的 `finally` 块中必须用 `DELETE` API 软删除所有测试创建的数据
2. **手动/探索性测试**：测试完成后手动调用 `DELETE` API 或直接物理删除
3. **测试项目命名**：测试创建的项目/设备使用 `_autotest_` 前缀或 `TestCo` 等明显测试标识，便于识别和清理

### 物理清理条件

以下情况执行物理 DELETE（而非软删除）：

- 项目 company 为 `TestCo`、`None`、`null` 等明显非生产值
- 项目/设备名称含 `_autotest_`、`test_` 前缀
- 软删除后仍在前端下拉列表中出现（如公司筛选器）
- 关联数据不完整（无系统、无设备、无传感器数据的项目）

### 物理清理步骤

```python
# 1. 备份数据库
shutil.copy2('smart_energy.db', 'smart_energy_backup_before_cleanup.db')

# 2. 按依赖顺序删除
DELETE FROM sensor_data WHERE device_id IN (SELECT id FROM devices WHERE project_id=?)
DELETE FROM control_logs WHERE device_id IN (SELECT id FROM devices WHERE project_id=?)
DELETE FROM devices WHERE project_id=?
DELETE FROM systems WHERE project_id=?
DELETE FROM faults WHERE project_id=?
DELETE FROM energy_consumption WHERE project_id=?
DELETE FROM projects WHERE id=?

# 3. VACUUM（大量删除后）
# 4. 记录到 work_summary（表、条件、行数）
```

### 验证清理结果

```sql
-- 确认无测试公司出现在活跃项目中
SELECT DISTINCT company FROM projects WHERE status=1;
-- 确认无残留设备
SELECT COUNT(*) FROM devices WHERE project_id NOT IN (SELECT id FROM projects);
```

## Documentation

- Link test files in docs using relative paths: `[test_xxx.py](../test_codes/test_xxx.py)`
- Record test results (total/passed/failed) in work_summary with date
- If a test reveals a bug, document the bug and fix in work_summary before committing