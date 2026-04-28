# Token Budget Rule — Token 预算监控与精简规则

## 背景

GLM-5.1 上下文约 500K token（Claude Code 1M 的一半）。固定基础设施（Rules + CLAUDE.md + MEMORY.md + 系统提示）约占 22%，每多一行 Rule 都增加全量加载成本。

## 硬性约束

| 指标 | 上限 | 当前值 | 状态 |
|------|------|--------|------|
| 全局 Rules 单文件 | 300 行 / 15KB | max 244 行 | OK |
| 项目 Rules 单文件 | 400 行 / 20KB | mcp-servers ~20KB | **警告** |
| Rules 总 token | 65,000 tokens | ~56,791 tokens | OK |
| 固定开销占总预算 | 30% | 22% | OK |
| Memory 单文件 | 100 行 | max 96 行 | OK |
| MEMORY.md 索引 | 50 行 | 43 行 | OK |

## 精简触发条件

以下任一条件满足时，必须执行精简：

1. **单文件超限**：任何 Rule 文件超过 300 行或 15KB
2. **总量超限**：Rules 总 token 超过 65,000
3. **固定占比超限**：固定开销超过总预算 30%
4. **重复检测**：发现 Rules/Memory/Skills 间内容重复率 > 30%
5. **季度审计**：每季度第一天执行 `/context-audit`

## 精简策略

### Rules 精简

| 策略 | 适用场景 | 操作 |
|------|---------|------|
| 拆分到 Memory | 详细目录/列表/速查表 | 保留决策树，列表移到 `reference_*.md` |
| 拆分到 Skill | 操作步骤/Checklist > 5 步 | 保留规则约束，步骤移到 SKILL.md |
| 拆分到全局 | 通用方法论不引用项目路径 | 移到 `~/.claude/rules/` |
| 合并同类 | 两个 Rule 内容重叠 > 30% | 合并保留更全面的那个 |
| 删除过时 | ChangeLog 中标记过时的内容 | 删除并记录理由 |

### Memory 精简

| 策略 | 适用场景 | 操作 |
|------|---------|------|
| 合并 reference | 同类外部资源指针 > 3 个 | 合并为 1-2 个综合 reference |
| 删除已覆盖 | Memory 内容已被 Rule 完整覆盖 | 删除 Memory，保留 Rule |
| 归档旧记录 | 超过 60 天的 project memory | 评估是否仍相关 |

## 计数维护

每次新增/删除/移动 Rule 后，必须更新以下文件中的计数：

| 文件 | 更新内容 |
|------|---------|
| `.claude/rules/subagents.md` | L2 Rules 行数 |
| `.claude/rules/config-review.md` | 同步范围表 |
| `memory/MEMORY.md` | 索引计数 |
| `CLAUDE.md` | ChangeLog |

## 与其他规则的关系

| 规则 | 互补关系 |
|------|---------|
| `context-hygiene.md` | context-hygiene 管文档体积，token-budget 管 Rules token 预算 |
| `config-review.md` | config-review 管修改后检查，token-budget 管定期审计 |
| `feedback_rules_no_duplication.md` (Memory) | 防止双重加载，本规则防止单文件膨胀 |

## ChangeLogs

- [2026-04-28] Initial: Token 预算硬性约束、精简触发条件、精简策略、计数维护规则
