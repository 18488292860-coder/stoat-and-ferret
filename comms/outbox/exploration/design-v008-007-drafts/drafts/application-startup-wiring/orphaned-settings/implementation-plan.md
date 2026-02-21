# Implementation Plan: orphaned-settings

## Overview

Wire `settings.debug` to the FastAPI constructor and replace the hardcoded `DEFAULT_HEARTBEAT_INTERVAL` constant in ws.py with `get_settings().ws_heartbeat_interval`, ensuring all defined settings fields are consumed.

## Files to Create/Modify

| File | Action | Change |
|------|--------|--------|
| `src/stoat_ferret/api/app.py` | Modify | Add `debug=get_settings().debug` to `FastAPI()` constructor |
| `src/stoat_ferret/api/routers/ws.py` | Modify | Replace `DEFAULT_HEARTBEAT_INTERVAL` with `get_settings().ws_heartbeat_interval`; add `get_settings` import |
| `src/stoat_ferret/api/__main__.py` | Modify | Add `debug=settings.debug` to `uvicorn.run()` call |
| `tests/test_orphaned_settings.py` | Create | New tests for debug mode and heartbeat wiring |

## Test Files

`tests/test_orphaned_settings.py`

## Implementation Stages

### Stage 1: Wire settings.debug to FastAPI

1. In `src/stoat_ferret/api/app.py`, modify the `FastAPI()` constructor to include `debug=get_settings().debug`
2. In `src/stoat_ferret/api/__main__.py`, add `debug=settings.debug` to the `uvicorn.run()` call
3. Write tests verifying debug mode is controlled by settings

**Verification:**
```bash
uv run pytest tests/test_orphaned_settings.py -v
```

### Stage 2: Wire settings.ws_heartbeat_interval to ws.py

1. In `src/stoat_ferret/api/routers/ws.py`, add `from stoat_ferret.api.settings import get_settings`
2. Replace `DEFAULT_HEARTBEAT_INTERVAL` usage with `get_settings().ws_heartbeat_interval`
3. Remove the `DEFAULT_HEARTBEAT_INTERVAL` constant
4. Write tests verifying heartbeat interval is read from settings

**Verification:**
```bash
uv run pytest tests/test_orphaned_settings.py -v
```

### Stage 3: Verify no orphaned settings remain

1. Audit all Settings fields against codebase usage
2. Confirm all 9 fields are consumed after this feature (database_path, api_host, api_port, debug, log_level, thumbnail_dir, gui_static_path, ws_heartbeat_interval, allowed_scan_roots)

**Verification:**
```bash
uv run pytest tests/ --cov=src --cov-fail-under=80
```

## Test Infrastructure Updates

- New test file: `tests/test_orphaned_settings.py` with unit and system tests
- No existing test modifications needed

## Quality Gates

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest tests/ --cov=src --cov-fail-under=80
```

## Risks

- No significant risks — both wiring changes are straightforward; see `006-critical-thinking/risk-assessment.md`
- Default values match existing behavior (debug=False, heartbeat=30s)

## Commit Message

```
feat: wire orphaned settings (debug, ws_heartbeat_interval) to consumers (BL-062)
```
