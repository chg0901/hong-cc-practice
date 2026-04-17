# DESIGN.md Templates — 品牌设计系统模板库

## 资源位置

**本地路径**: `D:/Proj/design-templates/awesome-design-md/design-md/`
**来源**: [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md)（MIT License）

58 个品牌设计系统模板，每个 `DESIGN.md` 包含 9 个标准章节：

| # | 章节 | 内容 |
|---|------|------|
| 1 | Visual Theme & Atmosphere | 设计哲学、氛围、密度 |
| 2 | Color Palette & Roles | 语义色名 + hex + 功能角色 |
| 3 | Typography Rules | 字体族、完整层级表 |
| 4 | Component Stylings | 按钮/卡片/输入框/导航 + 状态 |
| 5 | Layout Principles | 间距比例、网格、留白哲学 |
| 6 | Depth & Elevation | 阴影系统、表面层级 |
| 7 | Do's and Don'ts | 设计护栏和反模式 |
| 8 | Responsive Behavior | 断点、触摸目标、折叠策略 |
| 9 | Agent Prompt Guide | 快速颜色参考、可直接使用的 prompt |

## 可用模板分类

### AI & LLM
claude, cohere, elevenlabs, minimax, mistral.ai, nvidia, ollama, opencode.ai, replicate, runwayml, together.ai, voltagent, x.ai

### 开发工具
cursor, expo, lovable, raycast, superhuman, vercel, warp

### 后端/数据库/DevOps
clickhouse, composio, hashicorp, mongodb, posthog, sanity, sentry, supabase

### 生产力 & SaaS
cal, intercom, linear.app, mintlify, notion, resend, zapier

### 设计 & 创意
airtable, clay, figma, framer, miro, webflow

### 金融 & 加密
coinbase, kraken, revolut, stripe, wise

### 电商 & 零售
airbnb, tesla

### 媒体 & 消费
apple, ibm, pinterest, spotify, uber

### 汽车
bmw, ferrari, lamborghini, renault

### 其他
airtable, raycast, spacex

## 使用方式

### 1. 引用特定品牌风格

当用户要求"做一个像 Stripe 的页面"或"Apple 风格的 dashboard"时：

```
1. 读取 D:/Proj/design-templates/awesome-design-md/design-md/{brand}/DESIGN.md
2. 按照其 9 个章节的设计规范生成 UI
3. 特别是第 2 章（Color Palette）和第 3 章（Typography）必须严格遵循
```

### 2. 混合多个品牌风格

可以混合多个品牌的元素，例如"Stripe 的配色 + Vercel 的排版"：

```
1. 读取 stripe/DESIGN.md 的 Color Palette
2. 读取 vercel/DESIGN.md 的 Typography Rules
3. 合并生成混合设计系统
```

### 3. 作为设计 Skill 的输入源

DESIGN.md 模板可以作为现有设计 Skill 的参考输入：

| Skill | 如何配合 DESIGN.md |
|-------|-------------------|
| `/ui-ux-pro-max` | 用 DESIGN.md 的色彩方案替代其内置调色板 |
| `/frontend-design` | 将 DESIGN.md 作为 style reference 传入 |
| `/ckm-design-system` | 用 DESIGN.md 的 token 替代其 primitive 层 |

## 与现有设计工具的决策矩阵

| 需求 | 首选工具 | 理由 |
|------|---------|------|
| 复刻特定品牌风格 | DESIGN.md 模板 | 精确的颜色/字体/组件规范 |
| 创新设计系统 | `/ui-ux-pro-max` | 数据驱动探索（161 调色板 × 57 字体） |
| 生产级前端 | `/frontend-design` | Anthropic 官方质量标准 |
| 设计 Token 体系 | `/ckm-design-system` | 三层 token 架构 |
| 本项目 SVG 场景 | [svg-design.md](svg-design.md) | 项目专用设计 token |

## 本项目适用场景

本项目的智慧农业/冷暖平台前端使用自定义 SVG 设计系统（见 [svg-design.md](svg-design.md)）。DESIGN.md 模板适用于：

1. **新增独立页面**（如报告导出、数据分析仪表盘）
2. **大屏展示模式**（cooling-screen）的品牌化
3. **文档站点**的视觉升级
4. **未来项目**的快速启动

**不适用于**：现有 SVG 设备场景（已有项目专用设计 token）、控制面板（已有统一组件库）。

## ChangeLogs

- [2026-04-16 — Initial: 58 品牌模板库、使用方式、决策矩阵、本项目适用场景](changes/2026-04-16)
