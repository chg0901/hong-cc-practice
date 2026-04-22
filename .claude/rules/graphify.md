# Graphify 知识图谱使用规范

## 触发条件

当以下场景出现时，优先考虑使用 `graphify query` 替代纯 grep/glob：

| 场景 | 传统方式 | 推荐 graphify |
|------|---------|-------------|
| 「找 X 相关代码」 | `grep -rn "X" --include="*.py"` | `graphify query "X related code"` |
| 「模块 A 调用了哪些模块」 | 手动 grep 函数定义 | `graphify query "what does module A call"` |
| 「理解 Y 的调用链/上下游」 | 多个 grep 追溯 | `graphify query "Y call chain and dependencies"` |
| 「哪些模块涉及 Z 功能」 | `glob + grep` 组合 | `graphify query "Z feature modules"` |
| 「项目的核心抽象是什么」 | 读代码直觉判断 | 直接查看 God Nodes |

## 使用时机

### 建议使用 graphify 的情况

1. **写作前的踩点阶段**：准备写某主题文档，需要快速了解涉及哪些模块、它们之间什么关系
2. **接手新模块时**：快速理解模块边界和依赖关系
3. **跨模块追溯**：功能横跨多个模块，grep 只知道单文件内容，不知道模块间连接
4. **评估重构影响**：需要了解某模块被哪些其他模块依赖

### 不建议使用 graphify 的情况

1. **精确查找已知关键词**：直接 grep 更高效（如 `grep -n "def foo"`）
2. **读文件理解代码逻辑**：graphify 只返回节点/边，不返回代码内容
3. **修改代码时**：需要读文件内容才能理解实现细节
4. **已知特定文件**：直接 Read 比 graphify 快

## 执行命令

```bash
# 踩点查询（默认 BFS 遍历，2000 token 预算）
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify query "<question>"

# 更深遍历（DFS，追踪特定路径）
... -m graphify query "<question>" --dfs

# 自定义 token 预算
... -m graphify query "<question>" --budget 1500

# 增量重建图谱（AST-only，不需要 LLM，利用 SHA256 cache）
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -c \
  "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"

# 重新生成可视化
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/gen_graph_viz.py

# 查询 Q&A 反馈保存
... -m graphify save-result --question "Q" --answer "A" --nodes N1 N2

# Git hooks 状态检查
... -m graphify hook status
```

## 更新机制

### 三层更新策略

| 层级 | 触发方式 | 机制 | 说明 |
|------|---------|------|------|
| **自动** | git commit / checkout | post-commit/post-checkout hook | 仅当有代码文件变更时触发，SHA256 cache 增量更新 |
| **手动** | `/graphify-workflow update` | `_rebuild_code()` | 大量变更后手动触发，如 merge 后 |
| **提醒** | SessionEnd hook | 提示重建 | 会话结束时有代码变更则提醒 |
| **不触发** | git push（Gitee/GitHub） | 无 | push 是服务端操作，客户端 hook 不在 push 时运行；图谱已在 commit 时更新，push 无需额外操作 |

### Git Hooks（已安装）

```bash
# 检查 hooks 状态
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify hook status
# 输出：post-commit: installed / post-checkout: installed
```

hook 仅在有 `.py .js .ts .go .rs .java .cpp .c` 等代码文件变更时触发，markdown/config 变更不会触发重建。

### 验证更新

更新后检查 `graphify-out/GRAPH_REPORT.md` 的日期和节点数：
```
规模：2589 nodes / 5612 edges / 157 communities
```

## Token 成本意识

| 模式 | 构建成本 | 查询成本 |
|------|---------|---------|
| AST-only（当前） | 0 token | ~2000 token/query |
| Phase 2 Semantic | ~27 子代理 x 45k token | ~2000 token/query |

**警告**：Phase 2 语义提取 token 成本极高（可能超过 $1），仅在确实需要跨模语义关联（如设计决策、文档概念）时才使用。

## 当前图谱状态

```
位置：     graphify-out/graph.json
规模：     2589 nodes / 5612 edges / 157 communities
Token 压缩：~1423x（vs 全量读取）
构建成本：  0 token (AST-only)
缓存：     graphify-out/cache/ (SHA256 content-addressable)
```

图谱基于 tree-sitter AST 提取，涵盖代码文件（.py/.js/.ts 等），不含文档和图片语义。

God Nodes（最核心抽象）：
1. `DeviceIconLibrary` — 设备图标系统
2. `SchematicGenerator` — 组态图生成
3. `DBManager` — 数据库管理

## 可视化增强

### 度加权边粗细

`scripts/gen_graph_viz.py` 生成的交互式 HTML 中，边粗细基于两端节点的平均度：

- 高度连接节点间的边更粗（最大 5px），视觉突出核心路径
- 低度节点间的边更细（最小 1px），减少视觉噪声
- 置信度低的边（非 EXTRACTED）使用虚线 + 低透明度

### 交互式可视化工作流

代码变更后更新可视化：`/graphify-workflow viz`

## 与五层体系集成

| Layer | 机制 | Graphify 角色 |
|-------|------|--------------|
| L1 Hooks | git post-commit/post-checkout | 自动增量重建图谱 |
| L2 Rules | 本文件（graphify.md） | 使用规范、触发条件、更新机制 |
| L3 Skills | graphify-workflow | 操作命令（query/update/viz/hook） |
| L4 Subagents | context-research (fork) | graphify query 可在隔离上下文中使用 |
| L5 Memory | GRAPH_REPORT.md | God nodes + community structure 持久化 |

### 使用流程

```
新会话开始 → 读 GRAPH_REPORT.md 了解全局结构
  |
  v
需要查找代码 → 先 graphify query 获取模块关系 → 再 Read 具体文件
  |
  v
代码变更 → git commit → post-commit hook 自动重建
  |
  v
会话结束 → 确认图谱已更新（如有大量变更，手动 /graphify-workflow update）
```

## ChangeLogs

- [2026-04-22 — 三层更新策略表补充"push 不触发"说明（git push 是服务端操作，客户端 hook 不触发）]
- [2026-04-17 15:30:00] 大幅更新：重建图谱（2589/5612/157）、新增更新机制章节、五层体系集成、修正命令（_rebuild_code 替代不存在的 --update）、安装 git hooks
- [2026-04-14 09:30:00] 新增度加权边粗细说明、可视化工作流、代码变更后更新规则
- [2026-04-13 — Initial: 触发条件、使用时机、执行命令、token 成本](changes/2026-04-13)