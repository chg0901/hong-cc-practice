---
name: device_params duplicate data pattern
description: device_params table prone to duplicates (365 found 2026-04-10); three-layer fix applied
type: project
---

## device_params 重复数据问题（2026-04-10）

`device_params` 表存在大量重复记录（2585 行中有 365 组重复），每个 `(device_id, param_code)` 组合出现 2 次。

**Why**: 可能的根因是设备复制（copy project）或 device_library 同步逻辑重复插入，但未确认具体触发路径。

**修复（三层防御）**:
1. **数据库清理**: `DELETE FROM device_params WHERE id NOT IN (SELECT MIN(id) FROM device_params GROUP BY device_id, param_code)` — 2585→2220 行
2. **后端 API** (`device_query_routes.py`): SQL 加 `GROUP BY param_code ORDER BY MIN(sort_order)`
3. **前端** (`device-query.js`): `renderParamsList()` 按 `param_code` 去重后再渲染

**SQLite GROUP BY 注意**: SQLite 允许 `SELECT` 非聚合列不放 `GROUP BY`（与 MySQL `ONLY_FULL_GROUP_BY` 不同）。重复行字段值完全一致时没问题，不一致时需显式 `MIN()`。

**How to apply**: 查询 `device_params` 的地方都应考虑加 `GROUP BY` 或 `DISTINCT`。如果将来排查到根因（哪个操作产生重复），在写入时加 `(device_id, param_code)` UNIQUE 约束。
