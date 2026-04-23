# Context Hygiene — CLAUDE.md 及 Rules 上下文卫生规则

## 目的

防止 CLAUDE.md 和 rules 文件无限膨胀导致 token 浪费。CLAUDE.md 每次会话全量加载，每多一行都增加基础设施成本。

## 硬性约束

| 指标 | 上限 | 检查方式 |
|------|------|----------|
| CLAUDE.md 总行数 | 200 行 | `wc -l CLAUDE.md` |
| Change Log 条目 | 最近 3 天 | 检查日期，旧条目移到 `docs/changelog.md` |
| 单个 rules 文件 | 400 行 | 超过则拆分或精简 |
| 重复条目 | 0 | 同一日期不得出现两次 |

## 精简原则

### CLAUDE.md 内容密度

| 内容类型 | 应保留 | 应移除（用指针替代） |
|----------|--------|---------------------|
| 项目名称和一句话描述 | 保留 | — |
| 架构图引用 | 保留 | — |
| 开发命令（常用 5-6 条） | 保留 | — |
| 数据库表分类概览 | 保留（表格形式） | 逐表详细描述 |
| API 端点 | 指针 | 完整列表 |
| 测试文件列表 | 指针 | 完整列表 |
| 前端模块列表 | 一行概述 | 逐模块描述 |
| Weather/LSTM 详细配置 | 指针 | 详细参数 |
| Change Log | 最近 3 天 | 旧条目 |
| 重要注意事项 | 保留（5-6 条） | — |

### Rules 文件卫生

| 问题 | 检测方式 | 处理 |
|------|----------|------|
| 过时快照（已完成待办表格） | 检查 `- [x]` 比例 > 80% | 删除已完成项，仅保留未完成 |
| 历史执行记录 | 检查"首次执行记录"等章节 | 保留最近一次，旧记录移到 changelog |
| ChangeLogs 过长 | 超过 10 条 | 保留最近 5 条 |

## 触发频率

**每周二和周五**，在会话开始时检查。

| 日期 | 检查内容 |
|------|----------|
| 周二 | CLAUDE.md 行数 + Change Log 裁剪 + 重复检测 |
| 周五 | 全量检查（含 rules 卫生 + friday-review 知识抽象） |

也可手动触发：`/doc-trim`

## README 联动更新规则

当 CLAUDE.md 中的以下内容变更时，需联动检查 README 一致性：

| 变更内容 | 联动文件 | 检查项 |
|----------|----------|--------|
| 项目名称/描述 | README.md, user_manual/README.md | 名称一致 |
| 数据库表数量 | README.md | 表数量一致 |
| 模块数量 | README.md, user_manual/README.md | 模块列表一致 |
| 设备/公司数量 | README.md | 业务规模数据一致 |

### 三个 README 的侧重点

| 文件 | 受众 | 侧重 | 不应包含 |
|------|------|------|----------|
| `README.md` | 开发者 | 技术架构、API、数据库、部署 | 操作截图、用户指南 |
| `docs/user_manual/README.md` | 运维/操作人员 | 功能模块、操作步骤、截图索引 | 代码结构、API 细节 |
| `hong-cc-practice/README.md` | Claude Code 用户 | 五层体系、配置经验、Skill 目录 | 项目业务逻辑 |

## 与其他规则的关系

| 规则 | 互补关系 |
|------|----------|
| `friday-review.md` | friday-review 管知识抽象（insight→rule），context-hygiene 管体积控制（裁剪膨胀） |
| `config-review.md` | config-review 管修改后检查，context-hygiene 管定期主动检查 |
| `work-summary-rules.md` | work-summary 定义 Change Log 格式，context-hygiene 定义 Change Log 保留策略 |
| `context-management.md` | context-management 管会话级 token 策略，context-hygiene 管文件级体积控制 |

## ChangeLogs

- [2026-04-22] Initial: CLAUDE.md 行数上限、Change Log 裁剪策略、Rules 卫生检查、README 联动规则
