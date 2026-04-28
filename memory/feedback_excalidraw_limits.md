---
name: Excalidraw skill output and CLI export
description: excalidraw-diagram-generator 输出 .excalidraw JSON，用 excalidraw-brute-export-cli 导出 SVG/PNG（Playwright+Firefox headless）
type: feedback
originSessionId: 2e8851c8-82a5-45e3-95b6-cf9efed80ff8
---
## Excalidraw Skill 输出与导出

**规则**：`excalidraw-diagram-generator` skill 输出 `.excalidraw` JSON 文件，不是 PNG。通过 `excalidraw-brute-export-cli`（第三方工具，Playwright+Firefox headless）导出 SVG/PNG。

**Why**：Excalidraw 官方 CLI 不存在（Issue #1261 仍未实现）。`excalidraw-brute-export-cli` 是唯一验证可用的导出工具，使用 Playwright 启动 headless Firefox 访问 excalidraw.com 完成导出。已在 Windows 上验证 SVG/PNG 导出均可用。

**How to apply**：
- 前置安装：`npm install -g excalidraw-brute-export-cli` + `npx playwright install firefox`（约 113MB）
- SVG 导出：`npx excalidraw-brute-export-cli -i diagram.excalidraw --background 0 --scale 2 --format svg -o diagram.svg`
- PNG 导出：`npx excalidraw-brute-export-cli -i diagram.excalidraw --background 1 --scale 2 --format png -o diagram.png`
- 注意：每次导出需联网访问 excalidraw.com（或 `--url` 指定本地实例）
- 导出后用 Vision MCP 直接分析 PNG 文件验证质量
- 备选查看方式：excalidraw.com 拖放 或 VS Code Excalidraw 插件

**三工具分工（2026-04-22 修正）**：

| 工具 | 场景 | 输出 | 嵌入文档 |
|------|------|------|---------|
| Mermaid | 文档内联、快速流程图 | 静态 PNG | 直接 `![]()` |
| fireworks-tech-graph | 出版级架构图 | 静态 PNG（SVG→png） | 直接 `![]()` |
| Excalidraw | 手绘/协作/概念图 | `.excalidraw` → CLI 导出 SVG/PNG | CLI 导出后 `![]()` |
