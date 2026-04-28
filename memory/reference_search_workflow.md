---
name: Search workflow reference
description: Quick reference for search tool selection, parallel combos, confidence levels, and report template. Full rules in .claude/rules/search-workflow.md
type: reference
originSessionId: 506c923c-ef11-46ef-b9e7-7ecaf3b58273
---
# Search Workflow Quick Reference

**Full rules**: [`.claude/rules/search-workflow.md`](.claude/rules/search-workflow.md)

## Tool Selection Quick Pick

| Need | First | Fallback | Cost |
|------|-------|----------|------|
| Library docs | context7 | jina search_web | Free |
| Chinese search | baidu-search (Tier 1.5) | web-search-prime | 100/day |
| Chinese policy/regulation | baidu-search --recency | web-search-prime | 100/day |
| English tech | jina search_web | web-search-prime | Free tier |
| Academic | jina search_arxiv | — | Free tier |
| URL content | web-reader | jina read_url | Shared quota |
| Login page | web-access | playwright | Free (CDP) |
| Anti-scraping | web-access | — | Free (CDP) |
| GitHub repo | zread | GitHub plugin | Independent |

## Parallel Combos (max 3)

| Scenario | Combo |
|----------|-------|
| General research | context7 + jina search_web + web-search-prime |
| Library deep dive | context7 + zread + jina search_web |
| Academic topic | jina search_arxiv + jina search_web + web-search-prime |
| Chinese market | baidu-search + web-search-prime |
| Chinese policy | baidu-search --recency + jina search_web |

## Confidence Levels

- HIGH: 3+ sources agree
- MEDIUM: 2 sources agree
- LOW: 1 source only (flag for user)

## Failure Fallback Chain

context7 → jina → web-search-prime → MiniMax → WebSearch built-in

## Report Template Keywords

Search Report = Summary + Key Findings (with source) + Sources (hyperlinks) + Cross-Validation + Quota + Discrepancies
