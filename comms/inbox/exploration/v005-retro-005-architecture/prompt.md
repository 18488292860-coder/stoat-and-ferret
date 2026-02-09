Read AGENTS.md first and follow all instructions there.

## Objective

Check whether changes implemented in v005 of stoat-and-ferret have caused drift from documented architecture. Update or create backlog items only if NEW drift is detected.

## Tasks

### 1. Check for Existing Open Architecture Backlog Items
Search for open backlog items related to architecture or C4:
- `list_backlog_items(project="stoat-and-ferret", status="open", search="C4")`
- `list_backlog_items(project="stoat-and-ferret", status="open", search="architecture")`
- `list_backlog_items(project="stoat-and-ferret", status="open", tags=["architecture"])`
- `list_backlog_items(project="stoat-and-ferret", status="open", tags=["c4"])`

If an open item exists, read its description and notes.

### 2. Gather Version Changes
Read version and theme retrospectives:
- `comms/outbox/versions/execution/v005/retrospective.md`
- All `comms/outbox/versions/execution/v005/<theme>/retrospective.md`

Extract new services/components, significant modifications, removals, and new dependencies.

### 3. Check Current Architecture Documentation
Read architecture docs if they exist:
- `docs/C4-Documentation/README.md`
- `docs/C4-Documentation/c4-context.md`
- `docs/C4-Documentation/c4-container.md`
- `docs/ARCHITECTURE.md`

Note what exists and what doesn't.

### 4. Detect Drift
Compare changes from step 2 against documentation from step 3.

### 5. Take Action Based on Findings
- If NO new drift: document "no additional architecture drift detected"
- If NEW drift AND existing open architecture backlog item: add note via `update_backlog_item`
- If NEW drift AND no existing item: create new backlog item via `add_backlog_item`

## Output Requirements

Save outputs to comms/outbox/exploration/v005-retro-005-architecture/

### README.md (required)
First paragraph: Architecture alignment summary.

Then:
- **Existing Open Items**: List of open architecture backlog items (or "none")
- **Changes in v005**: Summary of architectural changes made
- **Documentation Status**: What architecture docs exist and their currency
- **Drift Assessment**: Specific drift items found (or "no additional drift")
- **Action Taken**: What was done

Commit the results in v005-retro-005-architecture with message "docs(v005): retrospective task 005 - architecture alignment".