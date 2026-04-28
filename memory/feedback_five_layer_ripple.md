---
name: Five-layer consistency ripple pattern
description: 五层体系中一个变更（如新增 skill）会波及 4+ 层文件的涟漪效应，以及 config-review 交叉引用检查的范围盲区
type: feedback
originSessionId: c1f1d1ab-2767-4c16-a21f-957bce2e31cc
---
# 五层一致性涟漪问题

**规则**：五层体系中的变更必须在所有相关层同步更新，否则产生过时引用。一个变更平均影响 4-6 个文件。

**Why**：五层体系（Hooks/Rules/Skills/Subagents/Teams）的配置分散在 `.claude/` 目录下的不同文件中，但还有大量引用散布在 `docs/` 和 `mermaid/` 目录。新增/删除/修改一个组件时，计数、描述、流程图都可能需要同步更新。

**How to apply**：

每次新增/删除/修改 skill、rule、agent、hook 后，检查以下文件是否需要更新：

| 文件 | 检查内容 |
|------|----------|
| `subagents.md` | L2/L3 计数、全景图表 |
| `config-review.md` | 同步范围（包含/排除清单） |
| `extension-onboarding.md` | 项目自建/第三方 skills 列表 |
| `MEMORY.md` | 索引计数 |
| `reference_skills_ecosystem.md` | Skill→Task 速查表 |

**容易被遗漏的范围**（不在 `.claude/` 目录中）：

| 文件 | 典型残留 |
|------|----------|
| `docs/claude_code_extensibility_tutorial.md` | Hook 描述、Rules 计数、Skills 列表 |
| `docs/claude_rules_skills_memory_guide.md` | 五层规模表、Hooks 配置表 |
| `mermaid/text/extensibility-*.mmd` | 流程图中的分支（如已删除 hook 残留） |
| `docs/graphify_usage_guide.md` | 硬编码统计数据 |

**案例**：04-22 graphify 五层审计发现 7 处过时引用（3 处硬编码数据 + 4 处已删除 Grep|Glob hook 残留），分布在 `.claude/rules/`、`.claude/skills/`、`docs/`、`mermaid/` 四个目录中。

**GRAPH_REPORT.md 作为唯一权威数据源**：graphify 统计数据（nodes/edges/communities）不得硬编码到其他文件，统一使用指针 "见 GRAPH_REPORT.md"。消除 DRY 违规。
