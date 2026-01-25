# Starter Prompt Generation Process

## Purpose

Create a comprehensive prompt for autonomous execution.

## Inputs

- REQUIREMENTS.md - Success criteria
- IMPLEMENTATION_PLAN.md - Stages and tasks

## Process

### 1. Synthesize Context

Combine:
- Objectives and deliverables
- Success criteria
- Prerequisites

### 2. Structure Stages

For each stage include:
- Clear objective
- Detailed tasks
- Verification commands
- Commit instructions

### 3. Add Prerequisites Check

Commands to verify starting state:
- Git status
- Dependencies installed
- Tests passing

### 4. Include Final Verification

Commands to verify completion:
- All tests pass
- Linting clean
- Type checking passes

### 5. Create Checklist

Summary checklist for tracking:
- All stages listed
- Gates identified

## Document Template

```markdown
# {Version} Implementation - {Title}

## Objective
{Clear statement}

**Key deliverables:**
1. {Deliverable}

**Success criteria:**
- {Criterion}

---

## Prerequisites Check
```bash
{verification commands}
```

---

## STAGE 0: {Name}

### Objective
{What this achieves}

### Tasks
1. {Detailed task}

### Verification
```bash
{commands}
```

### Commit
```bash
git add -A
git commit -m "{message}"
```

---

## Final Verification
```bash
{final checks}
```

## Checklist
- [ ] Stage 0
- [ ] All gates passing
```

## Outputs

- `v{version}/STARTER_PROMPT.md` created
- Ready for autonomous execution

## Next Step

[05-IMPLEMENTATION.md](./05-IMPLEMENTATION.md) - Execute the implementation
