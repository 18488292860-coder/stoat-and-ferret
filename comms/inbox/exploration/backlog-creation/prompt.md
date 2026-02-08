Create backlog items and update plan.md based on the gap analysis at comms/outbox/exploration/backlog-gap-analysis/.

## Inputs

Read all documents in comms/outbox/exploration/backlog-gap-analysis/:
- README.md (overview)
- v004-testing-infrastructure.md
- v005-gui-shell.md
- v006-effects-engine.md
- v007-effect-workshop.md
- existing-backlog-mapping.md

## Tasks

### 1. Create new backlog items

For each suggested backlog item in the v004–v007 documents, create it using the `add_backlog_item` MCP tool.

**You MUST use the MCP `add_backlog_item` tool for every item. Do NOT directly edit BACKLOG.json or BACKLOG.md. Those files are managed by the MCP tools.**

Use `tool_help` if you need guidance on tool parameters.

### 2. Update existing backlog items

Per existing-backlog-mapping.md, use `update_backlog_item` to add version tags to existing items that should be associated with planned versions (e.g. adding `v004` tag to BL-009, BL-010, BL-012, BL-014, BL-016).

### 3. Update plan.md

Edit docs/auto-dev/plan.md to reflect the new backlog coverage:
- Update the "Backlog Integration" section or add notes about backlog items now existing for v004–v007
- Add any relevant scoping decisions from the gap analysis documents

## Output Requirements

Create findings in comms/outbox/exploration/backlog-creation/:

### README.md (required)
Summary of what was created: count of items per version, any issues encountered.

### items-created.md
List of all backlog items created with their IDs and titles, grouped by version.

## When Complete
git add comms/outbox/exploration/backlog-creation/
git commit -m "exploration: backlog-creation - created v004-v007 backlog items and updated plan.md"
