# Friday Review — 每周五工具/方法自进化规则

## 触发条件

**每周五**（用户说"周五总结"、"weekly review"、"自进化"时触发）。也可在任意一天执行，当积累了足够多的新踩坑/模式时。

## 执行流程

```
Step 1: 读取本周所有 docs/work_summary_YYYYMMDD.md
Step 2: 提取 ★ Insight、踩坑、解决方案、可复用模式
Step 3: 对比现有 .claude/rules/ 和 memory/ 的覆盖度
Step 4: 将未覆盖的模式抽象为 Rules 或 Memory
Step 5: 去重检查 — 新内容不与已有 rules/memory 重复
Step 6: 更新 subagents.md（L2 Rules 计数）
Step 7: 更新 MEMORY.md 索引
Step 8: Commit + Push + Sync
```

## 抽象优先级

| 优先级 | 抽象目标 | 条件 |
|--------|---------|------|
| P1 | **Rule** | 可复用的操作规范、检查清单、格式约束 |
| P2 | **Memory (feedback)** | 踩坑教训、用户偏好、Why 理由 |
| P3 | **Memory (reference)** | 外部资源指针、速查表 |
| P4 | **Skill** | 多步骤操作流程（需要 checklist 引导时） |

## 去重原则

- **Rules** 是单一事实来源（single source of truth），Memory 只存"Why"
- 新 Rule 的内容如果与已有 Rule 超过 30% 重叠 → 合并到已有 Rule
- 新 Memory 如果已有同名 Memory → 更新而非新建
- **谨慎删除**：删减前必须说明理由，检查所有引用该内容的 agents/skills/CLAUDE.md

## 输出清单

每周五执行后，输出以下报告：

```markdown
## 周五自进化报告 YYYY-MM-DD

### 本周新增
| 类型 | 文件 | 内容摘要 |
|------|------|----------|
| Rule | xxx.md | ... |
| Memory | xxx.md | ... |

### 更新（非新建）
| 文件 | 更新内容 |
|------|----------|
| xxx.md | 新增 XX 章节 |

### 去重检查
- 无重复 / 发现重复及处理方式

### 覆盖度
- 本周 Insights 总数: N
- 已覆盖: M (P%)
- 未覆盖: K（说明原因：临时性问题/已过时/不属于本项目）
```

## 与其他规则的关系

| 规则 | 管辖范围 | 互补关系 |
|------|----------|----------|
| `daily-review.md` | 每日待办审查 | friday-review 管知识抽象，daily-review 管任务执行 |
| `config-review.md` | 修改后检查 | friday-review 可能触发 config-review 的去重检查 |
| `context-hygiene.md` | 上下文卫生 | friday-review 管知识抽象（insight→rule），context-hygiene 管体积控制（裁剪膨胀） |
| `todo-tracking.md` | 待办格式 | friday-review 的产出项（如"下次补 rule"）可进入待办 |
| `work-summary-rules.md` | 文档格式 | friday-review 的报告写入 work_summary |

## 首次执行记录（2026-04-16）

### 第一轮：04-09~04-16 日志（8 个文件，45+ insight）

- Rule: `database-patterns.md`（SQLite 去重/VACUUM/级联防护）
- Memory: `feedback_playwright_patterns.md`（5 个 Playwright 踩坑）
- Memory: `feedback_settings_hierarchy.md`（全局 vs 项目级配置）
- Memory 更新: `feedback_windows_env.md`（追加 .env + load_dotenv 陷阱）

### 第二轮：04-03~04-08 日志（5 个文件，30+ insight）

补充到已有 Rule/Memory：
- Rule 更新: `database-patterns.md`（+sensor_data 单行约束、三写模型、PARAM_RANGES 默认陷阱）
- Rule 更新: `testing.md`（+Pre-cleanup 测试隔离原则）
- Rule 更新: `svg-design.md`（+SVG Rendering Pitfalls：pointer-events、CJK 截断、Z-order）
- Memory 新建: `feedback_flask_gotchas.md`（debug reloader 双进程、AJAX 竞态、dict.get(None) 陷阱）

### 覆盖度

- 两轮合计提取 75+ insight
- 已覆盖：12 项（5 新建 + 7 更新）
- 未覆盖原因：多数为一次性 debug、临时性配置、已被代码修复的问题

## ChangeLogs

- [2026-04-16 12:02:00] 更新首次执行记录：第二轮 04-03~04-08 日志补充，覆盖度统计
- [2026-04-16 11:15:00] Initial: 每周五自进化规则 + 首次执行记录
