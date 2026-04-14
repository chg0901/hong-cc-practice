# Subagent & Agent Team 使用规则

## 五层可扩展体系总览

Claude Code 提供分层的自动化体系，从底层事件响应到高层并行协作：

| Layer | 机制 | 加载方式 | 项目现状 | 本项目文件 |
|-------|------|----------|----------|-----------|
| L1 | Hooks | 事件触发（自动） | 6 个 hooks（PreToolUse x2, PostToolUse x2, SubagentStart x1, SubagentStop x1, Stop x1） | `.claude/settings.json` + `~/.claude/settings.json` |
| L2 | Rules | 全量加载（自动） | 18 个 rules 文件 | `.claude/rules/*.md` |
| L3 | Skills | 按需加载（手动/自动） | 5 个项目 skills | `.claude/skills/*/SKILL.md` |
| L4 | Subagents | 隔离执行（按需调用） | 3 个自定义 agents | `.claude/agents/*.md` |
| L5 | Agent Teams | 并行协作（已启用） | 已配置 3 个 team templates | `.claude/agents/teams.md` |

Foundation: Memory（经验积累，自动记录）-- `memory/*.md`（9 个文件）

## Hooks 与所有层的协作

| Hook | 事件类型 | 协作对象 | 作用 |
|------|----------|----------|------|
| PreToolUse (Bash) | 工具执行前 | L2 Rules | 拦截危险操作（rm -rf, DROP TABLE, git push --force） |
| PreToolUse (Edit\|Write) | 工具执行前 | L2 Rules | 拦截敏感文件修改（.env, settings.json） |
| PostToolUse (Edit\|Write) | 工具执行后 | L3 Skills | 自动触发 /visual-check, /interaction-check |
| PostToolUse (Grep\|Glob) | 工具执行后 | L3 Skills | 建议使用 /graphify 进行模块搜索 |
| SubagentStart | 子代理启动 | L4 Subagents | 注入项目上下文（分支、NO_PROXY、规则摘要） |
| SubagentStop | 子代理完成 | L4 Subagents | 触发后续动作（测试失败修复、文档验证） |
| Stop | 对话结束 | L5 Teams + L2 Rules | 自动提交文档、汇总 TODO、提醒分支合并 |

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

- [2026-04-14 09:30:00] 新增 Agent Frontmatter 参考、Command→Agent→Skill 模式、五种多 Agent 协调模式
- [2026-04-13 20:56:00] Hooks 全面配置（6 个 hooks 覆盖全部事件类型）、Agent Teams 启用 + 团队模板、Memory 清理（19->9 文件）、Rules 去重（22->18 文件）
- [2026-04-13 — Initial: 五层体系总览、3 个 subagent 定义、模型选择指南、Agent Teams 规则](changes/2026-04-13-initial)
