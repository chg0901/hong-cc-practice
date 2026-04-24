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

## GLM-5.1 + Codex CLI 配置指南

### 前提条件

- Codex CLI 已安装：`npm install -g @openai/codex`
- 智谱 API Key 已获取（[open.bigmodel.cn](https://open.bigmodel.cn)）

### 配置与 Claude Code 零冲突

| 工具 | 配置目录 | API Key 文件 | 环境变量 |
|------|----------|-------------|----------|
| Codex CLI | `~/.codex/` | `~/.codex/auth.json` | `ZHIPU_API_KEY` |
| Claude Code | `~/.claude/` | `.claude.json` + `.env` | 独立环境变量 |

两者配置完全独立，可在不同终端同时使用。

### 配置步骤

**Step 1: 创建配置目录和文件**

```bash
mkdir -p ~/.codex
```

**Step 2: 创建 `~/.codex/config.toml`**

```toml
model_provider = "zhipu"
model = "glm-5.1"
model_reasoning_effort = "high"

[model_providers.zhipu]
name = "Zhipu AI"
base_url = "https://open.bigmodel.cn/api/paas/v4/"
wire_api = "chat"
env_key = "ZHIPU_API_KEY"
preferred_auth_method = "apikey"
```

**Step 3: 创建 `~/.codex/auth.json`**

```json
{
  "ZHIPU_API_KEY": "your-zhipu-api-key-here"
}
```

**Step 4: 验证配置**

```bash
codex --version
codex "Hello, test GLM-5.1 connection"
```

### 关键配置说明

| 参数 | 值 | 说明 |
|------|------|------|
| `model_provider` | `"zhipu"` | 自定义 provider 名称，引用 `[model_providers.zhipu]` 节 |
| `model` | `"glm-5.1"` | 模型 ID，需与智谱控制台一致 |
| `wire_api` | `"chat"` | **必须用 `"chat"`**，不能用 `"responses"`（智谱不支持 Codex 专有协议） |
| `base_url` | `https://open.bigmodel.cn/api/paas/v4/` | 智谱 OpenAI 兼容端点 |
| `env_key` | `"ZHIPU_API_KEY"` | auth.json 中对应的 key 名 |

### 已知限制

| 限制 | 说明 |
|------|------|
| `wire_api = "responses"` 不可用 | 智谱不支持 Codex 专有的 Responses API，必须用 `chat` |
| Codex 专有功能可能缺失 | reasoning summaries、tool calls 等高级功能 GLM-5.1 可能不支持 |
| 模型名需确认 | 可能是 `glm-5.1` 或 `glm-5.1-preview`，以智谱控制台为准 |

### 替代方案：使用 `OPENAI_BASE_URL` 环境变量

如果不修改 `config.toml`，可通过环境变量快速切换：

```bash
# 在 PowerShell 中（Terminal 2）临时设置
$env:OPENAI_API_KEY = "your-zhipu-api-key"
$env:OPENAI_BASE_URL = "https://open.bigmodel.cn/api/paas/v4/"
codex -m glm-5.1 "your prompt here"
```

注意：此方式可能需要 `wire_api = "chat"` 配置配合。

## ChangeLogs

- [2026-04-26] 新增 GLM-5.1 + Codex CLI 配置指南（TOML 配置、auth.json、隔离说明、已知限制）
- [2026-04-26] Initial: Cross-model 4 步流程 + Orchestration 调用链
