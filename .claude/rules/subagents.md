# Subagent & Agent Team 使用规则

## 五层可扩展体系总览

Claude Code 提供分层的自动化体系，从底层事件响应到高层并行协作：

| Layer | 机制 | 加载方式 | 项目现状 | 本项目文件 |
|-------|------|----------|----------|-----------|
| L1 | Hooks | 事件触发（自动） | 5 个 hooks（PreToolUse x2, PostToolUse x1, SubagentStart x1, SubagentStop x1, SessionEnd x1） | `.claude/settings.json` + `~/.claude/settings.json` |
| L2 | Rules | 全量加载（自动） | 32 个 rules 文件 | `.claude/rules/*.md` |
| L3 | Skills | 按需加载（手动/自动） | 21 个 skills（项目自建 9 + 第三方 5 + 第三方 symlink 7） | `.claude/skills/*/SKILL.md` |
| L4 | Subagents | 隔离执行（按需调用） | 3 个自定义 agents | `.claude/agents/*.md` |
| L5 | Agent Teams | 并行协作（已启用） | 已配置 3 个 team templates | `.claude/agents/teams.md` |

Foundation: Memory（经验积累，自动记录）-- `memory/*.md`（9 个文件）

## Skills 全景图

### 项目自建 Skills（9 个，`.claude/skills/` 目录内，同步到 GitHub）

| Skill | 触发方式 | 用途 | 与其他层协作 |
|-------|---------|------|------------|
| `visual-check` | 自动（PostToolUse hook）+ `/visual-check` | SVG/CSS 渲染验证 | Playwright + MiniMax/ZAI Vision MCP |
| `interaction-check` | 自动（PostToolUse hook）+ `/interaction-check` | JS 交互行为测试 | Playwright |
| `graphify-workflow` | `/graphify-workflow [query\|update\|viz\|hook]` | 知识图谱导航、增量重建、可视化 | graphify CLI + git hooks 自动重建 |
| `manual-review` | `/manual-review` | 用户手册截图与文档一致性审查 | Vision MCP |
| `long-screenshot` | 脚本调用 | 长页面滚动截图 | Playwright |
| `doc-trim` | `/doc-trim` 或每周二/五自动提醒 | CLAUDE.md/rules 精简 + README 联动 | context-hygiene rule |
| `context-research` | 自动（"deep search codebase"）+ 手动 | 深度代码搜索（fork 隔离） | Explore agent |
| `book2skills` | 手动 | 书籍内容转为 Skills | — |
| `create-colleague` | 手动 | 同事能力蒸馏为 AI Skill | — |

### 第三方安装 Skills（5 个，git clone/npm 安装，不同步）

book2skills, create-colleague, context-research, baidu-search, excalidraw-diagram-generator

### 第三方安装 Skills（7 个，symlink 到 `.agents/skills/`，不同步）

| Skill | 来源 | 触发方式 | 用途 | 项目集成建议 |
|-------|------|---------|------|------------|
| `code-review-expert` | sanyuan-skills | `/code-review-expert` 或自动（"review code"） | SOLID + 安全 + 性能多维代码审查 | 代码变更后、commit 前使用 |
| `sigma` | sanyuan-skills | `/sigma <topic>` 或自动（"teach me / learn"） | Bloom 2-Sigma 苏格拉底式 AI 导师 | 学习新技术（Flask、LSTM 等） |
| `skill-forge` | sanyuan-skills | `/skill-forge` 或自动（"create skill"） | 元技能：创建高质量 Skill | 开发新项目 Skill 时使用 |
| `book-study` | sanyuan-skills | `/book-study <book>` | 阅读教练：知识编译 + 间隔重复 | 技术书籍学习 |
| `wiki-ingest` | sanyuan-skills | `/wiki-ingest` | 文档编译为结构化 wiki 知识库 | 项目文档系统化整理 |
| `fireworks-tech-graph` | yizhiyanhua-ai | 自动（"画图/架构图/流程图"）或手动 | 7 styles x 14 types 出版级 SVG+PNG 技术图 | 替代 draw.io 画架构图、流程图 |
| `web-access` | eze-is | 自动（"搜索/浏览/抓取"）或手动 | Real Chrome CDP 自动化，登录态，反爬虫 | 需登录/反爬虫场景，Tier 4 重量级工具 |

### Skill 触发控制

| 控制方式 | Frontmatter 字段 | 适用场景 |
|---------|-----------------|---------|
| 仅手动触发 | `disable-model-invocation: true` | 有副作用的操作（deploy、commit） |
| 仅自动触发 | `user-invocable: false` | 背景知识类 |
| 隔离上下文执行 | `context: fork` + `agent: Explore` | 深度搜索不污染主上下文 |
| 限制工具范围 | `allowed-tools: Read, Grep` | 只读审查任务 |
| 动态注入 | `` !`command` `` | PR 摘要等需要实时数据的场景 |

## Hooks 与所有层的协作

| Hook | 事件类型 | 协作对象 | 作用 |
|------|----------|----------|------|
| PreToolUse (Bash) | 工具执行前 | L2 Rules | 拦截危险操作（rm -rf, DROP TABLE, git push --force） |
| PreToolUse (Edit\|Write) | 工具执行前 | L2 Rules | 拦截敏感文件修改（.env, settings.json） |
| PostToolUse (Edit\|Write) | 工具执行后 | L3 Skills | command 类型，检查文件扩展名输出视觉检查建议 |
| SubagentStart | 子代理启动 | L4 Subagents | 注入项目上下文（分支、NO_PROXY、规则摘要） |
| SubagentStop | 子代理完成 | L4 Subagents | 触发后续动作（测试失败修复、文档验证） |
| SessionEnd | 对话结束 | L5 Teams + L2 Rules | 自动提交文档、汇总 TODO、提醒分支合并 |

### Prompt Hook 注意事项

- **Prompt hooks 不能加 JSON 约束**：`type: "prompt"` 的 hooks 由框架内部解析 LLM 输出，末尾不能加 `"Respond with JSON only."`，否则触发 JSON validation failed
- **Prompt 以自然语言结尾**：推荐用 `Otherwise proceed.` 结尾，让 LLM 自然响应
- **需要确定性输出时用 command hooks**：`type: "command"` + shell 脚本 + exit code（0=放行，2=阻塞）
- **PostToolUse 建议类 hooks 用 command + exit 0**：不需要阻止操作，只输出建议文字。见 `scripts/hooks/post_edit_visual_check.sh`
- **详细记录**：见 `memory/feedback_prompt_hooks_no_json.md`

## Subagent 使用规则

### 本项目自定义 Subagents

| Agent | 模型 | 用途 | Hook 集成 |
|-------|------|------|-----------|
| `test-runner` | haiku | 运行 Python 测试套件 | SubagentStart 注入 NO_PROXY + Python 路径 |
| `doc-writer` | inherit | 生成/更新项目文档 | SubagentStart 指向 work-summary-rules.md |
| `mcp-search` | haiku | 搜索外部技术文档 | SubagentStart 优先 context7 |

### 何时使用 Subagent

**适合 subagent 的场景**：
- 运行测试（test-runner, haiku -- 机械任务不需高级推理）
- 搜索外部文档（mcp-search, haiku -- 搜索+读取是机械任务）
- 深度代码搜索不污染主上下文（context-research skill, context: fork）
- 文档生成（doc-writer, inherit -- 需要理解上下文）

**不适合 subagent 的场景**：
- 需要主对话上下文的决策（如架构选择、用户交互）
- 简单的单文件编辑（直接操作更快）
- 需要用户确认的危险操作（如 git push、DELETE）

### 缓存窗口差异

| 上下文 | Prompt Cache 寿命 | 说明 |
|--------|------------------|------|
| 主会话 | **1 小时** | 每次交互刷新计时器 |
| 子代理 | **5 分钟** | 独立窗口，不共享主会话缓存 |

**影响**：子代理是"清洁工具"（隔离噪音），不是"省钱工具"。短任务适合用 subagent，长时间运行的分析任务缓存优势不如主会话。详见 [context-management.md](context-management.md)。

### 模型选择指南

| 模型 | 适用场景 | 理由 |
|------|----------|------|
| `haiku` | 测试执行、文档搜索、数据提取 | 机械任务，更快更便宜 |
| `inherit` | 文档生成、代码审查、架构分析 | 需要高级推理和上下文理解 |
| 不指定 | 未知复杂度时 | 使用父对话模型（通常最优） |

### context: fork Skill 模式

Skills 可通过 `context: fork` 在子代理中运行，保护主对话上下文：

```yaml
# SKILL.md frontmatter
context: fork       # 在独立子代理中运行
agent: Explore      # 使用 Explore 代理类型
```

本项目使用此模式的 skill：`context-research`（深度代码搜索）

## Agent Frontmatter 参考

Agent 定义文件（`.claude/agents/*.md`）支持以下 frontmatter 字段：

| 字段 | 类型 | 说明 | 本项目使用 |
|------|------|------|-----------|
| `name` | string | Agent 名称（必须匹配文件名） | test-runner, doc-writer, mcp-search |
| `description` | string | 用途描述（Agent tool 选择器使用） | 所有 agents |
| `model` | haiku/inherit | 模型选择 | haiku (机械任务), inherit (推理任务) |
| `tools` | list | 可用工具白名单 | 按需配置 |
| `mcpServers` | list | 可访问的 MCP 服务器 | mcp-search 配置 3 个 zhipu 服务器 |
| `permissionMode` | string | 权限模式 | dontAsk (测试), acceptEdits (文档) |
| `maxTurns` | int | 最大对话轮数 | 10-15 防止失控 |
| `memory` | string | 记忆范围 | user (跨项目共享) |

### 模型选择决策

| 任务类型 | 推荐模型 | 理由 |
|----------|---------|------|
| 测试执行、数据提取 | haiku | 机械任务，快+便宜 |
| 文档搜索、代码搜索 | haiku | 搜索+读取不需高级推理 |
| 文档生成、代码审查 | inherit | 需理解上下文和推理 |
| 架构分析、复杂调试 | inherit | 需要高级推理 |

### Command → Agent → Skill 模式

复杂工作流推荐使用三层分发：

1. **Command**（用户触发 `/graphify-workflow`）→
2. **Agent**（按需调用 test-runner / mcp-search）→
3. **Skill**（执行具体步骤的 checklist）

本项目实例：
- `/graphify-workflow` → 按需调用 `mcp-search` → 执行 graphify query/update/viz 步骤
- 代码变更完成 → 自动调用 `test-runner` → 执行测试
- 任务完成 → 调用 `doc-writer` → 执行文档更新步骤

### 五种多 Agent 协调模式

| 模式 | 适用场景 | 本项目实例 |
|------|---------|-----------|
| **Generator-Verifier** | 生成+验证 | main 实现 + test-runner 验证 |
| **Orchestrator-Subagent** | 分工执行 | main 分配 + test-runner/doc-writer 执行 |
| **Agent Teams** | 并行协作 | Full-Stack Feature (3 agents) |
| **Message Bus** | 事件驱动 | Hooks → Skills 自动触发 |
| **Shared State** | 共享上下文 | TaskList + Memory 跨 agent 共享 |

## Agent Teams 使用规则

已通过 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 启用。

### 团队模板

See [teams.md](../agents/teams.md) for 3 pre-defined team templates:
1. **Full-Stack Feature** (3 agents): Backend + Frontend + QA
2. **Research + Implementation** (2 agents): context-research + main
3. **Competitive Debugging** (2-3 agents): parallel hypothesis testing

### Agent Teams 使用原则

1. **3-5 个队友最优**，避免过多协调开销
2. **每个队友独立负责不同文件集合**，避免冲突
3. **研究和审查任务最适合** agent teams
4. **team lead 负责整合结果**，队友负责独立执行
5. **用 `SendMessage` 通信**，不要直接修改共享文件
6. **TaskList 共享状态**：所有队友通过 `~/.claude/tasks/{team-name}/` 协调

## ChangeLogs

- [2026-04-22] L2 Rules 30→32（+context-hygiene.md, +github-mcp-workflow.md）；L3 Skills 20→21（项目自建 8→9，+doc-trim）；项目名修正为"中暖智慧双碳管理平台"
- [2026-04-22] L3 Skills 15→16（第三方 4→5，新增 excalidraw-diagram-generator）
- [2026-04-20 13:35:00] L3 Skills 15→15（第三方 3→4，新增 baidu-search；symlink 不变 7）；L2 Rules 30→30（secrets.md 已在 04-07 创建但首次纳入 tracking）
- [2026-04-18] L2 Rules 29→30（新增 context-management.md）；补充子代理缓存窗口差异（1hr vs 5min）
- [2026-04-17 00:10:00] L2 Rules 28→29（新增 design-templates.md 品牌设计系统模板库）
- [2026-04-16 16:35:00] L2 Rules 27→28（新增 search-workflow.md）；L3 Skills 修正 14→15（项目自建 8 + 第三方 symlink 7，新增 web-access）；Skills 全景图修正项目自建 5→8
- [2026-04-16 11:15:00] L2 Rules 计数 25→27（新增 database-patterns.md + friday-review.md）
- [2026-04-16 10:54:00] L1 Hooks 计数 6→5（删除 PostToolUse:Grep|Glob），PostToolUse:Edit|Write 从 prompt 迁移到 command
- [2026-04-16 10:00:00] L2 Rules 计数 24→25（新增 daily-review.md 每日待办审查规则）
- [2026-04-15 16:30:00] L2 Rules 计数 23→24（新增 fireworks-tech-graph.md）
- [2026-04-15 15:00:00] 新增 fireworks-tech-graph 到第三方 symlink skills（5→6 个），L3 总计数 13→14
- [2026-04-15 14:00:00] 新增 Prompt Hook 注意事项（不能加 JSON 约束、自然语言结尾、需要确定性时用 command hooks）
- [2026-04-15 14:30:00] L2 Rules 计数 22→23（新增 zhihu-article.md）
- [2026-04-15 10:30:00] 新增 Skills 全景图（14 个 skills 分类表）、Skill 触发控制参考、更新 L3 技能计数（5→14）
- [2026-04-15 09:00:00] 修复 Stop → SessionEnd：Stop 事件名已废弃（触发频率异常），改为 SessionEnd；生命周期事件 matcher 改为空字符串 ""
- [2026-04-14 09:30:00] 新增 Agent Frontmatter 参考、Command→Agent→Skill 模式、五种多 Agent 协调模式
- [2026-04-13 20:56:00] Hooks 全面配置（6 个 hooks 覆盖全部事件类型）、Agent Teams 启用 + 团队模板、Memory 清理（19->9 文件）、Rules 去重（22->18 文件）
- [2026-04-13 — Initial: 五层体系总览、3 个 subagent 定义、模型选择指南、Agent Teams 规则](changes/2026-04-13-initial)
