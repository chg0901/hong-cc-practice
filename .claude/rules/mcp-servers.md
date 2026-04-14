# MCP Server Usage Rules

## Available MCP Servers (24 total)

### Custom Servers (.claude.json) — 6 servers

| Server | Type | Status | Primary Use |
|--------|------|--------|-------------|
| MiniMax | stdio | connected | Vision analysis, web search |
| ZAI | stdio | connected | Image analysis, OCR, UI diff, data viz |
| 4_5v_mcp | HTTP | connected | Image analysis |
| web-search-prime | HTTP (zhipu) | connected | Web search |
| web-reader | HTTP (zhipu) | connected | URL to markdown |
| zread | HTTP (zhipu) | connected | GitHub repo browser |

### Plugin Servers (settings.json) — 8 servers

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

### Built-in Servers (claude.ai) — 7 servers

| Server | Status | Primary Use |
|--------|--------|-------------|
| Hugging Face | connected | ML models, datasets, papers |
| Gamma | connected | AI presentations, documents |
| Figma | connected | Design-to-code, screenshots |
| Linear | connected | Project management, issues |
| Gmail | connected | Email management |
| Vercel | connected | Deployment, logs |
| Mermaid Chart | partial | Diagram rendering (Puppeteer issue) |

## Usage Guidelines

### Image Analysis (use FIRST for visual verification)

```
MiniMax understand_image  → Full-page UI analysis, layout, icons
ZAI extract_text          → OCR for Chinese/numbers verification
ZAI ui_diff_check         → Regression comparison (baseline vs current)
ZAI analyze_data_viz      → Chart/graph data correctness
```

### Web Search & Documentation

```
MiniMax web_search        → General web search
web-search-prime          → Alternative web search
web-reader webReader      → URL to markdown conversion
context7 query-docs       → Library/framework documentation
zread read_file           → GitHub repo file reader
```

### Browser Automation

```
playwright browser_navigate   → Open URL
playwright browser_snapshot   → DOM accessibility tree
playwright browser_click      → Click element
playwright browser_screenshot → Capture PNG
```

### Project Management

```
Linear list_issues/create_issue → Track bugs and features
Gmail gmail_search/create_draft → Email communication
```

### Design & Presentation

```
Figma get_design_context → Design-to-code translation
Gamma generate           → AI presentations
Mermaid render           → Diagram rendering (use mmdc CLI as fallback)
```

## Proxy Bypass Rules

For NO_PROXY patterns and environment variables, see [proxy-rules.md](proxy-rules.md).

## Authentication Reference

| Server | Auth Method | Where to Configure |
|--------|------------|-------------------|
| GitHub | PAT | `settings.json` → env → GITHUB_PERSONAL_ACCESS_TOKEN |
| Greptile | API Key | `settings.json` → env → GREPTILE_API_KEY |
| Atlassian | OAuth | Browser redirect (one-time) via `authenticate` tool |
| Supabase | OAuth | Browser redirect (one-time) via `authenticate` tool |
| MiniMax | API Key | `.claude.json` → mcpServers → MiniMax → env |
| ZAI | API Key | `.claude.json` → mcpServers → zai-mcp-server → env |
| zhipu APIs | Bearer token | `.claude.json` → mcpServers → web-search-prime/reader/zread → headers |

## Cache Locations (D Drive)

All MCP-related caches consolidated at `D:\claude_code_mcp\`:

| Cache | Env Var | Size | Used By |
|-------|---------|------|---------|
| npm-cache | npm config | 1.8G | npx (MiniMax, mermaid-local, zai) |
| uv-cache | `UV_CACHE_DIR` | 1.7G | uvx (MiniMax) |
| puppeteer-cache | `PUPPETEER_CACHE_DIR` | 909M | Puppeteer |
| puppeteer-chrome | `PUPPETEER_EXECUTABLE_PATH` | 409M | mermaid-local MCP |
| playwright | `PLAYWRIGHT_BROWSERS_PATH` | 31M | Playwright plugin |

For env vars and migration rules, see [proxy-rules.md](proxy-rules.md) and [file-migration.md](file-migration.md).

## Zhipu MCP Usage Rules

### Vision Tool Selection (zai-mcp-server — 8 tools)

| User Need | Tool | Key Parameters |
|-----------|------|----------------|
| UI screenshot to code | `ui_to_artifact` | `output_type=code` |
| UI screenshot to spec | `ui_to_artifact` | `output_type=spec` |
| Design to AI prompt | `ui_to_artifact` | `output_type=prompt` |
| OCR text extraction | `extract_text_from_screenshot` | `programming_language` (optional) |
| Two screenshots diff | `ui_diff_check` | `expected_image_source` + `actual_image_source` |
| Chart/dashboard analysis | `analyze_data_visualization` | `analysis_focus` (trends/anomalies/comparisons) |
| Error screenshot diagnosis | `diagnose_error_screenshot` | `context` (when it happened) |
| Architecture/flow diagram | `understand_technical_diagram` | `diagram_type` (architecture/flowchart/uml/er) |
| Video analysis | `analyze_video` | MP4/MOV/M4V, max 8MB |
| Anything else (fallback) | `analyze_image` | `prompt` describes what to analyze |

**Rule**: Always use the most specific tool first. Only fall back to `analyze_image` when no specialized tool fits.

### Search + Read Pipeline

1. **Search first**: `web-search-prime` with `search_query` (max 70 chars)
   - Chinese results: add `location=cn`
   - Recent info: add `search_recency_filter=oneMonth`
   - Specific site: add `search_domain_filter=docs.python.org`
   - Detailed summary: add `content_size=high`

2. **Read full page**: `webReader` with URL from search results
   - Tech docs: `retain_images=false` (faster, cleaner)
   - Design pages: `retain_images=true`
   - Latest content: `no_cache=true`
   - Follow links: `with_links_summary=true`

3. **GitHub repos**: Use `zread` instead of search+reader
   - `search_doc(repo_name, query)` → search issues/docs/commits
   - `get_repo_structure(repo_name, dir_path)` → directory tree
   - `read_file(repo_name, file_path)` → full file content

### Quota Management

- `web-search-prime` + `web-reader` share monthly quota (Lite 100 / Pro 1000 / Max 4000)
- `zread` has independent quota (same tiers)
- **Never call search in loops** — batch search once, cache results in context
- Prefer `context7` for library docs (no quota cost, more precise)

### Search Tool Selection Decision

| Need | First Choice | Fallback | Reason |
|------|-------------|----------|--------|
| Library/framework docs | `context7` | `web-search-prime` + `webReader` | context7 is precise, no quota |
| Tech blog/article | `web-search-prime` | MiniMax `web_search` | zhipu has sufficient quota |
| Chinese content | `web-search-prime` + `location=cn` | — | Chinese search optimization |
| Latest news | `web-search-prime` + `recency_filter` | — | Time-bounded results |
| Specific URL content | `webReader` | Playwright navigate | webReader is lighter |
| GitHub repo exploration | `zread` | GitHub MCP plugin | zread needs no PAT |

## ChangeLogs

- [2026-04-13 20:56:00] Deduplicated: NO_PROXY section replaced with pointer to proxy-rules.md, cache section updated with cross-references
- [2026-04-13 — Added Zhipu MCP usage rules](changes/2026-04-13-zhipu)
- [2026-04-09 — Initial: 24 MCP servers catalog, usage guidelines, auth reference](changes/2026-04-09)
