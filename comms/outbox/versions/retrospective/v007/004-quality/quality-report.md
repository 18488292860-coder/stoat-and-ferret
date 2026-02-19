# v007 Quality Gate Full Report

## Run Configuration

- **Project:** stoat-and-ferret
- **Version:** v007
- **Version start commit:** `5634ab3` (v006 finalization)
- **Python files changed:** 13
- **Run mode:** Full (Python files changed)

## Quality Gate Run: `run_quality_gates(project="stoat-and-ferret")`

### ruff (PASS)

- **Return code:** 0
- **Duration:** 0.05s
- **Output:**
```
All checks passed!
```

### mypy (PASS)

- **Return code:** 0
- **Duration:** 0.41s
- **Output:**
```
Success: no issues found in 49 source files
```

### pytest (PASS)

- **Return code:** 0
- **Duration:** 21.57s
- **Tests collected:** 884
- **Output (summary):**
```
platform win32 -- Python 3.13.11, pytest-9.0.2, pluggy-1.6.0
rootdir: C:\Users\grant\Documents\projects\stoat-and-ferret
configfile: pyproject.toml
testpaths: tests
plugins: anyio-4.12.1, hypothesis-6.151.5, asyncio-1.3.0, cov-7.0.0

884 tests collected, all passed.
```

## Unconditional Test Categories

### Golden Scenarios

- **Status:** N/A
- **Reason:** `tests/system/scenarios/` directory does not exist in this project. No golden scenario tests are configured.

### Contract Tests (`tests/test_contract/`)

- **Command:** `uv run pytest tests/test_contract/ -v`
- **Result:** 30 passed, 11 skipped
- **Output (summary):**
```
tests/test_contract/test_ffmpeg_contract.py - 11 skipped (conditional: FFmpeg availability)
tests/test_contract/test_repository_parity.py - 15 passed
tests/test_contract/test_search_parity.py - 15 passed
```

### Parity Tests

- **Command:** `uv run pytest tests/test_contract/test_repository_parity.py tests/test_contract/test_search_parity.py -v`
- **Result:** 15 passed
- **Note:** Parity tests are located within `tests/test_contract/` in this project.

## Fixes Applied

None. All checks passed on the first run.

## Before/After Comparison

Not applicable — no fixes were needed.

## Summary

All quality gates passed cleanly on the first run. The v007 codebase is in good shape:

- **Static analysis (ruff):** No lint violations
- **Type checking (mypy):** No type errors across 49 source files
- **Test suite (pytest):** 884 tests passing
- **Contract tests:** 30 passed (11 conditionally skipped)
- **Parity tests:** 15 passed

No test problems or code problems were identified. No backlog items need to be created from this quality gate run.
