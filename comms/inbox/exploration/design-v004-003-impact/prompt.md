You are performing Task 003: Impact Assessment for stoat-and-ferret version v004.

Read AGENTS.md first and follow all instructions there.

PROJECT=stoat-and-ferret
VERSION=v004

## Objective

Identify documentation, tooling, and process impacts that the planned version changes will produce, so they can be scoped as features during logical design rather than discovered post-implementation.

## Context

Tasks 001-002 gathered environment context and backlog details. This task reviews the planned changes and proactively identifies impacts before research investigation begins.

Read the backlog analysis outputs from the design artifact store:
- `comms/outbox/versions/design/v004/002-backlog/backlog-details.md` — backlog details and scope
- `comms/outbox/versions/design/v004/002-backlog/README.md` — backlog overview

## Tasks

### 1. Read Phase 1 Outputs

Read backlog analysis outputs from the design artifact store to understand all planned changes.

### 2. Generic Impact Checks

#### 2.1 tool_help Currency
For each MCP tool whose parameters or behavior will change based on the planned backlog items, check current tool_help content and identify if planned changes will make help text inaccurate.

#### 2.2 Tool Description Accuracy
For each MCP tool affected by planned changes, review tool descriptions and flag those that will become stale.

#### 2.3 Documentation Review
Identify which documentation files reference areas that will change. Search for references to changed tools, services, or APIs.

### 3. Project-Specific Impact Checks

Read `docs/auto-dev/IMPACT_ASSESSMENT.md` if it exists. If not, document "No project-specific impact checks configured" and continue.

### 4. Classify Impacts

For each identified impact, classify:
- **small**: Sub-task within existing feature (update help string, fix doc reference)
- **substantial**: Own feature in version plan (rewrite tool reference, update multiple guides)
- **cross-version**: Too large for this version, add to backlog

### 5. Generate Work Items

For each impact, produce a work item description for logical design (Task 005).

## Output Requirements

Create findings in comms/outbox/exploration/design-v004-003-impact/:

### README.md (required)

First paragraph: Summary of impact assessment — total impacts found, breakdown by classification.

Then:
- **Generic Impacts**: Count and categories
- **Project-Specific Impacts**: Count and categories (or "N/A")
- **Work Items Generated**: Total count by classification
- **Recommendations**: How impacts should be incorporated into logical design

### impact-table.md

Complete impact assessment table with columns: #, Area, Description, Classification, Resulting Work Item, Caused By.

### impact-summary.md

Summary by classification (Small Impacts, Substantial Impacts, Cross-Version Impacts).

## Guidelines

- ALL planned backlog items must be reviewed for impacts — do not skip any
- Be thorough but practical — only flag genuine impacts, not speculative ones
- Each impact must have a concrete work item
- Keep each document under 200 lines

## When Complete

After writing all output files to comms/outbox/exploration/design-v004-003-impact/, also COPY the README.md, impact-table.md, and impact-summary.md to comms/outbox/versions/design/v004/003-impact-assessment/ so they are in the design artifact store.

git add comms/outbox/exploration/design-v004-003-impact/
git commit -m "exploration: design-v004-003-impact - impact assessment complete"
