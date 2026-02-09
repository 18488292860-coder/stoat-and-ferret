# v005 Quality Gate Report

All quality gates pass. Ruff lint and format, mypy type checking, and pytest (627 tests) all succeed with zero failures. Test coverage is 93.28%, well above the 80% threshold. No fixes were needed.

## Initial Results

| Check | Status | Return Code | Details |
|-------|--------|-------------|---------|
| ruff check | PASS | 0 | All checks passed |
| ruff format | PASS | 0 | 103 files already formatted |
| mypy | PASS | 0 | No issues found in 44 source files |
| pytest | PASS | 0 | 627 passed, 15 skipped, 93.28% coverage |

## Failure Classification

No failures to classify. All checks passed on the initial run.

| Test | File | Classification | Action | Backlog |
|------|------|----------------|--------|---------|
| _(none)_ | — | — | — | — |

## Test Problem Fixes

No test problems detected. All 627 tests pass.

## Code Problem Deferrals

No code problems detected. All lint, type, and test checks pass.

## Final Results

No additional runs were needed. Initial results are the final results.

| Check | Status | Return Code | Details |
|-------|--------|-------------|---------|
| ruff check | PASS | 0 | All checks passed |
| ruff format | PASS | 0 | 103 files already formatted |
| mypy | PASS | 0 | No issues found in 44 source files |
| pytest | PASS | 0 | 627 passed, 15 skipped, 93.28% coverage |

## Outstanding Failures

None. All quality gates pass.

## Notes

- 15 tests were skipped (expected — these are platform-specific or require external dependencies like FFmpeg)
- 12 warnings relate to unclosed SQLite database connections in `test_blackbox/test_core_workflow.py::TestProjectLifecycle::test_create_and_retrieve_project` — these are ResourceWarnings, not test failures
- Test coverage of 93.28% significantly exceeds the 80% minimum threshold
