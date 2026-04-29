# Five-Layer Integrity — 五层体系新增内容强制检查规则

## 核心约束

**每次新增或修改五层体系内容后，必须调用 `/five-layer-add` 完成增量健康检查。**

五层体系内容包括：
- L1 Hooks：`settings.json` 中的 hooks 配置
- L2 Rules：`~/.claude/rules/*.md` 或 `.claude/rules/*.md`
- L3 Skills：`.claude/skills/*/SKILL.md`
- L4 Agents：`.claude/agents/*.md`
- L5 Memory：`memory/*.md`（含 `MEMORY.md` 索引）

## 触发条件

| 操作 | 必须执行 |
|------|---------|
| 新建 rule 文件 | `/five-layer-add 新增 rules/<filename>.md` |
| 删除 rule 文件 | `/five-layer-add 删除 rules/<filename>.md` |
| 新建 skill 目录 | `/five-layer-add 新增 skills/<name>/SKILL.md` |
| 安装第三方 skill | `/five-layer-add 安装 <skill-name>` |
| 新建 memory 文件 | `/five-layer-add 新增 memory/<filename>.md` |
| 新建 agent 文件 | `/five-layer-add 新增 agents/<filename>.md` |
| 修改 hooks 配置 | `/five-layer-add 修改 hooks` |

**豁免条件**（不需要执行）：
- 只修改已有 rule/memory 的内容（不新增/删除文件）
- 只修改项目业务代码（`.py`、`.js`、`.html` 等）
- 只修改 `docs/`、`test_codes/` 等非五层体系目录

## 检查内容速查

`/five-layer-add` 执行以下 5 项检查（详见 `.claude/skills/five-layer-add/SKILL.md`）：

1. **双重加载** — 新文件名是否同时存在于全局和项目目录？
2. **计数同步** — `subagents.md` / `config-review.md` / `MEMORY.md` 是否需要更新？
3. **单文件超限** — 新文件是否超过行数/大小限制？
4. **内容归位** — 内容类型是否放对了层（rule/memory/skill 分工）？
5. **同步标记** — 是否需要 sync 到 `hong-cc-practice`？

## 与其他工具的分工

| 工具 | 触发时机 | 范围 | 耗时 |
|------|---------|------|------|
| `/five-layer-add` | 每次新增内容后（即时） | 增量：只检查刚变更的内容 | <5 分钟 |
| `/context-audit` | 季度第一天 / 发现异常时 | 全量：所有 rules/skills/memory | 15-30 分钟 |
| `/doc-trim` | 每周二/五 | CLAUDE.md + rules 体积 | 5-10 分钟 |

三者互补，不替代：`five-layer-add` 防止问题积累，`doc-trim` 管周级体积，`context-audit` 做季度全面体检。

## 违反后果

跳过 `/five-layer-add` 会导致：
- 双重加载悄悄积累（今日修复的 13 个 skills 就是典型案例）
- 计数不一致（`subagents.md` 声明 14 个 skills，实际 21 个）
- 文件超限无人发现（`subagents.md` 16.8KB 超限 12%）

## ChangeLogs

- [2026-04-29] Initial: 从今日 context-audit 修复流程抽象，强制每次新增五层体系内容后执行增量检查
