# Backlog Details — v005

## BL-003: EXP-003: FastAPI static file serving for GUI
- **Priority:** P2 | **Size:** M | **Tags:** investigation, v005-prerequisite, gui, fastapi
- **Description:**
  > Investigate serving the React/Svelte GUI from FastAPI: How do we configure FastAPI to serve static files from Vite build output? What's the development workflow — proxy setup for hot reload? How do we handle the /gui/* route mounting? What about index.html fallback for SPA routing? This informs v005 (GUI shell).
- **Acceptance Criteria:**
  1. FastAPI StaticFiles mount configuration documented
  2. Vite build output integration explained
  3. Development workflow (hot reload) documented
  4. Production deployment pattern shown
- **Quality Assessment:**
  - Description: 55 words — adequate depth with clear investigation questions
  - AC testability: All criteria use action verbs (documented, explained, shown) — PASS
  - Use case: Was formulaic — updated with authentic developer scenario in notes
- **Complexity:** Low-medium. Investigation/documentation task, no code changes to production

## BL-028: Frontend framework selection and Vite project setup
- **Priority:** P1 | **Size:** M | **Tags:** v005, gui, investigation
- **Description:**
  > No frontend project exists — no gui/ directory, package.json, or framework choice has been finalized. The 08-gui-architecture.md suggests React 18+ or Svelte 4+ with Vite and Tailwind. BL-003 covers investigating FastAPI static file serving but not the framework decision or project scaffolding. All v005 GUI milestones (M1.10–M1.12) are blocked until a frontend project is in place.
- **Acceptance Criteria:**
  1. Framework selected (React vs Svelte) with documented rationale
  2. gui/ project scaffolded with Vite, Tailwind CSS, and TypeScript
  3. npm run build produces gui/dist/ with a working bundle
  4. FastAPI serves the built frontend at /gui/* routes
  5. Dev proxy configured for Vite HMR during development
- **Quality Assessment:**
  - Description: 60 words — good context explaining the gap and blocking relationship
  - AC testability: All criteria have verbs (selected, scaffolded, produces, serves, configured) — PASS
  - Use case: Was formulaic — updated with authentic scenario in notes
- **Complexity:** Medium. Framework decision + scaffolding + FastAPI integration + dev workflow

## BL-029: WebSocket endpoint for real-time events
- **Priority:** P1 | **Size:** M | **Tags:** v005, websocket, api
- **Description:**
  > M1.10 requires a /ws WebSocket endpoint for real-time event broadcasting, but no WebSocket support exists in the current FastAPI app. The application shell needs live health status updates and the dashboard needs an activity feed. Without WebSocket support, the GUI must resort to polling, creating unnecessary load and delayed feedback.
- **Acceptance Criteria:**
  1. /ws endpoint accepts WebSocket connections with proper handshake
  2. Health status changes broadcast to all connected clients
  3. Activity events (scan started/completed, project created) broadcast in real time
  4. Connection lifecycle tested: connect, disconnect, reconnect scenarios
  5. WebSocket messages include correlation IDs from existing middleware
- **Quality Assessment:**
  - Description: 62 words — explains problem, impact, and alternative clearly
  - AC testability: All criteria have verbs (accepts, broadcast, tested, include) — PASS
  - Use case: Was formulaic — updated with real-time scan scenario in notes
- **Complexity:** Medium-high. New transport layer, connection management, event broadcasting, middleware integration

## BL-030: Application shell and navigation components
- **Priority:** P1 | **Size:** M | **Tags:** v005, gui, shell
- **Description:**
  > M1.10 specifies an application shell with navigation tabs, status bar, and health indicator, but no frontend components exist. The shell is the frame that hosts all other GUI panels (library browser, project manager, effect workshop). Without it, individual components have no layout structure or navigation context.
- **Acceptance Criteria:**
  1. Navigation between Dashboard, Library, and Projects tabs works with URL routing
  2. Health indicator polls /health/ready and displays green/yellow/red status
  3. Status bar displays WebSocket connection state
  4. Progressive tabs — only shows features whose backends are available
  5. Component unit tests pass in Vitest
- **Quality Assessment:**
  - Description: 62 words — clearly explains the shell's role as host frame
  - AC testability: All criteria have verbs (works, polls/displays, displays, shows, pass) — PASS
  - Use case: Was formulaic — updated with navigation and degraded-state scenario in notes
- **Complexity:** Medium. Layout, routing, health polling, WebSocket state display, progressive disclosure

## BL-031: Dashboard panel with health cards and activity log
- **Priority:** P2 | **Size:** M | **Tags:** v005, gui, dashboard
- **Description:**
  > M1.10 specifies a dashboard with system health cards, recent activity log, and metrics overview. No dashboard component exists. The dashboard is the landing page and primary system health visibility tool. Without it, users have no centralized view of component status (Python, Rust core, FFmpeg) or recent system activity.
- **Acceptance Criteria:**
  1. Health cards display individual component status (Python, Rust core, FFmpeg)
  2. Activity log receives and displays WebSocket events in real time
  3. Metrics cards show API request count and Rust operation timing from /metrics
  4. Dashboard auto-refreshes on a configurable interval
- **Quality Assessment:**
  - Description: 58 words — adequate context for a dashboard landing page
  - AC testability: All criteria have verbs (display, receives/displays, show, auto-refreshes) — PASS
  - Use case: Was formulaic — updated with concrete health status scenario in notes
- **Complexity:** Medium. Multiple data sources (health API, WebSocket, metrics), auto-refresh logic

## BL-032: Thumbnail generation pipeline for video library
- **Priority:** P1 | **Size:** M | **Tags:** v005, thumbnails, ffmpeg
- **Description:**
  > The library browser spec (M1.11) assumes thumbnail display for videos, but no thumbnail generation pipeline exists. Videos are scanned and stored with metadata, but no representative frame is extracted. Without thumbnails, the library browser would display a text-only list, degrading the visual browsing experience that is core to a video editing tool.
- **Acceptance Criteria:**
  1. Thumbnail generated during video scan or on first access
  2. GET /api/videos/{id}/thumbnail returns the thumbnail image
  3. Configurable thumbnail size with default 320x180
  4. Graceful fallback returns a placeholder for videos where extraction fails
  5. RecordingFFmpegExecutor captures thumbnail generation commands for testing
- **Quality Assessment:**
  - Description: 58 words — good problem statement linking to library browser dependency
  - AC testability: All criteria have verbs (generated, returns, configurable, returns, captures) — PASS
  - Use case: Was formulaic — updated with visual browsing scenario in notes
- **Complexity:** Medium-high. FFmpeg integration, storage, API endpoint, error handling, test infrastructure

## BL-033: Library browser with video grid, search, and scan UI
- **Priority:** P1 | **Size:** M | **Tags:** v005, gui, library
- **Description:**
  > M1.11 specifies a library browser with video grid, search, sort/filter, and scan controls. No frontend components for video display exist. The library browser is the primary entry point for working with media — users need to find, browse, and select videos before creating projects. The backend API endpoints (search, list, scan) exist from v003 but have no GUI consumer.
- **Acceptance Criteria:**
  1. Video grid displays thumbnails, filename, and duration for each video
  2. Search bar calls /api/videos/search with debounced input
  3. Sort by date, name, or duration updates the grid
  4. Scan modal triggers directory scan and shows progress feedback
  5. Virtual scrolling or pagination handles libraries with 100+ videos
- **Quality Assessment:**
  - Description: 68 words — strong context linking backend availability to frontend gap
  - AC testability: All criteria have verbs (displays, calls, updates, triggers/shows, handles) — PASS
  - Use case: Was formulaic — updated with large-library search/scan scenario in notes
- **Complexity:** High. Multiple UI interactions, API integration, virtual scrolling, real-time scan feedback

## BL-034: Fix pagination total count for list endpoints
- **Priority:** P2 | **Size:** M | **Tags:** v005, api, pagination
- **Description:**
  > Paginated list endpoints return page-based results but lack a true total count. This was identified as tech debt in the v003 retrospective. The library browser needs the total count for virtual scrolling (to size the scroll container) and for displaying "X of Y results" feedback. Without it, the frontend cannot accurately represent the full dataset size.
- **Acceptance Criteria:**
  1. List endpoints return a total field with the full count of matching items
  2. Search endpoint returns total matching results separate from page size
  3. Existing pagination tests updated to verify total count accuracy
- **Quality Assessment:**
  - Description: 58 words — clear tech debt lineage from v003 with concrete impact
  - AC testability: All criteria have verbs (return, returns, updated) — PASS
  - Use case: Was formulaic — updated with "Showing X of Y" UI scenario in notes
- **Complexity:** Low-medium. Backend-only change to existing endpoints, well-defined scope

## BL-035: Project manager with list, creation, and details views
- **Priority:** P1 | **Size:** M | **Tags:** v005, gui, projects
- **Description:**
  > M1.12 specifies a project manager with project list, creation modal, and details view showing Rust-calculated timeline positions. No frontend components for project management exist. The project API endpoints exist from v003, but users cannot create, browse, or inspect projects through the GUI. This is the last GUI milestone in Phase 1.
- **Acceptance Criteria:**
  1. Project list displays name, creation date, and clip count
  2. New Project modal validates output settings (resolution, fps, format)
  3. Project details view displays clip list with Rust-calculated timeline positions
  4. Delete action requires confirmation dialog before execution
  5. Component unit tests pass in Vitest
- **Quality Assessment:**
  - Description: 60 words — good context noting this is the final Phase 1 GUI milestone
  - AC testability: All criteria have verbs (displays, validates, displays, requires, pass) — PASS
  - Use case: Was formulaic — updated with full project lifecycle scenario in notes
- **Complexity:** Medium-high. Multiple views, form validation, Rust timeline data display, delete confirmation

## BL-036: Playwright E2E test infrastructure for GUI
- **Priority:** P2 | **Size:** M | **Tags:** v005, testing, e2e
- **Description:**
  > The 08-gui-architecture.md quality requirements specify Playwright for E2E testing, but no E2E test infrastructure exists. Unit tests (Vitest) cover individual components, but there are no tests verifying that the full stack (FastAPI + built frontend) works end-to-end. Without E2E tests, regressions in the integration between frontend and backend go undetected.
- **Acceptance Criteria:**
  1. Playwright configured with CI integration in GitHub Actions
  2. At least 3 E2E tests covering navigation, scan trigger, and project creation
  3. Tests run against FastAPI serving the built frontend bundle
  4. Accessibility checks (WCAG AA) included in test assertions
- **Quality Assessment:**
  - Description: 50 words — borderline but adequate; explains the testing gap clearly
  - AC testability: All criteria have verbs (configured, covering, run, included) — PASS
  - Use case: Was formulaic — updated with CI regression-catching scenario in notes
- **Complexity:** Medium. Infrastructure setup, CI integration, test authoring, accessibility checks
