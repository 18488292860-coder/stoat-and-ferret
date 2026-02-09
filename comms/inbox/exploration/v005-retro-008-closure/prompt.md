Read AGENTS.md first and follow all instructions there.

## Task
Execute Task 008 (Generic Closure) for the stoat-and-ferret v005 post-version retrospective.

## Context
PROJECT=stoat-and-ferret, VERSION=v005

## Instructions

### 1. Update plan.md
Read `docs/auto-dev/PLAN.md` (or `docs/auto-dev/plan.md`) and make these changes:
- Mark v005 as completed in the version list
- Update "Current Version" section to reflect the next planned version
- Move completed items from "Planned" to "Completed" section
If PLAN.md does not exist, document this in the output.

### 2. Verify CHANGELOG.md
Read `docs/CHANGELOG.md` and verify:
- v005 section exists with date
- Section contains categorized entries (Added, Changed, Fixed)
- Entries match the actual changes made (cross-reference with theme retrospectives)
If entries are missing or incorrect, add/fix them.

### 3. Review README.md
Read the project root `README.md` and check:
- Do any new capabilities from v005 need to be reflected?
- Are any described features now removed or changed?
- Is the project description still accurate?
If README needs updates: make the specific changes. If no changes needed: document "README current, no updates required."

### 4. Repository Cleanup
Verify repository state using git commands:
- Check for open PRs (all version-related PRs should be merged)
- Check for stale branches from this version
- Check working tree is clean
If stale branches exist, document them for cleanup.

## Output Requirements
Save ALL outputs to `comms/outbox/exploration/v005-retro-008-closure/`

### README.md (required)
First paragraph: Closure summary (X items completed, Y documents updated).

Then:
- Plan.md: Updated/not found/no changes needed
- CHANGELOG: Verified complete / entries added
- README: Updated / no changes needed
- Repository: Clean / stale branches found

### closure-report.md
Detailed report of all changes made: exact diffs for plan.md changes, CHANGELOG changes, README changes, and repository cleanup actions taken.

## Commit Instructions
After writing all files, commit with message referencing folder: v005-retro-008-closure

## Guidelines
- Make changes directly — do not propose and wait for approval
- For documentation review, check every document that could be affected
- If a document needs substantial rewriting, make the changes
- Do NOT create backlog items for documentation changes — just fix them