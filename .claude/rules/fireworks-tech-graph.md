# fireworks-tech-graph 技术图生成规则

## 适用场景

当需要生成独立的高清技术架构图（非 inline markdown 图表）时使用 fireworks-tech-graph Skill。

## 与 Mermaid 的分工

| 工具 | 用途 | 文件格式 | 质量验证 |
|------|------|---------|---------|
| Mermaid (`mmdc`) | 嵌入 markdown 文档的 inline 图表 | `.mmd` → `.png` | Vision MCP 检查文字渲染 |
| fireworks-tech-graph | 独立出版级 SVG 架构图 | `.svg` → `.png` | Vision MCP + OCR 交叉验证 |

**选择规则**：
- 文档内联图表、流程图 → Mermaid
- PPT 用图、README 头图、独立架构图 → fireworks-tech-graph
- 需要多风格切换 → fireworks-tech-graph

## 生成 Workflow

```
1. 分类图表类型（Architecture / Data Flow / Sequence / Agent / Memory 等）
2. 提取结构（layers, nodes, edges, flows）
3. 规划布局（viewBox, 分层方向）
4. 加载风格参考（references/style-N.md）
5. 映射形状语义（参考 SKILL.md Shape Vocabulary）
6. 检查图标需求（references/icons.md）
7. 生成 SVG（Python list method）
8. 导出 PNG（scripts/svg2png.py）
9. Vision MCP 质量验证
10. 引用到文档
```

## PNG 导出命令

```bash
# 生成后导出 PNG
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/svg2png.py \
    <input>.svg -o <output>.png -w 1920
```

## 质量验证（mandatory）

生成 PNG 后**必须**执行以下检查：

### 1. 基础检查（自动）

```python
# 检查 PNG 文件大小和像素分布
from PIL import Image
import numpy as np, os

img = Image.open(png_path)
arr = np.array(img)
size_kb = os.path.getsize(png_path) / 1024
white_ratio = (arr.mean(axis=2) > 250).mean()

# 断言：文件 > 50KB，非全白（< 95%），宽度 >= 1920px
```

### 2. Vision MCP 检查

使用 MiniMax `under_image` 检查：
- 文字清晰度、重叠、溢出
- 节点布局平衡
- 箭头可见性和语义正确性
- 整体专业质量评分

### 3. OCR 交叉验证（中文字符场景）

使用 ZAI `extract_text_from_screenshot` 验证中文渲染正确性。

## Jina MCP 协作场景

当需要从外部资料生成技术图时，Jina MCP 提供"调研 → 出图"一站式流水线：

| 场景 | Jina 工具 | → 图表类型 |
|------|----------|-----------|
| 技术架构调研 | `search_web` + `read_url` 读取技术文档 | → Architecture Diagram |
| 论文方法对比 | `search_arxiv` + `read_url` 读 PDF | → Comparison Matrix |
| 开源项目文档 | `read_url` 读 README/API 文档 | → Data Flow / Module Diagram |
| 竞品功能分析 | `search_web` + `sort_by_relevance` | → Feature Matrix |
| 系统设计参考 | `parallel_search_web` 批量搜索 | → Architecture / Sequence |

**典型流程**：
```
1. Jina search_web "microservices architecture best practices 2026"
2. Jina read_url 阅读 top 3 结果
3. 提取架构分层、组件关系、数据流信息
4. fireworks-tech-graph 生成对应图表（指定风格）
5. Vision MCP 验证 PNG 质量
6. 引用到文档
```

**Jina vs 其他搜索工具选择**：
- 库/框架文档 → `context7`（最精确，无配额）
- 英文技术内容 → `jina search_web`（返回完整内容）
- 学术论文 → `jina search_arxiv`（唯一支持）
- PDF 读取 → `jina read_url`（原生 PDF 支持）

## 输出文件位置

- SVG 源文件：`docs/samples/<name>_<style>.svg`
- PNG 导出：`docs/samples/<name>_<style>.png`

## 风格选择参考

| 图表类型 | 推荐风格 | 原因 |
|----------|---------|------|
| 架构图 | 1, 2, 3 | 彩色/深色对比/正式 |
| 数据流图 | 1, 2, 3 | 彩色箭头分类 |
| 时序图 | 2, 4, 7 | 等宽字体对齐 |
| Agent/Memory | 2, 5 | 科技感/AI 演示 |
| UML 类图 | 1, 4, 7 | 结构清晰 |
| 对比矩阵 | 4, 6, 7 | 表格嵌入 wiki |

完整风格-图表适配矩阵：`~/.agents/skills/fireworks-tech-graph/references/style-diagram-matrix.md`

## ChangeLogs

- [2026-04-15 17:00:00] 新增 Jina MCP 协作场景（search_web/read_url/search_arxiv → 架构图生成流水线）
- [2026-04-15 16:30:00] Initial: 生成规则、Mermaid 分工、质量验证三步流程、风格选择参考
