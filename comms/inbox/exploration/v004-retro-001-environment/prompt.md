Read AGENTS.md first and follow all instructions there.

## Objective

Verify the environment is ready for retrospective execution for stoat-and-ferret version v004.

## Tasks

### 1. Git Status
Run git status and verify:
- Working tree state (clean or dirty)
- Current branch (should be main)
- Remote sync status

If working tree is dirty, document all uncommitted files.

### 2. PR Status
Check for open PRs: run `git_read(project="stoat-and-ferret", operation="prs", state="open")` and verify no open PRs related to v004 remain.

### 3. Version Execution Status
Read the version state file at `comms/state/version-executions/stoat-and-ferret-v004-exec-1770592736/version-state.json` and verify:
- Version status is "completed"
- All 5 themes (test-foundation, blackbox-contract, async-scan, security-performance, devex-coverage) show completed status
- All 15 features completed

### 4. Branch Verification
Run `git_read(project="stoat-and-ferret", operation="branches")` and verify no stale feature branches from v004 remain.

### 5. Health Check
Run `health_check()` and document the result.

## Output Requirements

Save outputs to comms/outbox/exploration/v004-retro-001-environment/:

### README.md (required)
First paragraph: Environment status summary (ready/blocked) with key findings.

Then:
- **Health Check**: Pass/fail with details
- **Git Status**: Branch, working tree state, remote sync
- **Open PRs**: List or "none"
- **Version Status**: Execution state summary
- **Stale Branches**: List or "none"
- **Blockers**: Any issues preventing retrospective continuation

### environment-report.md
Detailed results from all checks, including raw outputs.

Do NOT commit — the calling prompt handles commits. Results folder: v004-retro-001-environment.