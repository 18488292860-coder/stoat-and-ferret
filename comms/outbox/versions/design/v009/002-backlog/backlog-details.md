# Backlog Details — v009

## Theme 1: observability-pipeline

### BL-059 — Wire ObservableFFmpegExecutor into dependency injection (P1)

> **Current state:** `ObservableFFmpegExecutor` exists with full metrics and structured logging instrumentation, but is never instantiated. The application uses the base executor directly, bypassing all observability.
>
> **Gap:** Created in v002/04-ffmpeg-integration but never wired into the dependency injection chain. The metrics and logging decorators are dead code.
>
> **Impact:** No visibility into FFmpeg execution — no duration metrics, no structured logs for render operations, no correlation ID tracking for debugging failed renders.

**Acceptance Criteria:**
1. ObservableFFmpegExecutor is instantiated and injected as the FFmpeg executor during startup
2. FFmpeg operations emit structlog calls with duration, command preview, and correlation ID
3. Prometheus metrics for FFmpeg execution count and duration are populated after render operations
4. Recording test double remains available for test injection

**Tags:** wiring-gap, observability, ffmpeg | **Notes:** None | **Use Case:** null

**Complexity:** Medium. Straightforward DI wiring in `create_app()` — the component already exists with full instrumentation. Main risk is ensuring the recording test double remains injectable for tests.

---

### BL-060 — Wire AuditLogger into repository dependency injection (P2)

> **Current state:** `AuditLogger` class exists and repository constructors accept it as a parameter, but it's never instantiated — the parameter is always None. No audit trail is produced for any database operation.
>
> **Gap:** Created in v002/03-database-foundation but never wired into the DI chain.
>
> **Impact:** No audit trail for data mutations. Debugging "what changed and when" requires manual database inspection.

**Acceptance Criteria:**
1. AuditLogger is instantiated and injected into repositories that accept it
2. Database mutations (create, update, delete) produce audit log entries
3. Audit entries include correlation ID, resource type, and operation type

**Tags:** wiring-gap, observability, database | **Notes:** None | **Use Case:** null

**Complexity:** Medium. Similar to BL-059 — instantiate existing class and pass to repository constructors. Need to verify which repositories accept the parameter and ensure audit entries are properly structured.

---

### BL-057 — Add file-based logging with rotation to logs/ directory (P2)

> **Current state:** After BL-056, structured logging will be wired up but outputs to stdout only. There is no persistent log output — when the process stops, all log history is lost.
>
> **Gap:** No file-based logging exists. Debugging issues after the fact requires the developer to have been watching stdout at the time. There's no way to review historical log output.
>
> **Impact:** Post-hoc debugging is impossible without log persistence. Auto-dev execution pipeline output is lost when sessions end.

**Acceptance Criteria:**
1. configure_logging() adds a RotatingFileHandler writing to logs/ directory at project root
2. Log files rotate at 10MB with a configurable backup count
3. logs/ directory is created automatically on startup if it doesn't exist
4. logs/ is added to .gitignore
5. File handler uses the same structlog formatter and log level as the stdout handler
6. Stdout logging continues to work alongside file logging

**Tags:** observability, logging | **Notes:** Depends on BL-056 (completed in v008). Use RotatingFileHandler with maxBytes=10MB. | **Use Case:** null

**Complexity:** Medium. Extends the `configure_logging()` function wired in v008. Must follow the idempotent handler pattern (LRN-040) established for the stdout handler. The RotatingFileHandler is stdlib — no new dependencies.

---

## Theme 2: gui-runtime-fixes

### BL-063 — Add SPA routing fallback for GUI sub-paths (P1)

> **Current state:** FastAPI serves the GUI via `StaticFiles(html=True)`, but this does not fall back to `index.html` for unknown sub-paths. Navigating directly to `/gui/library` or refreshing on `/gui/projects` returns 404. React Router handles client-side navigation fine, but only when starting from the root.
>
> **Gap:** Standard SPA routing fallback was not implemented in v005/01-frontend-foundation. The e2e test suite explicitly works around this by navigating via tab clicks only.
>
> **Impact:** Bookmarks, page refreshes, and shared URLs to any GUI sub-page are broken.

**Acceptance Criteria:**
1. Direct navigation to /gui/library, /gui/projects, and other GUI sub-paths returns the SPA index.html instead of 404
2. Page refresh on any GUI sub-path preserves the current view
3. Bookmarked and shared GUI URLs load the correct page

**Tags:** wiring-gap, gui, routing | **Notes:** None | **Use Case:** null

**Complexity:** Low-Medium. Standard SPA fallback pattern — catch-all route serving index.html for /gui/* paths. Must coexist with the conditional static mount pattern (LRN-020). Fixing this also unblocks direct-navigation E2E tests (LRN-023).

---

### BL-064 — Fix projects endpoint pagination total count (P2)

> **Current state:** `GET /api/v1/projects` returns `total=len(projects)` which equals the current page size, not the true database total. A `count()` method was added to `AsyncVideoRepository` but not to `AsyncProjectRepository`.
>
> **Gap:** Pagination total count implementation was incomplete in v005/02-backend-services — only one of two repositories received the method.
>
> **Impact:** Frontend pagination metadata is incorrect when there are more projects than the page limit. Users see wrong page counts and may not discover all their projects.

**Acceptance Criteria:**
1. AsyncProjectRepository has a count() method matching AsyncVideoRepository's implementation
2. GET /api/v1/projects returns correct total count reflecting all projects in the database, not just the current page
3. Frontend pagination displays correct page count when projects exceed the page limit

**Tags:** wiring-gap, api, pagination | **Notes:** None | **Use Case:** null

**Complexity:** Low. Copy the existing `count()` pattern from AsyncVideoRepository. Wire it into the projects endpoint. Frontend may need minimal adjustment if it already uses the `total` field.

---

### BL-065 — Wire WebSocket broadcast calls into API operations (P2)

> **Current state:** The backend defines WebSocket event types (SCAN_STARTED, SCAN_COMPLETED, PROJECT_CREATED, HEALTH_STATUS) and the frontend ActivityLog listens for them, but no API operation ever calls `ConnectionManager.broadcast()`. The only messages sent are heartbeat pings.
>
> **Gap:** WebSocket infrastructure was built in v005 across 02-backend-services and 03-gui-components, but the broadcast calls were never added to the API operations that should trigger them.
>
> **Impact:** The Dashboard ActivityLog renders correctly but only ever shows heartbeat entries. Real-time feedback for scan progress, project creation, and other operations is non-functional.

**Acceptance Criteria:**
1. Library scan operations trigger SCAN_STARTED and SCAN_COMPLETED WebSocket broadcasts
2. Project creation triggers PROJECT_CREATED WebSocket broadcast
3. Dashboard ActivityLog displays real application events, not just heartbeat pings
4. WebSocket event payloads include relevant context (project ID, scan path, etc.)

**Tags:** wiring-gap, websocket, gui | **Notes:** None | **Use Case:** null

**Complexity:** Medium-High. Touches multiple API endpoints (scan, project creation). Must inject ConnectionManager into route handlers and call broadcast at the right points. Higher risk than other items due to the cross-cutting nature — multiple endpoints need modification and the frontend ActivityLog behavior needs verification.
