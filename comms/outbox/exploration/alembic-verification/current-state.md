# Current Migration State Analysis

## Summary

The project has **2 migrations** in `alembic/versions/`, both with complete `upgrade()` and `downgrade()` functions.

## Migration Inventory

| Migration ID | Description | Downgrade | Status |
|-------------|-------------|-----------|--------|
| `c3687b3b7a6a` | Initial schema | Yes | Reversible |
| `44c6bfb6188e` | Add audit log | Yes | Reversible |

## Migration Chain

```
base → c3687b3b7a6a (initial_schema) → 44c6bfb6188e (add_audit_log) → head
```

## Detailed Analysis

### Migration 1: `c3687b3b7a6a_initial_schema.py`

**Creates:**
- `videos` table with 13 columns
- `idx_videos_path` index on videos.path
- `videos_fts` FTS5 virtual table for full-text search
- 3 triggers: `videos_fts_insert`, `videos_fts_delete`, `videos_fts_update`

**Downgrade Implementation:**
```python
def downgrade() -> None:
    op.execute("DROP TRIGGER IF EXISTS videos_fts_update")
    op.execute("DROP TRIGGER IF EXISTS videos_fts_delete")
    op.execute("DROP TRIGGER IF EXISTS videos_fts_insert")
    op.execute("DROP TABLE IF EXISTS videos_fts")
    op.execute("DROP INDEX IF EXISTS idx_videos_path")
    op.execute("DROP TABLE IF EXISTS videos")
```

**Quality Notes:**
- Uses `IF EXISTS` for safe downgrade (idempotent)
- Correct order: triggers → FTS table → index → main table
- Raw SQL used (appropriate for SQLite-specific features like FTS5)

### Migration 2: `44c6bfb6188e_add_audit_log.py`

**Creates:**
- `audit_log` table with 7 columns
- `idx_audit_log_entity` composite index

**Downgrade Implementation:**
```python
def downgrade() -> None:
    op.execute("DROP INDEX IF EXISTS idx_audit_log_entity")
    op.execute("DROP TABLE IF EXISTS audit_log")
```

**Quality Notes:**
- Uses `IF EXISTS` for safe downgrade
- Correct order: index → table

## Configuration

**alembic.ini:**
- Default SQLite database: `sqlite:///stoat_ferret.db`
- Standard script location: `alembic/`
- Path separator: `os` (cross-platform)

**env.py:**
- Standard online/offline mode support
- No custom target_metadata (manual migrations)
- NullPool for connection pooling

## Reversibility Assessment

Both existing migrations are **fully reversible**:

1. All `upgrade()` operations have corresponding `downgrade()` operations
2. `IF EXISTS` guards prevent errors on repeated downgrades
3. Drop order respects foreign key dependencies
4. No data-dependent migrations (pure DDL)

## Potential Issues for Future Migrations

Watch for these patterns that break reversibility:

1. **Data migrations** - `INSERT/UPDATE/DELETE` may not be reversible
2. **Column renames** - Old column names lost after downgrade
3. **NOT NULL additions** - Requires default or data backfill
4. **FTS5 rebuilds** - Content may be lost on downgrade
