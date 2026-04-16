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

## 维护操作记录

| 操作 | 记录内容 |
|------|----------|
| VACUUM | 前后文件大小、清理行数 |
| 物理 DELETE | 删除条件、影响行数、涉及表 |
| PRAGMA integrity_check | 检查结果 |
| ALTER TABLE | 表名、列名、类型 |
| 备份恢复 | 备份文件名、恢复时间 |

## ChangeLogs

- [2026-04-16 11:15:00] Initial: SQLite 去重模式、VACUUM 流程、级联重复防护、GROUP BY 陷阱
