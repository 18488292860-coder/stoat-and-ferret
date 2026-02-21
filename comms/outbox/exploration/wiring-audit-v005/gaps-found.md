# Wiring Gaps Found - v005

## Gap 1: WebSocket Event Broadcast Never Triggered

- **Version/Theme/Feature:** v005 / 01-frontend-foundation / 002-websocket-endpoint + 03-gui-components / 002-dashboard-panel
- **Severity:** minor (degraded)

### Designed

The WebSocket feature defined five event types in `src/stoat_ferret/api/websocket/events.py`: `HEALTH_STATUS`, `SCAN_STARTED`, `SCAN_COMPLETED`, `PROJECT_CREATED`, `HEARTBEAT`. The `ConnectionManager.broadcast()` method exists at `src/stoat_ferret/api/websocket/manager.py:49`. The Dashboard's `ActivityLog` component (`gui/src/components/ActivityLog.tsx:24-42`) consumes WebSocket messages and displays them with the Zustand activity store.

### What's Actually Wired

- `ConnectionManager` is created and stored on `app.state.ws_manager` (app.py:54-55)
- The `/ws` endpoint accepts connections and sends heartbeat pings (ws.py:42)
- The frontend `useWebSocket` hook connects and receives messages (Shell.tsx:13)
- The `ActivityLog` component renders messages into a scrollable list

**Missing link:** No API endpoint (scan, project creation, health) ever accesses `app.state.ws_manager` to call `broadcast()`. A grep for `broadcast` across `src/stoat_ferret/` returns only the definition and docstrings, never an invocation. The only messages clients receive are `HEARTBEAT` pings from ws.py.

### Completion Report Coverage

The 002-websocket-endpoint handoff-to-next.md explicitly identified this: "Integration points identified for scan endpoints, project creation, and health endpoint to emit WebSocket events." This was documented as future work, not a missed requirement.

---

## Gap 2: Projects List Returns Wrong Pagination Total

- **Version/Theme/Feature:** v005 / 02-backend-services / 002-pagination-total-count
- **Severity:** minor (degraded)

### Designed

Feature 002-pagination-total-count required adding `count()` to repository protocols and fixing the `total` field in list/search endpoints to return the true database count rather than the page length.

### What's Actually Wired

- `AsyncVideoRepository.count()` was added (async_repository.py:100, 210, 365)
- `GET /api/v1/videos` correctly calls `repo.count()` and returns the true total (videos.py:73)
- `GET /api/v1/projects` returns `total=len(projects)` at projects.py:113 - this is the count of items in the current page, not the total in the database
- `AsyncProjectRepository` (project_repository.py) has no `count()` method

### Impact

When a user has more projects than `limit` (default 20), the frontend's ProjectList will show incorrect pagination info (e.g., "20 projects" when there are 50).

### Completion Report Coverage

Not flagged. The 002-pagination-total-count feature scope was limited to videos. The projects endpoint was not mentioned in the requirements or completion report. This gap exists because the same pattern wasn't applied consistently.

---

## Gap 3: SPA Routing Fallback Missing for Direct URL Navigation

- **Version/Theme/Feature:** v005 / 01-frontend-foundation / 001-frontend-scaffolding + 03-gui-components / 001-application-shell
- **Severity:** minor (degraded)

### Designed

The frontend uses React Router with `basename="/gui"` and routes for `/`, `/library`, `/projects`. FastAPI mounts the built frontend at `/gui` with `StaticFiles(directory=gui_dir, html=True)`.

### What's Actually Wired

- Starlette's `StaticFiles` with `html=True` serves `index.html` for the root path and directory paths, but does NOT fall back to `index.html` for arbitrary sub-paths
- Navigating to `/gui/` works (serves `gui/dist/index.html`)
- Navigating directly to `/gui/library` returns 404 (no file `library` or `library.html` in `gui/dist/`)
- Client-side navigation via React Router (clicking tabs) works perfectly

### Impact

Users cannot bookmark, refresh, or share direct URLs to sub-pages. Browser refresh on any page other than Dashboard returns 404.

### Completion Report Coverage

The 002-e2e-test-suite completion report explicitly documents this: "Tests navigate via client-side routing (click tabs) rather than direct URLs because FastAPI StaticFiles doesn't handle SPA fallback for sub-paths." This is a known limitation documented but not resolved.

### Fix

Add a catch-all route in FastAPI before the StaticFiles mount that serves `index.html` for any `/gui/{path}` not matching a static asset. This is a standard SPA hosting pattern.
