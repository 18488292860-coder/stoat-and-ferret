---
status: complete
acceptance_passed: 4
acceptance_total: 4
quality_gates:
  ruff: pass
  mypy: pass
  pytest: pass
---
# Completion Report: 003-migration-support

## Summary

Configured Alembic for database migrations with full rollback capability. Added alembic dependency to pyproject.toml, initialized alembic configuration for SQLite, and created the initial migration that establishes the v001 schema (videos table, path index, FTS5 virtual table, and synchronization triggers).

## Implementation Details

### FR-001: Alembic Configuration
- Added `alembic>=1.13` dependency to pyproject.toml
- Initialized alembic with `alembic init alembic`
- Configured `alembic.ini` with SQLite URL: `sqlite:///stoat_ferret.db`
- Default env.py works correctly for SQLite without modifications

### FR-002: Initial Migration
- Created migration `c3687b3b7a6a_initial_schema.py`
- Migration creates:
  - `videos` table with all metadata columns
  - `idx_videos_path` index on path column
  - `videos_fts` FTS5 virtual table for full-text search
  - Three triggers to keep FTS in sync (insert, update, delete)
- Uses raw SQL via `op.execute()` for SQLite-specific features (FTS5, triggers)

### FR-003: Upgrade/Downgrade
- `alembic upgrade head` creates all tables and triggers
- `alembic downgrade -1` removes everything in correct order
- Full rollback to empty database verified working

### FR-004: Add Dependency
- Added `alembic>=1.13` to pyproject.toml dependencies
- Ran `uv sync` to install alembic and its dependencies (SQLAlchemy, Mako, etc.)

## Acceptance Criteria

| Criterion | Status |
|-----------|--------|
| `alembic upgrade head` creates all tables | PASS |
| `alembic downgrade -1` removes tables | PASS |
| `alembic current` shows migration status | PASS |
| Migration files in alembic/versions/ | PASS |

## Files Created/Modified

- `pyproject.toml` - Added alembic dependency
- `uv.lock` - Updated lockfile
- `alembic.ini` - Alembic configuration for SQLite
- `alembic/env.py` - Migration environment (default, unchanged)
- `alembic/script.py.mako` - Migration template
- `alembic/versions/c3687b3b7a6a_initial_schema.py` - Initial schema migration

## PR Reference

- PR #27: https://github.com/gwickman/stoat-and-ferret/pull/27
- Merged to main via squash merge
