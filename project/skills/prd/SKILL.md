---
name: prd
description: |
  Product Requirements Document workflow for structured feature development.
  Use before implementing new features (>3 files), API endpoints, or database schema changes.
  Trigger: "write PRD", "create PRD", "new feature requirements", "product requirements".
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
---

# PRD — Product Requirements Document

A lightweight PRD workflow adapted from BMAD-METHOD. Simplified for single-project use — no 6-role Agile, just structured requirements.

## When to Use

- New feature development involving >3 files
- Database schema changes
- New API endpoints
- Cross-module changes
- When superpowers:writing-plans needs more structure

## When NOT to Use

- Bug fixes (use systematic-debugging)
- Simple UI tweaks (use /visual-check)
- Quick experiments (use /spike)

## PRD Template

Generate the following sections when invoked:

```markdown
## PRD: <feature-name>

### 1. Problem Statement
- What problem does this solve?
- Who is affected?
- Current pain points

### 2. User Stories
- As a <role>, I want <action>, so that <benefit>
- Acceptance criteria for each story

### 3. Acceptance Criteria
- [ ] Criterion 1 (must be testable)
- [ ] Criterion 2
- [ ] Criterion 3

### 4. Technical Constraints
- Affected modules / files
- Database changes (if any)
- API changes (if any)
- Dependencies

### 5. API Design (if applicable)
- Endpoint: METHOD /api/xxx
- Request body / params
- Response format
- Error codes

### 6. Data Model Changes (if applicable)
- New/modified tables
- Migration strategy (see goal-backward.md)
- Backward compatibility

### 7. Test Plan
- API tests (test_codes/)
- Manual UI tests (see manual-testing.md)
- Verification criteria
```

## Workflow

```
1. Describe the feature in natural language
2. Fill in PRD template sections
3. Review with user (confirm scope)
4. Convert to implementation plan (superpowers:writing-plans)
5. Execute (superpowers:executing-plans)
```

## Integration with Five-Layer System

| Step | Five-Layer Component |
|------|---------------------|
| Write PRD | This skill (L3) |
| Plan implementation | superpowers:writing-plans |
| Handle deviations | deviation-handling.md |
| Verify completion | verification-before-completion |
| Document changes | doc-writer agent |

## Pre-commit Strategic Review (plan-ceo-review)

Before committing, check:

| Dimension | Question |
|-----------|----------|
| Scope | Is this change within the original PRD scope? |
| Strategy | Is the approach sound? Any better alternatives? |
| Impact | Does this affect other modules? Need联动更新? |
| Risk | Any rollback risk? Data loss potential? |

This complements code-review-expert (code-level) with strategic-level review.
