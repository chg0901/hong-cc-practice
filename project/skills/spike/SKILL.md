---
name: spike
description: |
  Quick experiment workflow for throwaway exploration. Isolates experiments in subagents
  so the main context stays clean. Record conclusions as Given/When/Then. Decide adopt or
  discard. Use for ML/LSTM exploration, tech selection, API feasibility checks, algorithm benchmarks.
when_to_use: |
  spike, experiment, quick test, try out, explore, feasibility, POC, proof of concept,
  benchmark, compare approaches, test hypothesis
allowed-tools:
  - Bash
  - Read
  - Write
  - Grep
  - Glob
---

# Spike — Quick Experiment Workflow

A spike is a time-boxed, throwaway experiment. The goal is to answer a specific question, not to produce production code.

## When to Use

- "Can library X do Y?" — API feasibility check
- "Which approach is faster?" — Algorithm benchmark
- "Does this LSTM architecture work with our data?" — ML exploration
- "Is this dependency worth adding?" — Tech evaluation
- "What does this error actually mean?" — Root cause isolation

## Workflow

### Step 1: Define the Spike (2 min)

Answer these questions before starting:

```
Spike: <one-line title>
Question: <what specific question are we answering?>
Timebox: <how long to spend, default 10 min>
Success criteria: <how do we know the answer?>
```

### Step 2: Execute in Isolation

Use a subagent to keep main context clean:

```
Execute this spike in a subagent:
- Create experiment code in .spike/<name>/
- Run the experiment
- Record raw results
- Do NOT modify any production files
```

### Step 3: Record Conclusions

Format results as Given/When/Then:

```markdown
## Spike: <title>

**Question**: <what we wanted to know>

**Given**: <setup/preconditions>
**When**: <what we did>
**Then**: <what happened>

**Verdict**: ADOPT | DISCARD | NEEDS_MORE

**Key findings**:
- Finding 1
- Finding 2
```

### Step 4: Decide

| Outcome | Action |
|---------|--------|
| ADOPT | Create a proper plan (superpowers:writing-plans) and implement |
| DISCARD | Delete `.spike/<name>/`, record why in work_summary |
| NEEDS_MORE | Schedule another spike with refined question |

## Rules

- **Never modify production files** during a spike
- **Time-box strictly** — default 10 min, max 30 min
- **One question per spike** — split complex questions into multiple spikes
- **Record even failures** — "doesn't work" is valuable information
- **Clean up** — delete spike code after recording conclusions

## File Organization

```
.spike/
├── <experiment-name>/
│   ├── spike.md          ← Question, results, verdict
│   ├── code/             ← Experiment code (throwaway)
│   └── output/           ← Generated artifacts
```

## Integration with Existing System

| Step | Existing Tool |
|------|--------------|
| Plan | superpowers:writing-plans (for ADOPT) |
| Execute | Agent tool with `context: fork` |
| Verify | test-runner agent |
| Document | doc-writer agent |
| Clean up | Bash `rm -rf .spike/<name>/` |
