# MCP Server Usage Rules

**Full server catalog (26+ servers), auth configs, per-server details**: [reference_mcp_catalog.md](memory/reference_mcp_catalog.md)
**Task-to-MCP quick mapping**: [reference_mcp_servers.md](memory/reference_mcp_servers.md)

## Tool Selection Rules

### Jina vs Search Tool Selection

| Scenario | First Choice | Reason |
|----------|-------------|--------|
| Library/framework docs | `context7` | Most precise, no quota cost |
| Chinese web search | `web-search-prime` | Chinese search optimization |
| English tech content | `jina search_web` | Returns full content |
| Academic papers | `jina search_arxiv` | Only tool supporting academic search |
| PDF reading | `jina read_url` | Native PDF support |
| GitHub repos | `zread` or `jina read_url` | zread is lighter |
| SPA/JS rendered pages | `jina read_url` | Puppeteer rendering, good SPA support |
| Login-required pages | `web-access` | Inherits Chrome login state |
| Anti-scraping sites | `web-access` | CDP harder to detect |
| Parallel browsing | `web-access` sub-agents | Each sub-agent opens own tab |

**Full search workflow spec**: `~/.claude/rules/search-workflow.md` (global rule: tool Tier 0-4, parallel combos, cross-validation, report template, fallback chain)

### web-access vs Playwright

| Need | First Choice | Reason |
|------|-------------|--------|
| Login-required page | web-access | Inherits Chrome login state |
| Anti-scraping site | web-access | Real browser fingerprint |
| Parallel browsing 3+ pages | web-access | Sub-agents naturally parallel |
| UI automation testing | Playwright | snapshot + screenshot + assertions |
| Screenshot verification | Playwright | fullPage + high-res + element precision |
| Form fill / dropdown select | Playwright | fill_form + select_option semantic API |
| Simple page interaction | Playwright | No Chrome pre-config needed |

## Proxy Bypass Rules

For NO_PROXY patterns and environment variables, see [proxy-rules.md](proxy-rules.md).

## Zhipu MCP Usage Rules

### Vision Tool Selection (zai-mcp-server -- 8 tools)

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
   - Chinese results: add `location=cn`; Recent info: add `search_recency_filter=oneMonth`
   - Specific site: add `search_domain_filter`; Detailed summary: add `content_size=high`

2. **Read full page**: `webReader` with URL from search results
   - Tech docs: `retain_images=false`; Design pages: `retain_images=true`

3. **GitHub repos**: Use `zread` instead of search+reader
   - `search_doc(repo_name, query)` / `get_repo_structure(repo_name, dir_path)` / `read_file(repo_name, file_path)`

### Quota Management

- `web-search-prime` + `web-reader` share monthly quota (Lite 100 / Pro 1000 / Max 4000)
- `zread` has independent quota (same tiers)
- **Never call search in loops** -- batch search once, cache results in context
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
| Login-required page | `web-access` | Playwright | web-access uses real Chrome login |
| Anti-scraping site | `web-access` | — | CDP harder to detect |
| Parallel browsing | `web-access` sub-agents | `jina parallel_read_url` | web-access spawns sub-agents |

## ChangeLogs

- [2026-04-28] Slim-down: extracted server catalog, per-server details, auth table, cache locations to reference_mcp_catalog.md; kept decision tables, selection rules, Zhipu usage rules; 363→~115 lines
- [2026-04-20 13:25:00] Added baidu-search CLI skill (Tier 1.5)
- [2026-04-16 16:30:00] Added web-access (eze-is); total 25→26
- [2026-04-15 16:20:00] Added jina-mcp-server; MCP total 24→25
- [2026-04-09 — Initial: 24 MCP servers catalog, usage guidelines, auth reference](changes/2026-04-09)
