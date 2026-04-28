---
name: Long screenshot tools and strategies
description: 4-level Playwright long screenshot tools: L1 fullPage, L2 viewport expansion for main.content, L3 CSS injection, L4 Python scroll+stitch. Key finding: body scrollHeight=704 but main.content scrollHeight=3011 in this project.
type: project
originSessionId: 61955e8a-0ed7-4869-bb56-81f48a1152c7
---
本项目前端使用 `main.content` 作为滚动容器，`body` 不滚动。这是 Playwright `fullPage` 截图的典型失败场景（Playwright #12962）。

**Why**: sidebar + content 布局导致 `main.content` 设置了 `overflow: auto`，内容在 main 内部滚动，body 的 scrollHeight 始终等于视口高度。

**How to apply**:
- 截图前必须检测滚动容器（用 `browser_run_code` 检查 main.scrollHeight vs body.scrollHeight）
- body SH=704, main SH=3011 -> 必须使用 viewport 扩展方法（设置 viewport 高度为 main.scrollHeight）
- 直接 `fullPage=true` 只截到 ~700px，丢失所有 main 内容

## 测试结果（2026-04-15）

| 方法 | 结果 | 截图高度 | 内容完整性 |
|------|------|---------|-----------|
| `fullPage=true` 直接使用 | FAIL | ~692px | 底部截断 |
| CSS overflow=visible + fullPage | FAIL | ~750px | 仍然截断 |
| Viewport 扩展到 main SH + CSS fix | SUCCESS | 3011px | 全部内容可见 |

## 工具文件

| 文件 | 用途 |
|------|------|
| `scripts/long_screenshot.py` | Python 滚动+拼接长截图（突破 8192px） |
| `scripts/stitch_segments.py` | 分段截图拼接 |
| `.claude/skills/long-screenshot/` | `/long-screenshot` skill（MCP 操作流程） |
| `.claude/rules/visual-long-screenshot.md` | 长截图规则（4 级工具选择） |
| `docs/playwright_fullpage_screenshot.md` | 完整技术文档 |

## 已安装的第三方 Skill

- `webpage-screenshotter`（rkreddyp/investrecipes，54 installs）— 高分辨率全页面截图，Cloudflare 绕过，1920x1080 viewport
  - 安装位置：`~/.agents/skills/webpage-screenshotter`（symlink）
  - 不处理非 body 滚动容器，本项目仍需 viewport 扩展方法
