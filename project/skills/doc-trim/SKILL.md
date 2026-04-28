---
name: doc-trim
description: 精简 CLAUDE.md 和 rules 文件，防止上下文膨胀浪费 token。检查行数、裁剪 Change Log、清理过时快照、联动更新 README。每周二/五自动提醒，也可手动 /doc-trim 触发。
user-invocable: true
---

# Doc Trim — 文档精简 Skill

## 触发方式

- 手动：`/doc-trim`
- 自动提醒：每周二/五会话开始时（由 `context-hygiene.md` rule 定义）

## 执行清单

### Step 1: CLAUDE.md 行数检查

```bash
wc -l CLAUDE.md
```

- 超过 200 行 → 执行精简
- 200 行以内 → 跳到 Step 3

### Step 2: CLAUDE.md 精简

按优先级裁剪：

1. **Change Log 裁剪**：只保留最近 3 天的条目
   - 计算今天日期，删除 3 天前的所有 `### YYYY-MM-DD` 条目
   - 确认 `Full history: [docs/changelog.md](docs/changelog.md)` 指针存在
   - 被删除的条目如果不在 `docs/changelog.md` 中，先追加过去

2. **重复条目检测**：同一日期不得出现两次
   - `grep -c "### YYYY-MM-DD" CLAUDE.md` 检查每个日期

3. **内容密度检查**：
   - 数据库表逐行列表 → 改为分类表格 + 指针
   - 测试文件完整列表 → 改为指针到 `testing.md`
   - 前端模块逐行描述 → 改为一行概述
   - Weather/LSTM 详细配置 → 改为指针

### Step 3: Rules 卫生检查

扫描 `.claude/rules/*.md`：

1. **过时快照**：检查含 `- [x]` 的表格，如果 >80% 已完成则建议删除
2. **ChangeLogs 过长**：超过 10 条则建议保留最近 5 条
3. **单文件行数**：超过 400 行则建议拆分

```bash
# 检查所有 rules 文件行数
wc -l .claude/rules/*.md | sort -rn | head -10
```

### Step 4: README 联动检查

检查三个 README 中的关键数据一致性：

| 检查项 | 文件 | 方法 |
|--------|------|------|
| 项目名称 | README.md, user_manual/README.md | grep "中暖\|智慧双碳\|Smart Agriculture" |
| 数据库表数量 | README.md | 与 CLAUDE.md 中的数字对比 |
| 模块数量 | README.md | 与 CLAUDE.md 中的数字对比 |

hong-cc-practice/README.md 不检查项目名（它是通用配置仓库），但检查：
- L2 Rules 计数与 `subagents.md` 一致
- L3 Skills 计数与 `subagents.md` 一致

### Step 5: 输出报告

```markdown
## Doc Trim Report YYYY-MM-DD

### CLAUDE.md
- 行数: N (before) → M (after) | STATUS: OK/TRIMMED
- Change Log: 保留 X 天，移除 Y 条旧条目
- 重复条目: 无/已修复

### Rules 卫生
| 文件 | 行数 | 问题 | 处理 |
|------|------|------|------|
| xxx.md | N | 过时快照 | 已清理/建议清理 |

### README 一致性
| 检查项 | 状态 |
|--------|------|
| 项目名称 | OK/MISMATCH |
| 表数量 | OK/MISMATCH |
```
