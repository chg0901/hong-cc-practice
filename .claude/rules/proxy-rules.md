# Proxy Rules (NO_PROXY)

## Background

This Windows machine has system `HTTP_PROXY`/`HTTPS_PROXY` environment variables that intercept outbound connections. All local and git operations MUST bypass the proxy using `NO_PROXY`.

## NO_PROXY Quick Reference

| Target | NO_PROXY value | Used by |
|--------|---------------|---------|
| Localhost API | `NO_PROXY=127.0.0.1,localhost` | Python tests, Flask server, curl health checks |
| Gitee (git remote) | `NO_PROXY=gitee.com` | git push/pull/fetch |
| GitHub (MCP tools) | `NO_PROXY=github.com` | uvx, Serena MCP |
| GitHub (git push mirror) | `git -c http.proxy= -c https.proxy=` | sync_github.sh, 临时清空代理直连 |
| Context7 MCP | `NO_PROXY=api.context7.com` | context7 documentation fetch |

## Python Test Commands

All test commands MUST be prefixed with `NO_PROXY=127.0.0.1,localhost` and use the direct Python path:

```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe test_codes/test_xxx.py
```

Do NOT use `conda run` (causes UnicodeEncodeError on Windows).

## Git Remote Operations

### Gitee (主仓库)

```bash
NO_PROXY=gitee.com git push origin main
NO_PROXY=gitee.com git pull --rebase origin main
NO_PROXY=gitee.com git fetch origin
```

### GitHub 镜像（独立脚本）

```bash
# 日常同步：推 main 到 GitHub
bash scripts/sync_github.sh

# 全量同步：所有分支 + 标签
bash scripts/sync_github.sh --all --tags

# 手动推送（脚本不可用时）
git -c http.proxy= -c https.proxy= push github main
```

脚本详情：[scripts/sync_github.sh](../../scripts/sync_github.sh)
研究报告：[docs/research_gitee_github_sync.md](../../docs/research_gitee_github_sync.md)

Without `NO_PROXY`, git fails: `Failed to connect to gitee.com port 443 via 127.0.0.1` (exit code 128).

Without clearing proxy for GitHub: `Failed to connect to github.com port 443 via 127.0.0.1` (exit code 128) when proxy port 7897 is not listening.

## Environment Variables (settings.json env)

| Variable | Value | Purpose |
|----------|-------|---------|
| `NO_PROXY` | `api.context7.com,github.com,localhost,127.0.0.1` | MCP server bypass |
| `UV_CACHE_DIR` | `D:\claude_code_mcp\uv-cache` | uv/uvx package cache |
| `PUPPETEER_CACHE_DIR` | `D:\claude_code_mcp\puppeteer-cache` | Puppeteer browsers |
| `PLAYWRIGHT_BROWSERS_PATH` | `D:\claude_code_mcp\playwright` | Playwright browsers |
| `PUPPETEER_EXECUTABLE_PATH` | `D:\claude_code_mcp\puppeteer-chrome\chrome-win64\chrome.exe` | Mermaid MCP Chrome |
| npm cache | `D:\claude_code_mcp\npm-cache` | npm packages |

Full migration details: [file-migration.md](file-migration.md)

## ChangeLogs

- [2026-04-16 11:00:00] Added GitHub mirror sync patterns, split Git Remote Operations into Gitee/GitHub sections
- [2026-04-13 20:56:00] Initial: extracted NO_PROXY rules from terminal.md, testing.md, workflow.md, file-migration.md, mcp-servers.md, visual-testing.md
