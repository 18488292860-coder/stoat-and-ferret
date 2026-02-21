# Wiring Audit v005 - Results

Three wiring gaps found across v005 themes. Two are minor (degraded functionality) and one is cosmetic (dead code/infrastructure). The known logging gap is excluded per audit scope. Overall, the frontend-to-backend integration is solid: all 22 GUI components are rendered, all API hooks call real endpoints, and all 5 routers plus the WebSocket endpoint are mounted. The gaps are at integration seams rather than missing fundamentals.

## Theme 01: Frontend Foundation

**No wiring gaps found.** Frontend scaffolding serves correctly from `/gui` via StaticFiles. WebSocket endpoint is mounted at `/ws` and accepts connections. Settings fields are registered.

### Gap: SPA Routing Fallback Missing (severity: minor)

- **Source:** 001-frontend-scaffolding + 001-application-shell interaction
- FastAPI `StaticFiles(html=True)` does NOT fall back to `index.html` for unknown sub-paths. Navigating directly to `/gui/library` or `/gui/projects` returns 404.
- React Router handles client-side navigation fine, but bookmarks, page refreshes, and shared URLs on sub-pages break.
- The 002-e2e-test-suite completion report documents this explicitly and works around it by navigating via tab clicks only.
- See `gaps-found.md` for details.

## Theme 02: Backend Services

### Gap: Projects Endpoint Returns Wrong Total Count (severity: minor)

- **Source:** 002-pagination-total-count scope
- The `count()` method was added to `AsyncVideoRepository` but not to `AsyncProjectRepository`. The `GET /api/v1/projects` endpoint returns `total=len(projects)` (page size) instead of the true database total.
- Frontend pagination metadata will be incorrect when there are more projects than `limit`.
- See `gaps-found.md` for details.

## Theme 03: GUI Components

**No component-level wiring gaps.** All pages, hooks, and stores are imported and used. API calls hit real endpoints. Zustand stores function correctly.

## Theme 04: E2E Testing

**No wiring gaps found.** Playwright configured with webServer auto-start, browser caching, and CI integration.

## Cross-Theme Gap

### Gap: WebSocket Broadcast Infrastructure Never Triggered (severity: minor)

- **Source:** 002-websocket-endpoint + 002-dashboard-panel interaction
- The backend defines event types (SCAN_STARTED, SCAN_COMPLETED, PROJECT_CREATED, HEALTH_STATUS) and the frontend ActivityLog listens for WebSocket messages, but no API operation ever calls `ConnectionManager.broadcast()`. Only heartbeat pings are sent.
- The Dashboard ActivityLog renders correctly but will only ever show heartbeat entries at runtime.
- See `gaps-found.md` for details.
