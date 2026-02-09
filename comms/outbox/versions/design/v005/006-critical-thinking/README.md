# Critical Thinking Review — v005 Design

This review investigated 6 risks and 4 unknowns from the v005 logical design. One risk (R-1: framework selection) was fully resolved through codebase evidence confirming React alignment. Five risks were accepted with documented mitigations. Two unknowns require runtime evaluation during implementation. No design-breaking issues were found; the logical design structure remains unchanged.

## Risks Investigated

- **Total items:** 10 (6 risks + 4 unknowns)
- **Investigate now (resolved):** 1 — R-1 (framework selection confirmed via design doc evidence)
- **Accept with mitigation:** 7 — R-2, R-3, R-4, R-5, UQ-1, UQ-3, UQ-4
- **TBD (requires runtime testing):** 2 — R-6, UQ-2 (virtual scrolling library)

## Resolutions

**R-1 (Framework selection — RESOLVED):** The `08-gui-architecture.md` design doc uses React-specific patterns throughout: JSX component names (`DashboardPanel.tsx`, `Navigation.tsx`), React hooks (`useWebSocket.ts`, `useApi.ts`), Zustand stores (`appStore.ts`), and TypeScript interfaces for WebSocket events. The file structure explicitly uses `.tsx` extensions. While the technology table says "React 18+ or Svelte 4+", all concrete specifications are React-aligned. React is the correct choice — no design change needed.

**R-4 (Thumbnail performance — ACCEPTED):** The FFmpeg executor pattern (`src/stoat_ferret/ffmpeg/executor.py`) already has `RealFFmpegExecutor`, `RecordingFFmpegExecutor`, and `FakeFFmpegExecutor`. The thumbnail pipeline can use this exact pattern. On-scan generation is acceptable for v005 (local, single-user app). The `thumbnail_path` field already exists on the `Video` model.

**BL-034 (Pagination fix — CONFIRMED):** Investigation confirmed `total=len(videos)` on lines 70 and 96 of `videos.py` returns page size, not true total. The `VideoListResponse` schema already has a `total` field. Adding `count()` to the repository protocol is the minimal fix. Both sync and async repositories need the method. The `AsyncVideoRepository` protocol and both implementations (`AsyncSQLiteVideoRepository`, `AsyncInMemoryVideoRepository`) need updating.

## Design Changes

**None.** The logical design from Task 005 is validated. No structural changes to themes, features, or execution order required. All mitigations fit within existing feature scopes.

## Remaining TBDs

1. **R-6 / UQ-2 (Virtual scrolling library):** Cannot be resolved pre-implementation. Three candidate libraries identified (react-window, react-virtuoso, tanstack-virtual). Decision deferred to BL-033 implementation. Mitigation: CSS Grid + Intersection Observer as fallback if none meet grid requirements.

2. **R-3 (WebSocket stability under real load):** 30s heartbeat with client-side reconnection is sufficient for design. Real stability testing requires runtime. Mitigation: configurable heartbeat interval via `STOAT_WS_HEARTBEAT_INTERVAL`.

## Confidence Assessment

**High confidence** in the refined design. Key factors:
- Framework choice validated with codebase evidence (not just ecosystem arguments)
- Pagination fix is well-scoped with clear implementation path in existing code
- Thumbnail pipeline leverages established FFmpeg executor pattern
- CI integration has clear caching strategy
- No backlog items need descoping or deferral
- No circular dependencies found
- All 10 backlog items remain fully covered
