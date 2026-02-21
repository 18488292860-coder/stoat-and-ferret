# Implementation Plan: logging-startup

## Overview

Make `configure_logging()` idempotent, call it during FastAPI lifespan startup before the `_deps_injected` guard, wire `settings.log_level` to control the root logger level, and change uvicorn's hardcoded log level to use the settings value.

## Files to Create/Modify

| File | Action | Change |
|------|--------|--------|
| `src/stoat_ferret/logging.py` | Modify | Make `configure_logging()` idempotent: guard `root.addHandler(handler)` with existing handler check |
| `src/stoat_ferret/api/app.py` | Modify | Add `configure_logging(level=getattr(logging, settings.log_level))` call before `_deps_injected` guard |
| `src/stoat_ferret/api/__main__.py` | Modify | Change `log_level="info"` to `log_level=settings.log_level.lower()` |
| `tests/test_logging.py` | Modify | Add root logger handler cleanup between tests |
| `tests/test_logging_startup.py` | Create | New tests for startup integration, log level conversion, idempotency |

## Test Files

`tests/test_logging_startup.py tests/test_logging.py`

## Implementation Stages

### Stage 1: Make configure_logging() idempotent

1. In `src/stoat_ferret/logging.py`, guard `root.addHandler(handler)` with a check: only add if no StreamHandler is already attached to the root logger
2. Add root logger handler cleanup to `tests/test_logging.py` teardown (remove handlers added by configure_logging between tests)

**Verification:**
```bash
uv run pytest tests/test_logging.py -v
```

### Stage 2: Wire configure_logging() into lifespan

1. In `src/stoat_ferret/api/app.py`, import `configure_logging` from `stoat_ferret.logging`
2. Add `import logging` to app.py imports
3. Add `configure_logging(level=getattr(logging, get_settings().log_level))` call BEFORE the `_deps_injected` guard in the lifespan function
4. Write tests verifying configure_logging() is called during startup

**Verification:**
```bash
uv run pytest tests/test_logging_startup.py -v
```

### Stage 3: Wire uvicorn log level

1. In `src/stoat_ferret/api/__main__.py`, change `log_level="info"` to `log_level=settings.log_level.lower()`
2. Write test verifying uvicorn receives the settings-based log level

**Verification:**
```bash
uv run pytest tests/test_logging_startup.py -v
```

## Test Infrastructure Updates

- New test file: `tests/test_logging_startup.py` with unit and system tests
- Existing test modification: `tests/test_logging.py` handler cleanup

## Quality Gates

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest tests/ --cov=src --cov-fail-under=80
```

## Risks

- Logging activation may cause test output noise — mitigated by idempotency fix; see `006-critical-thinking/risk-assessment.md`
- configure_logging() placement is critical — must be before _deps_injected guard; see `006-critical-thinking/investigation-log.md`

## Commit Message

```
feat: wire structured logging into application startup (BL-056)
```