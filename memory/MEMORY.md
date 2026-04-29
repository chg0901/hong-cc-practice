# Memory Index

## Feedback (workflow preferences)
- [Context management cache-aware decisions](feedback_context_management.md) — Prompt Cache 策略、6 个上下文管理指令、rewind 进阶、compact 主动 vs 被动
- [Windows environment gotchas](feedback_windows_env.md) — Three terminals, NO_PROXY, startup lessons, .env + load_dotenv 陷阱
- [Password masking](password_masking.md) — First 2 chars + "(还有N位)" format; `.env` for real values
- [Function deletion checks](feedback_function_deletion.md) — Grep callers, backup, verify, record in work_summary
- [Delete files separately commit](feedback_delete_commit.md) — 删除文件自行操作但必须独立 commit，不混入其他变更
- [Task-batch commit checkpoint](feedback_commit_checkpoint.md) — 每完成一个任务批次，自动日志记录 + git commit + push + GitHub sync
- [Mermaid 对比文档规则](feedback_mermaid_comparison.md) — 相对路径 `../`、HTML table 并列、旧版从 git 恢复、classDef 颜色必须
- [Agent memory mechanism best practices](feedback_memory_mechanism.md) — Instructional vs learned memory, layered management, rule writing principles
- [Doc deletion rules](feedback_doc_deletion.md) — 删除文档内容必须说明理由、原因、影响，交叉更新引用
- [No Claude co-author](feedback_no_claude_coauthor.md) — Never show Claude as collaborator/co-author in git commits
- [Screenshot rules](feedback_screenshot_rules.md) — 不调viewport、长页面多张滚动截图、必须包含子标题
- [Mermaid display ratio](feedback_mermaid_display.md) — 在线文档中 mermaid 图与截图显示差异处理；max-height 600px + 点击放大
- [Prompt hooks no JSON](feedback_prompt_hooks_no_json.md) — prompt hooks 不能要求 JSON 输出；精简加 JSON 约束会触发 validation failed
- [Daily morning review](feedback_daily_review.md) — 每次新会话开始先检查最近 3 天 work_summary 待办，按 P0-P3 优先级整理
- [Timestamp honesty](feedback_timestamp_honesty.md) — 日志时间戳必须用 `date` 获取实际主机时间，不得编造
- [Playwright MCP patterns](feedback_playwright_patterns.md) — alert 覆盖、非 body 滚动、JS click 失效、8192px 限制
- [Settings hierarchy](feedback_settings_hierarchy.md) — 全局 vs 项目级 settings.json 覆盖关系，修改时同时检查两处
- [Flask + Frontend gotchas](feedback_flask_gotchas.md) — debug reloader 双进程、AJAX 竞态、dict.get(None)、数量型备份保留
- [Excalidraw output and CLI export](feedback_excalidraw_limits.md) — 输出 .excalidraw JSON，@excalidraw/cli 自动导出 SVG/PNG；三工具均有完整闭环
- [Context hygiene and doc-trim](feedback_context_hygiene.md) — CLAUDE.md 精简经验（669→107行），/doc-trim 每周二/五，Change Log 3 天保留
- [Graphify incremental rebuild](feedback_graphify_incremental.md) — git diff 判断是否需要重建，SHA256 cache 文件级增量，决策表
- [Five-layer ripple consistency](feedback_five_layer_ripple.md) — 一个变更波及 4+ 层文件，config-review 需覆盖 docs/mermaid，GRAPH_REPORT.md 为 graphify 唯一权威源
- [PS script pitfalls](feedback_ps_script_pitfalls.md) — here-string 闭合必须 column 0、引号剥除用临时 .py 文件、.ps1 只用 ASCII 英文
- [SQLite optimization patterns](feedback_db_optimization.md) — 窗口函数陷阱、OR IS NULL 反模式、字符串→数字 FK 迁移、AJAX 并行化；复用 /db-optimization skill
- [Rules no duplication](feedback_rules_no_duplication.md) — Rules 不得同时存在于全局和项目目录（双重加载灾难），分类标准，当前 18+19 状态

## Project (architecture decisions)
- [CLAUDE.md restructured](project_claudemd_structure.md) — API/LSTM/changelog moved to rules files (2026-04-04)
- [device_params dedup fix](project_device_params_dedup.md) — 365 duplicate rows cleaned; three-layer defense (DB+API+frontend)
- [Long screenshot tools](project_long_screenshot.md) — 4-level Playwright screenshot strategy; main.content non-body scroll container; viewport expansion method

## Reference (external resources)
- [MCP servers quick reference](reference_mcp_servers.md) — Task-to-MCP mapping, cross-validation strategy; full catalog in rules
- [Search workflow](reference_search_workflow.md) — Tool Tier 0-4, parallel combos, confidence levels, report template; full rules in search-workflow.md
- [Zhipu MCP workflows](reference_zhipu_mcp_workflows.md) — Search->Read pipeline, vision trio, GitHub 3-step exploration
- [GitHub .claude/ config repo](reference_github_claude_config.md) — github.com/chg0901/hong-cc-practice, sync script, dual-repo maintenance
- [Skills ecosystem](reference_skills_ecosystem.md) — 16 skills 全景图（+excalidraw-diagram-generator 2026-04-22），三工具图表分工，Skill→Task 速查
- [fireworks-tech-graph](reference_fireworks_tech_graph.md) — SVG+PNG 技术图 skill; 7 styles, 14 types; Windows 用 Playwright wrapper 替代 rsvg-convert
- [Design templates](reference_design_templates.md) — 58 品牌设计系统 DESIGN.md 模板库; 与现有设计 Skill 决策矩阵
- [GitHub MCP quick ref](reference_github_mcp.md) — 26 tools overview, tool selection decision tree, 4 workflow templates; full guide in docs/github_mcp_guide.md
- [MCP Server Catalog](reference_mcp_catalog.md) — 26+ servers inventory, auth configs, per-server tools, cache locations (extracted from mcp-servers.md)
- [Test file catalog](reference_test_catalog.md) — 23 test files with coverage descriptions and run commands (extracted from testing.md)
- [Superpowers 触发矩阵](reference_superpowers_matrix.md) — 13 个 superpowers skills 触发时机和协作链（从 subagents.md 移出）
