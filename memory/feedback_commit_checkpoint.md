---
name: Task-batch commit checkpoint
description: 每完成一个任务批次，等待用户下一步指令前，必须自动执行日志记录 + git commit + push + GitHub sync
type: feedback
originSessionId: 78f5f996-566c-4bda-9a23-947cb947735f
---
**规则**：每完成一个任务批次、准备等待用户下一步指令时，**必须自动执行**以下流程：

1. 更新 `docs/work_summary_YYYYMMDD.md`（确保所有完成任务有编号章节）
2. 更新 ChangeLogs（工作日志 + CLAUDE.md）
3. `git add` + `git commit`
4. `NO_PROXY=gitee.com git push origin main`
5. **GitHub 同步**：`bash scripts/sync_claude_config.sh --push`（仅当修改了 `.claude/` 文件时）
6. 报告状态（commit hash + 文件数 + 是否同步 GitHub）

**Why**: 用户明确要求（2026-04-15）——防止上下文压缩或会话中断导致未提交工作丢失。每个任务批次是逻辑完整性边界。
**How to apply**: 不等用户说"提交"或"commit"，完成任务批次后立即执行。详见 `.claude/rules/work-summary-rules.md` 的 "Task-Batch Commit Checkpoint" 章节。
