# Boris Cherny Tips — Claude Code 作者实战建议

## 来源

提炼自 [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) 中 Boris Cherny（Claude Code 作者）的多次分享（2026-01~04）。

## 精选 Tips（与现有规则去重后）

### 1. 复杂任务先 Plan，争取 1-shot 实现

把精力倾注在 plan 上，让 Claude 一次通过实现。好 plan 的标志：
- 文件路径、函数名、数据流都已明确
- 已考虑边界情况和错误处理
- 有验证标准（怎么确认做对了）

**与现有规则的关系**：superpowers:writing-plans 已覆盖流程，本 tip 强调"plan 质量 = 实现质量"。

### 2. 每次纠错后更新 CLAUDE.md

当 Claude 犯了错被纠正后，以"更新 CLAUDE.md，以后不再犯这个错误"结束。

Claude 擅长为自己的错误写规则。持续迭代直到错误率明显下降。

**与现有规则的关系**：friday-review 管周级知识抽象，本 tip 管实时纠错。两者互补。

### 3. 给 Claude 验证工作的手段

Claude 能验证自己的工作，质量会提升 2-3 倍。验证方式因任务而异：
- 后端：运行服务器 + API 测试
- 前端：Playwright 截图 + 交互测试
- 数据库：query 验证数据完整性

**与现有规则的关系**：testing.md 管测试规范，本 tip 强调"每次变更都要有验证闭环"。已有 verification-before-completion skill。

### 4. 方案失败后不要原地换方案

当方案 A 失败：
1. 先 `summarize from here` 总结失败原因
2. `/rewind` 回到决策点之前
3. 带着总结作为新 prompt 开方案 B

不要在方案 A 的失败上下文中直接试方案 B，那样方案 A 的噪音会干扰方案 B。

**与现有规则的关系**：context-management.md 已有 rewind 用法，本 tip 强调"失败后先总结再回滚"。

### 5. 重复操作自动化为 Skill

如果某个操作一天内做超过一次，就把它变成 skill 或 command。

检查清单：
- 是否有重复的 prompt 模式？
- 是否每次都在手动组合相同的步骤？
- 是否可以用 `allowed-tools` 减少权限确认？

**与现有规则的关系**：本项目已有 23 个 skills，本 tip 提醒持续观察自动化机会。

### 6. Subagent 保持主会话上下文干净

将以下工作交给 subagent：
- 只需要结论不需要过程的搜索/分析
- 大量文件读取的深度调研
- 需要独立 200k context 的大范围重构

**注意**：子代理缓存窗口仅 5 分钟（主会话 1 小时），适合短任务不适合长分析。

**与现有规则的关系**：subagents.md 已有详细规范，本 tip 强调"隔离噪音保护主上下文"。

### 7. 不要 micromanage — 说目标不说方法

说"修复失败的 CI 测试"而不是"先读这个文件，然后改那个函数，然后运行那个命令"。

Claude 比你更擅长选择工具和步骤。给目标，信任过程。

**例外**：当你明确知道某个方法更好时（如"用 bcrypt 不要用 MD5"），直接指定。

### 8. AI 代码清理检查清单（来自 OMC ai-slop-cleaner）

每次实现完成后，检查以下 5 项常见 AI 代码问题：

| 检查项 | 问题描述 | 检测方法 |
|--------|----------|----------|
| 过度抽象 | 只有一处调用的函数不需要单独抽象 | grep 函数名，确认调用次数 > 1 |
| 冗余注释 | 代码本身已清晰的不需要注释解释 | 检查注释是否只是重复代码逻辑 |
| 过度防御 | 不会发生的场景不需要 try-catch | 检查 catch 块是否处理不可能发生的错误 |
| 不必要泛型 | 只为当前需求服务的不需要设计为可扩展 | 检查是否有未使用的配置参数或策略模式 |
| AI 痕迹 | "Let me...", "I'll...", placeholder TODO 注释 | grep -i "let me\|i'll\|TODO.*implement\|FIXME" |

**触发时机**：实现完成后、commit 前。与 deviation-handling.md 规则 4（架构变更需确认）互补 — 规则 4 管"该不该做"，本清单管"做完了是否干净"。

## 与现有 rules 的去重确认

| Tip | 已有覆盖 | 新增价值 |
|-----|---------|---------|
| Plan 先行 | writing-plans skill | 强调 plan 质量 |
| 纠错更新 CLAUDE.md | friday-review | 实时 vs 周级 |
| 验证闭环 | testing.md + verification skill | 2-3x 质量提升 |
| 失败后 rewind | context-management.md | 先总结再回滚 |
| 自动化 Skill | 23 skills 已有 | 持续观察提醒 |
| Subagent 隔离 | subagents.md | 强调保护主上下文 |
| 不要 micromanage | 无 | 目标导向 |

## ChangeLogs

- [2026-04-26] 新增 Tip 8: AI 代码清理检查清单（5 项：过度抽象、冗余注释、过度防御、不必要泛型、AI 痕迹），来自 OMC ai-slop-cleaner
- [2026-04-26] Initial: 7 条精选 tips（去重后），来自 Boris Cherny 4 次分享
