---
name: MCP Server Catalog
description: Complete MCP server listings (26+ servers), auth configurations, per-server tool references, cache locations. Decision rules stay in .claude/rules/mcp-servers.md
type: reference
originSessionId: 836cb3d4-b894-4748-bd6d-60c75ed2f1ad
---
# MCP Server Catalog

**Decision rules and tool selection**: [`.claude/rules/mcp-servers.md`](../../.claude/rules/mcp-servers.md)
**Task-to-MCP quick mapping**: [`reference_mcp_servers.md`](reference_mcp_servers.md)

## Server Inventory (26 MCP + 1 Skill-based + 1 GitHub)

### Custom Servers (.claude.json) — 8 servers

| Server | Primary Use |
|--------|-------------|
| MiniMax | Vision analysis, web search |
| ZAI | Image analysis, OCR, UI diff, data viz (8 tools) |
| 4_5v_mcp | Image analysis (cross-validation) |
| web-search-prime | Zhipu web search |
| web-reader | URL to markdown |
| zread | GitHub repo browser |
| jina-mcp-server | Web read/search, arXiv, PDF, rerank, dedup (20 tools) |
| web-access (eze-is) | Real Chrome CDP, login state, anti-scraping |

### Plugin Servers (settings.json) — 8 servers

| Server | Auth | Primary Use |
|--------|------|-------------|
| context7 | None | Library docs (Tier 0) |
| playwright | None | Browser automation, UI testing |
| GitHub | PAT in env | Repo management, issues, PRs |
| Greptile | API Key | Cross-repo code search |
| Serena | None (needs NO_PROXY) | Code analysis agent |
| Atlassian | Browser OAuth | Jira/Confluence |
| Supabase | Browser OAuth | Database management |
| Figma plugin | Built-in | Design-to-code |

### Built-in Servers (claude.ai) — 7 servers

Hugging Face, Gamma, Figma, Linear, Gmail, Vercel, Mermaid Chart

## Per-Server Tool Quick Reference

```
MiniMax understand_image  → Full-page UI, layout, icons (primary visual tool)
ZAI extract_text          → OCR Chinese/numbers
ZAI ui_diff_check         → Regression comparison
ZAI analyze_data_viz      → Chart/graph data
context7 query-docs       → Library/framework docs (Tier 0, no quota)
jina read_url             → URL→markdown (Puppeteer, SPA/PDF)
jina search_web           → Web search with full content
jina search_arxiv         → Academic papers
web-search-prime          → Chinese web search
zread read_file           → GitHub repo file reader
playwright browser_*      → UI automation, screenshots, assertions
```

**Full tool selection rules**: → [`mcp-servers.md`](../../.claude/rules/mcp-servers.md)

## Server Configuration Details

### web-access (eze-is)

Skill-based real browser automation via Chrome CDP Proxy.
**Install**: `npx skills add eze-is/web-access` (installed)
**Prerequisite**: Chrome enable `chrome://inspect/#remote-debugging`

Core: inherits Chrome login state, anti-scraping, parallel sub-agents.
**Decision table vs Playwright**: → [`mcp-servers.md`](../../.claude/rules/mcp-servers.md#web-access-vs-playwright)

### baidu-search (CLI Skill)

`~/.claude/skills/baidu-search/` — Baidu Qianfan AI Search.
**API Key**: `scripts/config.json` + `.env` (`BAIDU_API_KEY`)

```bash
D:/miniconda3/envs/ene/python.exe ~/.claude/skills/baidu-search/scripts/search.py "keyword" --json
... search.py "keyword" --recency week   # week/month/semiyear/year
... search.py "keyword" --api-type baike
```

Use only for: Chinese policy/regulation/subsidy (Baidu index best), time-filtered Chinese news.
**Full decision table**: → [`search-workflow.md`](../../.claude/rules/search-workflow.md)

### GitHub MCP (github@claude-plugins-official)

26 tools: file read, code search, issues/PRs CRUD, reviews, push.
**Config**: `settings.json` env `GITHUB_PERSONAL_ACCESS_TOKEN`.
**Full workflow spec**: → [`github-mcp-workflow.md`](../../.claude/rules/github-mcp-workflow.md)

### MiniMax Coding Plan MCP

`minimax-coding-plan-mcp` — `understand_image` (local/URL) + `web_search`.
API: `https://api.minimaxi.com` (CN) / `https://api.minimax.io` (intl). Key and host must match region.

### ZAI MCP (GLM-4V Vision)

Package: `@z_ai/mcp-server` (NPM). Env: `Z_AI_API_KEY`, `Z_AI_MODE=ZHIPU`. 8 specialized vision tools.

## Authentication Reference

| Server | Auth Method | Where to Configure |
|--------|------------|-------------------|
| GitHub | PAT | `settings.json` → env → GITHUB_PERSONAL_ACCESS_TOKEN |
| Greptile | API Key | `settings.json` → env → GREPTILE_API_KEY |
| Atlassian | OAuth | Browser redirect (one-time) via `authenticate` tool |
| Supabase | OAuth | Browser redirect (one-time) via `authenticate` tool |
| MiniMax | API Key | `.claude.json` → mcpServers → MiniMax → env |
| ZAI | API Key | `.claude.json` → mcpServers → zai-mcp-server → env |
| Jina | API Key (optional) | `.claude.json` → mcpServers → jina-mcp-server → headers |

## Cache Locations (D Drive)

All MCP-related caches consolidated at `D:\claude_code_mcp\`:

| Cache | Env Var | Size | Used By |
|-------|---------|------|---------|
| npm-cache | npm config | 1.8G | npx (MiniMax, mermaid-local, zai) |
| uv-cache | `UV_CACHE_DIR` | 1.7G | uvx (MiniMax) |
| puppeteer-cache | `PUPPETEER_CACHE_DIR` | 909M | Puppeteer |
| puppeteer-chrome | `PUPPETEER_EXECUTABLE_PATH` | 409M | mermaid-local MCP |
| playwright | `PLAYWRIGHT_BROWSERS_PATH` | 31M | Playwright plugin |

For env vars and migration rules: [proxy-rules.md](../../.claude/rules/proxy-rules.md) + [file-migration.md](../../.claude/rules/file-migration.md)

## ChangeLogs

- [2026-04-28] Extracted from mcp-servers.md: server catalog, per-server details, auth table, cache locations
