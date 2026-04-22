# Hong's Claude Code Practice

Personal Claude Code configuration and extensibility practices — rules, skills, agents, and hooks accumulated from real-world project development. Currently applied to a [Flask-based IoT management platform](https://gitee.com/chg0901/energy) but the patterns are reusable across projects.

## Structure

```
.claude/
├── agents/            # L4 Subagent definitions (4 files)
│   ├── doc-writer.md
│   ├── mcp-search.md
│   ├── teams.md
│   └── test-runner.md
├── rules/             # L2 Rules (auto-loaded, 32 files)
│   ├── api-endpoints.md
│   ├── config-review.md
│   ├── context-hygiene.md
│   ├── context-management.md
│   ├── daily-review.md
│   ├── database.md
│   ├── database-patterns.md
│   ├── data-scripts.md
│   ├── design-templates.md
│   ├── extension-onboarding.md
│   ├── file-migration.md
│   ├── fireworks-tech-graph.md
│   ├── friday-review.md
│   ├── github-mcp-workflow.md
│   ├── graphify.md
│   ├── manual-testing.md
│   ├── mcp-servers.md
│   ├── mermaid.md
│   ├── proxy-rules.md
│   ├── search-workflow.md
│   ├── secrets.md
│   ├── subagents.md
│   ├── svg-design.md
│   ├── terminal.md
│   ├── testing.md
│   ├── todo-tracking.md
│   ├── training.md
│   ├── visual-long-screenshot.md
│   ├── visual-testing.md
│   ├── workflow.md
│   ├── work-summary-rules.md
│   └── zhihu-article.md
├── skills/            # L3 Skills (on-demand, 6 project-created)
│   ├── doc-trim/          # CLAUDE.md/rules trimming + README linkage
│   ├── graphify-workflow/  # Knowledge graph navigation & visualization
│   ├── interaction-check/  # JS interaction testing (Playwright)
│   ├── long-screenshot/    # Long page screenshot (Playwright)
│   ├── manual-review/      # User manual consistency check
│   └── visual-check/       # SVG/CSS rendering verification (Playwright + Vision MCP)
└── settings.json      # Project hooks (no secrets)
```

## Five-Layer Extensibility

| Layer | Mechanism | Count | Files |
|-------|-----------|-------|-------|
| L1 | Hooks | 5 | `settings.json` (project) + `~/.claude/settings.json` (global) |
| L2 | Rules | 32 | `rules/*.md` (auto-loaded per session) |
| L3 | Skills | 21 | 6 project-created + 5 third-party installed + 3 baidu/excalidraw/book2skills + 7 third-party symlinks |
| L4 | Subagents | 4 | `agents/*.md` (test-runner, doc-writer, mcp-search, teams) |
| L5 | Agent Teams | 3 templates | `agents/teams.md` (experimental) |
| Foundation | Memory | 17 files | `~/.claude/projects/d--Proj-energy/memory/` (not in this repo) |

## External Projects

These projects are referenced by rules but live outside the energy repo:

| Project | Location | Purpose | Referenced by |
|---------|----------|---------|---------------|
| [graphify](https://github.com/nicholasgasior/ggraph) | `D:/Proj/graphify/graphify/` | Code-to-knowledge-graph CLI tool (AST analysis) | `rules/graphify.md`, skill `graphify-workflow` |
| [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | `D:/Proj/design-templates/` | 58 brand design system DESIGN.md templates | `rules/design-templates.md` |

## Skills Not Synced (Third-Party)

### Installed (local clone, in `.claude/skills/`)

| Skill | Source | Purpose |
|-------|--------|---------|
| `book2skills` | Manual install | Convert book content to Claude Code skills |
| `create-colleague` | Manual install | Distill colleague capabilities into AI skill |
| `context-research` | Manual install | Deep codebase search with Explore agent |

### Symlinks (npx-managed, in `.agents/skills/` or `~/.claude/skills/`)

| Skill | Source | Purpose |
|-------|--------|---------|
| `book-study` | sanyuan-skills | Reading coach with spaced repetition |
| `code-review-expert` | sanyuan-skills | SOLID + security + performance code review |
| `sigma` | sanyuan-skills | Bloom 2-Sigma personalized AI tutor |
| `skill-forge` | sanyuan-skills | Meta-skill: create production-grade skills |
| `wiki-ingest` | sanyuan-skills | Compile documents into structured wiki |
| `web-access` | eze-is | Real Chrome CDP automation (login state, anti-scraping) |
| `fireworks-tech-graph` | yizhiyanhua-ai | Publication-quality SVG+PNG technical diagrams |

## Sync

From the energy project root:

```bash
bash scripts/sync_claude_config.sh          # commit only
bash scripts/sync_claude_config.sh --push   # commit + push to GitHub
```

Synced content:
- `agents/*.md` (4 agents)
- `rules/*.md` (32 rules)
- `skills/*/SKILL.md` (6 project-created skills only)
- `settings.json` (hooks config, no secrets)

## Excluded (never synced)

- `settings.local.json` - contains API tokens
- `book2skills/`, `create-colleague/`, `context-research/` - third-party installed skills
- Symlink skills (`book-study`, `code-review-expert`, `sigma`, `skill-forge`, `wiki-ingest`, `web-access`, `fireworks-tech-graph`)
- `memory/` - local machine state

## Related Repos

| Repo | Role | URL |
|------|------|-----|
| energy (Gitee) | Main project code + `.claude/` | `gitee.com/chg0901/energy` |
| energy (GitHub) | Read-only mirror of Gitee | `github.com/chg0901/energy` |
| hong-cc-practice | `.claude/` config backup | `github.com/chg0901/hong-cc-practice` |

## Changelog

### 2026-04-22
- Decoupled README from specific project name — now describes generic Claude Code practices
- Updated counts: rules 29->32 (+context-hygiene, +context-management, +github-mcp-workflow), skills 14->21 (+doc-trim, +excalidraw-diagram-generator, +baidu-search, +fireworks-tech-graph)
- Added doc-trim skill for automated CLAUDE.md/rules trimming

### 2026-04-17
- Updated README: rules 18->29, skills 4->14 (5 project + 3 installed + 6 symlinks + 1 global)
- Added External Projects section (graphify, design-templates)
- Added Skills Not Synced section with full third-party catalog
- Added Related Repos table

### 2026-04-14
- Initial: migrated from energy project .claude/ directory
- 18 rules, 4 agents, 4 skills, settings.json
