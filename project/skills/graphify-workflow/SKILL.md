---
name: graphify-workflow
description: Use graphify knowledge graph for token-efficient code navigation. After code changes, update the graph. Trigger: /graphify-workflow
user-invocable: true
argument-hint: "[query|update|viz|hook]"
---

# Graphify Workflow Skill

Use the project knowledge graph for efficient code navigation, and update it after code changes.

## Commands

### `/graphify-workflow query <question>`

Search the knowledge graph for module-level connections:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify query "<question>"
```

Options:
- `--dfs` — depth-first traversal (trace specific paths)
- `--budget N` — cap output at N tokens (default 2000)
- `--graph <path>` — custom graph.json path

Best for: "what calls function X", "which modules import Y", "trace the call chain for Z"

### `/graphify-workflow update`

Incrementally rebuild the graph (AST-only, no LLM needed). Uses SHA256 content-addressable cache — only changed files are re-extracted.

**Smart update**: Check git status first to decide whether rebuild is needed:
```bash
# Step 1: Check if code files changed
CODE_CHANGES=$(git diff --name-only HEAD -- '*.py' '*.js' '*.ts' | wc -l)

# Step 2: Skip if no code changes (only docs/config changed)
if [ "$CODE_CHANGES" -eq 0 ]; then
    echo "No code file changes — skipping graphify rebuild"
else
    echo "$CODE_CHANGES code files changed — rebuilding..."
    NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -c \
      "from graphify.watch import _rebuild_code; from pathlib import Path; print(_rebuild_code(Path('.')))"
fi
```

**Force rebuild** (skip git check):
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -c \
  "from graphify.watch import _rebuild_code; from pathlib import Path; print(_rebuild_code(Path('.')))"
```

Run after: adding/removing functions, creating new files, modifying imports.

After rebuild, regenerate visualization:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/gen_graph_viz.py
```

### `/graphify-workflow viz`

Regenerate the interactive HTML visualization:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/gen_graph_viz.py
```

Open: `start graphify-out/graph_viz.html`

### `/graphify-workflow hook`

Check graphify git hooks installation status:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify hook status
```

To install: `NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify hook install`

## When to Rebuild

| Scenario | Auto/Manual | Trigger | Skip condition |
|----------|------------|---------|----------------|
| git commit | Auto | post-commit hook | No code files in diff |
| git checkout/switch branch | Auto | post-checkout hook | No code files in diff |
| After bulk file changes | Manual | `/graphify-workflow update` | Only docs/config changed |
| After merge/rebase | Manual | `/graphify-workflow update` (force) | Never skip |
| Need latest visualization | Manual | `/graphify-workflow viz` | — |
| Session end with code changes | Reminder | SessionEnd hook | No code files changed |

## When to Use graphify vs Alternatives

| Scenario | Use graphify? | Alternative |
|----------|--------------|-------------|
| Find exact function definition | No | Grep directly |
| Find what modules call function X | Yes | graphify query |
| Understand cross-module dependencies | Yes | graphify query |
| Trace data flow through system | Yes | graphify query --dfs |
| Find all files related to feature Y | Yes | graphify query |
| Read specific file contents | No | Read tool |
| After creating/modifying files | Yes (update) | Manual grep |

## Token Savings

Graphify returns module-level connections, not full file contents:
- Current stats: see `graphify-out/GRAPH_REPORT.md` (authoritative source)
- vs ~50,000+ tokens to read all source files
- ~1423x compression ratio

## Five-Layer Integration

| Layer | Mechanism | Graphify Role |
|-------|-----------|---------------|
| L1 Hooks | git post-commit/post-checkout hooks | Auto-rebuild on code changes |
| L2 Rules | graphify.md | Usage norms, triggers, incremental strategy |
| L3 Skills | This skill | Operation steps and commands |
| L4 Subagents | context-research | graphify query in forked context |
| L5 Memory | GRAPH_REPORT.md | God nodes + community structure |

## Graph Stats

Check current stats: read `graphify-out/GRAPH_REPORT.md`
Source: `graphify-out/graph.json`
Visualization: `graphify-out/graph_viz.html` (vis.js interactive)