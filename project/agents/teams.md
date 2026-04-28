# Agent Team Templates

## When to Use Agent Teams

Enable via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in settings.json env.

**Use when**:
- Multi-module parallel development (frontend + backend + test simultaneously)
- Competitive hypothesis debugging (2-3 agents testing different theories)
- Cross-repository code search (parallel search across multiple repos)

**Do NOT use when**:
- Simple tasks (overhead > benefit)
- Tasks with strong dependencies (need sequential execution)
- Token budget limited (multi-agent consumes significantly more)

## Template 1: Full-Stack Feature (3 agents)

Use when implementing a feature touching frontend + backend + tests.

| Role | Agent | Files | Task |
|------|-------|-------|------|
| Backend | subagent (inherit) | `*.py`, API routes | Implement API endpoints, database changes |
| Frontend | main conversation | `static/**`, `js/**`, `css/**` | Implement UI components, SVG |
| QA | test-runner (haiku) | `test_codes/*.py` | Write and run tests |

## Template 2: Research + Implementation (2 agents)

Use when exploring unfamiliar code before implementing changes.

| Role | Agent | Task |
|------|-------|------|
| Researcher | context-research (fork) | Deep codebase exploration, dependency mapping |
| Implementer | main conversation | Implement based on research findings |

## Template 3: Competitive Debugging (2-3 agents)

Use when debugging with multiple hypotheses.

| Role | Agent | Task |
|------|-------|------|
| Hypothesis A | main conversation | Test hypothesis A |
| Hypothesis B | subagent (fork) | Test hypothesis B in parallel |
| Validator | test-runner (haiku) | Run regression tests after fix |

## Cross-Layer Integration

Agent Teams (L5) coordinate with other layers via Hooks (L1):

| Hook | Event | Integration with Teams |
|------|-------|----------------------|
| SubagentStart | Agent launch | Inject project context into each team member |
| SubagentStop | Agent completion | Validate output, trigger follow-up actions |
| PreToolUse | Before Bash/Edit | Safety check on all agents' operations |
| PostToolUse | After Edit/Write | Auto-trigger /visual-check for UI changes |
| SessionEnd | Session end | Consolidate all agents' results, auto-commit docs |
