# Hong's Claude Code Configuration

This repo contains the `.claude/` directory configuration for a personal project.

## Structure

```
.claude/
├── agents/           # L4 Subagent definitions
│   ├── doc-writer.md
│   ├── mcp-search.md
│   ├── teams.md
│   └── test-runner.md
├── rules/            # L2 Rules (auto-loaded, 18 files)
│   ├── api-endpoints.md
│   ├── config-review.md
│   ├── data-scripts.md
│   ├── database.md
│   ├── file-migration.md
│   ├── graphify.md
│   ├── manual-testing.md
│   ├── mcp-servers.md
│   ├── mermaid.md
│   ├── proxy-rules.md
│   ├── secrets.md
│   ├── subagents.md
│   ├── svg-design.md
│   ├── terminal.md
│   ├── testing.md
│   ├── todo-tracking.md
│   ├── training.md
│   ├── visual-testing.md
│   ├── work-summary-rules.md
│   └── workflow.md
├── skills/           # L3 Skills (on-demand, 4 project skills)
│   ├── context-research/
│   ├── graphify-workflow/
│   ├── interaction-check/
│   └── visual-check/
└── settings.json     # Project hooks (no secrets)
```

## Five-Layer Extensibility

| Layer | Mechanism | Files |
|-------|-----------|-------|
| L1 | Hooks | `settings.json` (project) + `~/.claude/settings.json` (global) |
| L2 | Rules | `rules/*.md` (18 files, auto-loaded) |
| L3 | Skills | `skills/*/SKILL.md` (4 project skills) |
| L4 | Subagents | `agents/*.md` (4 agents) |
| L5 | Agent Teams | `agents/teams.md` (3 templates) |
| Foundation | Memory | `~/.claude/projects/d--Proj-energy/memory/` (not in this repo) |

## Sync

From the energy project root:

```bash
bash scripts/sync_claude_config.sh          # commit only
bash scripts/sync_claude_config.sh --push   # commit + push to GitHub
```

## Excluded (not synced)

- `settings.local.json` - contains API tokens
- `book2skills/` - third-party installed skills
- `create-colleague/` - third-party installed skills
- `memory/` - local machine state

## Changelog

### 2026-04-14
- Initial: migrated from energy project .claude/ directory
- 18 rules, 4 agents, 4 skills, settings.json
