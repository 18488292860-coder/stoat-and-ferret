# Wiring Gaps Found

## Critical Severity (feature non-functional)

### GAP-1: create_tables() never called at startup
- **Version/Theme/Feature:** v002 / 03-database-foundation / 001-sqlite-schema
- **Designed:** `create_tables(conn)` creates the videos table, FTS5 index, and path index
- **Actually wired:** Function exists in `src/stoat_ferret/db/schema.py` and is exported from `src/stoat_ferret/db/__init__.py`, but never called in `lifespan()` or anywhere in the startup path
- **Completion report caught it?** No — report marked all acceptance criteria as passing
- **Impact:** A fresh database has no schema. App only works because tables exist from prior manual setup or test runs.

### GAP-2: Alembic migrations never run at startup
- **Version/Theme/Feature:** v002 / 03-database-foundation / 003-migration-support
- **Designed:** Alembic config + initial migration + audit_log migration, with startup integration
- **Actually wired:** `alembic.ini` and migration scripts exist, but `alembic upgrade head` is never executed programmatically. No migration call in lifespan or __main__.
- **Completion report caught it?** No — report verified migration files exist and CLI works
- **Impact:** Database schema changes require manual `alembic upgrade head`. Conflicts with GAP-1: schema creation is split across two unexecuted paths.

### GAP-3: configure_logging() never called
- **Version/Theme/Feature:** v002 / 04-ffmpeg-integration / 004-ffmpeg-observability
- **Designed:** Structured logging via structlog with JSON formatting, configurable log level
- **Actually wired:** Function exists in `src/stoat_ferret/logging.py` with full structlog configuration. Never called in `__main__.py`, `lifespan()`, or anywhere else.
- **Completion report caught it?** No — report verified function exists and unit tests pass
- **Impact:** Application runs with default Python logging. All `structlog.get_logger()` calls throughout the codebase produce unconfigured output. This is the known issue that motivated this audit.

### GAP-4: ObservableFFmpegExecutor never instantiated
- **Version/Theme/Feature:** v002 / 04-ffmpeg-integration / 004-ffmpeg-observability
- **Designed:** Decorator wrapping FFmpegExecutor to add structured logging and Prometheus metrics
- **Actually wired:** Class exists in `src/stoat_ferret/ffmpeg/observable.py`. Lifespan creates `RealFFmpegExecutor()` directly without wrapping it. Prometheus metrics module exists but is never activated.
- **Completion report caught it?** No — report verified class and metrics code exists
- **Impact:** No execution metrics or structured logging for ffmpeg operations despite complete implementation.

## Minor Severity (degraded behavior)

### GAP-5: settings.log_level never consumed
- **Version/Theme/Feature:** v002 / 04-ffmpeg-integration / 004-ffmpeg-observability
- **Designed:** Configurable log level via settings (DEBUG/INFO/WARNING/ERROR/CRITICAL)
- **Actually wired:** Defined in `src/stoat_ferret/api/settings.py` line 53. Would be consumed by `configure_logging()` but that function is never called. `uvicorn.run()` in `__main__.py` uses hardcoded `log_level="info"`.
- **Completion report caught it?** No
- **Impact:** Log level cannot be configured via environment variables as designed.

### GAP-6: AuditLogger never instantiated
- **Version/Theme/Feature:** v002 / 03-database-foundation / 004-audit-logging
- **Designed:** AuditLogger records INSERT/UPDATE/DELETE operations in audit_log table
- **Actually wired:** Class exists in `src/stoat_ferret/db/audit.py`. `AsyncSQLiteVideoRepository` accepts an `audit_logger` parameter but it's never provided — defaults to None. No AuditLogger is created in lifespan.
- **Completion report caught it?** No
- **Impact:** Data mutations are not audited despite complete implementation.

### GAP-7: execute_command() never used
- **Version/Theme/Feature:** v002 / 04-ffmpeg-integration / 003-command-integration
- **Designed:** Bridge function connecting Rust FFmpegCommand.build() to Python FFmpegExecutor.run()
- **Actually wired:** Function exists in `src/stoat_ferret/ffmpeg/integration.py` and is exported from `src/stoat_ferret/ffmpeg/__init__.py`. No production code imports or calls it. Actual code path uses direct executor calls with pre-built args.
- **Completion report caught it?** No
- **Impact:** Integration layer exists but is bypassed. Not blocking since direct executor usage works.

## Cosmetic Severity (dead code, no runtime impact)

### GAP-8: settings.debug never referenced
- **Version/Theme/Feature:** v002 / 03-database-foundation (settings expansion)
- **Actually wired:** Defined in settings.py, never read by any application code
- **Impact:** None — setting has no effect

### GAP-9: settings.ws_heartbeat_interval hardcoded instead of read
- **Version/Theme/Feature:** v002 / 04-ffmpeg-integration (settings expansion)
- **Actually wired:** Defined in settings.py as configurable, but `ws.py` uses `DEFAULT_HEARTBEAT_INTERVAL = 30` constant instead of reading from settings
- **Impact:** WebSocket heartbeat interval cannot be configured via environment variables

### GAP-10: TimeRange list operations have zero production callers
- **Version/Theme/Feature:** v001 / 02-timeline-math / 003-range-arithmetic
- **Actually wired:** find_gaps, merge_ranges, total_coverage are exposed via PyO3 and tested, but no production Python code calls them. Design-ahead infrastructure for future features.
- **Impact:** None — intentional forward design

### GAP-11: Input sanitization functions have zero production callers
- **Version/Theme/Feature:** v001 / 03-command-builder / 003-input-sanitization
- **Actually wired:** 8 validation functions (validate_crf, validate_speed, etc.) exposed via PyO3 but never called from Python. Rust-side validation in FFmpegCommand.build() covers overlapping checks.
- **Impact:** None — validation still occurs at the Rust layer

## Pattern Analysis

All 7 non-cosmetic gaps share the same root cause: **quality gates test individual pieces in isolation but never verify startup wiring.** Each function has passing unit tests, each class is well-structured, and each completion report verified that code exists — but none checked whether the function is actually called at runtime.

No completion report across all themes caught any of these gaps.
