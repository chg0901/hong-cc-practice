---
name: doc_deletion_rules
description: When deleting documentation content, must explain reason, cause, and impact. Update related Claude tools and docs.
type: feedback
---

## Rule: 删除文档内容必须说明理由

删除或大幅删减文档内容时（rules/memory/skills/agents 中的任何内容），必须：

1. **说明理由**：为什么删除（去重、过时、错误？）
2. **说明原因**：导致删除的根本原因
3. **说明影响**：删除后哪些功能/引用会受影响
4. **交叉更新**：检查其他引用该内容的文件是否需要同步更新（CLAUDE.md、agents、skills、memory）

**Why**: 曾出现过删除 rules 内容后，引用该规则的 agents 和 skills 未同步更新，导致引用断裂。

**How to apply**: 每次删除文档内容前，grep 检查所有引用，删除后更新所有相关文件。
