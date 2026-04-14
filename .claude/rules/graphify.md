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
graphify query "<question>"

# 更深遍历（DFS，追踪特定路径）
graphify query "<question>" --dfs

# 自定义 token 预算
graphify query "<question>" --budget 1500

# 图谱已存在时，手动触发增量更新
graphify --update

# 重建（文件未变化时直接命中缓存）
graphify
```

## Token 成本意识

| 模式 | 构建成本 | 查询成本 |
|------|---------|---------|
| AST-only（当前） | 0 token | ~1983 token/query |
| Phase 2 Semantic | ~27 子代理 × 45k token | ~1983 token/query |

**警告**：Phase 2 语义提取 token 成本极高（可能超过 $1），仅在确实需要跨模语义关联（如设计决策、文档概念）时才使用。

## 当前图谱状态

```
位置：     graphify-out/graph.json
规模：     1,875 nodes / 2,907 edges / 149 communities
Token 压缩：1423x（vs 全量读取）
构建成本：  0 token (AST-only)
缓存：     graphify-out/cache/ (158 个文件，SHA256 content-addressable)
```

图谱基于 tree-sitter AST 提取，仅涵盖 161 个代码文件，不含文档和图片语义。

## 可视化增强

### 度加权边粗细

`scripts/gen_graph_viz.py` 生成的交互式 HTML 中，边粗细基于两端节点的平均度：

```python
avg_deg = (degree[src] + degree[tgt]) / 2
edge_width = max(1, min(5, 1 + 4 * (avg_deg / max_deg)))
```

- 高度连接节点间的边更粗（最大 5px），视觉突出核心路径
- 低度节点间的边更细（最小 1px），减少视觉噪声
- 置信度低的边（非 EXTRACTED）使用虚线 + 低透明度

### 交互式可视化工作流

代码变更后更新可视化：`/graphify-workflow viz`（或手动运行 `scripts/gen_graph_viz.py`）

## 代码变更后更新图谱

代码变更（新增/删除函数、修改 import、创建新文件）后，应运行 `/graphify-workflow update` 增量更新图谱，保持节点/边与代码同步。

## ChangeLogs

- [2026-04-14 09:30:00] 新增度加权边粗细说明、可视化工作流、代码变更后更新规则
- [2026-04-13 — Initial: 触发条件、使用时机、执行命令、token 成本](changes/2026-04-13)
