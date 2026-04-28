---
name: Agent memory mechanism best practices
description: Zhipu official recommendations for Agent memory management (instructional vs learned, layered management, verifiable rules)
type: feedback
---

# Agent Memory Mechanism Best Practices

Source: [智谱记忆机制文档](https://docs.bigmodel.cn/cn/coding-plan/learning-resources/memory-mechanism)

## Core Distinction: Instructional vs Learned Memory

**Why**: Claude Code has two distinct memory types that serve different purposes. Mixing them leads to bloated CLAUDE.md and stale memory.

- **Instructional memory** (CLAUDE.md + rules/*.md) = written by humans, tells Agent "how to do things"
  - Coding conventions, build commands, test workflows, naming style, commit rules
  - These are **rules** that should be followed exactly
- **Learned memory** (memory/*.md + MEMORY.md) = accumulated from interactions, user corrections, preferences, failures
  - User's preferences ("don't use emoji"), gotchas discovered ("Windows segfault on keras import")
  - These are **experiences** that inform behavior

**How to apply**: When adding new information, ask: "Is this a rule I'm imposing, or something I learned from experience?" Rules go to rules/, experiences go to memory/.

## Layered Management

**Why**: Avoids noise in context and keeps rules maintainable.

| Layer | Location | Scope | Example |
|-------|----------|-------|---------|
| Project | CLAUDE.md + rules/ | All team members, in Git | API conventions, test rules |
| Organization | settings.json | IT/DevOps enforced | Security policies |
| User | ~/.claude/CLAUDE.md | Personal only | Personal coding style |
| Local | .local.md | Machine-specific | Path configurations |
| Role/Agent | skills/, agents/ | Specific agent type | visual-check skill |

**How to apply**: Project rules (shared, in Git) go to `.claude/rules/`. User preferences (personal) go to `~/.claude/` or `memory/`. Don't mix them.

## Rule Writing Principles

**Why**: Vague rules lead to inconsistent Agent behavior. Specific rules reduce interpretation space.

1. **Be specific and verifiable**: "Use `NO_PROXY=127.0.0.1,localhost` before Python tests" not "avoid proxy issues"
2. **Keep main file under 200 lines**: CLAUDE.md should be concise; details go to rules/*.md
3. **Use Markdown structure**: Headings + lists for readability
4. **Consistent formatting**: Same pattern for similar rules (see this project's rules/ directory)

**How to apply**: Before writing a new rule, check if it's specific enough to be unambiguous. If it could be interpreted multiple ways, make it more precise.

## Mapping to This Project

This project already follows these principles well:
- CLAUDE.md = project-level instructional memory (architecture, commands, DB schema)
- .claude/rules/ = modular rules by topic (testing.md, database.md, mcp-servers.md, etc.)
- memory/ = learned experiences (feedback_*.md, reference_*.md, project_*.md)
- MEMORY.md = index (under 200 lines)

**Gap (2026-04-16)**: Some rules/*.md files remain long: mermaid.md ~243 lines, mcp-servers.md ~237 lines, visual-testing.md ~144 lines. Consider whether mermaid.md or mcp-servers.md should be further split by subtopic.
