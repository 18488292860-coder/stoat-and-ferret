Read AGENTS.md first and follow all instructions there.

## Objective

Verify all retrospective tasks completed for stoat-and-ferret version v004, run final quality gates, and produce the finalization summary. Do NOT commit or push — the calling prompt handles that.

## Tasks

### 1. Verify All Previous Tasks Completed
Read the README.md from each task folder and verify it exists and indicates successful completion:

| Task | Folder | Required |
|------|--------|----------|
| 001 | comms/outbox/versions/retrospective/v004/001-environment/ | Yes |
| 002 | comms/outbox/versions/retrospective/v004/002-documentation/ | Yes |
| 003 | comms/outbox/versions/retrospective/v004/003-backlog/ | Yes |
| 004 | comms/outbox/versions/retrospective/v004/004-quality/ | Yes |
| 005 | comms/outbox/versions/retrospective/v004/005-architecture/ | Yes |
| 006 | comms/outbox/versions/retrospective/v004/006-learnings/ | Yes |
| 007 | comms/outbox/versions/retrospective/v004/007-proposals/ | Yes |
| 008 | comms/outbox/versions/retrospective/v004/008-closure/ | Yes |
| 009 | comms/outbox/versions/retrospective/v004/009-project-closure/ | Conditional (skipped) |

### 2. Run Final Quality Gates
Call run_quality_gates(project="stoat-and-ferret") and verify all checks pass.

If any checks fail, check if they have corresponding backlog items from Task 007.

### 3. Generate Final Summary
Create a summary of the entire retrospective process:
- Total tasks executed
- Findings count by category
- Remediation actions taken
- Backlog items created/completed
- Learnings saved

## Output Requirements

Save outputs to comms/outbox/exploration/v004-retro-010-finalization/:

### README.md (required)
First paragraph: Finalization summary (version officially closed / blocked with reason).

Then:
- **Task Verification**: All tasks confirmed complete
- **Final Quality Gates**: All passed
- **Retrospective Summary**: High-level metrics

### final-summary.md
Complete retrospective summary with overview, verification results table, actions taken, and outstanding items.

Do NOT commit or push — the calling prompt handles commits. Results folder: v004-retro-010-finalization.