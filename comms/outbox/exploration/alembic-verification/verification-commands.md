# Alembic Migration Verification Commands

## Recommended Verification Sequence

The canonical sequence to verify migrations are reversible:

```bash
# Use an in-memory SQLite database for testing
export DATABASE_URL="sqlite:///:memory:"

# Or use a temporary file that gets cleaned up
export DATABASE_URL="sqlite:///test_migrations.db"
rm -f test_migrations.db

# Full cycle verification
alembic upgrade head && \
alembic downgrade base && \
alembic upgrade head

# Cleanup (if using file-based SQLite)
rm -f test_migrations.db
```

## Step-by-Step Verification

### Step 1: Upgrade to Head

```bash
alembic upgrade head
```

**Expected Output:**
```
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> c3687b3b7a6a, initial_schema
INFO  [alembic.runtime.migration] Running upgrade c3687b3b7a6a -> 44c6bfb6188e, add_audit_log
```

### Step 2: Downgrade to Base

```bash
alembic downgrade base
```

**Expected Output:**
```
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running downgrade 44c6bfb6188e -> c3687b3b7a6a, add_audit_log
INFO  [alembic.runtime.migration] Running downgrade c3687b3b7a6a -> , initial_schema
```

### Step 3: Re-upgrade to Head

```bash
alembic upgrade head
```

**Expected Output:**
Same as Step 1 - proves migrations are idempotent after downgrade.

## Alternative Commands

### Test Specific Migration

```bash
# Upgrade to specific revision
alembic upgrade c3687b3b7a6a

# Downgrade one revision
alembic downgrade -1

# Downgrade to specific revision
alembic downgrade c3687b3b7a6a
```

### Check Current State

```bash
# Show current revision
alembic current

# Show migration history
alembic history --verbose
```

### Dry Run (SQL Only)

```bash
# Generate upgrade SQL without executing
alembic upgrade head --sql

# Generate downgrade SQL without executing
alembic downgrade base --sql
```

## Using Custom Database URL

Override the default `alembic.ini` database URL:

```bash
# Method 1: Command-line argument
alembic -x sqlalchemy.url=sqlite:///test.db upgrade head

# Method 2: Environment variable (requires env.py modification)
DATABASE_URL="sqlite:///test.db" alembic upgrade head
```

**Note:** The current `env.py` reads from `alembic.ini` only. For CI, either:
1. Override via `-x` flag
2. Modify `env.py` to read `DATABASE_URL` environment variable

## Error Scenarios

### Missing Downgrade Function

```
FAILED: Target database is not up to date.
```

Or more specifically:
```
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) no such table: some_table
```

### Incomplete Downgrade

If a migration's `downgrade()` doesn't fully reverse `upgrade()`, the re-upgrade will fail:

```
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) table videos already exists
```

### SQL Syntax Error

```
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) near "INVALID": syntax error
```

## Complete CI-Ready Script

```bash
#!/bin/bash
set -euo pipefail

# Use temporary database
DB_FILE="$(mktemp).db"
trap "rm -f $DB_FILE" EXIT

export SQLALCHEMY_URL="sqlite:///$DB_FILE"

echo "Testing migration reversibility..."

echo "Step 1: Upgrade to head"
alembic -x sqlalchemy.url="sqlite:///$DB_FILE" upgrade head

echo "Step 2: Downgrade to base"
alembic -x sqlalchemy.url="sqlite:///$DB_FILE" downgrade base

echo "Step 3: Re-upgrade to head"
alembic -x sqlalchemy.url="sqlite:///$DB_FILE" upgrade head

echo "All migrations are reversible!"
```
