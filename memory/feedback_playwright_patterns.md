---
name: Playwright MCP 踩坑集
description: Playwright MCP 使用中的常见陷阱和解决方案，避免重复踩坑
type: feedback
originSessionId: 85b62cf5-e098-4ce2-b282-fb41489b8bb7
---
## Playwright MCP 踩坑集

### 1. window.alert 弹窗阻塞

**问题**：登录后 alert 弹窗阻塞 Playwright，MCP 的 modal state 检测是同步的，异步 dialog handler 来不及阻止。

**解决方案**：登录前注入 alert 覆盖：
```javascript
await page.evaluate(() => { window.alert = function(){}; });
```
比 `page.on('dialog')` 异步 handler 更可靠。

**Why**：Playwright MCP 的 dialog 检测是同步阻塞式的，如果不在 JS 层面消除 alert，MCP 会一直等待 dialog 关闭。

### 2. 非 body 滚动容器 fullPage 失效

**问题**：本项目使用 `main.content` 作为滚动容器（sidebar + content 布局），`body` 的 `scrollHeight` 不随内容增长。直接 `fullPage=true` 只截到 ~700px。

**解决方案**：检测并使用 viewport 扩展方法：
```javascript
const h = await page.evaluate(() => document.querySelector('main.content').scrollHeight);
await page.setViewportSize({ width: 1920, height: h });
```

**Why**：Playwright #12962 issue — fullPage 基于 `document.documentElement.scrollHeight`，不感知内部滚动容器。

### 3. 标签页切换不能用 JS click

**问题**：`page.evaluate(() => element.click())` 点击标签按钮可能不触发内容切换。

**解决方案**：使用 Playwright 原生 click：
```javascript
await page.getByText('标签名').click();
```

**Why**：JS click 只触发 DOM 事件，不经过 Playwright 的 action pipeline。有些框架的标签切换依赖 Playwright 的等待/导航机制。

### 4. Chromium 8192px 硬限制

**问题**：CDP `Page.captureScreenshot` 超过 ~8192px 输出损坏。

**解决方案**：超长页面使用 scroll + stitch 方法（Python `scripts/long_screenshot.py`）。

### 5. 截图验证 UI 行为比看代码更可靠

**场景**：确认"控制模块组态图在控制面板上方"是预期行为。

**方法**：Playwright snapshot（看 DOM 结构）+ screenshot（看视觉效果）比直接读代码更直观。当不确定是 bug 还是 feature 时，先截图看实际效果。
