# Search & Research Workflow Standard

统一管理本项目所有搜索/研究工具的选择、并行策略、交叉验证、报告格式。

## 工具分级目录（Tier 0-4）

按 cost/quota 从低到高排列，**优先使用低 Tier 工具**。

| Tier | Tool | Cost | Quota | Best For |
|------|------|------|-------|----------|
| 0 | context7 `query-docs` | Free | Unlimited | 库/框架官方文档 |
| 0 | WebSearch (built-in) | Free | ~30/mo | 快速事实查询 |
| 1 | jina `search_web` | Free tier | 1000/mo | 英文技术内容搜索 |
| 1 | jina `read_url` | Free tier | 1000/mo | URL→Markdown（含 PDF、SPA） |
| 1 | jina `search_arxiv` | Free tier | 1000/mo | 学术论文搜索 |
| 2 | web-search-prime | Shared | Lite 100/Pro 1K/Max 4K | 中文网页搜索 |
| 2 | web-reader | Shared | 同上 | URL→Markdown |
| 3 | zread | Independent | Same tiers | GitHub 仓库浏览 |
| 3 | MiniMax `web_search` | Independent | API plan | 通用搜索（query 参数） |
| 4 | web-access (eze-is) | Free | CDP session | 需登录、反爬虫、复杂 JS 交互 |
| 4 | Playwright | Free | Browser | 截图、JS 交互、UI 测试 |

**决策树**：

```
IF 库/框架文档        → context7 (Tier 0)
IF 快速事实           → WebSearch built-in (Tier 0)
IF 中文内容           → web-search-prime (Tier 2) → fallback jina search_web (Tier 1)
IF 英文技术内容       → jina search_web (Tier 1) → fallback web-search-prime (Tier 2)
IF 学术论文           → jina search_arxiv (Tier 1)
IF PDF 提取           → jina read_url (Tier 1)
IF GitHub 仓库        → zread (Tier 3)
IF 需登录页面         → web-access (Tier 4)
IF 反爬虫站点         → web-access (Tier 4)
IF SPA/JS 渲染页面    → jina read_url (Tier 1) → fallback playwright (Tier 4)
IF 复杂网页交互       → web-access (Tier 4) → fallback playwright (Tier 4)
```

## 并行搜索策略

### 可并行组合（max 3 并发）

| Scenario | Parallel Combo | Why |
|----------|---------------|-----|
| 通用研究 | context7 + jina search_web + web-search-prime | 3 独立源，无配额冲突 |
| 库深度研究 | context7 + zread + jina search_web | 文档 + 源码 + 网页 |
| 学术主题 | jina search_arxiv + jina search_web + web-search-prime | 3 源交叉验证 |
| 中文市场 | web-search-prime + MiniMax web_search | 2 中文源 |
| 登录态+公开对比 | web-access + jina search_web + web-search-prime | 登录内容 + 公开搜索 |

### 禁止并行

| 禁止组合 | 原因 |
|----------|------|
| web-search-prime + web-reader | 共享配额池，reader 应在 search 之后 |
| 同一工具不同查询 | 串行即可，并行浪费配额 |

### 并行执行方式

使用 Claude Code Agent tool 并行调用多个搜索工具：

```
# 在单个 message 中调用多个工具
mcp__plugin_context7_context7__query-docs(libraryId="...", query="...")
mcp__web-search-prime__web_search_prime(search_query="...")
mcp__plugin_jina-mcp-server__search_web(query="...")
```

## 交叉验证协议

### 置信度级别

| Level | 条件 | 报告标注 |
|-------|------|----------|
| HIGH | 3+ 独立源一致 | `Confidence: HIGH` |
| MEDIUM | 2 源一致 | `Confidence: MEDIUM` |
| LOW | 单一来源 | `Confidence: LOW` — 提示用户验证 |

### 分歧处理

| 情况 | 处理方式 |
|------|----------|
| 2v1（两源一致一源不同） | 报告多数 + 标记分歧：`DISPUTED (2v1): Source C says Y` |
| 全部分歧 | 报告所有源 + 无置信度 + 提示用户判断 |
| 源超过 6 个月 | 置信度降一级 |

### 交叉验证触发规则

- **视觉相关**: 总是至少 2 MCP（MiniMax + ZAI）
- **事实查询**: 至少 2 独立源
- **技术文档**: context7 + 至少 1 网络源
- **学术/安全**: 至少 3 独立源（EXHAUSTIVE depth）

## 搜索报告模板

所有搜索结果**必须**使用以下标准格式：

```markdown
## Search Report: [Topic]

**Date**: YYYY-MM-DD | **Confidence**: HIGH/MEDIUM/LOW | **Sources**: N tools | **Depth**: SHALLOW/MEDIUM/DEEP/EXHAUSTIVE

### Summary
[2-3 句综合概述]

### Key Findings
1. [Finding] -- Source: [tool_name], Cross-validated: [YES by toolX / NO]
2. [Finding] -- Source: [tool_name], Cross-validated: [YES/NO]

### Sources
| # | Title | URL | Tool | Type |
|---|-------|-----|------|------|
| 1 | [Title](URL) | URL | context7 | Documentation |
| 2 | [Title](URL) | URL | jina search_web | Blog |

### Cross-Validation Status
| Claim | Source A | Source B | Status |
|-------|----------|----------|--------|
| [Claim] | tool1: agrees | tool2: agrees | CONFIRMED |

### Quota Consumed
| Tool | Calls | Pool | Notes |
|------|-------|------|-------|
| web-search-prime | 1 | shared | — |
| jina search_web | 1 | independent | Free tier |

### Discrepancies / Caveats
- [None / List of unresolved disagreements]
```

**源链接规则**：
- 所有引用**必须包含超链接** `[Title](URL)`
- 超链接来源：搜索结果的原始 URL，不编造
- 如果搜索工具未返回 URL，标注 `[Source: tool_name, no URL]`

## 失败回退链

当首选工具失败时，按顺序尝试备选：

| Primary | Fallback 1 | Fallback 2 | Fallback 3 |
|---------|-----------|-----------|-----------|
| context7 | jina search_web | web-search-prime | MiniMax web_search |
| web-search-prime | jina search_web | MiniMax web_search | WebSearch built-in |
| web-reader | jina read_url | playwright navigate | web-access |
| zread | jina read_url | GitHub plugin | web-access |
| jina search_web | web-search-prime | MiniMax web_search | WebSearch built-in |
| web-access | playwright | jina read_url | web-reader |

## 配额感知规则

| Pool | Tools | Budget Rule |
|------|-------|-------------|
| Shared | web-search-prime + web-reader | 月末保留 20% 供关键搜索 |
| Independent (zhipu) | zread | 独立配额，不过度使用 |
| Independent (jina) | jina 20 工具 | Free tier，无紧迫限制 |
| Independent (MiniMax) | MiniMax | API plan 内 |
| Free | context7, WebSearch, Playwright, web-access | 无限制 |

**规则**：
- 禁止循环搜索（批量搜一次，结果缓存到上下文）
- 库文档**优先 context7**（无配额消耗，最精确）
- 搜索前评估深度（Shallow 用 1 工具，Exhaustive 用 3+）

## 触发规则

### 自动触发（无需用户指令）

| 用户行为 | 自动动作 |
|----------|----------|
| 询问库/框架用法 | context7 auto-query |
| 提供一个 URL | web-reader 或 jina read_url 读取 |
| 提及 GitHub repo | zread 浏览 |
| mcp-search subagent 被调用 | 执行完整搜索管线 |

### 手动触发（用户明确要求）

| 用户指令 | 动作 |
|----------|------|
| "Research X" / "调研 X" | 全并行多源搜索 + 交叉验证 |
| "Compare sources on X" / "对比来源" | 交叉验证协议 |
| "Deep dive into X" / "深入了解" | 并行搜索 + 读取 top 结果 |
| "Search Chinese sources" | web-search-prime + MiniMax |
| "Check login-required site" | web-access |

## 自适应深度规则

| Depth | Tools | Queries | When |
|-------|-------|---------|------|
| SHALLOW | 1 tool, 1 query | 单次 | 简单事实（"What is X?"） |
| MEDIUM | 2 tools, parallel | 1-2 次 | 功能比较、API 用法 |
| DEEP | 3 tools, parallel + follow-up reads | 2-3 次 | 架构决策、安全审计、竞品分析 |
| EXHAUSTIVE | All available + cross-validation | 3+ 次 | 研究论文、生产环境关键决策 |

## web-access 使用指南

### 前置条件

1. Chrome 必须启用 remote debugging：打开 `chrome://inspect/#remote-debugging`，勾选 "Allow remote debugging"
2. 在 Chrome 中登录目标网站
3. web-access 继承该 Chrome 实例的登录状态

### 适用场景

| 场景 | 用 web-access | 原因 |
|------|-------------|------|
| 需登录的页面 | YES | 继承 Chrome 登录状态 |
| 反爬虫站点（小红书、微信、知乎） | YES | CDP 更难被检测 |
| 并行浏览 5+ 页面 | YES | 子代理各自开 tab |
| 简单 URL 读取 | NO | jina read_url 或 web-reader 更轻量 |
| SPA 内容提取 | MAYBE | 先试 jina read_url（Puppeteer 渲染） |
| 截图 | NO | Playwright 更专用 |

### 安全注意

- 所有操作在**新后台 tab** 中执行，不影响用户已有 tab
- 代理完成后自动关闭自己的 tab
- **不要**通过 web-access 访问银行/支付等敏感站点

## 与其他规则的关系

| 规则 | 管辖范围 | 互补关系 |
|------|----------|----------|
| `mcp-servers.md` | MCP 服务器完整目录、认证、配额详情 | 本文件管搜索工作流，mcp-servers 管服务器配置 |
| `proxy-rules.md` | NO_PROXY 代理绕过 | 所有外部搜索需 NO_PROXY |
| `subagents.md` | L3 Skills 全景图 | 本文件是 L2 Rule，subagents 管层间协调 |
| `visual-testing.md` | 视觉验证交叉验证策略 | 搜索交叉验证借鉴视觉验证模式 |

## ChangeLogs

- [2026-04-16 — Initial: 工具分级 Tier 0-4、并行搜索策略、交叉验证协议、搜索报告模板、失败回退链、配额感知、触发规则、自适应深度、web-access 指南](changes/2026-04-16)
