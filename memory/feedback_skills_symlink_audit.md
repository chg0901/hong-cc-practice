---
name: Skills symlink audit trap on Windows
description: Windows Git Bash symlinks look like real dirs in plain `ls`. Deleting ~/.claude/skills/ dirs breaks project symlinks. Always use `ls -la` when auditing skills.
type: feedback
originSessionId: 16243a13-138c-42c6-abb6-b9d2c6f71e3c
---
# Skills 双重加载审计：Windows Symlink 陷阱

**规则**：审计 skills 双重加载时，必须用 `ls -la ~/.claude/skills/` 检查，不能用 `ls`。

**Why**: 2026-04-29 context-audit 事故：
- `ls ~/.claude/skills/` 显示 13 个目录，看起来像真实目录副本
- 实际上项目 `.claude/skills/` 中的 skills 是 symlinks，指向 `~/.claude/skills/`
- 删除 `~/.claude/skills/` 中的目录后，项目目录中的 symlinks 变成悬空链接（dangling symlinks）
- 需要从 `hong-cc-practice` 备份恢复

**How to apply**:
1. 审计前：`ls -la ~/.claude/skills/` — 看 `->` 箭头判断是否 symlink
2. 审计前：`ls -la .claude/skills/` — 项目目录中的 skills 通常是 symlinks
3. 删除策略：
   - 项目专用 skills（visual-check, interaction-check 等）→ 真实目录在 `~/.claude/skills/`，项目目录是 symlink，**不要删 `~/.claude/skills/` 中的**
   - global skills（context-research, doublecheck 等）→ 真实目录在 `~/.claude/skills/`，项目目录**不应有副本**
4. 双重加载的真正来源：`~/.claude/skills/` 中有真实目录，同时 `.claude/skills/` 中有同名 symlink 指向它 → 两边都被加载

**正确的双重加载修复方案（2026-04-29 实际执行）**：
- 11 个项目专用 skills：从项目 `.claude/skills/` 删除（symlinks 指向全局，全局保留）
- 2 个 global skills（context-research, create-colleague）：从项目 `.claude/skills/` 删除（它们属于全局，不应在项目目录）
- 结果：项目专用 12 个 skills 保留在 `~/.claude/skills/`（symlink 目标），项目目录中的 symlinks 正常工作

**当前正确状态（2026-04-29 修复后）**：
- `~/.claude/skills/` 中有 12 个项目专用 skills（真实目录）
- `.claude/skills/` 中有 12 个 symlinks 指向上面的真实目录
- `~/.claude/skills/` 中另有 5 个 global skills（context-research, doublecheck, excalidraw-diagram-generator, find-skills, graphify）
- 无双重加载
