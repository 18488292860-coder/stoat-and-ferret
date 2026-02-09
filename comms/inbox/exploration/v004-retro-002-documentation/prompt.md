Read AGENTS.md first and follow all instructions there.

## Objective

Verify all required documentation artifacts exist for every theme and feature in stoat-and-ferret version v004.

## Tasks

### 1. Identify Version Structure
Read `comms/inbox/versions/execution/v004/THEME_INDEX.md` to determine all themes and features in this version.

### 2. Check Feature Completion Reports
For each feature in each theme, verify this file exists:
- `comms/outbox/versions/execution/v004/<theme>/<feature>/completion-report.md`

Record: feature path, exists (yes/no), status from report if exists.

### 3. Check Theme Retrospectives
For each theme, verify this file exists:
- `comms/outbox/versions/execution/v004/<theme>/retrospective.md`

Record: theme name, exists (yes/no).

### 4. Check Version Retrospective
Verify this file exists:
- `comms/outbox/versions/execution/v004/retrospective.md`

### 5. Check CHANGELOG.md
Read `docs/CHANGELOG.md` and verify:
- A section for v004 exists
- Section contains at least one entry

### 6. Check version-state.json
Verify `comms/state/version-executions/stoat-and-ferret-v004-exec-1770592736/version-state.json` exists and contains correct version identifier and status.

## Output Requirements

Save outputs to comms/outbox/exploration/v004-retro-002-documentation/:

### README.md (required)
First paragraph: Documentation completeness summary (X/Y artifacts present).

Then:
- **Completion Reports**: Table of feature to status
- **Theme Retrospectives**: Table of theme to status
- **Version Retrospective**: Present/missing
- **CHANGELOG**: Present/missing, has version section
- **Version State**: Present/missing, status value
- **Missing Artifacts**: List of all missing documents with full paths

### completeness-report.md
Detailed table of all artifacts checked with paths, existence status, and notes.

Do NOT commit — the calling prompt handles commits. Results folder: v004-retro-002-documentation.