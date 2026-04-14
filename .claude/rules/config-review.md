# Config Review Rules — Rules/Skills/Memory 修改后检查

## Trigger

当修改或新建以下目录中的文件时，**必须**执行本规则中的检查清单：

- `.claude/rules/*.md`
- `.claude/skills/*/SKILL.md`
- `memory/*.md`（即 `~/.claude/projects/d--Proj-energy/memory/`）

## Checklist

### 1. 去重检查（mandatory）

修改完成后，检查新内容是否与已有 rules/memory/skills 重复：

| 检查项 | 方法 |
|--------|------|
| Rules vs Memory 重复 | grep 新内容关键词到 `memory/` 目录 |
| Rules vs Rules 重复 | grep 新内容关键词到 `.claude/rules/` 目录 |
| Skills vs Rules 重复 | 检查 SKILL.md 中的规则是否已在 rules 文件中 |

**去重原则**：
- Rules = 权威规范（single source of truth）
- Memory = 只存"为什么"（Why）和个性化偏好，不重复规则内容
- Skills = 只存操作步骤（How/Checklist），不重复规则约束

### 2. 精简检查（recommended）

- 新增内容是否可以用一行指针替代？（如 `→ 见 rules/xxx.md`）
- 是否有已存在的 memory 可以更新而非新建？
- MEMORY.md 索引是否需要更新？

### 3. 位置检查（recommended）

| 内容类型 | 应该放在 |
|----------|----------|
| 项目规范/约束/格式 | `.claude/rules/` |
| 用户偏好/教训/Why | `memory/` |
| 操作步骤/Checklist | `.claude/skills/` |
| 外部资源指针 | `memory/` (reference type) |

### 4. Hooks 配置检查（mandatory）

修改 rules/skills/agents 后，检查 `.claude/settings.json` 和 `~/.claude/settings.json` 中的 hooks 是否需要更新：
- 新增 rules 文件 -> SubagentStart hook 的上下文注入是否需要引用？
- 新增 skill -> PostToolUse hook 是否需要新增 matcher？
- 删除 rules 文件 -> 其他文件中是否有引用？grep 验证

### 5. 交叉引用检查（recommended）

- 新文件是否在相关文件中被引用？
- 如果删除了 memory 文件，MEMORY.md 索引是否已更新？
- 如果修改了 rules，是否需要更新 CLAUDE.md？

### 6. 删除内容检查（mandatory）

**删除或大幅删减文档内容时，必须记录**：

| 必须说明 | 示例 |
|----------|------|
| 理由 | 去重/过时/错误/迁移到其他文件 |
| 原因 | 导致删除的根本原因（如合并了两个文件） |
| 影响 | 哪些功能/引用会受影响，已如何处理 |
| 交叉更新 | grep 检查所有引用该内容的 agents/skills/CLAUDE.md 是否已同步 |

**原则**：不允许静默删除内容。每次删除必须在 ChangeLogs 中记录理由。

## GitHub 同步（mandatory）

`.claude/` 目录配置在两个 repo 中维护：

| Repo | URL | 内容 | 自动/手动 |
|------|-----|------|----------|
| Gitee（主项目） | `gitee.com/chg0901/energy` | 部分 `.claude/` 文件（`git add -f` 强制添加） | 随项目提交 |
| GitHub（配置专用） | `github.com/chg0901/hong-cc-practice` | 完整 `.claude/` 配置（排除敏感文件） | 手动同步 |

### 同步时机

修改 `.claude/` 下的文件后，**必须**执行同步：

```bash
bash scripts/sync_claude_config.sh --push
```

### 同步范围

| 包含 | 排除（原因） |
|------|-------------|
| `agents/*.md`（4 个项目 agents） | `settings.local.json`（含 API tokens） |
| `rules/*.md`（18 个 rules） | `book2skills/`（第三方安装的 skills） |
| `skills/*/SKILL.md`（4 个项目 skills） | `create-colleague/`（第三方安装的 skills） |
| `settings.json`（hooks 配置，无敏感信息） | — |

同步脚本：[scripts/sync_claude_config.sh](../../scripts/sync_claude_config.sh)
本地 clone：`$HOME/.claude-github/hong-cc-practice/`

## Execution

本规则在每次修改 rules/skills/memory 文件后**手动执行**（不是自动化 hook）。在提交前快速过一遍 checklist 即可。

## ChangeLogs

- [2026-04-14 10:30:00] 新增 GitHub 同步规则（mandatory）：双 repo 维护、同步时机、同步范围、排除清单
- [2026-04-14 10:00:00] 新增删除内容检查（mandatory）：理由/原因/影响/交叉更新
- [2026-04-13 21:00:00] 新增 Hooks 配置检查项（mandatory）
- [2026-04-09 — Initial: 修改 rules/skills/memory 后的去重+精简+位置检查规则](changes/2026-04-09)