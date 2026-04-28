---
name: Delete files separately commit
description: 删除文件可以自行操作，但必须作为独立 commit 单独提交，不能混入其他变更
type: feedback
---

**规则：删除文件必须作为独立 commit 单独提交**

Why: 删除操作难逆，混在功能 commit 中不易发现和回滚。独立 commit 方便 git revert 只撤销删除而不影响功能代码。

How to apply:
- 删除临时文件/旧文件时，可以自行 `rm` 操作，无需询问
- 但提交时必须单独 `git commit`，不与功能代码、文档更新混在同一个 commit
- commit message 格式：`chore: 删除临时文件（说明原因）`
