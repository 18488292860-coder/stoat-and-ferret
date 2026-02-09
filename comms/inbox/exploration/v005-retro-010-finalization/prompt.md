Read AGENTS.md first and follow all instructions there.

## Task
Execute Task 010 (Finalization) for the stoat-and-ferret v005 post-version retrospective.

## Context
PROJECT=stoat-and-ferret, VERSION=v005

## Instructions

### 1. Verify All Previous Tasks Completed
Read the README.md from each task folder and verify it exists and indicates successful completion:

| Task | Folder | Required |
|------|--------|----------|
| 001 | comms/outbox/versions/retrospective/v005/001-environment/ | Yes |
| 002 | comms/outbox/versions/retrospective/v005/002-documentation/ | Yes |
| 003 | comms/outbox/versions/retrospective/v005/003-backlog/ | Yes |
| 004 | comms/outbox/versions/retrospective/v005/004-quality/ | Yes |
| 005 | comms/outbox/versions/retrospective/v005/005-architecture/ | Yes |
| 006 | comms/outbox/versions/retrospective/v005/006-learnings/ | Yes |
| 007 | comms/outbox/versions/retrospective/v005/007-proposals/ | Yes |
| 008 | comms/outbox/versions/retrospective/v005/008-closure/ | Yes |
| 009 | comms/outbox/versions/retrospective/v005/009-project-closure/ | Conditional (skipped - no VERSION_CLOSURE.md) |

If any required task is missing or reports failure, document the gap and STOP.

### 2. Run Final Quality Gates
Run quality gates using run_quality_gates tool. If ALL checks pass, proceed. If any fail, check if they were classified as code problems in Task 004 with corresponding backlog items from Task 007. If ALL failures have backlog item references, proceed. Otherwise STOP.

### 3. Commit All Closure Changes
Stage and commit all retrospective and closure changes:
```bash
git add comms/outbox/versions/retrospective/v005/
git add docs/
git add comms/
git commit -m "chore(v005): version closure housekeeping"
```

### 4. Push to Remote
```bash
git push
```
Note: If push fails due to GitHub server errors, retry once. If it still fails, document the error and proceed — the commit is local.

### 5. Mark Version Complete
Call complete_version(project="stoat-and-ferret", version_number=5)

### 6. Generate Final Summary
Create a summary of the entire retrospective process including total tasks executed, findings count, remediation actions, backlog items, learnings saved.

## Output Requirements
Save ALL outputs to `comms/outbox/exploration/v005-retro-010-finalization/`

### README.md (required)
First paragraph: Finalization summary (version officially closed / blocked with reason).
Then: Task Verification, Final Quality Gates, Commit SHA, Version Status, Retrospective Summary.

### final-summary.md
Complete retrospective summary with overview, verification results table, actions taken, outstanding items.

## Commit Instructions
After writing all output files, commit with message referencing folder: v005-retro-010-finalization

## Guidelines
- Do NOT proceed past a failing quality gate — STOP and report
- Do NOT skip the commit step
- Do NOT call complete_version if quality gates fail
- This task produces the final record of the retrospective — be thorough in the summary