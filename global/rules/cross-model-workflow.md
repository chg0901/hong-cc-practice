# Cross-Model Workflow — Claude Code + Codex CLI 协作规范

## 来源

基于 [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) 的 Cross-Model Workflow 和 Orchestration Workflow。

## 前提条件

- Claude Code 已配置（当前使用 GLM-5.1）
- Codex CLI 已安装：`npm install -g @openai/codex`
- 两个独立终端可用

## 四步协作流程

```
Terminal 1 (Claude Code)          Terminal 2 (Codex CLI)
========================          =====================

Step 1: PLAN
  /plan mode
  生成 plans/<feature>.md
         │
         ▼
Step 2: QA REVIEW ───────────────→ 读取 plans/<feature>.md
                                    在 plan 中插入
                                    "Codex Finding" 标记
                                    不修改原始 phase
         │
         ▼
Step 3: IMPLEMENT
  按 plan 逐 phase 执行
  test gates at each phase
         │
         ▼
Step 4: VERIFY ──────────────────→ 验证实现与 plan 一致
                                    检查测试覆盖率
                                    输出验证报告
```

## 各步骤详细说明

### Step 1: Claude Code Plan

```bash
# Terminal 1
# 进入 plan mode，描述需求
# 输出: plans/<feature-name>.md
```

Plan 应包含：
- Phase 划分（每 phase 有 test gate）
- 文件变更清单
- 依赖关系
- 验证标准

### Step 2: Codex QA Review

```bash
# Terminal 2
codex "Review the plan in plans/<feature>.md against the actual codebase.
Insert intermediate phases with 'Codex Finding' headings where the plan
misses edge cases or dependencies. Do NOT rewrite existing phases."
```

Codex 的角色：**独立视角的 QA 审查**，补充 Claude 可能遗漏的场景。

### Step 3: Claude Code Implement

```bash
# Terminal 1 (new session)
# 按 plan 逐 phase 执行
# 每个 phase 完成后运行对应测试
```

### Step 4: Codex Verify

```bash
# Terminal 2 (new session)
codex "Verify that the implementation matches the plan in plans/<feature>.md.
Check: 1) All phases implemented 2) Test gates passed 3) No scope creep"
```

## Orchestration Workflow — Command→Agent→Skill 调用链

```
用户指令 (/command)
    │
    ├── Command（入口，用户交互）
    │     │
    │     ├── Agent Tool → Agent（带预载 Skill）
    │     │     └── 返回数据
    │     │
    │     ├── Skill Tool → Skill（独立执行）
    │     │     └── 产出文件
    │     │
    │     └── 汇总结果
    │
    └── 输出给用户
```

**两种 Skill 模式**：
1. **Agent Skill（预载）**：Skill 内容注入 Agent 上下文，Agent 按指令执行
2. **Direct Skill（直接调用）**：通过 Skill tool 独立调用

## 当前项目中的 Orchestration 实例

| Command | Agent | Skill | 数据流 |
|---------|-------|-------|--------|
| `/graphify-workflow` | Explore | graphify query | Command 分配 → Agent 搜索 → Skill 操作 |
| 代码变更完成 | test-runner | 测试步骤 | Agent 执行 → 返回结果 |
| 任务完成 | doc-writer | 文档格式 | Agent 读取规则 → Skill 格式化 |

## 适用场景

| 场景 | 使用方式 |
|------|----------|
| 新功能开发（>5 文件） | 完整 4 步流程 |
| Bug 修复 | Claude 直接修复 + Codex 验证 |
| 重构（>10 文件） | Claude plan + Codex QA + Claude 执行 |
| 快速实验 | 仅 Claude，不涉及 Codex |

## Codex CLI 配置指南（AgentRouter + wire_api = "responses"）

### 前提条件

- Codex CLI 已安装：`npm install -g @openai/codex`（当前版本 0.125.0）
- AgentRouter Token 或 OpenAI API Key

### 配置与 Claude Code 零冲突

| 工具 | 配置目录 | API Key 文件 | 环境变量 |
|------|----------|-------------|----------|
| Codex CLI | `~/.codex/` | `~/.codex/auth.json` | `AGENT_ROUTER_TOKEN` |
| Claude Code | `~/.claude/` | `.claude.json` + `.env` | 独立环境变量 |

两者配置完全独立，可在不同终端同时使用。

### wire_api 变更说明（2026-04-27）

Codex CLI v0.125.0 **移除了 `wire_api = "chat"` 支持**，只支持 `wire_api = "responses"`。这意味着：

- 旧配置 `wire_api = "chat"` 会报错：`wire_api = "chat" is no longer supported`
- 必须使用支持 **Responses API** 的 API 代理或 OpenAI 直连
- 智谱 GLM-5.1 的 `/v1/chat/completions` 兼容端点**不再可用**（仅支持 chat completions，不支持 responses）

**当前方案**：使用 AgentRouter（`agentrouter.org`）作为 API 代理，它支持 Responses API 并可路由到多种模型。

### 配置步骤

**Step 1: 创建配置目录和文件**

```bash
mkdir -p ~/.codex
```

**Step 2: 创建 `~/.codex/config.toml`**

```toml
model = "gpt-5.1-codex-max"
model_provider = "openai-chat-completions"
preferred_auth_method = "apikey"
model_reasoning_effort = "high"

[model_providers.openai-chat-completions]
name = "OpenAI using Chat Completions"
base_url = "https://agentrouter.org/v1"
env_key = "AGENT_ROUTER_TOKEN"
wire_api = "responses"
query_params = {}
stream_idle_timeout_ms = 300000
```

**Step 3: 创建 `~/.codex/auth.json`**

```json
{
  "AGENT_ROUTER_TOKEN": "your-agent-router-token-here"
}
```

**Step 4: 验证配置**

```bash
codex --version
codex "Hello, test connection"
```

### 关键配置说明

| 参数 | 值 | 说明 |
|------|------|------|
| `model_provider` | `"openai-chat-completions"` | 引用 `[model_providers.openai-chat-completions]` 节 |
| `model` | `"gpt-5.1-codex-max"` | AgentRouter 支持的模型 ID |
| `wire_api` | `"responses"` | **v0.125.0+ 必须用 `"responses"`** |
| `base_url` | `https://agentrouter.org/v1` | AgentRouter API 代理端点 |
| `env_key` | `"AGENT_ROUTER_TOKEN"` | auth.json 中对应的 key 名 |

### Git Proxy 配置（Windows 代理环境必需）

本机有系统 HTTP_PROXY，git 操作需绕过代理才能访问 GitHub：

```bash
# SSH→HTTPS 重写
git config --global url."https://github.com/".insteadOf "git@github.com:"

# GitHub 不走代理
git config --global http.https://github.com/.proxy ""
git config --global https.https://github.com/.proxy ""
```

## Codex Plugin for Claude Code（codex-plugin-cc）

### 安装

OpenAI 官方发布的 Claude Code 插件，可在 Claude Code 中直接调用 Codex 功能。

```bash
# 在 Claude Code 终端中运行
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

`/codex:setup` 会检查本地 Codex CLI 安装和认证状态。

### 可用命令

| 命令 | 功能 | 是否修改代码 |
|------|------|------------|
| `/codex:review` | 代码审查（对比分支差异） | 只读 |
| `/codex:adversarial-review` | 对抗性审查（质疑设计方向、竞态条件） | 只读 |
| `/codex:rescue` | 把任务甩给 Codex 执行（唯一会改代码的命令） | **会修改** |
| `/codex:status` | 查看后台任务状态 | 只读 |
| `/codex:cancel` | 取消后台任务 | 只读 |
| `/codex:result` | 获取任务结果 + session ID | 只读 |

### 典型用法

```
# 后台审查当前改动
/codex:review --background

# 对抗性审查：质疑缓存策略和竞态条件
/codex:adversarial-review challenge whether this was the right caching design

# 让 Codex 修复失败的测试
/codex:rescue fix the failing test with the smallest safe patch

# 查看后台任务进度
/codex:status
```

### Review Gate（可选）

开启后每次 Claude 回复前自动触发 Codex 审查：

```bash
/codex:setup --enable-review-gate   # 开启
/codex:setup --disable-review-gate  # 关闭
```

### 与独立 Codex CLI 的关系

| 维度 | 独立 Codex CLI | codex-plugin-cc |
|------|---------------|-----------------|
| 运行方式 | Terminal 2 独立终端 | Claude Code 内嵌 |
| 交互方式 | 命令行对话 | 斜杠命令 |
| 配置来源 | `~/.codex/config.toml` | 复用本地 Codex 安装 |
| 适用场景 | 四步协作流程的独立审查 | 快速审查 + rescue 修复 |
| 模型 | 由 config.toml 指定 | 同左 |

## 已知限制

| 限制 | 说明 |
|------|------|
| `wire_api = "chat"` 已废弃 | codex-cli v0.125.0+ 只支持 `"responses"` |
| 智谱 GLM 直连不再可用 | 智谱仅支持 chat completions，需通过 AgentRouter 等支持 Responses API 的代理 |
| Codex 专有功能可能受限 | reasoning summaries、tool calls 等高级功能取决于 API 代理支持 |
| codex-plugin-cc 需要 OpenAI 认证 | 插件使用本地 Codex 安装，需 Codex CLI 已认证 |

## ChangeLogs

- [2026-04-27] **wire_api 重大变更**：`"chat"` 已废弃，改为 `"responses"` + AgentRouter；新增 codex-plugin-cc 插件集成（6 个斜杠命令）；新增 Git proxy 绕过配置
- [2026-04-26] 新增 GLM-5.1 + Codex CLI 配置指南（TOML 配置、auth.json、隔离说明、已知限制）
- [2026-04-26] Initial: Cross-model 4 步流程 + Orchestration 调用链
