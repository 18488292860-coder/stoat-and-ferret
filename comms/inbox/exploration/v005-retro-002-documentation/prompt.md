Read AGENTS.md first and follow all instructions there.

## Objective

Verify all required documentation artifacts exist for every theme and feature in v005 of stoat-and-ferret.

## Tasks

### 1. Identify Version Structure
Read `comms/inbox/versions/execution/v005/THEME_INDEX.md` to determine all themes and features in this version.

### 2. Check Feature Completion Reports
For each feature in each theme, verify this file exists:
- `comms/outbox/versions/execution/v005/<theme>/<feature>/completion-report.md`

Record: feature path, exists (yes/no), status from report if exists.

### 3. Check Theme Retrospectives
For each theme, verify this file exists:
- `comms/outbox/versions/execution/v005/<theme>/retrospective.md`

Record: theme name, exists (yes/no).

### 4. Check Version Retrospective
Verify this file exists:
- `comms/outbox/versions/execution/v005/retrospective.md`

### 5. Check CHANGELOG.md
Read `docs/CHANGELOG.md` and verify a section for v005 exists with at least one entry.

### 6. Check version-state.json
Verify the version execution state file exists and contains correct version identifier and status field. Check `comms/state/version-executions/` for v005 state files.

## Output Requirements

Save outputs to comms/outbox/exploration/v005-retro-002-documentation/

### README.md (required)
First paragraph: Documentation completeness summary (X/Y artifacts present).

Then:
- **Completion Reports**: Table of feature -> status
- **Theme Retrospectives**: Table of theme -> status
- **Version Retrospective**: Present/missing
- **CHANGELOG**: Present/missing, has version section
- **Version State**: Present/missing, status value
- **Missing Artifacts**: List of all missing documents with full paths

### completeness-report.md
Detailed table:
| Artifact Type | Path | Exists | Notes |
|---------------|------|--------|-------|

Commit the results in v005-retro-002-documentation with message "docs(v005): retrospective task 002 - documentation completeness".