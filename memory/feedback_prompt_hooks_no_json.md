---
name: Prompt hooks must not require JSON output
description: Claude Code prompt hooks 不需要 LLM 返回 JSON；加 "Respond with JSON only" 会触发 JSON validation failed 错误。这是 2026-04-15 的实际踩坑。
type: feedback
originSessionId: 78f5f996-566c-4bda-9a23-947cb947735f
---
**规则**：Claude Code 的 `type: "prompt"` hooks **绝不能**在 prompt 末尾加 `"Respond with JSON only."` 或要求 LLM 返回 JSON 格式输出。

**Why**: Claude Code 的 prompt hooks 框架有自己的 LLM 响应解析逻辑，不需要 JSON 输出。加上 JSON 约束后，框架会尝试将 LLM 输出解析为 JSON，导致 "JSON validation failed" 错误。原始的自然语言 prompt（如 `[Safety Check] Before executing...\nIf matched, STOP and ask user for confirmation. Otherwise proceed.`）是正确格式。

**How to apply**:
1. prompt hooks 的 prompt 应该用自然语言描述检查逻辑，以 `Otherwise proceed.` 结尾
2. **不要**加 `Respond with JSON only`、`Return JSON` 等约束
3. 如果需要确定性输出，应改用 `type: "command"` hooks（shell 脚本 + exit code）
4. 修改 hooks 前先用 `git diff` 对比原始版本，确认不破坏已有功能

**反面案例**（2026-04-15）：
- 原始 prompt: `[Safety Check] Before executing this Bash command, check for dangerous patterns:\n- rm -rf /...\nIf matched, STOP and ask user for confirmation. Otherwise proceed.` — **正常工作**
- 精简后: `Block if the Bash command contains: rm -rf /, ...\nRespond with JSON only.` — **JSON validation failed**

**排查方法**：
1. `git log --oneline -- .claude/settings.json` 找到修改的 commit
2. `git diff <old_commit>..HEAD -- .claude/settings.json` 对比差异
3. 恢复原始 prompt 即可修复
