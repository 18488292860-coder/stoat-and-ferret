# Refined Logical Design — v005: GUI Shell, Library Browser & Project Manager

## Version Overview

v005 delivers the first GUI layer for stoat-and-ferret: a React frontend served from FastAPI, WebSocket real-time events, application shell with navigation, library browser with thumbnails, project manager, and E2E test infrastructure. This completes Phase 1 (Milestones M1.10-M1.12).

**Goals:**
- Scaffold a React 18 / TypeScript / Vite frontend project with CI integration
- Add WebSocket support for real-time event broadcasting
- Build the application shell, dashboard, library browser, and project manager
- Add thumbnail generation for the video library
- Fix pagination total count (tech debt from v003)
- Establish Playwright E2E test infrastructure

**Structure:** 4 themes, 13 features, covering all 10 backlog items.

**Risk status:** Framework confirmed as React (R-1 resolved). All remaining risks accepted with mitigations or deferred to runtime testing. No design-breaking issues found.

---

## Theme 01: Frontend Foundation (`01-frontend-foundation`)

**Goal:** Scaffold the frontend project, configure static file serving from FastAPI, set up CI pipeline integration, and add WebSocket real-time event support.

**Backlog Items:** BL-003, BL-028, BL-029

| Feature | Name | Goal | Backlog | Dependencies |
|---------|------|------|---------|--------------|
| 001 | `001-frontend-scaffolding` | Scaffold React/Vite/TypeScript project in `gui/`, configure FastAPI static file serving with `StaticFiles(html=True)`, and update CI pipeline with new `frontend` job | BL-028, BL-003 | None |
| 002 | `002-websocket-endpoint` | Implement `/ws` WebSocket endpoint with ConnectionManager, event broadcasting, heartbeat (30s default), and correlation ID support | BL-029 | 001 (app factory changes) |
| 003 | `003-settings-and-docs` | Add new settings fields (`thumbnail_dir`, `gui_static_path`, `ws_heartbeat_interval`) and update architecture/API/AGENTS docs | BL-003, BL-029 | 001, 002 |

**Risk mitigations integrated:**
- R-1: React confirmed — scaffolding uses `create-vite` with `react-ts` template
- R-2: `StaticFiles(html=True)` sufficient for flat v005 routes; `SPAStaticFiles` subclass documented as fallback in implementation plan
- R-5: CI uses separate `frontend` job on ubuntu-latest with `node_modules` caching, parallel to existing test matrix

---

## Theme 02: Backend Services (`02-backend-services`)

**Goal:** Add backend capabilities that the GUI components consume: thumbnail generation pipeline and pagination total count fix.

**Backlog Items:** BL-032, BL-034

| Feature | Name | Goal | Backlog | Dependencies |
|---------|------|------|---------|--------------|
| 001 | `001-thumbnail-pipeline` | Implement thumbnail generation service using FFmpeg executor pattern, add `GET /api/videos/{id}/thumbnail` endpoint, integrate with scan service, store thumbnails in configurable `thumbnail_dir` | BL-032 | Theme 01 complete (settings fields) |
| 002 | `002-pagination-total-count` | Add `count()` to async repository protocol and implementations, fix `list_videos` and `search` endpoints to return true total via `SELECT COUNT(*)` | BL-034 | None (independent of Theme 01) |

**Risk mitigations integrated:**
- R-4: On-scan thumbnail generation. Scan is already an async job (`AsyncioJobQueue`), so latency adds to background job, not API response. `FakeFFmpegExecutor` enables fast testing. `thumbnail_path` field already exists on `Video` model — no migration needed.
- BL-034 fix scope: async protocol + `AsyncSQLiteVideoRepository` + `AsyncInMemoryVideoRepository` + router (2 lines). Schema already has `total: int`.

---

## Theme 03: GUI Components (`03-gui-components`)

**Goal:** Build the four main GUI panels: application shell with navigation, dashboard, library browser, and project manager.

**Backlog Items:** BL-030, BL-031, BL-033, BL-035

| Feature | Name | Goal | Backlog | Dependencies |
|---------|------|------|---------|--------------|
| 001 | `001-application-shell` | Build application shell with navigation tabs, health indicator, status bar, progressive tab disclosure, and WebSocket reconnection via `useWebSocket` hook | BL-030 | Theme 01 complete |
| 002 | `002-dashboard-panel` | Build dashboard with health cards, real-time activity log via WebSocket, and metrics overview | BL-031 | 001 (shell provides layout), Theme 01 (WebSocket) |
| 003 | `003-library-browser` | Build library browser with video grid, thumbnails, search (300ms debounce), sort/filter, scan modal, and pagination or virtual scrolling | BL-033 | 001 (shell), Theme 02 (thumbnails + pagination) |
| 004 | `004-project-manager` | Build project manager with list, creation modal, details view with timeline positions, and delete confirmation | BL-035 | 001 (shell) |

**Risk mitigations integrated:**
- R-3: WebSocket reconnection logic in `useWebSocket.ts` hook (Feature 001). Client detects disconnect, auto-reconnects with exponential backoff.
- R-6/UQ-2: Feature 003 allows "pagination or virtual scrolling" per BL-033 AC#5. Implementation will evaluate tanstack-virtual first, fall back to pagination if grid support is insufficient.

---

## Theme 04: E2E Testing (`04-e2e-testing`)

**Goal:** Establish Playwright E2E test infrastructure with CI integration.

**Backlog Items:** BL-036

| Feature | Name | Goal | Backlog | Dependencies |
|---------|------|------|---------|--------------|
| 001 | `001-playwright-setup` | Configure Playwright with CI integration (ubuntu-latest only), `webServer` config for FastAPI, browser installation with caching | BL-036 | Theme 03 complete |
| 002 | `002-e2e-test-suite` | Write E2E tests for navigation, scan trigger, project creation, and WCAG AA accessibility checks via `@axe-core/playwright` | BL-036 | 001 |

**Risk mitigations integrated:**
- R-5: Playwright CI runs on ubuntu-latest only (not full 3x3 matrix). Browser binaries cached via `actions/cache`. Separate `e2e` job runs parallel to existing test matrix.
- UQ-4: No frontend coverage threshold enforced in v005. E2E tests provide integration validation. Coverage baseline established after Theme 03 for ratcheting in v006+.

---

## Execution Order

### Theme Order

```
Theme 01: Frontend Foundation  →  Theme 02: Backend Services  →  Theme 03: GUI Components  →  Theme 04: E2E Testing
```

**Unchanged from Task 005.** No risk findings altered the dependency structure.

### Feature Order Within Themes

- **Theme 01:** 001 → 002 → 003 (linear; each depends on prior)
- **Theme 02:** 001 → 002 (linear; thumbnail pipeline first, then pagination fix)
- **Theme 03:** 001 → 002 → 003 → 004 (linear; shell first, then panels in dependency order)
- **Theme 04:** 001 → 002 (linear; setup before test authoring)

### Dependency Graph

```
BL-028/BL-003 (frontend scaffolding)
    ├── BL-029 (WebSocket) ──── BL-031 (dashboard)
    ├── BL-030 (app shell) ──┬─ BL-031 (dashboard)
    │                        ├─ BL-033 (library browser)
    │                        ├─ BL-035 (project manager)
    │                        └─ BL-036 (E2E tests)
    └── BL-032 (thumbnails) ── BL-033 (library browser)
BL-034 (pagination fix) ────── BL-033 (library browser)
```

---

## Backlog Coverage

All 10 backlog items mapped. No items deferred. **Mandatory scope fully preserved.**

| Backlog | Theme | Feature(s) |
|---------|-------|------------|
| BL-003 | 01 | 001, 003 |
| BL-028 | 01 | 001 |
| BL-029 | 01 | 002, 003 |
| BL-030 | 03 | 001 |
| BL-031 | 03 | 002 |
| BL-032 | 02 | 001 |
| BL-033 | 03 | 003 |
| BL-034 | 02 | 002 |
| BL-035 | 03 | 004 |
| BL-036 | 04 | 001, 002 |
