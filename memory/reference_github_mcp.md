---
name: GitHub MCP quick reference
description: GitHub MCP 26 tools 速查，工作流模板，工具选择决策树
type: reference
originSessionId: 6e41932c-9a41-4a8c-8a1b-7f542185a45d
---
# GitHub MCP Quick Reference

## 26 Tools at a Glance

**Repo**: `get_file_contents` / `create_or_update_file` / `push_files` / `search_repositories`
**Issues**: `create_issue` / `list_issues` / `update_issue` / `add_issue_comment`
**PRs**: `create_pull_request` / `get_pull_request_files` / `create_pull_request_review` / `merge_pull_request`
**Search**: `search_code` (GitHub 语法) / `search_issues` / `search_users`
**CI**: `list_workflow_runs` / `trigger_workflow`

## 4 Workflows (W1-W4)

- W1 调研：search_repos → get_file_contents README → list_commits
- W2 Bug追踪：search_issues → get_issue → update_issue / create_issue
- W3 PR审查：get_pull_request → get_pull_request_files → create_pull_request_review
- W4 备份：get_file_contents 批量 → push_files 到目标仓库

## Git CLI 分工

读文件/搜索/Issue/PR/调研 → GitHub MCP；日常 commit/push/branch 管理 → `git` CLI

## Full Guide

完整文档：`docs/github_mcp_guide.md`（知乎格式 + mermaid 图）
工作流规范：`.claude/rules/github-mcp-workflow.md`
