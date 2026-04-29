---
name: context-audit
description: 审计五层体系的 Token 预算、Rules/Skills/Memory 膨胀、双重加载问题。触发：/context-audit 或每季度第一天
user-invocable: true
---

# Context Audit — 上下文体系健康检查

## Checklist

### 1. Token 预算扫描

```bash
# 全局 rules token
for f in ~/.claude/rules/*.md; do wc -c < "$f"; done | paste -sd+ | bc

# 项目 rules token  
for f in .claude/rules/*.md; do wc -c < "$f"; done | paste -sd+ | bc

# MEMORY.md token
wc -c < memory/MEMORY.md
```

报告格式：
```
| 组件 | Bytes | ~Tokens | 占 500K% |
|------|-------|---------|---------|
| 全局 Rules (N) | X | ~Y | Z% |
| 项目 Rules (N) | X | ~Y | Z% |
| CLAUDE.md | X | ~Y | Z% |
| MEMORY.md | X | ~Y | Z% |
| 固定合计 | X | ~Y | Z% |
```

判断：固定合计 > 30% → **必须精简**

### 2. 双重加载检测

```bash
# Rules 重叠
comm -12 <(ls ~/.claude/rules/*.md | xargs -I{} basename {} | sort) \
         <(ls .claude/rules/*.md | xargs -I{} basename {} | sort)

# Skills 重叠（.claude/skills/ vs .agents/skills/）
comm -12 <(ls .claude/skills/ | sort) <(ls .agents/skills/ 2>/dev/null | sort)
```

判断：任何重叠 → **立即修复**

### 3. 单文件超限检测

```bash
# Rules > 300 行 or > 15KB
find ~/.claude/rules/ .claude/rules/ -name "*.md" -size +15k -exec wc -l {} \;

# Memory > 100 行
find memory/ -name "*.md" ! -name "MEMORY.md" -exec sh -c 'l=$(wc -l < "$1"); [ $l -gt 100 ] && echo "$1: $l lines"' _ {} \;
```

判断：任何文件超限 → **标记精简候选**

### 4. 内容重复检测

```bash
# Rules vs Memory 关键词重复
for r in .claude/rules/*.md; do
  name=$(basename "$r" .md)
  grep -rl "$name" memory/*.md 2>/dev/null | head -3
done
```

判断：重复率 > 30% → **合并或删除**

### 5. 计数一致性

检查以下文件中的计数是否一致：

| 文件 | 检查项 |
|------|--------|
| `subagents.md` | L2 Rules 计数 = `ls ~/.claude/rules/*.md | wc -l` + `ls .claude/rules/*.md | wc -l` |
| `subagents.md` | L3 Skills 项目自建计数 = 实际 `.claude/skills/` 中项目自建数量 |
| `config-review.md` | 同步范围表中的 rules 计数 = `.claude/rules/` 中文件数 |
| `MEMORY.md` | 索引行数 = `ls memory/*.md | wc -l` - 1（减去 MEMORY.md 自身） |

### 6. 输出报告

```markdown
## Context Audit Report — YYYY-MM-DD

### Token Budget
| 组件 | ~Tokens | 占比 | 状态 |
|------|---------|------|------|
| 全局 Rules (18) | ~22,936 | 4.6% | OK |
| 项目 Rules (19) | ~33,855 | 6.8% | OK |
| 固定合计 | ~110K | 22% | OK |

### Issues Found
| 问题 | 严重性 | 文件 | 建议 |
|------|--------|------|------|
| ... | ... | ... | ... |

### Action Items
- [ ] 精简 xxx.md（~Y tokens 可节省）
- [ ] 修复 yyy 计数不一致
```

### 7. 修复指引

发现问题后，按严重性顺序修复：

#### 🔴 双重加载修复（最优先）

**Rules 双重加载**：

```bash
# 确认哪边是权威版本
diff ~/.claude/rules/xxx.md .claude/rules/xxx.md

# 判断标准：
# - 内容引用项目路径（smart_energy.db、D:/Proj/energy/）→ 项目专用，删全局
# - 通用方法论，不引用项目路径 → 全局，删项目
rm ~/.claude/rules/xxx.md   # 删全局（项目专用 rule）
# 或
rm .claude/rules/xxx.md     # 删项目（通用 rule）
```

**Skills 双重加载**（Windows symlink 陷阱）：

```bash
# 必须用 ls -la 检查，普通 ls 看不出 symlink
ls -la .claude/skills/<name>
ls -la ~/.claude/skills/<name>

# 项目目录中的 skills 应该是 symlinks（-> ~/.claude/skills/<name>）
# 如果项目目录有真实 SKILL.md（非 symlink），说明有双重加载

# 正确状态：
# ~/.claude/skills/<name>/SKILL.md  ← 真实文件
# .claude/skills/<name>             ← symlink 指向上面

# 修复：从 hong-cc-practice 备份恢复真实目录到 ~/.claude/skills/
# 不要直接删除 ~/.claude/skills/<name>，会破坏项目 symlinks
cp -r ~/.claude-github/hong-cc-practice/project/skills/<name> ~/.claude/skills/<name>
```

#### 🟡 计数不一致修复

```bash
# 获取实际数量
echo "Global rules:" && ls ~/.claude/rules/*.md | wc -l
echo "Project rules:" && ls .claude/rules/*.md | wc -l
echo "Project skills:" && ls .claude/skills/ | wc -l
echo "Memory files:" && ls memory/*.md | grep -v MEMORY.md | wc -l
```

更新 `subagents.md` 中的 L2 Rules 行、L3 Skills 行、Memory 文件数。
更新 `config-review.md` 中的同步范围表。
更新 `memory/MEMORY.md` 索引（新增/删除对应行）。

#### 🟡 文件超限修复

```bash
# 找出可移出的内容（列表/速查表/触发矩阵）
# 1. 创建 memory/reference_xxx.md，写入被移出的内容
# 2. 在原文件中替换为一行指针：→ 见 memory/reference_xxx.md
# 3. 裁剪 ChangeLogs（保留最近 5 条）
# 4. 验证
wc -l <file> && wc -c <file>
```

精简策略优先级：
1. 移出到 Memory（详细列表/速查表）
2. 裁剪 ChangeLogs（保留最近 5 条）
3. 删除与其他 rule 重复的内容（留指针）
4. 拆分文件（内容超过 2 个独立主题时）
