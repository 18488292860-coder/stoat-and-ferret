# Risks and Unknowns — v009

## Risk: WebSocket broadcast touches multiple routers

- **Severity:** medium
- **Description:** BL-065 requires injecting `ws_manager` and adding `broadcast()` calls in both the scan endpoint (`routers/videos.py`) and project creation endpoint (`routers/projects.py`). Cross-cutting changes across multiple routers increase the risk of partial implementation or inconsistent event payloads.
- **Investigation needed:** Verify all operation points where broadcasts should fire. Check if any other operations (e.g., project deletion, video deletion) should also broadcast events beyond the 4 types currently defined.
- **Current assumption:** Only the 4 defined event types (SCAN_STARTED, SCAN_COMPLETED, PROJECT_CREATED, HEALTH_STATUS) need broadcast calls. No additional event types are needed for v009.

## Risk: SPA fallback route ordering in FastAPI

- **Severity:** medium
- **Description:** The catch-all route `@app.get("/gui/{path:path}")` must not intercept requests for actual static files (JS, CSS, images) in `gui/dist/`. Route ordering matters — the catch-all must coexist with `StaticFiles` mount without conflicts.
- **Investigation needed:** Test that static asset requests (e.g., `/gui/assets/main.js`) are still served by `StaticFiles` and not caught by the fallback route. Confirm the correct registration order: API routers → SPA fallback → StaticFiles mount.
- **Current assumption:** Registering the catch-all route before the `StaticFiles` mount will serve `index.html` for paths that don't match real files. FastAPI/Starlette routes take priority over mounted sub-applications, and `StaticFiles` will handle actual file requests. This needs validation during implementation.

## Risk: AuditLogger requires synchronous SQLite connection

- **Severity:** medium
- **Description:** `AuditLogger.__init__` takes a `sqlite3.Connection` (synchronous), but the async repositories use `aiosqlite` connections. The lifespan creates an async connection. Wiring AuditLogger may require maintaining a separate synchronous connection or adapting the AuditLogger.
- **Investigation needed:** Check if `aiosqlite` exposes the underlying `sqlite3.Connection` object, or if AuditLogger needs to be adapted for async usage. Review how `AuditLogger` is called from within async repository methods.
- **Current assumption:** The async repo's audit logger usage (at `async_repository.py:169-170`) likely runs the audit call within the aiosqlite executor. If the underlying connection is accessible, no adapter is needed. UNVERIFIED — requires implementation-time investigation.

## Risk: Lifespan function complexity growth

- **Severity:** low
- **Description:** The lifespan function in `app.py` already handles DB setup, structured logging, and DI initialization. v009 adds 3 more concerns: FFmpeg executor wrapping, AuditLogger instantiation, and file logging directory creation. Growing complexity may reduce maintainability.
- **Investigation needed:** Monitor the lifespan function size after v009. If it exceeds ~50 lines, consider extracting a `setup_observability()` helper in a future version.
- **Current assumption:** The additions are small (2-5 lines each). Complexity remains manageable for v009 but should be watched.

## Risk: Log backup count setting addition

- **Severity:** low
- **Description:** BL-057 AC2 specifies "configurable backup count." This may require adding a new `log_backup_count` field to the Settings model. Per LRN-044, any new settings field must be wired to a consumer immediately.
- **Investigation needed:** Decide whether to add the setting to the Settings model or use a hardcoded default. If adding, ensure the consumer (RotatingFileHandler) reads from it during `configure_logging()`.
- **Current assumption:** A `log_backup_count` setting with default 5 will be added to Settings and consumed by `configure_logging()`. This is a small, self-contained change.

## Risk: Test double injection after ObservableFFmpegExecutor wiring

- **Severity:** low
- **Description:** BL-059 AC4 requires that the recording test double remains available. After wiring `ObservableFFmpegExecutor` as the default, tests that inject a mock executor via `create_app(ffmpeg_executor=...)` must still bypass the observable wrapper.
- **Investigation needed:** Verify that test injection via `create_app()` kwargs overrides the lifespan default completely (not wrapping the test double in another observable layer).
- **Current assumption:** The `create_app()` kwarg pattern (check `app.state` first, skip lifespan setup if injected) will handle this correctly, as it does for other dependencies.

## Unknown: Frontend pagination adjustment for BL-064

- **Severity:** low
- **Description:** The `total` field in `GET /api/v1/projects` response currently equals the page size. Changing it to the true database total should be seamless if the frontend already uses `total` for pagination math, but there could be edge cases.
- **Investigation needed:** Check frontend pagination component to confirm it uses the `total` field correctly and won't break when the value changes from page-count to true-count.
- **Current assumption:** Frontend pagination already expects `total` to be the true total (it works correctly for videos where `count()` is implemented). No frontend changes needed. UNVERIFIED for the projects page specifically.
