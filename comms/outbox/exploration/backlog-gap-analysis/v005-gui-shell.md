# v005 Gap Analysis: GUI Shell, Library Browser & Project Manager

**Milestones:** M1.10 (Application Shell & Dashboard), M1.11 (Library Browser), M1.12 (Project Manager)
**Design docs:** 08-gui-architecture.md (primary), 01-roadmap.md

## Existing Coverage

- BL-003: EXP-003 FastAPI static file serving for GUI (P2) — prerequisite exploration only

## Gaps Identified

### Pre-requisite: Frontend Project Setup

**Gap 1: No frontend project exists.** No `gui/` directory, no package.json, no framework choice finalized. 08-gui-architecture.md suggests React 18+ or Svelte 4+ with Vite and Tailwind. BL-003 covers investigation but not implementation.

### M1.10: Application Shell & Dashboard

**Gap 2: WebSocket endpoint.** M1.10 requires `/ws` WebSocket endpoint for real-time events. No WebSocket support exists in the current FastAPI app.

**Gap 3: Application shell and dashboard components.** No navigation, status bar, health indicator, metrics cards, or activity log components exist.

### M1.11: Library Browser

**Gap 4: Video grid/list UI.** No frontend components for video display, search, sort, or filter.

**Gap 5: Thumbnail generation.** The library browser spec assumes thumbnail display. No thumbnail generation pipeline exists (FFmpeg frame extraction → storage → serving).

**Gap 6: Pagination for large libraries.** Backend returns page-based results but lacks true total count (identified in v003 retrospective). Virtual scrolling needs proper count support.

### M1.12: Project Manager

**Gap 7: Project manager UI.** No frontend components for project list, creation modal, or details view.

**Gap 8: Timeline position display.** M1.12 requires displaying Rust-calculated timeline positions in the project details view. The clip model exists but no timeline visualization.

## Suggested Backlog Items

### 1. EXP: Frontend Framework Selection and Vite Setup
- **Priority:** P1 | **Tags:** `v005`, `gui`, `investigation` | **Size:** S
- **Description:** Finalize React vs Svelte decision. Set up Vite project in `gui/` with Tailwind CSS, TypeScript, and test framework (Vitest). Configure FastAPI to serve `gui/dist/` static files. This extends BL-003 into an actionable decision.
- **Acceptance criteria:**
  1. Framework selected with documented rationale
  2. `gui/` project scaffolded with Vite, Tailwind, TypeScript
  3. `npm run build` produces `gui/dist/`
  4. FastAPI serves built frontend at `/gui/*`
  5. Dev proxy configured for Vite HMR
- **Depends on:** BL-003 (EXP-003)

### 2. Implement WebSocket Endpoint for Real-Time Events
- **Priority:** P1 | **Tags:** `v005`, `websocket`, `api` | **Size:** M
- **Description:** Add `/ws` WebSocket endpoint to FastAPI per M1.10 and 08-gui-architecture.md. Support event types: system health changes, activity feed, and future render progress.
- **Acceptance criteria:**
  1. `/ws` endpoint accepts WebSocket connections
  2. Health status changes broadcast to connected clients
  3. Activity events (scan started/completed, project created) broadcast
  4. Connection lifecycle tests (connect, disconnect, reconnect)
  5. Integration with existing correlation ID middleware
- **Depends on:** None (backend only)

### 3. Build Application Shell and Navigation
- **Priority:** P1 | **Tags:** `v005`, `gui`, `shell` | **Size:** M
- **Description:** Create the application shell per M1.10: navigation tabs (Dashboard, Library, Projects), status bar with system info, health status indicator (green/yellow/red) connected to `/health/ready`.
- **Acceptance criteria:**
  1. Navigation between Dashboard, Library, Projects tabs
  2. Health indicator polls `/health/ready` and shows colored status
  3. Status bar displays connection state
  4. Progressive tabs — only show features with working backends
  5. Component unit tests (Vitest)
- **Depends on:** Item 1

### 4. Build Dashboard Panel
- **Priority:** P2 | **Tags:** `v005`, `gui`, `dashboard` | **Size:** M
- **Description:** Create dashboard per M1.10: system health cards (Python, Rust core, FFmpeg status), recent activity log via WebSocket, and metrics overview from `/metrics`.
- **Acceptance criteria:**
  1. Health cards show individual component status
  2. Activity log receives and displays WebSocket events
  3. Metrics cards show API request count, Rust op timing
  4. Auto-refresh on configurable interval
- **Depends on:** Items 2, 3

### 5. Implement Thumbnail Generation Pipeline
- **Priority:** P1 | **Tags:** `v005`, `thumbnails`, `ffmpeg` | **Size:** M
- **Description:** Extract representative frames from videos using FFmpeg for library browser display. Store thumbnails and serve via API endpoint.
- **Acceptance criteria:**
  1. Thumbnail generated during video scan (or on-demand)
  2. GET `/api/videos/{id}/thumbnail` returns image
  3. Configurable thumbnail size (default 320x180)
  4. Graceful fallback for videos where extraction fails
  5. Recording test double captures thumbnail generation commands
- **Depends on:** None (backend only)

### 6. Build Library Browser
- **Priority:** P1 | **Tags:** `v005`, `gui`, `library` | **Size:** L
- **Description:** Create library browser per M1.11: video grid with thumbnails, search bar with debounced FTS5 query, sort/filter controls, scan directory modal with progress, and video detail view.
- **Acceptance criteria:**
  1. Video grid displays thumbnails, filename, duration
  2. Search calls `/api/videos/search` with debounce
  3. Sort by date, name, duration works
  4. Scan modal triggers directory scan and shows progress
  5. Virtual scrolling or pagination for 100+ videos
- **Depends on:** Items 1, 5

### 7. Fix Pagination Total Count
- **Priority:** P2 | **Tags:** `v005`, `api`, `pagination` | **Size:** S
- **Description:** Add true total count to paginated list endpoints. Identified as tech debt in v003 retrospective. Required for library browser virtual scrolling.
- **Acceptance criteria:**
  1. List endpoints return `total` field with full count
  2. Search endpoint returns total matching results
  3. Existing pagination tests updated
- **Depends on:** None

### 8. Build Project Manager
- **Priority:** P1 | **Tags:** `v005`, `gui`, `projects` | **Size:** L
- **Description:** Create project manager per M1.12: project list with stats, "New Project" modal (output resolution, fps, format), project details with clip list showing Rust-calculated timeline positions, and open/delete actions.
- **Acceptance criteria:**
  1. Project list shows name, creation date, clip count
  2. New project modal validates output settings
  3. Project details displays clip list with timeline positions
  4. Delete action with confirmation dialog
  5. Component unit tests
- **Depends on:** Items 1, 3

### 9. E2E Test Infrastructure
- **Priority:** P2 | **Tags:** `v005`, `testing`, `e2e` | **Size:** M
- **Description:** Set up Playwright for E2E testing per 08-gui-architecture.md quality requirements. Cover critical user paths: navigation, scan, search, project creation.
- **Acceptance criteria:**
  1. Playwright configured with CI integration
  2. At least 3 E2E tests covering core paths
  3. Tests run against FastAPI + built frontend
  4. Accessibility checks (WCAG AA) included
- **Depends on:** Items 3, 6, 8
