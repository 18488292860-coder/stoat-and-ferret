## Exploration: Alembic Migration Verification

### Context
stoat-and-ferret v003 backlog item BL-015 requires CI verification that migrations are reversible. Current Alembic setup is in `alembic/` directory with migrations in `alembic/versions/`.

### Questions to Answer

1. **Current migration state**: What migrations exist? Are they currently reversible (have downgrade functions)?

2. **Verification command sequence**: What's the exact command sequence to verify reversibility? `alembic upgrade head && alembic downgrade base && alembic upgrade head`?

3. **CI integration**: How do we run this in GitHub Actions? Do we need a test database? Can we use in-memory SQLite?

4. **Error handling**: What happens if a migration isn't reversible? How do we get good error messages?

5. **Best practices**: Should we add a pre-commit hook or just CI? Any common gotchas?

### Research Sources
- DeepWiki: sqlalchemy/alembic
- Existing code: alembic/, alembic/versions/, alembic.ini

### Output Requirements

Write findings to: `comms/outbox/exploration/alembic-verification/`

Required files:
- `README.md` - Summary with recommended CI approach
- `current-state.md` - Analysis of existing migrations
- `verification-commands.md` - Exact commands and expected output
- `ci-integration.md` - GitHub Actions workflow addition
- `best-practices.md` - Additional recommendations

### Commit Instructions
Commit all files with message: "docs(exploration): Alembic migration verification for BL-015"