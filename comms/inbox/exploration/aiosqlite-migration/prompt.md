## Exploration: aiosqlite Migration Patterns

### Context
stoat-and-ferret has an existing sync VideoRepository using sqlite3. For v003 FastAPI integration, we need async database access. The current implementation is in `src/stoat_ferret/db/repository.py`.

### Questions to Answer

1. **aiosqlite API compatibility**: How closely does aiosqlite mirror the sqlite3 API? Can we maintain the same patterns (row_factory, execute, fetchone, fetchall)?

2. **Protocol compatibility**: Can we define an AsyncVideoRepository protocol that mirrors VideoRepository but with async methods? Show concrete examples.

3. **Contract test strategy**: The existing `tests/test_repository_contract.py` tests both SQLite and InMemory implementations. How should we structure async contract tests? Can we use the same parametrized fixture pattern?

4. **Alembic async support**: Does Alembic work with aiosqlite? If not, what's the recommended pattern (sync connection for migrations, async for runtime)?

5. **Connection management**: What's the recommended pattern for managing aiosqlite connections in FastAPI (lifespan, dependency injection)?

### Research Sources
- DeepWiki: omnilib/aiosqlite
- DeepWiki: encode/databases (alternative to consider)
- Existing code: src/stoat_ferret/db/repository.py, tests/test_repository_contract.py

### Output Requirements

Write findings to: `comms/outbox/exploration/aiosqlite-migration/`

Required files:
- `README.md` - Summary of findings with recommendations
- `api-comparison.md` - Side-by-side sqlite3 vs aiosqlite API comparison
- `async-repository-pattern.md` - Concrete AsyncVideoRepository implementation sketch
- `contract-test-strategy.md` - How to test async repositories
- `alembic-compatibility.md` - Alembic + async findings

### Commit Instructions
Commit all files with message: "docs(exploration): aiosqlite migration patterns for v003"