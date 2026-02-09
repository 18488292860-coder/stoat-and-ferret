Read AGENTS.md first and follow all instructions there.

## Task
Execute Task 007 (Stage 1 Proposals) for the stoat-and-ferret v005 post-version retrospective.

## Context
PROJECT=stoat-and-ferret, VERSION=v005

## Instructions

### 1. Read All Previous Task Outputs
Read the README.md from each completed task:
- comms/outbox/versions/retrospective/v005/001-environment/README.md
- comms/outbox/versions/retrospective/v005/002-documentation/README.md
- comms/outbox/versions/retrospective/v005/003-backlog/README.md
- comms/outbox/versions/retrospective/v005/004-quality/README.md
- comms/outbox/versions/retrospective/v005/005-architecture/README.md
- comms/outbox/versions/retrospective/v005/006-learnings/README.md

Read detailed reports where the README references them.

### 2. Identify All Findings Requiring Action
From each task, extract items that need remediation:
- 001: Environment blockers (stale branches, dirty working tree)
- 002: Missing documentation artifacts
- 003: Backlog items that failed to complete
- 004: Quality gate failures not yet fixed
- 005: Architecture drift items (already handled via backlog — reference only)
- 006: Learning extraction issues (unlikely but check)

### 3. Create Backlog Items for Quality Gate Code Problems
For each failure classified as a "code problem" in Task 004's README.md, create a backlog item using add_backlog_item. If Task 004 reports no code problems, skip this section and note "No quality gate backlog items needed."

### 4. Create Proposals Document
For each finding, use this exact format:

```markdown
### Finding: [Brief description]

**Source:** Task [NNN] - [task name]

**Current State:**
[What exists now — specific file paths and content]

**Gap:**
[What's missing or incorrect]

**Proposed Actions:**
- [ ] [Action 1]: `exact/file/path` — [exact change to make]
- [ ] [Action 2]: `exact/file/path` — [exact change to make]

**Auto-approved:** Yes
```

Anti-Patterns (DO NOT USE): "Review and update if needed", "Consider adding documentation", "May need to check X", "Should probably update Y". Every action must specify: exact file path, exact change, no ambiguity.

### 5. Categorize Proposals
Group proposals by type:
- Immediate Fixes: Can be executed by the remediation exploration
- Backlog Items: Already created by previous tasks (reference only)
- User Action Required: Items requiring human intervention

### 6. Summary Statistics
Count: Total findings across all tasks, findings with no action needed, immediate fix proposals, backlog items created/updated, user actions required.

## Output Requirements

Save ALL outputs to `comms/outbox/exploration/v005-retro-007-proposals/`

### README.md (required)
First paragraph: Proposals summary (X findings, Y immediate fixes, Z backlog items, W user actions).

Then: Status by Task table, Immediate Fixes list, Backlog References, User Actions.

### proposals.md
Complete proposals document with all findings in Crystal Clear Actions format.

## Commit Instructions
After writing all files, commit with message referencing folder: v005-retro-007-proposals

## Allowed MCP Tools
- read_document
- add_backlog_item