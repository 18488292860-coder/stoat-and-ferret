# Retrospective Process

## Purpose

Review the implementation and capture learnings.

## Inputs

- Completed implementation
- Commit history
- Any issues encountered

## Process

### 1. Summarize Delivery

Document:
- What was delivered
- Key metrics (LOC, coverage, etc.)
- Timeline (if relevant)

### 2. Review Key Decisions

For significant decisions:
- What was the context?
- What was decided?
- What was the rationale?
- What was the outcome?

### 3. Capture Learnings

Identify:
- What went well (to repeat)
- What could improve
- Patterns discovered
- Pitfalls encountered

### 4. Define Action Items

Create actionable items for:
- Process improvements
- Technical debt
- Future enhancements

### 5. Update LEARNINGS.md

Move relevant learnings to the project's LEARNINGS.md.

## Document Template

```markdown
# Version {version} Retrospective

## Summary
{What was delivered}

## Deliverables
| Deliverable | Status | Notes |
|-------------|--------|-------|
| {Item} | Complete | {Notes} |

## Key Decisions
### {Decision}
**Context:** {Why needed}
**Choice:** {What chosen}
**Outcome:** {Result}

## Metrics
- Lines of code: {n}
- Test coverage: {%}
- Stages: {n}

## Learnings
### What Went Well
- {Item}

### What Could Improve
- {Item}

## Action Items
- [ ] {Action}
```

## Outputs

- `v{version}/RETROSPECTIVE.md` created
- LEARNINGS.md updated

## Next Step

[07-VERSION-CLOSE.md](./07-VERSION-CLOSE.md) - Close the version
