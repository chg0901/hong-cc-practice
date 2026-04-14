# 数据库操作规则

## 三库架构

本项目使用三个数据库，各有分工：

| 数据库 | 角色 | 端口 | 何时使用 |
|--------|------|------|----------|
| **SQLite** | 主业务库 | 文件 | 所有 CRUD、权限、项目、设备管理 |
| **InfluxDB** | 原始时序库 | 8086 | 传感器数据写入/查询（秒级精度） |
| **MariaDB** | 聚合分析库 | 3307 | 聚合指标查询（分钟/小时/天级） |

详细文档：
- [docs/database_sqlite.md](../../docs/database_sqlite.md)
- [docs/database_influxdb.md](../../docs/database_influxdb.md)
- [docs/database_mariadb.md](../../docs/database_mariadb.md)

## SQLite 操作规则

### 连接

```python
# 必须包含 timeout=30 和 check_same_thread=False
conn = sqlite3.connect('smart_energy.db', check_same_thread=False, timeout=30)
```

- **永远不要** 用默认 timeout（5s），reloader 重启时会锁超时
- **永远不要** 省略 `check_same_thread=False`，Flask 多线程环境会报错
- 每次请求新建连接，操作完 `conn.commit()` + `conn.close()`

### PRAGMA

```python
# 仅在 init_database() 中设置一次
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA synchronous=NORMAL")
```

- 不要在 API 路由中重复设置 PRAGMA
- `foreign_keys` 未启用（默认 OFF），外键仅作文档约束

### 软删除

所有表使用 `status` 列做软删除：
- `1` = 正常
- `0` = 已删除

查询时**必须**过滤 `WHERE status = 1`。

例外：`production_plans` 和 `production_activities` 使用 `is_deleted` 列。

### 迁移

使用内联 ALTER TABLE 模式，先检查列是否存在：

```python
existing = [col[1] for col in c.execute("PRAGMA table_info(table_name)").fetchall()]
if 'new_column' not in existing:
    c.execute("ALTER TABLE table_name ADD COLUMN new_column TYPE")
```

### 维护操作记录

执行以下操作时，**必须在 work_summary 中记录**：

| 操作 | 何时执行 | 记录内容 |
|------|----------|----------|
| `VACUUM` | 大量 DELETE 后 | 清理行数、前后文件大小 |
| 物理 DELETE | 清理测试数据 | 删除条件、影响行数、涉及表 |
| `PRAGMA integrity_check` | 数据疑似损坏时 | 检查结果 |
| `ALTER TABLE` | 新增/修改列 | 表名、列名、类型 |
| 备份恢复 | 数据恢复时 | 备份文件名、恢复时间 |

### 备份

自动备份每 6 小时执行一次，最多保留 5 个。手动备份：

```python
import sqlite3, shutil
shutil.copy2('smart_energy.db', 'smart_energy_backup_manual.db')
```

## InfluxDB 操作规则

### 写入

- 使用 `SYNCHRONOUS` 写入模式确保可靠性
- 批量写入大小：实时 5000 点，批量生成 50000 点
- Tag 用于过滤（device_id, param_code），Field 用于数值（value）

### 查询

- 所有 Flux 查询必须包含 `range()`（InfluxDB 要求时间范围）
- 使用 `filter()` 缩小范围，避免全表扫描
- 大结果集加 `limit(n: N)`

### 维护

- 当前未配置保留策略（默认无限保留）
- 数据目录迁移使用 `scripts/migrate_influxdb_data.ps1`

## MariaDB 操作规则

### 连接

```python
import mysql.connector
conn = mysql.connector.connect(
    host="localhost", port=3307,
    user="root", password="...",
    database="smart_energy"
)
```

- 使用 `is_connected()` + `reconnect()` 处理连接断开
- 批量写入每 1000 条 commit 一次

### UPSERT

聚合数据使用 `ON DUPLICATE KEY UPDATE` 保证幂等：

```sql
INSERT INTO device_metrics_aggregated (...) VALUES (...)
ON DUPLICATE KEY UPDATE value=VALUES(value), data_points=VALUES(data_points)
```

### 启动

MariaDB 启动需要 `D:\MariaDB_Data\my.ini`，可能需要管理员权限。

## RealtimeSimulator 数据管道

`RealtimeSimulator` 每秒处理 20 台设备，写入三条路径：

```
_generate_tick() / _replay_tick()
  ├── InfluxDB           (SYNCHRONOUS 写入, 秒级时序)
  ├── device_runtime_data (JSON 格式: 所有参数)
  └── sensor_data         (每设备1行: 参数平均值)
```

关键约束：
- **sensor_data 只写 1 行/设备/tick**: API 用 `ROW_NUMBER() PARTITION BY device_id` 取最新值，多行同时间戳会导致随机取值
- **device_params 是前提**: 设备必须在 `device_params` 表有定义才被仿真器加载，缺少条目的设备（如 COP）会被跳过
- **PARAM_RANGES 控制值域**: 缺少定义的 param_code 落到 `default (0,100)`，COP 必须定义 `(2.5, 5.5)`
- **数据断层恢复**: `scripts/fill_sensor_gap.py` 检测并填补历史断层（对标 `generate_test_data()` 相同数值模式）

## 测试数据清理规则

### 清理原则

1. 测试数据使用 **软删除 API**（`DELETE /api/xxx/{id}`）清理
2. 大量残留数据需要**物理 DELETE + VACUUM**
3. 物理清理后**必须在 work_summary 中记录**（表、条件、行数）

### 测试隔离

- 测试使用当天日期（不用 2099 远期日期）
- 每个测试组在开始前 pre-cleanup（清理同 project+type 已有记录）
- reporter 字段标记为 `test_xxx_{timestamp}`，便于识别
