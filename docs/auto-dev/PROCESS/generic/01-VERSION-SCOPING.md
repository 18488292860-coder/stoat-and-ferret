# Version Scoping Process

## Purpose

Define the scope and objectives for a new version.

## Inputs

- ROADMAP.md - Current state and planned work
- BACKLOG.md - Prioritized work items
- Previous retrospectives - Learnings to apply

## Process

### 1. Review Current State

Examine:
- What was delivered in previous versions
- What's currently in progress
- What's been learned

### 2. Identify Objectives

Select 2-5 key objectives for the version:
- Must be achievable in reasonable timeframe
- Should address high-priority backlog items
- Consider dependencies between items

### 3. Define Success Criteria

For each objective:
- What does "done" look like?
- How will success be measured?
- What are the acceptance criteria?

### 4. Estimate Complexity

Assess:
- Technical complexity
- Risk factors
- Dependencies on external systems

### 5. Create Version Folder

```bash
mkdir -p docs/auto-dev/v{version}
```

## Outputs

- Version folder created
- ROADMAP.md updated with version entry
- Clear objectives and success criteria

## Next Step

[02-REQUIREMENTS.md](./02-REQUIREMENTS.md) - Document detailed requirements
