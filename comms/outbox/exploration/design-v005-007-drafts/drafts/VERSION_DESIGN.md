# VERSION_DESIGN - v005: GUI Shell, Library Browser & Project Manager

## Overview

v005 delivers the first GUI layer for stoat-and-ferret: a React frontend served from FastAPI, WebSocket real-time events, application shell with navigation, library browser with thumbnails, project manager, and E2E test infrastructure. This completes Phase 1 (Milestones M1.10-M1.12).

## Goals

- Scaffold a React 18 / TypeScript / Vite frontend project with CI integration
- Add WebSocket support for real-time event broadcasting
- Build the application shell, dashboard, library browser, and project manager
- Add thumbnail generation for the video library
- Fix pagination total count (tech debt from v003)
- Establish Playwright E2E test infrastructure

## Scope

**Backlog Items:** BL-003, BL-028, BL-029, BL-030, BL-031, BL-032, BL-033, BL-034, BL-035, BL-036 (10 items: 6x P1, 4x P2)

**Structure:** 4 themes, 13 features

## Themes

| # | Theme | Goal | Backlog Items |
|---|-------|------|---------------|
| 01 | Frontend Foundation | Scaffold frontend, static file serving, WebSocket support | BL-003, BL-028, BL-029 |
| 02 | Backend Services | Thumbnail pipeline, pagination fix | BL-032, BL-034 |
| 03 | GUI Components | App shell, dashboard, library browser, project manager | BL-030, BL-031, BL-033, BL-035 |
| 04 | E2E Testing | Playwright infrastructure and test suite | BL-036 |

## Execution Order

```
Theme 01: Frontend Foundation -> Theme 02: Backend Services -> Theme 03: GUI Components -> Theme 04: E2E Testing
```

Theme 01 first (infrastructure-first per LRN-019), Theme 02 provides backend prerequisites for GUI panels, Theme 03 builds user-facing components, Theme 04 validates the integrated system.

## Key Technical Decisions

| Decision | Rationale | Evidence |
|----------|-----------|----------|
| React 18+ with TypeScript | Design docs React-aligned (JSX examples, hooks, Zustand) | `004-research/external-research.md` |
| Vite with `react-ts` template | Standard tooling, dev proxy for API/WS | `004-research/external-research.md` |
| `StaticFiles(html=True)` for SPA | Built-in FastAPI support, sufficient for flat routes | `006-critical-thinking/risk-assessment.md` R-2 |
| ConnectionManager for WebSocket | Injectable via `create_app()` DI pattern (LRN-005) | `004-research/codebase-patterns.md` |
| FFmpeg executor for thumbnails | Reuses Real/Recording/Fake testing pattern (LRN-008) | `004-research/codebase-patterns.md` |
| Playwright for E2E | Design doc specification, WCAG AA via @axe-core | `004-research/external-research.md` |

## Dependencies

- **v004 (complete):** Backend infrastructure, 571 tests at 92.86% coverage
- **BL-028 -> BL-030, BL-031, BL-033, BL-035:** All GUI panels need the frontend project
- **BL-032, BL-034 -> BL-033:** Library browser needs thumbnails and pagination fix
- **BL-036 -> All GUI:** E2E tests need all components to exist

## Risks

See `comms/outbox/versions/design/v005/006-critical-thinking/risk-assessment.md` for full analysis.

| Risk | Severity | Status |
|------|----------|--------|
| R-1: Framework selection | High | Eliminated (React confirmed) |
| R-2: SPA routing edge cases | Medium | Accepted (html=True sufficient for flat routes) |
| R-3: WebSocket stability | Medium | Accepted (local app, 30s heartbeat + client reconnect) |
| R-4: Thumbnail generation performance | Medium | Accepted (on-scan, async job queue) |
| R-5: CI build time | Low | Accepted (caching, parallel jobs, ubuntu-only E2E) |
| R-6: Virtual scrolling compatibility | Low | TBD (pagination fallback available) |

## Constraints

- Frontend is greenfield with no existing code
- Framework selection must resolve first as all GUI work depends on it
- CI build time managed via caching and parallel jobs
- Playwright E2E on ubuntu-latest only (not full OS matrix)

## References

- Design artifact store: `comms/outbox/versions/design/v005/`
- Architecture docs: `docs/design/08-gui-architecture.md`
- v004 retrospective: `comms/outbox/versions/execution/v004/retrospective.md`
