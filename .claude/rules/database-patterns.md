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

## 维护操作记录

| 操作 | 记录内容 |
|------|----------|
| VACUUM | 前后文件大小、清理行数 |
| 物理 DELETE | 删除条件、影响行数、涉及表 |
| PRAGMA integrity_check | 检查结果 |
| ALTER TABLE | 表名、列名、类型 |
| 备份恢复 | 备份文件名、恢复时间 |

## ChangeLogs

- [2026-04-16 11:52:00] 补充：sensor_data 单行约束、三写模型、PARAM_RANGES 默认陷阱（来自 04-07~04-08 日志抽象）
- [2026-04-16 11:15:00] Initial: SQLite 去重模式、VACUUM 流程、级联重复防护、GROUP BY 陷阱
