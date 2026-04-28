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

### Custom Servers (.claude.json) -- 7 servers

| Server | Type | Status | Primary Use |
|--------|------|--------|-------------|
| MiniMax | stdio | connected | Vision analysis, web search |
| ZAI | stdio | connected | Image analysis, OCR, UI diff, data viz |
| 4_5v_mcp | HTTP | connected | Image analysis |
| web-search-prime | HTTP (zhipu) | connected | Web search |
| web-reader | HTTP (zhipu) | connected | URL to markdown |
| zread | HTTP (zhipu) | connected | GitHub repo browser |
| jina-mcp-server | HTTP (remote) | connected | Web read/search, arXiv, PDF, rerank, dedup (20 tools) |

### Skill-based Servers -- 1 server

| Server | Type | Status | Primary Use |
|--------|------|--------|-------------|
| web-access (eze-is) | CDP Proxy | connected | Real Chrome automation, login state, anti-scraping, parallel sub-agents |

### Plugin Servers (settings.json) -- 8 servers

| Server | Status | Auth | Primary Use |
|--------|--------|------|-------------|
| context7 | connected | None | Library documentation query |
| playwright | connected | None | Browser automation, UI testing |
| GitHub | configured | PAT in env | Repo management, issues, PRs |
| Greptile | needs key | API Key | Cross-repo code search |
| Serena | installed | None (needs NO_PROXY) | Code analysis agent |
| Atlassian | needs OAuth | Browser OAuth | Jira/Confluence |
| Supabase | needs OAuth | Browser OAuth | Database management |
| Figma plugin | needs auth | Built-in | Design-to-code |

### Built-in Servers (claude.ai) -- 7 servers

| Server | Status | Primary Use |
|--------|--------|-------------|
| Hugging Face | connected | ML models, datasets, papers |
| Gamma | connected | AI presentations, documents |
| Figma | connected | Design-to-code, screenshots |
| Linear | connected | Project management, issues |
| Gmail | connected | Email management |
| Vercel | connected | Deployment, logs |
| Mermaid Chart | partial | Diagram rendering (Puppeteer issue) |

## Per-Server Tool References

### Image Analysis

```
MiniMax understand_image  → Full-page UI analysis, layout, icons
ZAI extract_text          → OCR for Chinese/numbers verification
ZAI ui_diff_check         → Regression comparison (baseline vs current)
ZAI analyze_data_viz      → Chart/graph data correctness
```

### Web Search & Documentation

```
MiniMax web_search        → General web search (query param, not search_query)
web-search-prime          → Zhipu web search (search_query param)
web-reader webReader      → URL to markdown conversion
context7 query-docs       → Library/framework documentation
zread read_file           → GitHub repo file reader
jina read_url             → URL to markdown (Puppeteer rendered, better for SPA/PDF)
jina search_web           → Web search with full content (not just snippets)
jina search_arxiv         → Academic paper search
jina search_ssrn          → Social science paper search
jina search_images        → Image search
jina extract_pdf          → PDF figure/table/equation extraction
```

### Jina MCP Tool Usage Priority

**Daily development (default)**:
`read_url`, `search_web`, `search_arxiv`, `primer`, `guess_datetime_url`

**Parallel operations**: `parallel_read_url`, `parallel_search_web`, `parallel_search_arxiv`/`parallel_search_ssrn`

**Rerank tools (on-demand only)**: `sort_by_relevance`, `deduplicate_strings`, `deduplicate_images`, `classify_text`

### Browser Automation

`playwright browser_navigate/snapshot/click/screenshot`

### Project Management

`Linear list_issues/create_issue`, `Gmail gmail_search/create_draft`

### Design & Presentation

`Figma get_design_context`, `Gamma generate`, `Mermaid render` (use mmdc CLI as fallback)

## Server Configuration Details

### web-access (eze-is)

Skill-based real browser automation via Chrome CDP Proxy.
**Install**: `npx skills add eze-is/web-access` (installed)
**Prerequisite**: Chrome enable `chrome://inspect/#remote-debugging`
**API**: HTTP localhost:3456 (`/new`, `/eval`, `/click`, `/screenshot`, `/scroll`, `/navigate`, `/close`)

**Core Capabilities**:
- Inherits user Chrome login state (cookies, sessions)
- Click, scroll, fill forms, upload files
- Anti-scraping support (Xiaohongshu, WeChat, Zhihu, etc.)
- Parallel sub-agents (each opens own tab)
- Video analysis (seek + frame capture)

### web-access vs Playwright Dimension Comparison

| Dimension | web-access (CDP) | Playwright (MCP Plugin) |
|-----------|-----------------|------------------------|
| Browser | User's daily Chrome (non-headless) | Built-in Chromium (isolated) |
| Login state | Inherits Chrome cookies/sessions | No login state, manual login required |
| Anti-scraping | Harder to detect (real Chrome fingerprint) | May be detected by anti-scraping |
| Screenshot quality | General (CDP captureScreenshot) | High (fullPage, element, CSS scale) |
| DOM operations | Via CDP Runtime.evaluate (JS injection) | Semantic API (click/type/fill/snapshot) |
| Parallelism | Sub-agents each open tab, naturally parallel | Single browser serial, needs multi-context |
| UI testing | Not suitable (no assertion framework) | Suitable (snapshot + screenshot + Vision MCP) |
| Speed | Fast (direct Chrome connection) | Fast (local Chromium) |
| Stability | Depends on Chrome CDP port being open | Stable (bundled browser) |
| Config complexity | Needs Chrome `--remote-debugging-port` | Zero config (auto-install) |

### baidu-search (CLI Skill)

Baidu Qianfan AI Search, Python CLI tool at `~/.claude/skills/baidu-search/`.
**API Key**: `scripts/config.json` + `.env` (`BAIDU_API_KEY`)

```bash
D:/miniconda3/envs/ene/python.exe ~/.claude/skills/baidu-search/scripts/search.py "keyword" --json
... search.py "keyword" --recency week   # week/month/semiyear/year
... search.py "keyword" --api-type baike   # baike/miaodong_baike/ai_chat
```

| vs | Chinese quality | Time filter | Quota |
|----|----------------|-------------|-------|
| baidu-search | Baidu index, best | `--recency` | 100/day (independent) |
| web-search-prime | Medium | `search_recency_filter` | Shared pool |
| jina search_web | Weak | None | Free tier |

**Use only for**:
1. Chinese policy/regulation/subsidy searches (Baidu index has best coverage)
2. Precise time-filtered Chinese searches (past 7 days/1 month news)
3. Fallback for web-search-prime when Chinese quality matters

### GitHub MCP (github@claude-plugins-official)

26 tools: `get_file_contents`, `search_code`, `search_repositories`, `create_issue`/`list_issues`/`update_issue`, `create_pull_request`/`get_pull_request_files`, `create_pull_request_review`, `merge_pull_request`, `push_files`.

**Config**: `~/.claude/settings.json` env `GITHUB_PERSONAL_ACCESS_TOKEN`.
**Workflow spec**: [github-mcp-workflow.md](../../.claude/rules/github-mcp-workflow.md)
**Full guide**: [docs/github_mcp_guide.md](../../docs/github_mcp_guide.md)

### MiniMax Coding Plan MCP

Package: `minimax-coding-plan-mcp` (GitHub: [MiniMax-AI/MiniMax-Coding-Plan-MCP](https://github.com/MiniMax-AI/MiniMax-Coding-Plan-MCP))

| Tool | Param | Description |
|------|-------|-------------|
| `understand_image` | `image_source` + `prompt` | General image analysis (local file or URL) |
| `web_search` | `query` | Web search returning organic results |

API Host: `https://api.minimaxi.com` (China) / `https://api.minimax.io` (international). Key and host **must match region**.
Docs: [platform.minimaxi.com/docs/guides/mcp-guide](https://platform.minimaxi.com/docs/guides/mcp-guide)

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
