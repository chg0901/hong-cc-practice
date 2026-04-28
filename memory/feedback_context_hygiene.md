---
name: Context hygiene and doc-trim workflow
description: CLAUDE.md 精简经验 — 669→107行，Change Log 裁剪策略，README 联动更新，/doc-trim skill 每周二/五执行
type: feedback
originSessionId: c1f1d1ab-2767-4c16-a21f-957bce2e31cc
---
CLAUDE.md 精简必须定期执行，否则几周内从 200 行膨胀到 600+ 行。

**Why:** CLAUDE.md 每次会话全量加载，每多一行都增加基础 token 成本。Change Log 是最大膨胀源（本次占 55%），但 Claude 几乎不需要历史记录来完成当前任务。

**How to apply:**
- 每周二/五执行 `/doc-trim`，检查 CLAUDE.md 行数（上限 200）
- Change Log 只保留最近 3 天，旧条目移到 docs/changelog.md
- 详细内容用指针替代（如 "See testing.md" 替代 20 行测试文件列表）
- 修改项目名称/数据时联动检查 README.md + user_manual/README.md 一致性
- hong-cc-practice README 不绑定项目名，只描述 Claude Code 配置经验
