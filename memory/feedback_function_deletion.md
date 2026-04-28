---
name: Function deletion checks
description: Before deleting functions/code: grep references, backup files+DB, verify syntax+import+startup, record in work_summary
type: feedback
---

## Function Deletion Checklist

**Why**: Deleted `generate_test_data()` + `force_regenerate_test_data()` from `smart_energy_platform.py` (300 lines). These were recreating company1/company2 test projects on every restart or new DB creation.

**How to apply**: Before deleting ANY function or large code block:

1. **Grep** all callers (`grep -rn "func_name" --include="*.py" .`)
2. **Backup** the file (`cp file.py file.py.bak`) and DB if data changes
3. **Identify** what depends on it — check that critical paths (admin user, templates, permissions, table creation) are NOT in the deleted code
4. **After deletion**: verify syntax (`py_compile`), import test (`import module`), startup test, data integrity check
5. **Record** in work_summary: deleted function names, line counts, impact analysis, backup paths, verification results

**Key lesson**: `init_database()` handles all critical initialization (tables, admin, templates, permissions). `generate_test_data()` was ONLY for test company1/company2 data generation. Never confuse the two.