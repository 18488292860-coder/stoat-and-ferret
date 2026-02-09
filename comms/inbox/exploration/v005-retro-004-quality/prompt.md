Read AGENTS.md first and follow all instructions there.

## Objective

Run all quality gate checks for stoat-and-ferret and verify the codebase passes after v005 completion. Attempt fixes for straightforward test failures only.

## Tasks

### 1. Run Quality Gates
Call `run_quality_gates(project="stoat-and-ferret")` and record the result for each check: pytest, ruff, mypy.

### 2. Evaluate Results
For each check, record: check name, pass/fail status, return code, key output (first 50 lines of failure output if failed).

### 3. Classify and Resolve Failures
If any check fails:

**Ruff/Mypy Failures — Classify by Location:**
- Errors in `tests/` files -> TEST PROBLEM. Fix directly.
- Errors in `src/` files -> CODE PROBLEM. Document for Task 007 backlog item creation. Do NOT auto-fix production code.

**Pytest Failures — Classify Each One:**
For each failing test, read BOTH the test code AND the production code it tests. Also read `comms/inbox/versions/execution/v005/THEME_INDEX.md` and relevant completion reports.

Apply this decision tree:
1. Does the test assert behavior intentionally changed by a feature in v005? -> TEST PROBLEM
2. Does the test reference APIs/parameters/types modified by v005? -> TEST PROBLEM
3. Does the test fail due to moved modules or renamed classes? -> TEST PROBLEM
4. Is the failing code path untouched by v005? -> Investigate further
5. Does the test pass in isolation but fail in the suite? -> TEST PROBLEM
6. None of the above? -> CODE PROBLEM

Fix test problems. Document code problems for Task 007.

### 4. Final Gate Check
After fixes, run `run_quality_gates` one final time. Max 2 additional rounds.

## Output Requirements

Save outputs to comms/outbox/exploration/v005-retro-004-quality/

### README.md (required)
First paragraph: Quality gate summary (all pass / X failures remain).

Then:
- **Initial Results**: Table of check -> pass/fail
- **Failure Classification**: Table of Test | File | Classification | Action | Backlog
- **Test Problem Fixes**: List of fixes applied with evidence citations
- **Code Problem Deferrals**: List of deferred items with classification reasoning
- **Final Results**: Table of check -> pass/fail after fixes
- **Outstanding Failures**: Detailed description of any remaining failures

### quality-report.md
Full quality gate output including complete output from each check run.

Commit the results in v005-retro-004-quality with message "docs(v005): retrospective task 004 - quality gates".