# Feature: database-startup

## Goal

Wire create_tables() into the FastAPI lifespan so database schema is created automatically on every application startup.

## Background

**Backlog Item:** BL-058 (P0)

Database schema creation (`create_tables()`) exists in `src/stoat_ferret/db/schema.py` and Alembic migrations are configured, but neither is called during application startup. A fresh database gets no schema — the application starts but any database operation fails. This was built in v002/03-database-foundation but never wired into the lifespan startup sequence.

Three test files contain duplicate `create_tables_async()` helpers with inconsistent DDL coverage (8, 10, and 12 statements respectively). The canonical `create_tables()` executes 12 DDL statements.

## Functional Requirements

**FR-001:** Application lifespan calls schema creation during startup
- Acceptance: Database tables are created automatically during application startup lifespan if they don't exist

**FR-002:** Async schema creation function available in schema.py
- Acceptance: `create_tables_async()` exists in `src/stoat_ferret/db/schema.py` and creates all 12 DDL objects using aiosqlite

**FR-003:** Schema creation is idempotent
- Acceptance: Calling `create_tables_async()` on an already-initialized database does not error (IF NOT EXISTS)

**FR-004:** Test helper consolidation
- Acceptance: All 3 duplicate `create_tables_async()` helpers in test files are replaced with imports from `schema.py`

**FR-005:** Fresh database is fully functional after single start
- Acceptance: A fresh database with no prior state becomes fully functional after a single application start

## Non-Functional Requirements

**NFR-001:** Schema creation completes within startup timeout
- Metric: Schema creation on fresh database completes in < 1 second

**NFR-002:** Test mode bypass preserved
- Metric: `_deps_injected=True` bypass skips schema creation (test mode behavior preserved)

## Handler Pattern

Not applicable for v008 — no new handlers introduced.

## Out of Scope

- Alembic migration integration at startup (documented as upgrade path per LRN-029)
- Schema evolution or migration strategy
- Database connection pooling changes

## Test Requirements

**Unit tests:**
- Test create_tables_async() creates all 12 expected DDL objects on a fresh in-memory database
- Test create_tables_async() is idempotent (calling twice does not error)

**System tests:**
- Test application lifespan creates schema on fresh database (integration test with real lifespan startup)
- Test _deps_injected=True bypass skips schema creation

**Existing test modifications:**
- Replace local create_tables_async() in test_async_repository_contract.py with import from schema.py
- Replace local create_tables_async() in test_project_repository_contract.py with import from schema.py
- Replace local create_tables_async() in test_clip_repository_contract.py with import from schema.py
- Verify all 3 test files pass with shared function (DDL coverage must match 12 statements)

## Reference

See `comms/outbox/versions/design/v008/004-research/` for supporting evidence.
