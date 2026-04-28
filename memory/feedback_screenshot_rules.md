---
name: screenshot rules for user manual
description: 截图规则：不调viewport、用多张截图覆盖长页面、必须包含子标题
type: feedback
---

## 截图规则

- **不调整 viewport**：使用 Playwright 默认窗口大小，不调用 `setViewportSize` 或 `browser_resize`
- **长页面用多张截图**：`fullPage` 参数在 Playwright MCP 中可能不生效，改用多张截图拼接方式：
  1. 先截顶部视口（默认位置）
  2. 用 `page.evaluate(() => window.scrollBy(0, window.innerHeight))` 滚动
  3. 再截下一屏
  4. 重复直到页面底部（`scrollY + innerHeight >= document.body.scrollHeight`）
- **必须包含子标题**：每个功能模块的子标题（如"统计卡片"、"双碳统计"、"项目地图"等）必须在截图中可见
- **命名规则**：`{module}_{section}_{N}.png`，如 `home_top_1.png`, `home_bottom_2.png`
- **不使用 `scale: 'css'`**：直接用默认 `page.screenshot({ path, type: 'png' })`

- **操作过程截图（mandatory）**：除页面静态截图外，必须截取：
  - 下拉选项选择后（如公司筛选、项目类型筛选）
  - 标签切换（如"数据管理"、"生产计划"、"生产活动"等不同标签页）
  - 弹窗/模态框（新建、编辑、添加操作），带滚动条的弹窗必须滚到底部再截一张
  - 日期选择器设置后
  - 查询/筛选结果
  - 不同状态（如空列表、有数据、错误提示）
- **页面内部滚动条处理（mandatory）**：
  - 除了主内容区的页面级滚动外，还要注意页面中间的内部滚动条：
    - 弹窗/模态框内的滚动：弹窗内容超过视口时，必须截取顶部和底部两张
    - 表格区域滚动：表格有独立滚动条时，滚动到底部再截一张
    - 侧边栏滚动：设备列表、参数列表等侧边滚动区域也要覆盖
  - 检测方法：`element.scrollHeight > element.clientHeight`
  - 操作方式：`element.scrollTop = element.scrollHeight` 再截图
- **弹窗滚动规则**：弹窗/模态框如果有滚动条，必须截取顶部和底部两张，确保所有字段可见
- **数据加载等待（mandatory）**：截图前必须确认页面数据已加载完成：
  - 不允许截取包含"--"、空白区域、"加载中..."、"暂无数据"的截图
  - 必须等待 API 请求完成且数据渲染到 DOM 后再截图
  - 检测方法：`page.evaluate(() => { const text = document.querySelector('main').textContent; return /\d+\.\d+/.test(text) && !text.includes('--'); })`
  - 轮询等待：最多 10 秒（20 次 x 500ms），确保数值数据出现
  - 适用模块：土壤墒情、气象信息、系统监控、冷暖监控、历史数据等所有需要 API 加载数据的页面
  - 图表类内容：等待 canvas/svg 渲染完成（至少等待 2 秒后再截图）

## How to apply:
- 所有用户手册截图遵循此规则
- 每个页面至少2张截图（顶部+下方内容）
- 每个弹窗至少1张（有滚动条则2张）
- 每个筛选/标签切换至少1张
- 手册中引用截图时用多图并排或顺序排列
