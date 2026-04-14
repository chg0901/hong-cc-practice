# Mermaid 图表规则

## 适用场景

凡涉及以下类型的图表，**必须同时保留文字版和 mermaid 渲染版**：

- 流程图（工作流、决策树、处理步骤）
- 架构图（系统组件、模块依赖、分层架构）
- 时序图（API 调用链、事件顺序）
- 关系图（数据库 ER 图、状态机）
- 框图/层级图（任何使用 `┌──┐` `││` `└──┘` 绘制的结构）
- 文字箭头流程（任何使用 `→` `├─→` `└─→` 绘制的流程）

**强制规则：ASCII 文字版 + Mermaid 图必须同时保留，方便对比确认。不允许删除原始文字版。**

## Aspect Ratio Constraint / 长宽比约束

**Rule**: Rendered PNG aspect ratio (width/height) MUST be between 0.33 and 3.0.
**规则**：渲染后的 PNG 长宽比（宽/高）必须在 0.33 到 3.0 之间。

**Preferred ratios / 优先比例**（按优先级排列）：

| Priority 优先级 | Ratio 比例 | Orientation 方向 | Rendering 渲染参数 |
|----------------|------------|-----------------|-------------------|
| 1st 首选 | 4:3 (1.33) | Landscape 横向 | `-w 1200 -H 900` |
| 2nd 次选 | 16:9 (1.78) | Landscape 横向 | `-w 1600 -H 900` |
| 3rd 长图才用 | 3:4 (0.75) | Portrait 纵向 | `-w 1000 -H 1400` |
| 4rd 长图才用 | 9:16 (0.56) | Portrait 纵向 | `-w 900 -H 1600` |

**Priority rule / 优先级规则**：
- EN: Always try 4:3 or 16:9 first. Only use 3:4 or 9:16 when the diagram is inherently tall (e.g., long sequential flow).
- CN: 始终优先尝试 4:3 或 16:9。只有当图表天然是长图时（如长顺序流程）才使用 3:4 或 9:16。

**Verification / 验证（mandatory, 渲染后必须执行）**：

```
# Pseudo-code / 伪代码 (Bilingual / 双语)

RULE check_aspect_ratio(image_path):
    img = Image.open(image_path)
    w, h = img.size
    ratio = w / h

    # EN: If ratio > 3.0 (too wide) or < 0.33 (too tall), MUST fix
    # CN: 如果比例 > 3.0（太宽）或 < 0.33（太高），必须修复
    IF ratio > 3.0 OR ratio < 0.33:
        RETURN "FAIL: ratio={ratio}, must be 0.33..3.0"
    ELSE:
        RETURN "OK: ratio={ratio}"

RULE fix_aspect_ratio(mmd_file):
    # EN: Strategies to fix aspect ratio violations
    # CN: 修复长宽比违规的策略
    STRATEGIES = [
        "1. Change direction: TB↔LR / 改变方向",
        "2. Split into multiple smaller diagrams / 拆分为多张小图",
        "3. Use subgraph with direction LR inside TB / 混合布局",
        "4. Use ~~~ for horizontal spacing / 使用 ~~~ 横向留白",
        "5. Reduce node text length / 缩短节点文字",
    ]
    APPLY strategy
    RE-RENDER and RE-CHECK
```

**Fix strategies / 修复策略**：

| Problem 问题 | Strategy 策略 | Example 示例 |
|-------------|--------------|-------------|
| ratio > 3 (too wide 太宽) | Change `LR` to `TB` | `flowchart TB` instead of `flowchart LR` |
| ratio > 3 (too wide 太宽) | Split into 2+ diagrams | One diagram per logical section |
| ratio < 0.33 (too tall 太高) | Change `TB` to `LR` | `flowchart LR` instead of `flowchart TB` |
| ratio < 0.33 (too tall 太高) | Use `direction LR` inside subgraphs | Nested horizontal layout within vertical |
| 星形太宽 | LR + `~~~` + 反向箭头 | `Ref ~~~ Projects ~~~ Data`；`Ref --> Projects`（左→右）让 Ref 在左侧 |
| 连线交叉 | 调整节点定义顺序 | 有跨区域连线的节点放最靠近连线目标的位置 |
| Any 任何 | Combine nodes with `~~~` | Force horizontal spacing |

**减少连线交叉的技巧**：

1. **节点顺序即布局顺序**：mermaid dagre 引擎按节点定义顺序排列，调换 subgraph 内节点顺序可改变上下位置
2. **跨区域连线优化**：把有最多跨区域连线的节点（如 Devices → SensorData）放在最靠近目标区域的位置（Core 顶部靠近 Tier2 顶部）
3. **连线顺序对齐**：`Projects --> Devices --> SensorData` 应紧跟在 Devices 定义之后，不要穿插无关连线

## Visual Quality Check / 视觉质量检查（mandatory, 渲染后必须执行）

**Rule**: After rendering any mermaid PNG, MUST use Vision MCP to verify text rendering quality.
**规则**：渲染任何 mermaid PNG 后，必须使用 Vision MCP 验证文字渲染质量。

```
RULE check_mermaid_visual(png_path):
    # EN: Use Vision MCP to check for common rendering issues
    # CN: 使用 Vision MCP 检查常见渲染问题

    CHECKLIST = [
        "1. Text overlap / 文字遮挡",
        "2. Text overflow beyond node border / 文字溢出节点边框",
        "3. Text truncation / 文字被截断",
        "4. Node overlap / 节点重叠",
        "5. Chinese characters garbled / 中文乱码",
        "6. Arrows obscured by text / 箭头被文字遮挡",
    ]

    # EN: Primary tool: ZAI analyze_image (best for text rendering issues)
    # CN: 首选工具：ZAI analyze_image（最适合检测文字渲染问题）
    result = mcp__zai_mcp_server.analyze_image(
        image_source = png_path,
        prompt = "Check ALL nodes for: text overlap, text overflow, text truncation, node overlap, garbled Chinese. List each node status OK or FAIL."
    )

    # EN: If any FAIL found, fix mermaid source and re-render
    # CN: 如果发现任何 FAIL，修复 mermaid 源文件并重新渲染
    IF result.has_failures():
        FIX source and RE-RENDER
        RE-CHECK until all nodes pass
```

**Common fixes / 常见修复**：

| Issue 问题 | Fix 修复 |
|-----------|---------|
| Text overflow 文字溢出 | Shorten node text / 缩短节点文字 |
| Node overlap 节点重叠 | Use `~~~` for spacing / 用 `~~~` 横向留白 |
| Text too small 文字太小 | Reduce node count or split diagram / 减少节点数或拆分图表 |
| Chinese garbled 中文乱码 | Check font in mmd file / 检查 mmd 文件编码 |

## 渲染命令

```bash
# 横向 16:9 / Landscape 16:9
mmdc -i input.mmd -o output.png -t neutral -w 1600 -H 900 -s 2

# 纵向 3:4 / Portrait 3:4
mmdc -i input.mmd -o output.png -t neutral -w 1000 -H 1400 -s 2

# 横向 4:3 / Landscape 4:3
mmdc -i input.mmd -o output.png -t neutral -w 1200 -H 900 -s 2
```

- `-s 2`：高清 scale / high-DPI scale
- `-t neutral`：中性主题 / neutral theme
- 渲染后必须验证长宽比 / MUST verify aspect ratio after rendering

## 文件存放

```
mermaid/
├── text/    ← .mmd 源文件
└── figure/  ← 渲染后的 .png
```

## 颜色规范

| 节点类型 | 填充色 | 边框色 | 用途 |
|----------|--------|--------|------|
| 输入/开始/结束 | `#e1f5fe` | `#0288d1` | 流程起止点 |
| 处理步骤 | `#e3f2fd` | `#1565c0` | 一般处理节点 |
| 输出/成功 | `#c8e6c9` | `#2e7d32` | 正常结果 |
| 判断/决策 | `#fff3e0` | `#e65100` | 菱形判断节点 |
| 特殊/放弃 | `#f3e5f5` | `#6a1b9a` | 特殊分支 |
| 错误/冲突 | `#ffcdd2` | `#c62828` | 异常路径 |

## 在文档中引用

必须同时保留**文字版**和**Mermaid 图**：

```markdown
**文字版**：
(原始 ASCII 框图或箭头流程)

**Mermaid 图**：
![图表标题](../mermaid/figure/xxx.png)
- 图：[mermaid/figure/xxx.png](../mermaid/figure/xxx.png)
- 源：[mermaid/text/xxx.mmd](../mermaid/text/xxx.mmd)
```

## 注意事项

- 节点标签中的 HTML 特殊字符需转义：`<` → `&lt;`，`>` → `&gt;`
- 纯文本换行使用 `<br/>`
- 加粗使用 `<b>文字</b>`（flowchart 节点内）
- 避免节点 id 与 mermaid 关键字冲突（如 `end`、`start`）
- 图表过于复杂时拆成多张，每张聚焦一个主题
- 渲染后必须验证长宽比（0.33-3.0），不符合则修复后重新渲染
- **每个 .mmd 文件必须有 classDef 颜色定义**，不允许黑白图（参考上方"颜色规范"表）
- **sequence 图也必须有颜色**：使用 `%%{init: {'themeVariables': {...}}}%%` 指令设置参与者/信号/备注颜色（flowchart 的 classDef 对 sequence 无效）
- **节点中英文混排时用 `<br/>` 换行**，不要用 ` - ` 分隔符（` - ` 会产生中文断行不连续 + 节点过宽过高）
- **不适合 mermaid 的场景直接用表格**：5+ 节点纯链式 TB 图（dagre 间距 overhead 70%+ 空白）、简单层级/枚举信息（表格更紧凑）。mermaid 适合有分支/并行/反馈环的复杂结构
- **过长的线性流程图**拆分为 2+ 张图，在文档中用 HTML table 并列展示（见下方"拆分并列展示"）

## 拆分并列展示

对于线性流程图，如果 TD 布局导致 ratio < 0.33，可拆分为多个阶段：

1. 创建 `xxx-a.mmd`、`xxx-b.mmd`，每张聚焦一个阶段
2. 渲染为 `xxx-a.png`、`xxx-b.png`（使用较小的 `-w 600 -H 700`）
3. 在文档中用 HTML table 并列展示：

```html
<table>
<tr><th>阶段一</th><th>阶段二</th></tr>
<tr>
<td><img src="../mermaid/figure/xxx-a.png" width="400"/></td>
<td><img src="../mermaid/figure/xxx-b.png" width="400"/></td>
</tr>
</table>
```

## 对比文档规则

修复旧图表长宽比时，创建 `docs/mermaid_ratio_fix_comparison_YYYYMMDD.md`：

- **相对路径**：文档在 `docs/` 目录下，图片路径必须用 `../mermaid/figure/xxx.png`（需要 `../` 返回项目根目录）
- **HTML table 左右对比**：用 `<table>` 实现新旧图片并排，`width="480"` 控制列宽
- **旧版来源**：用 `git show HEAD:path > xxx_old.mmd` 从 git 提取旧版源文件，渲染为 `*_old.png`
- **不要用新版 mmd 渲染旧图**：否则新旧图完全相同，失去对比意义

## ChangeLogs

- 2026-04-13 — 新增：sequence 图颜色（themeVariables）、`<br/>` 换行规则、不适合 mermaid 的场景直接用表格（dagre 间距 overhead）
- 2026-04-10 — 新增减少连线交叉技巧（节点定义顺序、跨区域连线优化、连线顺序对齐）、sqlite_foreign_keys 布局优化、system-flow 颜色修复
- 2026-04-10 — 新增：classDef 颜色必须（禁止黑白图）、拆分并列展示、对比文档规则（相对路径 `../`、HTML table、旧版从 git 恢复）
- 2026-04-09 — 新增视觉质量检查（Vision MCP mandatory）、优先比例 4:3>16:9>3:4>9:16
- 2026-04-09 — 黄金比例约束、长宽比验证规则、双语伪代码、文字版+Mermaid 同时保留
- 2026-04-09 — 强制规则：ASCII 框线/箭头流程图必须同时创建 mermaid 版本并渲染 PNG
- 2026-04-08 — 初始：渲染命令、颜色规范、文件存放、引用格式