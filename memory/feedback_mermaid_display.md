---
name: Mermaid 图片显示比例规则
description: 在线文档中 mermaid 图与截图的显示差异处理规则。mermaid 图文字需与正文文字大小协调。
type: feedback
originSessionId: fe3e437f-ae40-46e8-9840-3b902d43201e
---
## 规则

Mermaid 图和截图在在线文档中需要不同的显示策略：

### 问题

Mermaid 图用 `-s 2` 高 DPI 渲染（实际像素 2x），在 `max-width: 900px` 的内容区域中：
- 宽图（LR 布局，如 1870x844）→ 缩到 900px 后文字只有正文文字的 ~48% 大小
- 窄图（TD 布局，如 526x1244）→ 不缩放但文字仍然偏小，且高度过长

### 解决方案

1. **截图**（screenshots/）：`max-width: 100%` — 自然缩放，保持纵横比
2. **Mermaid 图**（mermaid/）：
   - CSS: `width: 100%; max-height: 600px; object-fit: contain;`
   - 添加 `mermaid-diagram` class 以区分截图
   - 点击可放大查看完整尺寸（`zoomed` class 移除 max-height）
   - 浅灰背景 + 边框，与截图视觉区分

### docs_server.py 实现

- JS 自动检测 `mermaid/` 路径的图片，添加 class 和点击事件
- 截图路径保持 `max-width: 100%` 不变
- 所有 `../` 前缀用正则 `(?:\.\.\/)+` 全部去除（兼容 `../` 和 `../../`）

### Why

Mermaid 图的文字在缩小后可读性下降，影响文档审美。限制高度 + 点击放大是折中方案：默认展示全局结构，点击查看细节。

### How to apply

- docs_server.py 中已实现此逻辑
- 如需调整，修改 `.content img.mermaid-diagram` 的 `max-height` 值
- 新增 mermaid 图时自动生效（路径包含 `mermaid/` 即可）
