The v004 design orchestrator (exploration design-v004-1770582098400) timed out after completing tasks 001-007. Tasks 008 and 009 were never launched.

Your job: pick up where it left off.

1. Read `docs/auto-dev/PROMPTS/design_version_prompt/000-master-prompt.md` to understand the full workflow
2. Review the completed sub-exploration results to understand current state:
   - design-v004-001-environment through design-v004-007-drafts (all complete, all committed)
3. Execute Task 008 (persist documents) and Task 009 (pre-execution validation) per the master prompt
4. After Task 009, do the commit described in the master prompt for phases 1-2 if it wasn't already done, and run the final completion steps

PROJECT=stoat-and-ferret, VERSION=v004.

## Output Requirements

Create findings in comms/outbox/exploration/design-v004-resume/:

### README.md (required)
First paragraph: What was done to complete the v004 design.

### resume-log.md
Task execution log with exploration IDs and outcomes.

## When Complete
git add comms/outbox/exploration/design-v004-resume/
git commit -m "exploration: design-v004-resume - completed tasks 008-009"
