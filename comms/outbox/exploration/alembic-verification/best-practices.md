# Alembic Migration Best Practices

## CI vs Pre-commit Hook

### Recommendation: CI Only (No Pre-commit)

**Reasons:**
1. **Speed** - Migration verification takes 1-3 seconds, acceptable in CI but annoying on every commit
2. **Scope** - Most commits don't touch migrations; running on every commit wastes time
3. **Reliability** - CI runs in clean environment; pre-commit has local state issues
4. **Simplicity** - One verification point, one source of truth

**Alternative considered:** Pre-commit hook with path filter
```yaml
# .pre-commit-config.yaml (NOT RECOMMENDED)
- repo: local
  hooks:
    - id: verify-migrations
      name: Verify migrations reversible
      entry: bash -c 'alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head && alembic -x sqlalchemy.url=sqlite:///:memory: downgrade base && alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head'
      language: system
      files: ^alembic/versions/.*\.py$
```

This only runs when migration files change, but still adds complexity. CI is simpler and sufficient.

## Common Gotchas

### 1. Empty Downgrade Functions

**Problem:**
```python
def downgrade() -> None:
    pass  # Forgot to implement!
```

**Solution:** CI verification catches this - upgrade works, downgrade is no-op, re-upgrade fails.

### 2. Wrong Drop Order

**Problem:**
```python
def downgrade() -> None:
    op.execute("DROP TABLE videos")  # Fails if triggers reference it
    op.execute("DROP TRIGGER videos_fts_insert")
```

**Solution:** Drop in reverse order of creation (triggers before tables).

### 3. Missing IF EXISTS

**Problem:**
```python
def downgrade() -> None:
    op.execute("DROP TABLE videos")  # Fails if already dropped
```

**Solution:** Always use `IF EXISTS`:
```python
def downgrade() -> None:
    op.execute("DROP TABLE IF EXISTS videos")
```

### 4. Data-Dependent Migrations

**Problem:**
```python
def upgrade() -> None:
    op.execute("ALTER TABLE users ADD COLUMN email TEXT")
    op.execute("UPDATE users SET email = username || '@example.com'")

def downgrade() -> None:
    # How do we restore original state? We can't!
    op.execute("ALTER TABLE users DROP COLUMN email")
```

**Solution:** For data migrations, consider:
- Keeping backup columns during transition
- Making such migrations one-way (document clearly)
- Using soft deletes instead of hard deletes

### 5. SQLite ALTER TABLE Limitations

**Problem:** SQLite doesn't support `DROP COLUMN` (pre-3.35.0) or `ALTER COLUMN`.

**Solution:** Use table recreation pattern:
```python
def downgrade() -> None:
    # Create temp table without column
    op.execute("CREATE TABLE users_new AS SELECT id, name FROM users")
    op.execute("DROP TABLE users")
    op.execute("ALTER TABLE users_new RENAME TO users")
```

### 6. FTS5 Virtual Tables

**Problem:** FTS5 tables have special requirements.

**Best Practice:**
- Always create FTS with `content='table_name'` for external content
- Create triggers to keep FTS in sync
- Drop triggers before FTS table in downgrade

### 7. In-Memory SQLite Differences

**Problem:** Some features behave differently in `:memory:`:
- File locking doesn't exist
- Connection pooling behaves differently
- Some PRAGMA settings don't persist

**Solution:** If tests pass with `:memory:` but fail with file DB, add file-based test:
```yaml
- name: Verify migrations (file)
  run: |
    uv run alembic -x sqlalchemy.url=sqlite:///test.db upgrade head
    uv run alembic -x sqlalchemy.url=sqlite:///test.db downgrade base
    rm -f test.db
```

## Migration Writing Checklist

When writing new migrations:

- [ ] Both `upgrade()` and `downgrade()` implemented
- [ ] `downgrade()` reverses `upgrade()` completely
- [ ] Uses `IF EXISTS` / `IF NOT EXISTS` for safety
- [ ] Drop order is reverse of create order
- [ ] Tested locally: `alembic upgrade head && alembic downgrade -1 && alembic upgrade head`
- [ ] No data loss in downgrade (or documented as one-way)
- [ ] SQLite-compatible syntax (if using SQLite)

## Monitoring Migration Health

### Regular Checks

```bash
# List all migrations and their status
alembic history --verbose

# Check current database state
alembic current

# Show pending migrations
alembic heads
```

### When Migrations Diverge

If multiple developers create migrations simultaneously:

```bash
# Show branch points
alembic branches

# Merge branches (creates new migration)
alembic merge heads -m "merge_branches"
```

## Documentation Standards

Each migration file should have:

1. **Clear description** in the module docstring
2. **What it creates/modifies** in `upgrade()` docstring
3. **What it removes** in `downgrade()` docstring

Example:
```python
"""Add user preferences table.

Revision ID: abc123
Revises: def456
Create Date: 2026-01-28
"""

def upgrade() -> None:
    """Create user_preferences table with columns: id, user_id, theme, notifications."""
    ...

def downgrade() -> None:
    """Drop user_preferences table."""
    ...
```
