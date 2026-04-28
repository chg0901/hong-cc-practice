---
name: db-optimization
description: SQLite 数据库优化工作流。触发：用户提到"数据库优化"、"查询慢"、"性能优化"、"索引"、"迁移"、"ALTER TABLE"、"全表扫描"时自动加载。也适用于发现慢查询、排查锁等待、设计增量迁移方案时。
user-invocable: true
---

# SQLite 数据库优化 Skill

中暖智慧双碳管理平台的数据库优化操作流程，覆盖：查询优化、索引设计、结构优化（增量迁移）、性能验证。

## 检查清单

### Phase 1: 问题定位

- [ ] **复现问题**：确认慢查询或卡死的触发路径（哪个 API、哪个操作）
- [ ] **测量基准**：记录优化前的查询耗时（用 `time.time()` 或浏览器 Network 面板）
- [ ] **分析 EXPLAIN QUERY PLAN**：对慢查询执行 `EXPLAIN QUERY PLAN <sql>` 确认是否全表扫描
- [ ] **检查表行数**：`SELECT COUNT(*) FROM table_name` 确认数据量级

### Phase 2: 查询优化

- [ ] **识别全表扫描模式**：
  - `ROW_NUMBER() OVER (PARTITION BY ...)` 在大表 → 改用逐条 `ORDER BY DESC LIMIT 1`
  - `OR column IS NULL` → 改用 `INNER JOIN` 消除 NULL 条件
  - `SELECT *` → 改用显式列名
  - `LEFT JOIN` 无 NULL 行 → 改用 `INNER JOIN`
- [ ] **设计复合索引**：
  - 覆盖 WHERE + ORDER BY 列：`CREATE INDEX idx ON table(col_where, col_order DESC)`
  - 用 `IF NOT EXISTS` 防止重复创建
  - 预估索引创建耗时（1000 万行约 40-60 秒）
- [ ] **前端并行化**：
  - 串行嵌套 AJAX → `$.when()` 并行（延迟从相加变取最大值）
  - 添加 `timeout: 10000` 防止无限等待

### Phase 3: 结构优化（增量迁移）

当需要添加列或修改表结构时：

- [ ] **安全三步走**：
  1. `CREATE TABLE IF NOT EXISTS` 创建新表
  2. `ALTER TABLE ADD COLUMN IF NOT EXISTS` 添加列（先 `PRAGMA table_info` 检查）
  3. `UPDATE SET ... WHERE ... IS NULL` 回填数据（仅未迁移行）
- [ ] **幂等保障**：
  - `INSERT OR IGNORE` 防止重复插入
  - `WHERE new_col IS NULL` 仅回填未迁移行
  - 重启不产生副作用
- [ ] **双列策略**：保留旧列（显示用）+ 新增优化列（过滤用）
- [ ] **Session 双存储**：旧字段用于 UI 显示，新字段用于数据库查询

### Phase 4: 验证

- [ ] **测量优化后耗时**：同一查询/操作对比
- [ ] **自动化测试**：新建或更新 `test_codes/test_xxx.py`
- [ ] **回归测试**：运行相关现有测试确认无破坏
- [ ] **记录优化报告**：在 `docs/` 下创建优化报告文档

## 常见反模式速查

| 反模式 | 症状 | 修复方案 |
|--------|------|----------|
| `ROW_NUMBER() OVER` 大表 | 查询 100s+ 级 | 逐条 `ORDER BY DESC LIMIT 1` + 索引 |
| `OR col IS NULL` | 全表扫描 | `INNER JOIN` 消除 NULL |
| `WHERE company=?` 中文比较 | 编码差异导致隔离失效 | 数字外键 `WHERE company_id=?` |
| `SELECT *` + `row[N]` | Schema 变更后取错列 | 显式列名 |
| 串行嵌套 AJAX | 延迟叠加 | `$.when()` 并行 |
| 无 timeout 的 AJAX | 页面卡死 | `timeout: 10000` |
| 硬编码字符串过滤 | 数据不一致 | 提取到独立表 + FK |

## 索引设计决策树

```
查询模式？ → WHERE a=? ORDER BY b DESC  →  CREATE INDEX idx ON t(a, b DESC)
         → WHERE a=? AND b=?            →  CREATE INDEX idx ON t(a, b)
         → WHERE a IN (...)             →  CREATE INDEX idx ON t(a)
         → 仅 ORDER BY b DESC           →  CREATE INDEX idx ON t(b DESC)
         → WHERE a=? + JOIN t2          →  确保两边都有索引
```

## 优化报告模板

```markdown
# 数据库优化报告 — [主题]
**日期**: YYYY-MM-DD | **类型**: 查询优化/结构优化/索引优化 | **影响范围**: [模块]

## 问题背景
[症状描述、用户影响]

## 根因分析
[EXPLAIN QUERY PLAN 结果、慢查询 SQL、为什么慢]

## 优化方案
[索引设计、查询重写、结构变更、前端并行化等]

## 性能对比
| 查询 | 优化前 | 优化后 | 提升 |
|------|--------|--------|------|

## 涉及文件
| 文件 | 变更类型 | 说明 |
|------|----------|------|

## 经验总结
[可复用的模式、踩坑、注意事项]
```

## 实际案例

### 案例 1: latest-data 查询（204.8s → 0.011s）

- **反模式**: `ROW_NUMBER() OVER (PARTITION BY device_id)` 在 14M 行 sensor_data
- **修复**: 47 次逐设备 `ORDER BY collect_time DESC LIMIT 1` + 复合索引 `(device_id, collect_time DESC)`
- **原理**: B-tree 最右叶子节点 O(log N) 定位，替代全表物化

### 案例 2: faults 查询（锁等待 → 0.001s）

- **反模式**: `LEFT JOIN ... OR f.project_id IS NULL` 触发全表扫描
- **修复**: `INNER JOIN ... AND p.status = 1` + 显式列名

### 案例 3: company_id 迁移（字符串比较 → 数字外键）

- **反模式**: `WHERE company=?` 中文编码差异导致隔离失效
- **修复**: `companies` 表 + `company_id INTEGER FK` + 6 表增量迁移 + 回填
- **要点**: 双列保留、Session 双存储、INSERT OR IGNORE 幂等
