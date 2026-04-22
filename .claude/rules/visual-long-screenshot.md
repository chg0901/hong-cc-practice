# Long Screenshot Rules — 长截图工具使用规范

## 工具层级

根据页面特征选择不同级别的工具：

| 级别 | 场景 | 工具 | Skill |
|------|------|------|-------|
| L1 快捷 | 简单页面（body 滚动，<8192px） | `browser_take_screenshot(fullPage=true)` | 无需 |
| L2 标准 | 非body滚动容器（本项目最常见的 main.content） | `browser_run_code` + viewport 扩展 | `/long-screenshot` |
| L3 增强 | 有固定头部/懒加载 | L2 + CSS 注入 + lazy load 强制 | `/long-screenshot` + 可选参数 |
| L4 超长 | 页面 >8192px | Python `scripts/long_screenshot.py` | `/long-screenshot` (Strategy C) |

## 本项目特殊问题

本项目前端使用 `main.content` 作为滚动容器（sidebar + content 布局），`body` 的 `scrollHeight` 不会随内容增长。因此：

- **直接 `fullPage=true` 会截断**：只捕获 ~700px（body 高度），丢失 main 中的所有内容
- **必须使用 viewport 扩展方法**：设置 viewport 高度为 main.scrollHeight，然后截图
- **这是 Playwright #12962 的典型表现**

## 检测方法

在截图前先检测滚动容器：

```javascript
async (page) => {
    const info = await page.evaluate(() => {
        const main = document.querySelector('main.content');
        return {
            bodySH: document.documentElement.scrollHeight,
            mainSH: main ? main.scrollHeight : 0,
            hasInnerScroll: main && main.scrollHeight > document.documentElement.scrollHeight
        };
    });
    return JSON.stringify(info);
}
```

- `hasInnerScroll === true` -> 使用 L2 方法
- `hasInnerScroll === false` && `bodySH <= 8192` -> 使用 L1 方法
- `bodySH > 8192` -> 使用 L4 方法

## Viewport 扩展标准操作

```javascript
async (page) => {
    const h = await page.evaluate(() => document.querySelector('main.content').scrollHeight);
    await page.setViewportSize({ width: 1920, height: h });
    await page.waitForTimeout(500);
    await page.evaluate(() => {
        document.querySelectorAll('main, [class*="content"]').forEach(el => {
            const s = getComputedStyle(el);
            if (s.overflow === 'auto' || s.overflowY === 'auto') {
                el.style.overflow = 'visible';
                el.style.height = 'auto';
                el.style.maxHeight = 'none';
            }
        });
        document.body.style.overflow = 'visible';
        document.body.style.height = 'auto';
    });
    await page.waitForTimeout(300);
    await page.screenshot({ path: 'output.png', scale: 'css', type: 'png' });
    await page.setViewportSize({ width: 1280, height: 720 }); // restore
}
```

## Python 脚本

```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/long_screenshot.py \
    --url http://127.0.0.1:5000 --output test_screenshots/xxx.png --expand-scroll
```

可选：`--fix-sticky` `--force-lazy` `--scroll-to-bottom`

分段拼接工具：

```bash
D:/miniconda3/envs/ene/python.exe scripts/stitch_segments.py seg_*.png -o output.png --overlap 50
```

## 截图命名规范

`test_screenshots/{area}_fullpage_{YYYYMMDD}.png`

- `{area}` = home, 3d, meteorology, control, cooling-dashboard 等
- 不用 viewport 截图（违反 memory/feedback_screenshot_rules.md）— 仅限长截图场景

## 验证

截图后用 Vision MCP 验证完整性：

```
mcp__MiniMax__understand_image -> "Verify: all content captured? No truncation? All rows visible?"
```

## 与图表工具的联动

截长图工具链针对**网页页面**截图。图表文件的验证路径不同：

| 图表工具 | 验证方式 | 是否需要截长图 |
|---------|---------|--------------|
| Mermaid (`.mmd` → `.png`) | Vision MCP 直接分析 PNG 文件 | 否 |
| fireworks-tech-graph (`.svg` → `.png`) | Vision MCP + ZAI OCR 分析 PNG | 否 |
| Excalidraw (`.excalidraw` → `.svg`/`.png`) | `@excalidraw/cli` 导出 → Vision MCP 直接分析 PNG | 否 |

**Excalidraw 导出命令**（`@excalidraw/cli`）：

```bash
# 推荐：透明背景 + 高清
excalidraw diagram.excalidraw -o diagram.svg --padding 15 --background transparent --scale 2
```

## ChangeLogs

- [2026-04-22] Excalidraw 验证路径更新：`@excalidraw/cli` 导出 PNG + Vision MCP，不再需要浏览器截图
- [2026-04-22] 新增"与图表工具的联动"章节：Mermaid/fireworks/Excalidraw 验证路径对比
- [2026-04-15] 初始版本：4 级工具层级、本项目 main.content 滚动容器检测、viewport 扩展标准操作
