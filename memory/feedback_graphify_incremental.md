---
name: Graphify incremental rebuild strategy
description: 基于 git diff 的增量图谱更新策略 — 跳过非代码变更，SHA256 cache 文件级增量，决策表按变更量选择策略
type: feedback
originSessionId: c1f1d1ab-2767-4c16-a21f-957bce2e31cc
---
Graphify 重建不需要每次都全量执行。利用 git diff 判断是否有代码文件变更，可以跳过纯文档/配置变更。

**Why:** `_rebuild_code()` 虽然有 SHA256 cache，但仍需扫描全部文件计算 hash。对于纯 markdown/config 变更，完全不需要调用。

**How to apply:**
- 先 `git diff --name-only HEAD -- '*.py' '*.js' '*.ts'` 检查代码文件变更数
- 0 文件变更 → 跳过重建
- 1-20 文件 → `_rebuild_code()` 增量（SHA256 cache 自动处理）
- 20+ 文件或 merge/rebase 后 → 全量重建 + 重新生成可视化
- post-commit hook 已内置代码文件过滤，只有代码变更才触发
- 详见 graphify.md rule "Git-Status-Based 增量更新" 章节
