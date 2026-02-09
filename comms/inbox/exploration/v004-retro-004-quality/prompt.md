Read AGENTS.md first and follow all instructions there.

## Objective

Run all quality gate checks for stoat-and-ferret and verify the codebase passes after version v004 completion. Attempt fixes for straightforward test failures.

## Tasks

### 1. Run Quality Gates
Call `run_quality_gates(project="stoat-and-ferret")` and record the result for each check: pytest, ruff, mypy.

### 2. Evaluate Results
For each check, record: check name, pass/fail status, return code, key output (first 50 lines of failure output if failed).

### 3. Classify and Resolve Failures
If any check fails:

**Ruff/Mypy Failures — Classify by Location:**
- Errors in tests/ files → TEST PROBLEM. Fix directly.
- Errors in src/ files → CODE PROBLEM. Document for backlog item creation. Do NOT auto-fix production code.

**Pytest Failures — Classify Each One:**
For each failing test, read BOTH the test code AND the production code it tests. Also read the version's THEME_INDEX.md and relevant completion reports.

Apply this decision tree:
1. Does the test assert behavior intentionally changed by v004? If YES → TEST PROBLEM.
2. Does the test reference APIs/parameters modified by v004? If YES → TEST PROBLEM.
3. Does the test fail due to moved modules from v004? If YES → TEST PROBLEM.
4. Is the failing code path untouched by v004? If YES → investigate further.
5. Does the test pass in isolation but fail in suite? If YES → TEST PROBLEM.
6. None of the above? Default: CODE PROBLEM.

Fix test problems. Document code problems for Task 007 backlog creation.

### 4. Final Gate Check
After all test-problem fixes, run `run_quality_gates` one final time. Max 2 additional rounds.

## Output Requirements

Save outputs to comms/outbox/exploration/v004-retro-004-quality/:

### README.md (required)
First paragraph: Quality gate summary (all pass / X failures remain).

Then:
- **Initial Results**: Table of check to pass/fail
- **Failure Classification**: Table of Test, File, Classification, Action, Backlog
- **Test Problem Fixes**: List of fixes applied with evidence citations
- **Code Problem Deferrals**: List of deferred items with classification reasoning
- **Final Results**: Table of check to pass/fail after fixes
- **Outstanding Failures**: Detailed description of any remaining failures

### quality-report.md
Full quality gate output including complete output from each check run, diff of any fixes applied, before/after comparison if fixes were made.

Do NOT commit — the calling prompt handles commits. Results folder: v004-retro-004-quality.