---
name: Superpowers 触发矩阵
description: superpowers 13 个 skills 的触发时机和协作链速查表，从 subagents.md 移出以节省 token
type: reference
originSessionId: 16243a13-138c-42c6-abb6-b9d2c6f71e3c
---
# Superpowers 触发矩阵（13 skills）

superpowers 是方法论型 skill 套件，与五层体系深度集成。

| superpowers Skill | 触发时机 | 协作链（→ 表示后续步骤） |
|------------------|---------|------------------------|
| brainstorming | 新功能/方案选择/架构决策 | → writing-plans → executing-plans |
| writing-plans | 非平凡实现任务（>3 文件） | → deviation-handling（偏差处理）→ goal-backward（需求倒推） |
| executing-plans | plan 批准后 | → test-driven-development → verification |
| test-driven-development | 新功能实现 | → test-runner agent → ralph（循环修复） |
| systematic-debugging | bug 修复/错误排查/测试失败 | → context-management（rewind）→ ralph（循环修复） |
| verification-before-completion | 任务完成前/commit 前 | → /visual-check + /interaction-check + 测试运行 |
| code-review | commit 前/PR 提交前 | → code-review-expert（增强审查：SOLID+安全+对抗性） |
| dispatching-parallel-agents | >5 文件变更/独立子任务 | → subagent 隔离 → Wave Execution |
| using-git-worktrees | 大特性开发/需要隔离 | → L5 Agent Teams |
| finishing-a-development-branch | 合并前 | → workflow.md（session end）→ strategic review |
| receiving-code-review | 收到审查反馈 | → deviation-handling（自动修 bug/补缺失） |
| writing-skills | 新建 skill | → skill-forge（第三方）→ skill-reviewer |
| subagent-driven-development | 大重构/>10 文件 | → context: fork → Wave Execution |

## ChangeLogs

- [2026-04-29] 从 subagents.md 移出，节省 ~19 行 rule token
