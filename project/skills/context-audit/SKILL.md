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
