---
name: Rules global/project separation anti-pattern
description: Rules 文件不得同时存在于全局和项目目录，否则会双重加载浪费 context。2026-04-28 事故教训。
type: feedback
originSessionId: 836cb3d4-b894-4748-bd6d-60c75ed2f1ad
---
# Rules 不得双重加载

**规则**：每个 rule 文件必须只存在于一个目录 — 全局（`~/.claude/rules/`）或项目（`.claude/rules/`），绝不能两个目录都有一份。

**Why**: Claude Code 从 `~/.claude/rules/` 和 `.claude/rules/` 两个目录全量加载所有 `.md` 文件，**不做去重**。37 个文件双重加载 = 每次会话浪费 ~27,600 token（~55,209 token 原始大小 / 2 的缓存节省）。在 GLM-5.1 的 ~500K context 窗口下，这是灾难性的（~11% 的上下文被浪费在重复内容上）。

**How to apply**:
1. 新增 rule 时，先判断是通用（全局）还是项目专属（项目）
2. 使用 `comm -12 <(ls ~/.claude/rules/*.md | xargs basename | sort) <(ls .claude/rules/*.md | xargs basename | sort)` 验证零重叠
3. **不要用 symlink**：Claude Code 从两个路径都加载，symlink 方向不影响双重加载
4. 通用规则 → 只放在 `~/.claude/rules/`，项目规则 → 只放在 `.claude/rules/`

**分类标准**：
- 引用项目路径（`.env`、`smart_energy.db`、`js/modules/`）→ 项目专属
- 引用项目名称（中暖、双碳）→ 项目专属
- 包含项目特定数据（API 端点、数据库表结构）→ 项目专属
- 通用方法论/工具用法（Mermaid 规范、搜索工作流、上下文管理）→ 全局

**当前状态（2026-04-28）**：
- 全局 18 个 rules（`~/.claude/rules/`）：boris-cherny-tips, context-hygiene, context-management, cross-model-workflow, daily-review, design-templates, deviation-handling, fireworks-tech-graph, friday-review, github-mcp-workflow, goal-backward, mermaid, search-workflow, todo-tracking, training, user-manual, visual-long-screenshot, zhihu-article
- 项目 19 个 rules（`.claude/rules/`）：api-endpoints, config-review, database, database-patterns, data-scripts, extension-onboarding, file-migration, graphify, manual-testing, mcp-servers, proxy-rules, secrets, subagents, svg-design, terminal, testing, visual-testing, workflow, work-summary-rules
