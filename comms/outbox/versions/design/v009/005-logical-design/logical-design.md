# Logical Design — v009: Observability & GUI Runtime

## Version Overview

**Version:** v009
**Description:** Complete the observability pipeline (FFmpeg metrics, audit logging, file-based logs) and fix GUI runtime gaps (SPA routing, pagination, WebSocket broadcasts).

**Goals:**
1. Wire existing but unused observability components (ObservableFFmpegExecutor, AuditLogger) into the DI chain
2. Add persistent file-based logging with rotation
3. Fix SPA routing so direct navigation and page refresh work
4. Correct pagination total count for projects
5. Activate WebSocket broadcast calls so the frontend receives real-time events

**Scope:** 2 themes, 6 features. All items are wiring gaps — no new functionality is being built.

---

## Theme 1: 01-observability-pipeline

**Goal:** Wire the three observability components that exist as dead code into the application's dependency injection chain and startup sequence. After this theme, FFmpeg operations emit metrics and structured logs, database mutations produce audit entries, and all logs are persisted to rotating files.

**Backlog Items:** BL-059, BL-060, BL-057

**Rationale for grouping:** All three features modify the DI/startup code path (`create_app()`, lifespan, `configure_logging()`). Grouping minimizes context-switching and reduces merge conflicts in `app.py` (LRN-042).

**Features:**

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | 001-ffmpeg-observability | Wire ObservableFFmpegExecutor into DI so FFmpeg operations emit metrics and structured logs | BL-059 | None |
| 2 | 002-audit-logging | Wire AuditLogger into repository DI so database mutations produce audit entries | BL-060 | None (but sequenced after 001 to avoid app.py conflicts) |
| 3 | 003-file-logging | Add RotatingFileHandler to configure_logging() for persistent log output | BL-057 | None (but sequenced after 002 to avoid app.py conflicts) |

---

## Theme 2: 02-gui-runtime-fixes

**Goal:** Fix three runtime gaps in the GUI and API layer: SPA routing fallback for direct navigation, correct pagination totals for the projects endpoint, and WebSocket broadcast calls for real-time event delivery to the frontend.

**Backlog Items:** BL-063, BL-064, BL-065

**Rationale for grouping:** All three features modify the API routing and frontend interaction layer. They are independent of Theme 1's DI/startup changes and can be developed after Theme 1 completes (LRN-042).

**Features:**

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | 001-spa-routing | Add catch-all SPA fallback route so /gui/* sub-paths serve index.html | BL-063 | None |
| 2 | 002-pagination-fix | Add count() to AsyncProjectRepository and fix projects endpoint total | BL-064 | None |
| 3 | 003-websocket-broadcasts | Wire ConnectionManager.broadcast() into scan and project API operations | BL-065 | None (but sequenced last due to cross-cutting complexity) |

---

## Execution Order

### Theme Order

**Theme 1 (01-observability-pipeline) before Theme 2 (02-gui-runtime-fixes).**

Rationale:
- Theme 1 modifies `app.py` startup/DI code. Theme 2 modifies API routers and adds routes to `app.py`. Sequential execution avoids concurrent modification of `app.py`.
- Theme 1 features are lower risk (straightforward DI wiring), making them a good warm-up.
- No functional dependency between themes — ordering is for code-conflict avoidance.

### Feature Order Within Theme 1

1. **001-ffmpeg-observability** (BL-059, P1): Adds `ffmpeg_executor` kwarg to `create_app()`. Wraps `RealFFmpegExecutor()` with `ObservableFFmpegExecutor()` in lifespan.
2. **002-audit-logging** (BL-060, P2): Adds `audit_logger` kwarg to `create_app()`. Instantiates `AuditLogger` in lifespan and passes to repositories.
3. **003-file-logging** (BL-057, P2): Extends `configure_logging()` with `RotatingFileHandler`. Adds `logs/` directory creation. Updates `.gitignore`.

Rationale: Features 001 and 002 both modify `create_app()` signature and lifespan. Sequential ordering prevents merge conflicts. Feature 003 is independent (modifies `logging.py`, not `app.py` DI) but is sequenced last as it has a dependency on BL-056 (completed in v008).

### Feature Order Within Theme 2

1. **001-spa-routing** (BL-063, P1): Adds catch-all route for `/gui/{path:path}` in `app.py`.
2. **002-pagination-fix** (BL-064, P2): Adds `count()` to `AsyncProjectRepository` protocol and implementations. Updates projects router.
3. **003-websocket-broadcasts** (BL-065, P2): Injects `ws_manager` into scan and project routers. Adds `broadcast()` calls at operation completion points.

Rationale: Feature 001 is highest priority (P1) and lowest complexity. Feature 003 is saved for last as the highest-complexity item (touches multiple routers, cross-cutting concern).

---

## Research Sources Adopted

### SPA Fallback Pattern
- **Finding:** `StaticFiles(html=True)` does NOT serve `index.html` for arbitrary sub-paths — it returns 404.
- **Decision:** Add a FastAPI catch-all route `@app.get("/gui/{path:path}")` that serves `index.html` content, registered after API routers but before/alongside the StaticFiles mount.
- **Constraint:** Route must be conditional on `gui_dir.is_dir()` (LRN-020).
- **Source:** `comms/outbox/versions/design/v009/004-research/external-research.md` (Section 1)

### Idempotent Handler Guard
- **Finding:** Existing pattern uses `type(h) is logging.StreamHandler` exact-type check.
- **Decision:** BL-057 uses `type(h) is RotatingFileHandler` following the same pattern.
- **Source:** `comms/outbox/versions/design/v009/004-research/codebase-patterns.md` (Section 2)

### Constructor DI Pattern
- **Finding:** `create_app()` accepts optional kwargs stored on `app.state`. Tests inject via same mechanism.
- **Decision:** BL-059 adds `ffmpeg_executor` kwarg; BL-060 adds `audit_logger` kwarg.
- **Source:** `comms/outbox/versions/design/v009/004-research/codebase-patterns.md` (Section 1)

### Repository count() Pattern
- **Finding:** `AsyncVideoRepository` has `count()` at protocol, SQLite, and InMemory levels. `AsyncProjectRepository` is missing it entirely.
- **Decision:** Copy the pattern exactly — add to protocol, both implementations, and wire into projects router.
- **Source:** `comms/outbox/versions/design/v009/004-research/codebase-patterns.md` (Section 4)

### WebSocket Broadcast via app.state
- **Finding:** `ws_manager` is stored on `app.state` at `app.py:149`. Events defined in `websocket/events.py`. `build_event()` constructs payloads. Zero broadcast calls exist in routers.
- **Decision:** Access via `request.app.state.ws_manager` and call `broadcast(build_event(...))` at operation completion points in scan and project routers.
- **Source:** `comms/outbox/versions/design/v009/004-research/codebase-patterns.md` (Section 5)

### Log Rotation Configuration
- **Value:** `maxBytes=10MB`, `backupCount=5` (configurable via settings).
- **Source:** `comms/outbox/versions/design/v009/004-research/evidence-log.md`

---

## Handler Concurrency Decisions

Not applicable — no new MCP tool handlers are introduced in v009. All changes are to the stoat-and-ferret application's FastAPI endpoints and internal wiring, not MCP server handlers.
