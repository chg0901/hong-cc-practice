# User Manual Version & Update Rules — 用户手册版本管理

## 版本号规范

| 版本类型 | 格式 | 触发条件 | 示例 |
|----------|------|----------|------|
| 主版本 | V{x}.0 | 架构重构、新增完整模块章节 | V2.0（新增授权系统章节） |
| 次版本 | V{x}.{y} | 内容更新、截图替换、章节重写 | V1.3（授权-登录顺序调整） |

## 版本更新触发条件

以下场景**必须**更新 `docs/user_manual/README.md` 中的版本号和更新日志：

| 触发条件 | 版本变化 | 说明 |
|----------|----------|------|
| 新增章节文件 | 次版本 +1 | 如新增授权许可独立章节 |
| 重写某章节的整节内容 | 次版本 +1 | 如 00_overview 第 7 节重写 |
| 替换 3+ 张截图 | 次版本 +1 | 批量截图更新 |
| 修正错别字或微调描述 | 不变 | 仅更新日期或在已有条目追加说明 |
| 修正操作步骤或界面描述 | 次版本 +1 | 操作流程变更 |

## 更新检查清单

每次修改 `docs/user_manual/` 下的 `.md` 文件后，必须执行：

1. **README.md 版本号**：检查 `**版本**: V{x}.{y}` 是否需要递增
2. **README.md 发布日期**：更新为当日日期
3. **README.md 更新日志**：新增 `### V{x}.{y} — YYYY-MM-DD` 条目
4. **README.md 目录导航**：如果新增/删除了章节文件，更新目录表
5. **交叉引用**：检查其他章节是否引用了被修改的内容，更新引用

## 文件结构

```
docs/user_manual/
├── README.md           ← 版本号 + 目录 + 更新日志
├── 00_overview.md      ← 各章节文件
├── 01_quickstart.md
├── ...
├── 25_troubleshooting.md
└── screenshots/        ← 截图目录（按章节分子目录）
    ├── license/
    ├── login/
    ├── home/
    └── ...
```

## 截图更新规则

- 截图文件名格式：`{area}_{description}_{YYYYMMDD}.png`（仅在同名截图冲突时加日期）
- 截图存放路径：`docs/user_manual/screenshots/{chapter_area}/`
- 截图引用格式：`![描述](screenshots/{area}/{filename}.png)` + 换行 + `- 图：描述`
- 替换截图时直接覆盖同名文件，不另存为新版本

## 与其他规则的关系

| 规则 | 管辖范围 | 互补关系 |
|------|----------|----------|
| `context-hygiene.md` | CLAUDE.md 精简 + README 联动 | 本文件管用户手册版本，context-hygiene 管 CLAUDE.md 体积 |
| `work-summary-rules.md` | 工作日志格式 | 手册更新必须在 work_summary 中记录 |
| `manual-testing.md` | UI 手动测试项 | 截图更新通常伴随 UI 变更测试 |

## ChangeLogs

- [2026-04-25] Initial: 版本号规范、更新触发条件、检查清单、截图更新规则
