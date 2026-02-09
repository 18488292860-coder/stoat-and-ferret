# Research — v005 Design Phase

Research investigation covering frontend framework selection, WebSocket integration, static file serving, thumbnail generation, and E2E testing infrastructure for the v005 GUI shell + library browser + project manager release. Ten backlog items (BL-003, BL-028 through BL-036) were investigated across codebase patterns, external library documentation, and impact analysis.

## Research Questions

| # | Question | Backlog | Status |
|---|----------|---------|--------|
| RQ-1 | React vs Svelte for frontend framework? | BL-028 | Resolved |
| RQ-2 | Vite project setup and proxy configuration? | BL-028 | Resolved |
| RQ-3 | FastAPI StaticFiles mount for SPA routing? | BL-003, BL-028 | Resolved |
| RQ-4 | WebSocket endpoint patterns in FastAPI? | BL-029 | Resolved |
| RQ-5 | WebSocket connection manager and broadcasting? | BL-029, BL-031 | Resolved |
| RQ-6 | Thumbnail extraction via FFmpeg subprocess? | BL-032 | Resolved |
| RQ-7 | Thumbnail API endpoint and storage pattern? | BL-032 | Resolved |
| RQ-8 | Playwright E2E setup and CI integration? | BL-036 | Resolved |
| RQ-9 | Accessibility testing (WCAG AA) with Playwright? | BL-036 | Resolved |
| RQ-10 | Pagination total count implementation? | BL-034 | Resolved |
| RQ-11 | How does existing DI pattern extend for new services? | BL-029, BL-003 | Resolved |
| RQ-12 | Virtual scrolling approach for large video libraries? | BL-033 | Resolved |

## Findings Summary

**Framework Decision:** React 18+ with TypeScript recommended over Svelte. React has a larger ecosystem, more TypeScript tooling, better component library availability (needed for video grid, virtual scrolling), and the design docs already lean toward React patterns (JSX examples in 08-gui-architecture.md, Zustand for state).

**Vite Setup:** `npm create vite@latest gui -- --template react-ts` scaffolds the project. Dev proxy via `server.proxy` routes `/api/*`, `/ws`, `/health/*`, `/metrics` to FastAPI at `localhost:8000`. Production build uses `base: '/gui/'` and outputs to `gui/dist/`.

**Static Files:** FastAPI `StaticFiles(directory="gui/dist", html=True)` mounted at `/gui` provides SPA routing with index.html fallback. Must be mounted after API routers to avoid path conflicts.

**WebSocket:** FastAPI native `@app.websocket("/ws")` decorator. ConnectionManager class maintains active connections set, provides `connect()`, `disconnect()`, `broadcast()` methods. Inject via `create_app(ws_manager=...)` following existing DI pattern.

**Thumbnails:** FFmpeg single-frame extraction via `ffmpeg -ss 5 -i input.mp4 -frames:v 1 -vf scale=320:-1 -q:v 5 thumb.jpg`. Integrate with existing `RealFFmpegExecutor`/`RecordingFFmpegExecutor` pattern. Store in configurable `data/thumbnails/` directory.

**Playwright E2E:** Initialize with `npm init playwright@latest`. CI via `webServer` config pointing to FastAPI serving built frontend. Accessibility via `@axe-core/playwright` with `withTags(['wcag2a', 'wcag2aa'])`.

**Pagination:** Current `list_videos` returns `total=len(videos)` (page count, not true total). Fix requires adding `count()` method to repository protocol and returning full count.

## Unresolved Questions

| # | Question | Reason | Resolution Path |
|---|----------|--------|-----------------|
| UQ-1 | WebSocket heartbeat optimal interval | Depends on deployment environment network characteristics | TBD - start with 30s, tune at runtime |
| UQ-2 | Virtual scrolling library choice | Multiple options (react-window, react-virtuoso, tanstack-virtual); depends on grid layout needs | Evaluate during BL-033 implementation |
| UQ-3 | Thumbnail generation timing | On-scan vs on-first-access tradeoff depends on library size | Implement on-scan; lazy fallback if scan is slow |

## Recommendations

1. **Select React 18+ with TypeScript** — larger ecosystem, better tooling, design docs already React-aligned
2. **Use Zustand for state management** — lightweight, TypeScript-friendly, recommended in design docs
3. **Scaffold `gui/` with `create-vite react-ts`** — standard tooling, well-documented
4. **Mount static files after API routers** — prevents route conflicts
5. **WebSocket ConnectionManager as injectable service** — follows existing `create_app()` DI pattern (LRN-005)
6. **Extend FFmpeg executor pattern for thumbnails** — reuses Recording/Fake testing infrastructure (LRN-008)
7. **Add `count()` to repository protocol** — minimal change, enables true pagination totals
8. **Playwright `webServer` config for E2E** — automatically starts FastAPI + serves built frontend
