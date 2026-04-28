---
name: feedback_mermaid_comparison
description: Mermaid 对比文档规则 + sequence 图颜色 + 节点换行规则（`<br/>` 而非 ` - `）
type: feedback
---

## Mermaid 对比文档规则

### 相对路径

文档在 `docs/` 目录下，图片路径必须用 `../mermaid/figure/xxx.png`（`../` 返回项目根目录）。

**错误**：`mermaid/figure/xxx.png`（图片不显示）
**正确**：`../mermaid/figure/xxx.png`

### HTML table 左右对比

用 `<table>` 实现新旧图片并排展示：

```html
<table>
<tr><th>旧版本（ratio=X.XX）</th><th>新版本（ratio=X.XX）</th></tr>
<tr>
<td><img src="../mermaid/figure/xxx_old.png" width="480"/></td>
<td><img src="../mermaid/figure/xxx.png" width="480"/></td>
</tr>
</table>
```

拆分图并列展示：`width="400"` 适合两张拆分图。

### 旧版图片恢复

**必须从 git 提取旧版 mmd**，不要用当前 mmd 渲染：

```bash
git show HEAD:path/to/xxx.mmd > mermaid/text/xxx_old.mmd
mmdc -i mermaid/text/xxx_old.mmd -o mermaid/figure/xxx_old.png -t neutral -w 1200 -H 900 -s 2
```

**Why**: 如果用已修改的 mmd 渲染"旧图"，新旧图会完全相同（MD5 一致），失去对比意义。

### classDef 颜色必须

**每个 .mmd 文件必须有 classDef 颜色定义**，不允许黑白图。缺少 classDef 会导致 mermaid 渲染为默认黑白样式，与其他彩色图风格不一致。

### 禁止使用 `---` 水平线

**Why**: `---` 是 HTML `<hr>` 的 Markdown 语法，在渲染后显示为一条灰色水平线。在文档中没有信息量，是纯视觉噪音。

**How to apply**: 用空行分隔章节即可。如果确实需要视觉分隔，用 blockquote 或表格边框代替。

## 拆分并列展示

线性流程图如果 TD 布局导致 ratio < 0.33：
1. 拆分为 `xxx-a.mmd` + `xxx-b.mmd`，每张聚焦一个阶段
2. 渲染为 `xxx-a.png` + `xxx-b.png`（用 `-w 600 -H 700`）
3. 在 md 中用 HTML table 并列展示

### 相关规则

- [mermaid.md](../.claude/rules/mermaid.md) — 颜色规范、`<br/>` 换行、信息密度规则

## Sequence 图必须有颜色

**Why**: mermaid 的 `classDef` + `class` 语法对 sequence 图无效（参与者不会上色），必须用 `%%{init: {'themeVariables': {...}}}%%` 指令。sequence 图不加颜色会渲染为黑白，与项目其他彩色图风格严重不一致。

**How to apply**: 在 sequence 图第一行添加：
```
%%{init: {'themeVariables': {'actorBkg': '#e3f2fd', 'actorBorder': '#1565c0', 'actorTextColor': '#0d47a1', 'actorLineColor': '#1565c0', 'noteBkgColor': '#fff3e0', 'noteBorderColor': '#e65100', 'activationBkgColor': '#e1f5fe', 'activationBorderColor': '#0288d1'}}}%%
```

## 节点中英文混排用 `<br/>` 换行

**Why**: 节点标签中用 ` - ` 分隔中英文（如 `L5: Agent Teams - 并行协作`）会导致两个问题：1) 中文可能在不自然的位置断行；2) 分隔符增加节点宽度，TB 布局时使图过高。

**How to apply**:
```mermaid
L5["L5: Agent Teams<br/>并行协作"]
```
而不是：
```mermaid
L5["L5: Agent Teams - 并行协作"]
```

## 不适合 mermaid 的场景直接用表格

**Why**: dagre 布局引擎给每个节点添加大量内边距（padding），5+ 节点的纯链式 TB 图中 70%+ 画面是空白间距。对于简单层级、枚举、线性信息，markdown 表格的信息密度远高于 mermaid 图。

**How to apply**: 判断标准——如果图的内容是"几个东西排成一排/一列，只有一条主线没有分支"，用 blockquote + 表格更紧凑。如果有分支、并行、反馈环、跨区域连线，才值得用 mermaid。

对于确实要用 mermaid 的线性图，考虑：subgraph 2xN 网格布局、缩短节点文本、去掉 `<b>` 减少字符宽度。
