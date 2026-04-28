---
name: Claude Code 配置层级规则
description: 全局 vs 项目级 settings.json 的优先级和覆盖关系，避免遗漏修改
type: feedback
originSessionId: 85b62cf5-e098-4ce2-b282-fb41489b8bb7
---
## Claude Code 配置层级规则

**规则**：Claude Code 有两级 `settings.json`，修改配置时必须同时检查两个文件。

| 层级 | 路径 | 作用范围 |
|------|------|----------|
| 全局 | `~/.claude/settings.json` | 所有项目共享（hooks、plugins、permissions） |
| 项目级 | `.claude/settings.json` | 当前项目覆盖（hooks、permissions） |

**Why**：项目级会覆盖全局的同名配置。之前只迁移了项目级的 PreToolUse hooks，遗漏了全局的 PostToolUse hooks，导致新 session 继续报错。

**How to apply**：
- 修改 hooks 时，先 `grep -r "hooks" ~/.claude/settings.json .claude/settings.json` 确认两个文件
- 安全检查类 hooks（PreToolUse）放在项目级（只影响本项目）
- 通用类 hooks（PostToolUse）放在全局（所有项目生效）
- SubagentStart/SubagentStop/SessionEnd 放在项目级（项目特定的上下文注入）

**同步规则**：
- 项目级 `.claude/settings.json` 通过 `scripts/sync_claude_config.sh --push` 同步到 GitHub
- 全局 `~/.claude/settings.json` 不在项目 repo 中，需手动确认一致性
- 同步脚本只处理项目级文件，全局修改需要单独关注
