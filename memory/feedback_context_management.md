---
name: Context management cache-aware decisions
description: Prompt Cache 策略和 6 个上下文管理指令的最佳实践，来自 AI 方寸山文章
type: feedback
originSessionId: 090b4035-4632-427e-967e-c4922c75ce3d
---
## 缓存感知决策模式

**规则**：缓存还热、任务没换 → 继续聊（默认选项）；缓存过期、任务切换、噪音太多 → 果断重开。

**Why**: 每次 /clear 重建 ~50K token 基础设施要全价；活跃会话中只付 1/10。一次 /clear 的沉没成本够在老会话里继续聊小半天。

**How to apply**: 做完一个小任务不 /clear。只有满足"任务切换/闲置 >1hr/噪音太多"之一时才开新会话。

## /rewind 是最高频的纠错工具

**规则**：方案失败时，先 summarize → rewind → 带着总结重下指令。不要直接说"换方案"。

**Why**: rewind 保留回滚点之前的缓存（前面读过的文件 token 没白花），清除失败方案的脏上下文。直接"换方案"会让失败的中间输出继续污染 Claude 的注意力。

**How to apply**: 每次方案失败时的操作序列：(1) 让 Claude summarize (2) /rewind (3) 用总结作为新 prompt 开头。

## /compact 主动优于被动

**规则**：在阶段性完成、缓存还热时主动 /compact，不要等自动压缩触发。

**Why**: 自动压缩可能在最不合适的时刻触发（长调试会话后），此时 Claude 自己也"最没脑子"，可能把重要信息当成次要内容砍掉。主动压缩可以带参数控制保留什么。

**How to apply**: 完成一个阶段后立即 `/compact focus on X, drop Y`。

## 最便宜的 token 是没进上下文的 token

**规则**：长内容（日志、数据文件、大段代码）只给路径，不粘贴内容。

**Why**: 让 Claude 自己用 Read/grep 提取相关几行，比粘贴一万行日志高效得多。未进入上下文的 token = 零成本。

**How to apply**: 永远给文件路径而非粘贴内容。例如："看 logs/app.log 里的 ERROR"而非粘贴日志。
