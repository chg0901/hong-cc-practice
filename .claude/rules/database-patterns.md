# Database Patterns — SQLite 常用操作模式

## 去重模式

### 安全去重（保留最新行）

```sql
DELETE FROM table_name
WHERE id NOT IN (
    SELECT MAX(id) FROM table_name
    GROUP BY col1, col2, col3
);
```

**要点**：
- `MAX(id)` 保留最新插入的行（自增 ID 场景）
- 幂等安全：重复执行不会多删
- 比 `ROW_NUMBER()` 窗口函数在 SQLite 中更高效
- **去重前必须备份**：`shutil.copy2('smart_energy.db', 'smart_energy_backup_before_dedup_YYYYMMDD.db')`

### 去重前检查

```sql
-- 查看重复组数和冗余行数
SELECT COUNT(*) as groups,
       COUNT(*) - COUNT(DISTINCT (col1||col2||col3)) as duplicates
FROM table_name;
```

## VACUUM

**时机**：大量 DELETE 后（>10% 数据量）

**条件**：必须无其他连接持有数据库锁。Flask 服务器运行时 VACUUM 会失败（`database is locked`）。

```python
# 需要停服后执行
conn = sqlite3.connect(db, timeout=60)
conn.execute('PRAGMA journal_mode=DELETE')  # VACUUM 要求非 WAL
conn.execute('VACUUM')
conn.execute('PRAGMA journal_mode=WAL')    # 恢复 WAL
conn.close()
```

**检查是否需要 VACUUM**：

```python
c.execute('PRAGMA freelist_count')
c.execute('PRAGMA page_count')
c.execute('PRAGMA page_size')
free_ratio = freelist * page_size / file_size
# >30% 空间可回收时建议 VACUUM
```

## WAL Checkpoint 假阳性

**问题**：WAL 文件增长到数百 MB 且 checkpoint 被活跃读者阻塞时，SQLite 报 `database or disk is full`，即使磁盘空间充足。

**防护**：长时间运行的 WAL 模式服务必须添加定期 checkpoint 后台线程：

```python
import threading, time

def start_wal_checkpoint_scheduler(db_path, interval=1800):
    def checkpoint_loop():
        while True:
            time.sleep(interval)
            try:
                conn = sqlite3.connect(db_path, timeout=10)
                conn.execute('PRAGMA wal_checkpoint(PASSIVE)')
                conn.close()
            except Exception:
                pass
    t = threading.Thread(target=checkpoint_loop, daemon=True)
    t.start()
```

**PASSIVE vs TRUNCATE**：`PASSIVE` 不阻塞读者，适合后台定期执行。`TRUNCATE` 更彻底但需要无活跃读者。

## 级联重复防护

**问题**：项目复制功能不检查同名设备 → 每次复制新增一套设备 → 级联产生 `device_params` + `sensor_data` 重复。

**防护模式**（Python set 去重键）：

```python
existing = {(r['name'], r['type']) for r in c.fetchall()}
for row in source_rows:
    key = (row['name'], row['type'])
    if key in existing:
        skipped += 1
        continue
    existing.add(key)
    # INSERT...
```

## SQLite GROUP BY 陷阱

SQLite 允许 SELECT 非聚合列而不放在 GROUP BY 中（不像 MySQL 的 `ONLY_FULL_GROUP_BY`）。取的是组内**第一行**的值，如果字段可能不一致需显式聚合函数。

## sensor_data 单行约束

**规则**：`sensor_data` 表每设备每 tick **只写 1 行**。

API 用 `ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY collect_time DESC)` 取最新值。多行同时间戳会导致非确定性行为（随机取值）。

**RealtimeSimulator 三写模型**：

```
_generate_tick() / _replay_tick()
  ├── InfluxDB           (SYNCHRONOUS 写入, 秒级时序, 全量参数)
  ├── device_runtime_data (JSON 格式: 所有参数合为一行)
  └── sensor_data         (每设备 1 行: 所有参数的平均值)
```

- API 端点（`/api/projects/<pid>/latest-data`、`/api/cooling/overview/<pid>`）从 `sensor_data` 读取
- 如果 `sensor_data` 写入路径缺失 → 前端数据显示空白

## RealtimeSimulator PARAM_RANGES 默认陷阱

**问题**：未定义的 `param_code` 落到默认范围 `(0, 100)`，产生无意义数据。

**案例**：COP 正常范围 `2.5-5.5`，默认 `(0, 100)` 产生 `29-96` 的无意义值。

**规则**：新增设备类型时，必须同时在 `PARAM_RANGES` 字典中添加显式范围定义：

```python
PARAM_RANGES = {
    'cop': (2.5, 5.5),     # 明确范围
    'power_meter': (0, 100),
    # 新设备类型必须在此定义
}
```

- `device_params` 表是模拟器加载的前提：设备缺少条目会被跳过
- 新增设备类型 → 先插入 `device_params` → 再加 `PARAM_RANGES`

## 增量迁移模式（安全三步走）

SQLite 不支持 `ALTER TABLE DROP/ALTER COLUMN`，结构变更必须用增量模式：

```python
# Step 1: 检查列是否存在
cols = [row[1] for row in c.execute(f"PRAGMA table_info({table})").fetchall()]
if 'new_column' not in cols:
    c.execute(f"ALTER TABLE {table} ADD COLUMN new_column TYPE")

# Step 2: 回填仅未迁移行（幂等）
c.execute(f"UPDATE {table} SET new_column = <expr> WHERE new_column IS NULL")

# Step 3: INSERT OR IGNORE 防止重复
c.execute("INSERT OR IGNORE INTO new_ref_table (name) VALUES (?)", (value,))
```

**要点**：
- `PRAGMA table_info` 检查列存在性 → 重启不报错
- `WHERE new_column IS NULL` → 仅回填未迁移行，幂等安全
- `INSERT OR IGNORE` → 参照表不重复
- **保留旧列**：双列策略（旧列显示 + 新列过滤），零停机
- 测试残留数据（如 `_nonexistent_`）自然不会被回填，company_id 保持 NULL → 自动标识

## 查询优化反模式

### 窗口函数陷阱（ROW_NUMBER() OVER）

**问题**：`ROW_NUMBER() OVER (PARTITION BY col ORDER BY other_col DESC)` 看起来优雅（每组取一条），但在大表上 SQLite 必须先物化全量结果再排序编号，即使只需要 `rn=1`。

**案例**：14M 行 `sensor_data` 表的窗口函数查询耗时 **204.8 秒**。

**替代方案**：N 次索引查找

```python
# 旧：全表窗口函数 — 204.8s
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY collect_time DESC) as rn
    FROM sensor_data
) WHERE rn = 1

# 新：逐设备取最新 — 0.011s（18600x 提速）
for device_id in device_ids:
    c.execute('SELECT value, collect_time FROM sensor_data '
              'WHERE device_id=? ORDER BY collect_time DESC LIMIT 1', (device_id,))
```

**适用条件**：
- **N < 100**：设备/分组数量少于 100 时，N 次索引查找远快于一次全表窗口函数
- **必须有复合索引**：`CREATE INDEX idx ON table(partition_col, order_col DESC)`，否则每次查找仍是全表扫描
- **原理**：`ORDER BY col DESC LIMIT 1` + 复合索引 = B-tree 最右叶子节点 O(log N) 定位

### 其他反模式速查

| 反模式 | 症状 | 修复方案 |
|--------|------|----------|
| `OR col IS NULL` | 全表扫描、锁等待 | `INNER JOIN` 消除 NULL |
| `WHERE company=?` 中文比较 | 编码差异导致隔离失效 | 数字外键 `WHERE company_id=?` |
| `SELECT *` + `row[N]` | Schema 变更后取错列 | 显式列名 |
| 无索引的 ORDER BY | 全表排序 | 覆盖索引 `(where_col, order_col DESC)` |

### 前端 AJAX 并行化（配合后端优化）

当后端查询已优化但仍需调用多个独立 API 时，前端串行嵌套 AJAX 会叠加延迟。

**反模式**：串行嵌套（总延迟 = T1 + T2 + ...）

```javascript
// 旧：串行嵌套，延迟叠加
$.ajax('/api/weather/...', success: function() {
    $.ajax('/api/projects/.../latest-data', success: function() {
        updateUI();
    });
});
```

**正确模式**：`$.when()` 并行（总延迟 = max(T1, T2)）

```javascript
// 新：并行，延迟取最大值
$.when(
    $.ajax({ url: '/api/weather/data/latest', data: { project_id } }),
    $.ajax({ url: `/api/projects/${id}/latest-data` })
).done(function(weatherRes, sensorRes) {
    updateEnvironmentStatsFromWeather(project, weatherRes[0].data, sensorRes[0].data);
}).fail(function() {
    updateEnvironmentStatsFallback(project);
});
```

**要点**：
- 两个 API 之间无依赖时才可并行，否则仍需串行
- `.done()` 回调参数是 `[data, status, jqXHR]` 的数组套数组，用 `res[0].data` 取数据
- 添加 `timeout: 10000` 防止单个慢请求拖垮整个页面

## 维护操作记录

| 操作 | 记录内容 |
|------|----------|
| VACUUM | 前后文件大小、清理行数 |
| 物理 DELETE | 删除条件、影响行数、涉及表 |
| PRAGMA integrity_check | 检查结果 |
| ALTER TABLE | 表名、列名、类型 |
| 备份恢复 | 备份文件名、恢复时间 |

## ChangeLogs

- [2026-04-25 14:10:00] 新增增量迁移模式 + 查询优化反模式（来自 company_id 迁移 + latest-data 查询优化经验）
- [2026-04-22 14:49:00] 新增 WAL Checkpoint 假阳性防护模式（来自 04-21 日志 Insight）
- [2026-04-16 11:52:00] 补充：sensor_data 单行约束、三写模型、PARAM_RANGES 默认陷阱（来自 04-07~04-08 日志抽象）
- [2026-04-16 11:15:00] Initial: SQLite 去重模式、VACUUM 流程、级联重复防护、GROUP BY 陷阱
