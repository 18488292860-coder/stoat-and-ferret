# Test Strategy — v009

## Theme 1: 01-observability-pipeline

### Feature 001-ffmpeg-observability (BL-059)

**Unit tests:**
- Verify `ObservableFFmpegExecutor` is the active executor when accessed via DI (check `app.state` or dependency injection chain)
- Verify `create_app(ffmpeg_executor=...)` stores the executor on `app.state`
- Verify recording test double remains injectable via `create_app()` kwargs (AC4)

**Integration tests:**
- Verify FFmpeg execution emits structlog calls with duration, command preview, and correlation ID (AC2)
- Verify Prometheus metrics (`ffmpeg_executions_total`, `ffmpeg_execution_duration_seconds`) are populated after render operations (AC3)

**Parity tests:** None — no API contract changes.

**Contract tests:** None — no new DTO models.

**Existing tests affected:**
- `tests/test_api/conftest.py`: May need update if `create_app()` signature changes
- `tests/test_api/test_di_wiring.py`: Verify new DI param stored on `app.state`

---

### Feature 002-audit-logging (BL-060)

**Unit tests:**
- Verify `AuditLogger` is instantiated and injected into repositories that accept it (AC1)
- Verify `create_app(audit_logger=...)` stores the logger on `app.state`

**Integration tests:**
- Verify database mutations (create, update, delete) produce audit log entries (AC2)
- Verify audit entries include correlation ID, resource type, and operation type (AC3)

**Parity tests:** None — no API contract changes.

**Contract tests:** None — no new DTO models.

**Existing tests affected:**
- `tests/test_api/conftest.py`: May need update for `audit_logger` kwarg
- Tests using `create_app()` should continue working with `audit_logger=None` default

---

### Feature 003-file-logging (BL-057)

**Unit tests:**
- Verify `configure_logging()` adds a `RotatingFileHandler` writing to `logs/` directory (AC1)
- Verify idempotent guard: calling `configure_logging()` twice does not duplicate file handlers
- Verify `logs/` directory is created automatically if it doesn't exist (AC3)
- Verify file handler uses same structlog formatter and log level as stdout handler (AC5)
- Verify stdout logging continues alongside file logging (AC6)

**Integration tests:**
- Verify log files rotate at 10MB with configurable backup count (AC2)

**Parity tests:** None.

**Contract tests:** None.

**Existing tests affected:**
- `tests/test_logging_startup.py`: Add file handler tests alongside existing stream handler tests
- `.gitignore`: Verify `logs/` is added (AC4)

---

## Theme 2: 02-gui-runtime-fixes

### Feature 001-spa-routing (BL-063)

**Unit tests:**
- Verify `/gui/library` returns 200 with index.html content (not 404) (AC1)
- Verify `/gui/projects` returns 200 with index.html content (AC1)
- Verify arbitrary `/gui/any-sub-path` returns SPA content (AC1)
- Verify SPA fallback only activates when `gui/dist/` directory exists (LRN-020)

**Integration tests:**
- Verify page refresh on GUI sub-path preserves the current view (AC2)

**E2E tests:**
- After SPA fallback works, E2E tests can navigate directly to sub-paths (LRN-023)
- New E2E tests must include explicit assertion timeouts (LRN-043)

**Parity tests:** None.

**Contract tests:** None.

**Existing tests affected:**
- E2E test suite may be simplified (remove client-side-only navigation workaround)

---

### Feature 002-pagination-fix (BL-064)

**Unit tests:**
- Verify `AsyncProjectRepository.count()` returns correct total for SQLite implementation (AC1)
- Verify `AsyncProjectRepository.count()` returns correct total for InMemory implementation (AC1)
- Verify protocol defines `count()` method

**Integration tests:**
- Verify `GET /api/v1/projects` returns correct total count reflecting all projects, not just current page (AC2)
- Verify frontend pagination displays correct page count when projects exceed page limit (AC3)

**Parity tests:** None — this fixes the existing `total` field, no new API fields.

**Contract tests:** None — no new DTO models.

**Existing tests affected:**
- Project repository tests: Add `count()` tests mirroring video repository test patterns

---

### Feature 003-websocket-broadcasts (BL-065)

**Unit tests:**
- Verify scan operations call `broadcast()` with `SCAN_STARTED` event type (AC1)
- Verify scan completion calls `broadcast()` with `SCAN_COMPLETED` event type (AC1)
- Verify project creation calls `broadcast()` with `PROJECT_CREATED` event type (AC2)
- Verify event payloads include relevant context (project ID, scan path, etc.) (AC4)

**Integration tests:**
- Verify WebSocket client receives SCAN_STARTED and SCAN_COMPLETED events after triggering a scan (AC1)
- Verify WebSocket client receives PROJECT_CREATED event after creating a project (AC2)

**E2E tests:**
- Verify Dashboard ActivityLog displays real application events, not just heartbeats (AC3)
- New E2E tests must include explicit assertion timeouts (LRN-043)

**Parity tests:** None.

**Contract tests:** None.

**Existing tests affected:**
- Router tests for scan and project endpoints: May need WebSocket manager mock injection

---

## Cross-Cutting Test Concerns

1. **mypy after every change** (LRN-041): All features are cross-module wiring. Run mypy as primary safety net for catching kwarg mismatches, wrong types, and API misuse.

2. **Idempotent startup** (LRN-040): BL-057 and BL-059/060 add startup logic. Tests that invoke lifespan multiple times must not see side-effect accumulation (duplicate handlers, duplicate metrics).

3. **Test double availability** (BL-059 AC4): Ensure `create_app()` kwargs still allow test injection of recording/mock executors after wiring ObservableFFmpegExecutor as default.

4. **E2E timeouts** (LRN-043): Any new E2E tests (BL-063, BL-065) must use explicit assertion timeouts to avoid CI flakes.
