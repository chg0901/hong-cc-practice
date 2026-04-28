---
name: feedback_no_claude_coauthor
description: Never show Claude as collaborator/co-author in git commits
type: feedback
---

Do NOT show Claude as collaborator or co-author in any git commits. This applies to ALL repositories, including forks and PRs.

**Why**: User preference — commits should appear as authored solely by the human contributor, without AI tool attribution.

**How to apply**:
- Never add `Co-Authored-By: Claude` lines to commit messages
- Never add `Generated with Claude Code` or similar AI attribution
- This applies to `git commit -m`, `gh pr create`, and any other commit/PR creation
- Also applies to CLAUDE.md global instructions: "Remove 'Generated with Claude Code' and 'Co-Authored-By' from commit messages"
