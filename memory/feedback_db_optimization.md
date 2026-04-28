---
name: SQLite 优化经验
description: 从两次数据库优化实践中总结的模式：查询优化（索引+重写）和结构优化（增量迁移），复用 /db-optimization skill
type: feedback
originSessionId: e0acf8cd-abd6-4c7e-ac65-ea954ed02fff
---
- **SQLite 窗口函数陷阱**：`ROW_NUMBER() OVER (PARTITION BY ...)` 在 14M+ 行表上物化全量结果，即使只需要 `rn=1`。逐条 `ORDER BY DESC LIMIT 1` + 复合索引是更好的模式，204.8s→0.011s（18,600x）
  - **Why**: SQLite 必须先排序全量数据再编号，无法利用索引
  - **How to apply**: 当需要"每组最新/第一条"时，避免窗口函数，改用 N 次索引查找

- **`OR IS NULL` 反模式**：`LEFT JOIN ... OR f.project_id IS NULL` 阻止索引使用，触发全表扫描 + 锁等待。改用 `INNER JOIN ... AND p.status=1`
  - **Why**: OR 条件使 SQLite 优化器放弃索引，退化为全表扫描
  - **How to apply**: 如果业务上外键不会为 NULL（如 faults 必须属于 project），直接 INNER JOIN

- **字符串→数字外键迁移**：中文字符串比较（`WHERE company=?`）受编码差异影响导致隔离失效。用 `companies` 表 + `company_id INTEGER FK` 替代，双列保留零停机
  - **Why**: 中文字符串可能有不可见的编码差异、同义异名、全角半角差异
  - **How to apply**: 增量迁移三步走（CREATE TABLE + ALTER ADD COLUMN + UPDATE 回填），INSERT OR IGNORE 幂等

- **前端 AJAX 并行化**：串行嵌套 AJAX 延迟叠加（T1+T2+T3），`$.when()` 并行化后延迟取最大值（max(T1,T2,T3)）
  - **Why**: 两个独立 API 调用不应串行等待
  - **How to apply**: 用 `$.when(a, b).done(...)` 替代嵌套 `$.ajax({ success: function() { $.ajax(...) } })`
