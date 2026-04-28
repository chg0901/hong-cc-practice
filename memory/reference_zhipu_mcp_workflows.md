---
name: Zhipu MCP workflows reference
description: Zhipu 4 MCP combo workflows and best practices (vision/search/reader/GitHub) — detailed usage in rules/mcp-servers.md and docs/mcp-configuration.md
type: reference
---

# Zhipu MCP Workflows Quick Reference

**Full details**: `.claude/rules/mcp-servers.md` (usage rules) + `docs/mcp-configuration.md` (advanced usage section 16)

## Search -> Read -> Analyze Pipeline

```
web-search-prime(query, location?, recency_filter?)
  -> extract URLs from results
  -> webReader(url, retain_images?, with_links_summary?)
    -> analyze markdown content
```

Use case: Research a tech topic
- Step 1: `web-search-prime(search_query="Flask SSE 2026 best practices", location="us", content_size="high")`
- Step 2: `webReader(url="<from results>", return_format="markdown", retain_images=false)`
- Step 3: Read and summarize the markdown content

## Vision Verification Trio

```
Playwright screenshot
  -> MiniMax understand_image (global UI analysis)
  -> ZAI specialized tool (OCR / diff / chart / error)
  -> GLM analyze_image (cross-validation if disagreement)
```

Cross-validation rule: If MiniMax and ZAI disagree, use GLM as tiebreaker. Retake screenshot at different zoom if still unclear.

## GitHub Repo 3-Step Exploration

```
zread search_doc(repo_name, query)       -> find relevant docs/issues
zread get_repo_structure(repo_name)     -> understand directory layout
zread read_file(repo_name, file_path)   -> read key implementation files
```

Example: Explore FastAPI middleware system
1. `search_doc(repo_name="fastapi/fastapi", query="middleware")`
2. `get_repo_structure(repo_name="fastapi/fastapi", dir_path="fastapi/middleware")`
3. `read_file(repo_name="fastapi/fastapi", file_path="fastapi/middleware/asyncexitstack.py")`

## Key Parameters Cheat Sheet

### web-search-prime
- `search_query` max 70 chars
- `search_recency_filter`: oneDay / oneWeek / oneMonth / oneYear / noLimit
- `search_domain_filter`: restrict to specific domain
- `location`: cn (Chinese) / us (English)

### webReader
- `retain_images`: false for tech docs (faster), true for design pages
- `with_links_summary`: true when you need to follow reference links
- `no_cache`: true for latest content

### zread
- `repo_name`: "owner/repo" format (e.g., "vitejs/vite")
- `language`: "zh" or "en" for search_doc
