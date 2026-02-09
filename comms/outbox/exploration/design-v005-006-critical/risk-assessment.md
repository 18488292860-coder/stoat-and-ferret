# Risk Assessment — v005

## R-1: Framework Selection Locks In Technology Choice

- **Original severity:** High
- **Category:** Investigate now
- **Investigation:** Read `docs/design/08-gui-architecture.md` for framework-specific patterns. Checked for prescriptive vs illustrative React usage.
- **Finding:** The design doc is overwhelmingly React-aligned despite the technology table listing "React 18+ or Svelte 4+". Evidence: all file examples use `.tsx` extensions (e.g., `DashboardPanel.tsx`, `Navigation.tsx`); hooks directory uses React hooks pattern (`useWebSocket.ts`, `useApi.ts`); stores use Zustand (`appStore.ts`); WebSocket event types use TypeScript interfaces (lines 433-465). No Svelte-specific patterns exist anywhere. The entire file structure section (lines 585-680) specifies React conventions.
- **Resolution:** Risk eliminated. React is the correct and only viable choice. No design change needed.
- **Affected themes:** Theme 01 (scaffolding confirmation)

## R-2: SPA Routing Edge Cases with StaticFiles

- **Original severity:** Medium
- **Category:** Accept with mitigation
- **Investigation:** Reviewed the v005 route structure. The design uses flat tab routes (`/gui/`, `/gui/library`, `/gui/projects`) — not deeply nested routes.
- **Finding:** `StaticFiles(html=True)` handles flat routes well. The risk materializes only with deep nesting (e.g., `/gui/library/video/123/details`) or routes that look like file paths. v005's route structure avoids these patterns.
- **Resolution:** Accept with mitigation. Start with `html=True`; document `SPAStaticFiles` subclass as fallback in feature 001 implementation plan if issues arise during testing.
- **Affected themes:** Theme 01, Feature 001

## R-3: WebSocket Connection Stability

- **Original severity:** Medium
- **Category:** Accept with mitigation
- **Investigation:** Assessed deployment context. This is a locally-served app (confirmed by settings: `api_host: 127.0.0.1`, `api_port: 8000`). No reverse proxies, NAT traversal, or load balancers in the path.
- **Finding:** Local deployment eliminates most WebSocket stability risks (proxy timeouts, NAT). The primary remaining risk is browser tab backgrounding, which is a standard problem solved by client-side reconnection logic.
- **Resolution:** 30s heartbeat with client-side auto-reconnect. Configurable via `STOAT_WS_HEARTBEAT_INTERVAL` (Theme 01, Feature 003). Client reconnect logic is part of `useWebSocket.ts` hook (Theme 03, Feature 001).
- **Affected themes:** Theme 01 (server), Theme 03 (client reconnect)

## R-4: Thumbnail Generation Performance

- **Original severity:** Medium
- **Category:** Accept with mitigation
- **Investigation:** Confirmed FFmpeg executor pattern exists at `src/stoat_ferret/ffmpeg/executor.py` with `RealFFmpegExecutor`, `RecordingFFmpegExecutor`, and `FakeFFmpegExecutor`. Confirmed `Video` model already has `thumbnail_path` field (line 136 of `repository.py`: `video.thumbnail_path`). Confirmed scan service at `src/stoat_ferret/api/services/scan.py` uses the async pattern.
- **Finding:** On-scan generation (1-2s per video) is acceptable for v005. The FFmpeg executor pattern enables easy testing via `FakeFFmpegExecutor`. The `thumbnail_path` field already exists, so no schema migration is needed — only a new service and endpoint.
- **Resolution:** Implement on-scan generation. For large libraries (100+ videos), the scan is already an async job (via `AsyncioJobQueue`), so thumbnail generation adds latency to the background job, not to the API response. BL-032 AC#1 explicitly permits either on-scan or on-first-access.
- **Affected themes:** Theme 02, Feature 001

## R-5: CI Build Time Increase

- **Original severity:** Low
- **Category:** Accept with mitigation
- **Investigation:** Read CI workflow (`.github/workflows/ci.yml`). Current CI runs on 3 OS x 3 Python versions = 9 matrix entries. The `changes` job filters by path, skipping CI for docs-only PRs.
- **Finding:** The path filter already includes `'.github/workflows/**'`. Adding `gui/**` to the filter would be needed. Node.js setup + npm install + Vite build adds ~30-60s. Playwright browser install adds ~60-90s. Total increase: 2-3 minutes per matrix entry if not cached. However, Playwright can run on ubuntu-latest only (LRN-015 precedent).
- **Resolution:** (1) Add `gui/**` and `gui/package*.json` to the path filter. (2) Cache `node_modules/` with `actions/cache`. (3) Cache Playwright browsers. (4) Run Playwright on ubuntu-latest only (single entry, not full matrix). (5) Add `gui` job separate from `test` job so frontend and backend CI parallelize.
- **Affected themes:** Theme 01 (CI update), Theme 04 (Playwright CI)

## R-6: Virtual Scrolling Library Compatibility

- **Original severity:** Low
- **Category:** TBD — requires runtime testing
- **Investigation:** Cannot resolve pre-implementation. Requires evaluating library APIs with the specific grid layout needed for the video library (thumbnail cards in a responsive grid).
- **Finding:** Three candidate libraries exist: react-window (well-established, limited grid), react-virtuoso (good grid support), tanstack-virtual (modern, headless). All have TypeScript support. The choice depends on how each handles responsive CSS Grid column counts, which requires prototyping.
- **Resolution:** Decision deferred to Theme 03, Feature 003 implementation. Fallback: CSS Grid with Intersection Observer for visibility detection (no virtualization library needed for <500 items). BL-033 AC#5 says "virtual scrolling or pagination" — pagination is an acceptable alternative.
- **Affected themes:** Theme 03, Feature 003

---

## UQ-1: WebSocket Heartbeat Interval

- **Original severity:** Low
- **Category:** Accept with mitigation
- **Resolution:** 30s default, configurable via `STOAT_WS_HEARTBEAT_INTERVAL`. Settings field added in Theme 01, Feature 003. No investigation needed — standard practice.

## UQ-2: Virtual Scrolling Library Choice

- **Original severity:** Medium
- **Category:** TBD — requires runtime testing
- **Resolution:** Combined with R-6 above. Three candidates identified. Decision at implementation time. Pagination is the acceptable fallback per BL-033 AC#5.

## UQ-3: Thumbnail Generation Timing

- **Original severity:** Low
- **Category:** Accept with mitigation
- **Resolution:** Combined with R-4 above. On-scan generation chosen. Scan is already async (job queue), so latency impact is on background job, not user-facing API. BL-032 AC#1 explicitly permits either approach.

## UQ-4: Frontend Test Coverage Baseline

- **Original severity:** Low
- **Category:** Accept with mitigation
- **Resolution:** No enforced Vitest coverage threshold in v005. Start with component tests per feature (Theme 03). Establish baseline after Theme 03 completes. Ratchet up in v006+. This aligns with LRN-013 (conservative coverage bootstrapping).
