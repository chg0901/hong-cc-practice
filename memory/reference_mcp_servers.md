---
name: MCP servers reference
description: Task-to-MCP quick mapping and cross-validation strategy; full catalog in .claude/rules/mcp-servers.md
type: reference
originSessionId: 6eadc642-d903-4d61-8c38-ba24847fbcdd
---
# MCP Servers — Quick Task Reference

**Full server catalog + auth details**: [`.claude/rules/mcp-servers.md`](.claude/rules/mcp-servers.md)

## Task → Primary MCP

| Task | Primary | Fallback |
|------|---------|----------|
| UI/layout visual check | MiniMax `understand_image` | ZAI `analyze_image` |
| OCR / Chinese text extraction | ZAI `extract_text_from_screenshot` | MiniMax |
| UI regression diff | ZAI `ui_diff_check` | Manual comparison |
| Chart/data viz analysis | ZAI `analyze_data_visualization` | MiniMax |
| Cross-validation (disagreement) | GLM `analyze_image` (URL only) | Third screenshot |
| Web search (Chinese) | baidu-search CLI (Tier 1.5) | web-search-prime → MiniMax `web_search` |
| Chinese policy/regulation | baidu-search --recency | web-search-prime |
| Web search (English/tech) | jina `search_web` | web-search-prime |
| URL → markdown | web-reader `webReader` | jina `read_url` (better for SPA/PDF) |
| Library docs | context7 `query-docs` | web-reader |
| Academic papers | jina `search_arxiv` | — |
| PDF reading | jina `read_url` | — |
| GitHub repo browsing | zread `read_file` | GitHub plugin |
| External doc research | mcp-search subagent (haiku) | web-search-prime + webReader |
| Deep research (rerank/dedup) | jina `sort_by_relevance` + `deduplicate_strings` | — |
| Login-protected site | web-access (eze-is) | playwright |
| Anti-scraping site | web-access (eze-is) | jina read_url |
| Parallel web browsing | web-access sub-agents | jina parallel_search_web |

| Browser automation | playwright `browser_*` | — |

**Full search workflow**: [`.claude/rules/search-workflow.md`](.claude/rules/search-workflow.md) (tool Tier 0-4, parallel combos, cross-validation, report template, fallback chain)

## Cross-Validation Strategy

When MiniMax and ZAI disagree on a visual finding:
1. Use GLM (`mcp__4_5v_mcp__analyze_image`) as tiebreaker (remote URL only, no local files)
2. If still unclear, retake screenshot at different zoom and re-analyze
3. Font/spacing consistency → always cross-validate with at least 2 MCPs

## Proxy Bypass

All external MCP servers need `NO_PROXY` on this machine. See `.claude/rules/mcp-servers.md` for details.
