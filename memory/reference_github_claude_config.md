---
name: github_claude_config_repo
description: GitHub repo for .claude/ config backup. Sync after modifying rules/agents/skills.
type: reference
originSessionId: daa3a2c3-f21d-45a8-a800-8b4e78caf249
---
## GitHub 配置专用 Repo

- **Repo**: https://github.com/chg0901/hong-cc-practice
- **本地 clone**: `$HOME/.claude-github/hong-cc-practice/`
- **同步脚本**: `scripts/sync_claude_config.sh [--push]`

### 同步范围

| 类别 | 数量 | 说明 |
|------|------|------|
| agents | 4 | doc-writer, mcp-search, teams, test-runner |
| rules | 24 | `rules/*.md` 通配，自动覆盖新增规则 |
| skills | 5 | **仅项目自建**（见下方列表） |
| settings | 1 | settings.json（hooks 配置，无敏感信息） |

**项目自建 skills（同步）**：graphify-workflow, interaction-check, visual-check, manual-review, long-screenshot

**第三方 skills（不同步）**：
- `book2skills`, `create-colleague`, `context-research`（git clone/npm 安装）
- `book-study`, `code-review-expert`, `sigma`, `skill-forge`, `wiki-ingest`（symlink from sanyuan-skills）
- `fireworks-tech-graph`（symlink from yizhiyanhua-ai）
- `webpage-screenshotter`（symlink, rkreddyp/investrecipes）

### 同步时机

每次修改 `.claude/` 下的文件后**必须**执行：

```bash
bash scripts/sync_claude_config.sh --push
```

### Gitee vs GitHub

- Gitee（主项目）：部分 `.claude/` 文件通过 `git add -f` 跟踪
- GitHub（配置专用）：仅同步项目自建内容，排除第三方和敏感信息
- 两个 repo 独立管理，互不影响
