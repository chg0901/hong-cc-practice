---
name: manual-review
description: 审查用户手册截图与文档一致性，验证截图内容，发现问题并生成修复计划
---

# Manual Review Skill — 用户手册截图与文档审查

## 使用场景

当完成以下工作时，执行本 skill 进行截图与文档一致性审查：
- 完成一批截图之后
- 发现文档与实际功能不符时
- 提交前最终验证

## 审查流程

### 1. 截图引用验证（必须首先执行）

使用 Python 脚本验证所有 MD 文件的截图引用与实际 PNG 文件匹配：

```bash
cd d:/Proj/energy
python -c "
import os, re, glob
screenshots_dir = 'docs/user_manual/screenshots'
png_files = set()
for root, dirs, files in os.walk(screenshots_dir):
    for f in files:
        if f.endswith('.png'):
            rel = os.path.relpath(os.path.join(root, f), 'docs/user_manual').replace(os.sep, '/')
            png_files.add(rel)
md_files = glob.glob('docs/user_manual/*.md')
referenced = set()
missing = []
for md in md_files:
    with open(md, 'r', encoding='utf-8') as f:
        content = f.read()
    refs = re.findall(r'!\[.*?\]\((.*?)\)', content)
    for ref in refs:
        if ref.startswith('screenshots/'):
            referenced.add(ref)
            if ref not in png_files:
                missing.append((os.path.basename(md), ref))
orphan = png_files - referenced
print('PNG:', len(png_files), 'Ref:', len(referenced), 'MISSING:', len(missing))
if missing:
    for m,r in missing: print('  MISSING:', m, r)
print('ORPHAN:', len(orphan))
"
```

**通过标准**：MISSING=0

### 2. Vision MCP 内容抽查

对关键模块截图用 Vision MCP 验证内容正确性：

```
MiniMax understand_image
  → 分析图片内容是否与 md 文档描述的功能/区域匹配
  → 检查：页面标题、子标题、可见元素是否与预期一致
```

**必须抽查的模块**：
- 3D展示（最容易导航错误的模块）
- 土壤墒情（内部滚动条页面代表）
- 冷暖监控（多标签页代表）
- 历史数据（3标签页，需验证每个标签页截图）

### 3. 内部滚动条检测（关键！）

**问题现象**：某些页面 body 不需要滚动，但 main 元素内部有滚动条

**已发现需要内部滚动的页面**：
- 土壤墒情（main.scrollHeight=2354, clientHeight=695）

**滚动方法（错误 vs 正确）**：
```javascript
// 错误：window 滚动对内部滚动条无效
window.scrollBy(0, 500);

// 正确：滚动 main 元素
document.querySelector('main').scrollTop = targetValue;
```

### 4. 模块导航注意点

| 模块 | 导航注意点 |
|------|-----------|
| 3D展示 | 点击菜单后需等待 SVG 渲染，验证 h2 标题为"3D展示" |
| 土壤墒情 | main 内部滚动条，非 window |
| 历史数据 | 3个标签页（数据导出/数据对比/LSTM），每标签截1张 |
| 弹窗/模态框 | 有内部滚动条时需上下各截1张 |

### 5. 截图命名规范

```
{module}_{position}.png   — 页面级截图（top/middle/bottom/end）
{module}_{operation}.png   — 操作级截图（如 home_create_cooling.png）
```

## 常见问题

| 问题 | 原因 | 修复 |
|------|------|------|
| 3张截图内容完全相同 | 页面 main 有内部滚动条但滚动的是 window | 用 element.scrollTop 滚动 main |
| 截图显示错误的页面标题 | navigate 后没等页面完全加载 | 加 waitForTimeout(1500) + 验证标题 |
| 弹窗内只看到一半内容 | 弹窗有内部滚动条但只截了1张 | 弹窗内部滚动时顶部和底部各截1张 |

## 输出格式

审查完成后报告：PNG总数/缺失数/Vision MCP 抽查结果/需重新截图列表/已更新MD列表

## ChangeLogs

- [2026-04-14] 初始：基于本轮用户手册编写经验
