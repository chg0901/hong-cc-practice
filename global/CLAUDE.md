# Personal Preferences

## Code Style
- No emoji in code
- Generated files (png/pkl/csv) must include timestamp (YYYYMMDDHHmmss) in filename
- Check code with linting tools before running
- Never use `killall python3`

## Python Visualization
- Loss curves: use EMA smoothing with confidence interval bands
- One plot per metric, do not use subplot for training curves
- Do not use `ax.text()` in subplots; print summaries to terminal

## Git
- Do not use `git add .` without asking first
- Remove "Generated with Claude Code" and "Co-Authored-By" from commit messages
- Commit and push after finishing tasks to track progress

## Markdown Reports
- Use markdown links `[filename](path)` for file paths in text (not in code blocks)
- Figures must be followed by source link (mmd file or python script)
- Add bullet `-` before emoji lines in lists
- Add changelog at the end of updated markdown files
- Add usage/reproduction commands for code results
- Do not read image files directly
- Do not use `---` (horizontal rule) in markdown files — use blank lines for section separation instead

## Mermaid Diagrams
- Always create mermaid text AND render to PNG for flowcharts, architecture, sequence, and ER diagrams
- Store source in `mermaid/text/*.mmd`, output in `mermaid/figure/*.png`
- Render commands (always use `-s 2` for high DPI):
  - Landscape 16:9: `mmdc -i input.mmd -o output.png -t neutral -w 1600 -H 900 -s 2`
  - Portrait 3:4 (tall flowcharts): `mmdc -i input.mmd -o output.png -t neutral -w 1000 -H 1400 -s 2`
  - Landscape 4:3: `mmdc -i input.mmd -o output.png -t neutral -w 1200 -H 900 -s 2`
- Node colors with bold text: blue=#e1f5fe (start/end), navy=#e3f2fd (process), green=#c8e6c9 (success/output), orange=#fff3e0 (decision), red=#ffcdd2 (error/conflict), purple=#f3e5f5 (special)
- Use `classDef` + `stroke-width:2px,font-weight:bold` on all node types
- In reports: show figure inline, put mmd source in appendix; always include source link after image

## Long-Running Tasks
- Run training/long scripts with `nohup` + log file (allowed without permission)
- Before stopping a training, report status and let user decide
- Keep at most 5 checkpoints per training session

## File Organization
- Group multiple files for one task in a new folder
- Backup code before major updates; remove backup after validation
# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.
