# CI Integration for Alembic Migration Verification

## Recommended Approach

Add a dedicated step to the existing CI workflow that verifies migration reversibility using an in-memory SQLite database.

## GitHub Actions Workflow Addition

Add this step to `.github/workflows/ci.yml` after the "Install dependencies" step:

```yaml
      - name: Verify migrations reversible
        run: |
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: downgrade base
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head
```

### Full Context in Workflow

```yaml
      - name: Install dependencies
        run: uv sync

      - name: Verify migrations reversible
        run: |
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: downgrade base
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head

      - name: Rust fmt
        run: cargo fmt --manifest-path rust/stoat_ferret_core/Cargo.toml -- --check
```

## Why In-Memory SQLite?

| Approach | Pros | Cons |
|----------|------|------|
| In-memory SQLite | Fast, no cleanup needed, isolated | Not exactly like production |
| File-based SQLite | Closer to production | Requires cleanup, slower |
| Real PostgreSQL/MySQL | Exact production match | Complex setup, slow, requires service container |

**Recommendation:** In-memory SQLite is sufficient for this project since:
1. Production uses SQLite (`sqlite:///stoat_ferret.db`)
2. All migrations use SQLite-compatible syntax
3. Speed is important in CI

## Alternative: File-Based SQLite

If in-memory has issues (some FTS5 features behave differently):

```yaml
      - name: Verify migrations reversible
        run: |
          uv run alembic -x sqlalchemy.url=sqlite:///ci_test.db upgrade head
          uv run alembic -x sqlalchemy.url=sqlite:///ci_test.db downgrade base
          uv run alembic -x sqlalchemy.url=sqlite:///ci_test.db upgrade head
          rm -f ci_test.db
```

## Alternative: Separate Job

For better visibility and parallel execution:

```yaml
  migrations:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Install dependencies
        run: uv sync --no-dev

      - name: Verify migrations reversible
        run: |
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: downgrade base
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head
```

**Pros of separate job:**
- Runs in parallel with tests
- Clear pass/fail visibility
- Faster feedback

**Cons:**
- Extra checkout and setup time
- More workflow complexity

## Recommendation for BL-015

Add the step inline to the existing `test` job (first option above) because:

1. **Simplicity** - Single step addition, minimal change
2. **Matrix coverage** - Runs on all OS/Python combinations
3. **Dependency reuse** - Uses already-installed uv/dependencies
4. **Order** - Runs early, fails fast if migrations broken

## Complete Modified Workflow Section

```yaml
      - name: Install dependencies
        run: uv sync

      - name: Verify migrations reversible
        run: |
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: downgrade base
          uv run alembic -x sqlalchemy.url=sqlite:///:memory: upgrade head

      - name: Rust fmt
```

## Expected CI Output

On success:
```
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> c3687b3b7a6a, initial_schema
INFO  [alembic.runtime.migration] Running upgrade c3687b3b7a6a -> 44c6bfb6188e, add_audit_log
INFO  [alembic.runtime.migration] Running downgrade 44c6bfb6188e -> c3687b3b7a6a, add_audit_log
INFO  [alembic.runtime.migration] Running downgrade c3687b3b7a6a -> , initial_schema
INFO  [alembic.runtime.migration] Running upgrade  -> c3687b3b7a6a, initial_schema
INFO  [alembic.runtime.migration] Running upgrade c3687b3b7a6a -> 44c6bfb6188e, add_audit_log
```

On failure (e.g., missing downgrade):
```
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) table videos already exists
Error: Process completed with exit code 1.
```

## Cross-Platform Notes

The `-x sqlalchemy.url=sqlite:///:memory:` syntax works on:
- Linux (ubuntu-latest)
- macOS (macos-latest)
- Windows (windows-latest)

No shell-specific escaping needed since `uv run` handles this.
