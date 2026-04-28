# Extension Onboarding Rule — 安装新 Skills/Agents 后必执行的文档整理

## Trigger

当执行以下操作后，**必须**执行本规则中的整理流程：

- `npx skills add ...` 安装第三方 skills
- 手动创建 `.claude/skills/*/SKILL.md` 新建项目 skills
- 手动创建 `.claude/agents/*.md` 新建 agents
- `npx skills remove ...` 或手动删除 skills/agents
- 从 `npm/pip` 安装新的 MCP server 插件

## 整理流程（mandatory）

### Step 1: 读取新内容

```
对新安装的每个 skill/agent：
1. 读取 SKILL.md 或 agent .md 文件
2. 理解其功能、触发方式、依赖资源
3. 识别与其他 skill/agent/rules 的潜在冲突或协作点
```

### Step 2: 更新 Rules（至少更新以下文件）

| 文件 | 更新内容 |
|------|---------|
| `.claude/rules/subagents.md` | L3 Skills 计数、Skills 全景图表、触发方式 |
| `.claude/rules/config-review.md` | GitHub 同步范围（包含/排除清单）、skills 计数 |
| `.claude/rules/mcp-servers.md` | 如涉及新 MCP server，更新可用服务器列表 |
| `~/.claude/rules/search-workflow.md` | 如涉及搜索工具变更，更新工具目录和回退链（全局 rule） |
| `CLAUDE.md` | 如涉及新开发命令或测试流程，更新对应章节 |

### Step 3: 更新 Memory

| 操作 | 条件 |
|------|------|
| 新建 `memory/reference_xxx.md` | 当新 skill 引入了重要外部资源或工作流 |
| 更新 `memory/MEMORY.md` 索引 | 当新建或删除了 memory 文件 |
| 更新现有 memory | 当新 skill 改变了已有工作流（如 code-review 替代了原有审查方式） |

### Step 4: 去重与冲突检查

```bash
# 检查新 skill 的触发词是否与已有 skill 冲突
grep -rn "触发词/trigger keyword" .claude/skills/*/SKILL.md

# 检查新 skill 的功能是否与已有 rules 重复
grep -rn "相关关键词" .claude/rules/*.md
```

**冲突处理原则**：
- 第三方 skill 与项目 rules 功能重复 → 优先使用项目 rules（更贴合项目上下文）
- 两个第三方 skill 触发词冲突 → 在 description 中明确区分，或设置 `disable-model-invocation: true`
- 第三方 skill 与项目 skill 功能重叠 → 保留两者，通过触发词区分

### Step 5: GitHub 同步

```bash
bash scripts/sync_claude_config.sh --push
```

仅同步项目自建的 rules/skills/agents，**不同步**第三方 symlink。

### Step 6: 记录到 Work Summary

在当日 `docs/work_summary_YYYYMMDD.md` 中记录：

```markdown
## N、扩展安装整理

### 新安装内容
| 名称 | 来源 | 类型 | 功能 |
|------|------|------|------|
| xxx | sanyuan-skills | skill | ... |

### 文档更新
| 文件 | 更新内容 |
|------|---------|
| subagents.md | Skills 全景图新增 x 条 |
| config-review.md | 同步范围更新 |
| extension-onboarding.md | 新建规则 |

### 冲突检查结果
- 无冲突 / 发现冲突及处理方式
```

## Skills 当前全景（快速参考）

### 项目自建 Skills（`.claude/skills/`，纳入 Git 同步）

graphify-workflow, interaction-check, visual-check, manual-review, long-screenshot, doc-trim, ps-script-dev

### 第三方 Skills（git clone/npm 安装，不同步到 GitHub）

book2skills, create-colleague, context-research, baidu-search, excalidraw-diagram-generator

### 第三方 Skills（symlink，npx 管理，不同步到 GitHub）

book-study, code-review-expert, sigma, skill-forge, wiki-ingest, fireworks-tech-graph, web-access

## ChangeLogs

- [2026-04-22] 新增 excalidraw-diagram-generator（github/awesome-copilot，手动 clone 安装，第三方 5 个）
- [2026-04-20 13:35:00] 新增 baidu-search 到第三方 skills（4个）；修正 symlink 列表补充 fireworks-tech-graph
- [2026-04-16 16:35:00] 新增 web-access 到第三方 symlink skills（7个）；新增 search-workflow.md 到 Step 2 更新清单
- [2026-04-15 14:30:00] 新增 fireworks-tech-graph 到第三方 symlink skills（7个）
- [2026-04-15 10:30:00] Initial: 安装新 skills/agents 后的 6 步整理流程、冲突检查、同步范围
