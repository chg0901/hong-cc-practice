---
name: graphify-workflow
description: Use graphify knowledge graph for token-efficient code navigation. After code changes, update the graph. Trigger: /graphify-workflow
user-invocable: true
argument-hint: "[query|update|viz]"
---

# Graphify Workflow Skill

Use the project knowledge graph for efficient code navigation, and update it after code changes.

## Commands

### `/graphify-workflow query <question>`

Search the knowledge graph for module-level connections:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify query "<question>"
```

Best for: "what calls function X", "which modules import Y", "trace the call chain for Z"

### `/graphify-workflow update`

Incrementally update the graph after code changes:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe -m graphify --update
```

Run after: adding/removing functions, creating new files, modifying imports

### `/graphify-workflow viz`

Regenerate the interactive HTML visualization:
```bash
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/gen_graph_viz.py
```

Open: `start graphify-out/graph_viz.html`

## When to Use

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
- 1875 nodes / 2907 edges compressed into ~2000 tokens
- vs ~50,000+ tokens to read all source files
- ~1423x compression ratio

## Graph Stats

Current: 1875 nodes, 2907 edges, 149 communities
Source: `graphify-out/graph.json`
Visualization: `graphify-out/graph_viz.html` (vis.js interactive)
