# Implementation Plan: database-startup

## Overview

Add an async schema creation function (`create_tables_async()`) to `schema.py` and call it from the FastAPI lifespan startup sequence inside the `_deps_injected` guard. Consolidate 3 duplicate test helpers into imports from the shared function.

## Files to Create/Modify

| File | Action | Change |
|------|--------|--------|
| `src/stoat_ferret/db/schema.py` | Modify | Add `create_tables_async(db: aiosqlite.Connection)` function |
| `src/stoat_ferret/api/app.py` | Modify | Add schema creation call in lifespan inside `_deps_injected` guard |
| `tests/test_async_repository_contract.py` | Modify | Replace local `create_tables_async()` with import from `schema.py` |
| `tests/test_project_repository_contract.py` | Modify | Replace local `create_tables_async()` with import from `schema.py` |
| `tests/test_clip_repository_contract.py` | Modify | Replace local `create_tables_async()` with import from `schema.py` |
| `tests/test_database_startup.py` | Create | New tests for create_tables_async() and lifespan integration |

## Test Files

`tests/test_database_startup.py tests/test_async_repository_contract.py tests/test_project_repository_contract.py tests/test_clip_repository_contract.py`

## Implementation Stages

### Stage 1: Add create_tables_async() to schema.py

1. Add `create_tables_async(db: aiosqlite.Connection)` to `src/stoat_ferret/db/schema.py`
2. Implement using the same 12 DDL statements as the existing sync `create_tables()`, executing via `await db.executescript()`
3. Export from `src/stoat_ferret/db/__init__.py`

**Verification:**
```bash
uv run pytest tests/test_database_startup.py -v
```

### Stage 2: Wire into lifespan

1. Import `create_tables_async` in `src/stoat_ferret/api/app.py`
2. Add `await create_tables_async(db)` inside the `_deps_injected` guard, after the aiosqlite connection is opened and before repository creation

**Verification:**
```bash
uv run pytest tests/test_database_startup.py -v
```

### Stage 3: Consolidate test helpers

1. Replace local `create_tables_async()` in `tests/test_async_repository_contract.py` with `from stoat_ferret.db.schema import create_tables_async`
2. Replace local `create_tables_async()` in `tests/test_project_repository_contract.py` with same import
3. Replace local `create_tables_async()` in `tests/test_clip_repository_contract.py` with same import
4. Verify all 3 test files still pass (DDL coverage unified to 12 statements)

**Verification:**
```bash
uv run pytest tests/test_async_repository_contract.py tests/test_project_repository_contract.py tests/test_clip_repository_contract.py -v
```

## Test Infrastructure Updates

- New test file: `tests/test_database_startup.py` with unit and integration tests
- No new test fixtures or conftest changes needed

## Quality Gates

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest tests/ --cov=src --cov-fail-under=80
```

## Risks

- Schema evolution strategy deferred — see `006-critical-thinking/risk-assessment.md`
- Shared modification point (app.py) — mitigated by sequential execution

## Commit Message

```
feat: wire database schema creation into application startup (BL-058)
```
