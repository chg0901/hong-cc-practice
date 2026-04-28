---
name: CLAUDE.md restructured — reference material in rules files
description: On 2026-04-04 CLAUDE.md was slimmed from 41KB to 10KB; API endpoints, LSTM config, changelog, project tree moved to separate files
type: project
---

CLAUDE.md was restructured on 2026-04-04 to stay under the 40KB performance limit.

**Where content moved:**
- API Endpoints → `.claude/rules/api-endpoints.md`
- LSTM Config + Data Processing + Visualization → `.claude/rules/data-scripts.md`
- Full Change Log → `docs/changelog.md` (CLAUDE.md keeps only the latest 1 entry + link)
- Project Structure tree → removed (derivable via `ls`/`Glob`)
- EDA Data Sources → merged into `.claude/rules/data-scripts.md`

**Why:** CLAUDE.md is loaded into every conversation's context window. At 41KB it exceeded the 40KB threshold causing a performance warning. Reference-only material (API lists, config params, historical changelog) is rarely needed for current tasks.

**How to apply:**
- When adding new API endpoints, update `.claude/rules/api-endpoints.md` (not CLAUDE.md)
- When adding changelog entries, add full entry to `docs/changelog.md` and only a 1-2 line summary to CLAUDE.md
- The changelog/work-summary rules are in `.claude/rules/work-summary-rules.md` (merged from changelog.md + doc-structure.md on 2026-04-13)
- Do not re-add project structure trees or long reference lists to CLAUDE.md
