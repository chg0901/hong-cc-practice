---
name: Skills ecosystem reference
description: 16 skills 全景图（+excalidraw-diagram-generator 2026-04-22）+ 58 DESIGN.md 模板库（非 skill），Skill→Task 速查，安装后 onboarding 流程，工具链协作
type: reference
originSessionId: 85b62cf5-e098-4ce2-b282-fb41489b8bb7
---
# Skills Ecosystem Quick Reference

## Skill → Task 速查表

| 用户意图 | 推荐 Skill | 触发方式 | 来源 |
|---------|-----------|---------|------|
| "审查这段代码" | `code-review-expert` | 自动或 `/code-review-expert` | sanyuan-skills |
| "学一下 X 技术" | `sigma` | `/sigma <topic>` | sanyuan-skills |
| "创建一个新 skill" | `skill-forge` | 自动或 `/skill-forge` | sanyuan-skills |
| "读这本书" | `book-study` | `/book-study <book-name>` | sanyuan-skills |
| "整理这些文档到 wiki" | `wiki-ingest` | `/wiki-ingest` | sanyuan-skills |
| "画架构图/流程图（出版级）" | `fireworks-tech-graph` | 自动（"画图/架构图/流程图"） | yizhiyanhua-ai |
| "画手绘/协作/概念图" | `excalidraw-diagram-generator` | 自动（"create diagram/flowchart/mind map"） | github/awesome-copilot |
| "验证 SVG 渲染" | `visual-check` | 自动（PostToolUse hook） | **项目自建** |
| "测试交互行为" | `interaction-check` | 自动（PostToolUse hook） | **项目自建** |
| "搜索代码模块关系" | `graphify-workflow` | `/graphify-workflow query <topic>` | **项目自建** |
| "更新知识图谱" | `graphify-workflow` | `/graphify-workflow update` 或 git commit 自动 | **项目自建** |
| "深度搜索代码" | `context-research` | `/context-research`（context: fork） | 第三方 |
| "审查手册截图" | `manual-review` | `/manual-review` | **项目自建** |
| "长截图" | `long-screenshot` | 脚本调用 | **项目自建** |
| "像 Stripe/Apple 风格" | DESIGN.md 模板 | 读取 `D:/Proj/design-templates/` | **参考库**（非 skill） |

## Skills 分类

**项目自建（9 个，同步到 GitHub）**：visual-check, interaction-check, graphify-workflow, manual-review, long-screenshot, doc-trim, context-research, book2skills, create-colleague

**第三方 clone/npm（5 个，不同步）**：book2skills, create-colleague, context-research, baidu-search, excalidraw-diagram-generator

**第三方 symlink（7 个，npx 管理，不同步）**：book-study, code-review-expert, sigma, skill-forge, wiki-ingest, fireworks-tech-graph, web-access

## 三工具图表分工（2026-04-22 修正 CLI 工具）

| 工具 | 场景 | 输出格式 | 嵌入文档 |
|------|------|---------|---------|
| Mermaid (`mmdc`) | 文档内联、快速流程图 | 静态 PNG | 直接 `![]()` |
| fireworks-tech-graph | 出版级架构图、多风格 | 静态 PNG（SVG→png） | 直接 `![]()` |
| Excalidraw | 手绘/协作/概念图 | `.excalidraw` → CLI 导出 SVG/PNG | CLI 导出后 `![]()` |

**Excalidraw 导出命令**：`npx excalidraw-brute-export-cli -i diagram.excalidraw --background 0 --scale 2 --format svg -o diagram.svg`

**Excalidraw 关键特性**：输出 `.excalidraw` JSON 可编辑文件，通过 `excalidraw-brute-export-cli`（Playwright+Firefox headless）导出 SVG/PNG。前置安装需 `npx playwright install firefox`。

## 工具链协作矩阵

### Mermaid 图表工具链
```
编辑 .mmd 文件 → mmdc 渲染 PNG → Vision MCP 验证 → 文档引用
```

### fireworks-tech-graph 工具链
```
自然语言描述 → Skill 生成 SVG → svg2png.py 导出 PNG → Vision MCP 验证 → 文档引用
```

### Excalidraw 图表工具链
```
Skill 生成 .excalidraw → excalidraw-brute-export-cli 导出 SVG/PNG → Vision MCP 验证 → 文档引用
```

### 技术调研 + 图表生成（含 Jina MCP）
```
Jina search_web 搜索技术资料
  → Jina read_url 读取技术文档
  → 提取架构/流程信息
  → fireworks-tech-graph 生成架构图
  → Vision MCP 验证质量
```

### 文档研究工具链（含 Jina MCP）
```
Jina search_arxiv 搜索论文
  → Jina read_url 读取 PDF
  → 提取方法论/架构描述
  → fireworks-tech-graph 生成对比图/流程图
  → ZAI OCR 交叉验证中文字符
```

## Jina MCP 协作场景

| 场景 | Jina 工具 | 协作对象 | 产出 |
|------|----------|---------|------|
| 技术架构调研 | `search_web` + `read_url` | fireworks-tech-graph | 架构图 SVG+PNG |
| 论文方法对比 | `search_arxiv` + `read_url` | fireworks-tech-graph | 对比矩阵 SVG+PNG |
| 竞品分析 | `search_web` + `sort_by_relevance` | fireworks-tech-graph | Feature Matrix SVG+PNG |
| 开源项目文档 | `read_url` 读取 README | fireworks-tech-graph | 模块关系图 SVG+PNG |

## 安装后整理流程

每次安装新 skill/agent 后，执行 `extension-onboarding.md` 规则中的 6 步流程：
1. 读取新内容 → 2. 更新 Rules → 3. 更新 Memory → 4. 去重检查 → 5. GitHub 同步 → 6. 记录 Work Summary
