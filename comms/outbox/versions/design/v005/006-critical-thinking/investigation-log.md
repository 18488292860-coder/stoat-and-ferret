# Investigation Log — v005 Critical Thinking

## Investigation 1: Framework Selection Evidence (R-1)

**Goal:** Determine whether `08-gui-architecture.md` is prescriptive about React or merely illustrative.

**Query:** Read `docs/design/08-gui-architecture.md` (822 lines)

**Findings:**
- Technology table (line 21): "React 18+ or Svelte 4+" — ambiguous
- File structure (lines 585-680): All `.tsx` extensions, React conventions exclusively
- Hooks directory: `useWebSocket.ts`, `useApi.ts`, `useTheaterMode.ts` — React hooks pattern
- Stores directory: `appStore.ts`, `projectStore.ts`, `theaterStore.ts` — Zustand pattern
- WebSocket events (lines 433-465): TypeScript interfaces (framework-agnostic but JS/TS ecosystem)
- Phase checklist (line 486): "Set up React/Svelte project with Vite" — ambiguous
- No Svelte-specific patterns (no `.svelte` files, no svelte stores, no `$:` reactive syntax)

**Conclusion:** React is prescriptive. The "or Svelte" phrasing in the table is aspirational; all concrete specifications are React-only. **Risk R-1 eliminated.**

## Investigation 2: App Factory Structure (R-2, Theme 01)

**Goal:** Understand how `StaticFiles` mount will integrate with existing `create_app()`.

**Query:** Read `src/stoat_ferret/api/app.py` (131 lines)

**Findings:**
- `create_app()` already mounts routes and metrics endpoint
- Existing mount at line 128: `app.mount("/metrics", metrics_app)` — precedent for mounting ASGI sub-apps
- Router includes: health, videos, projects, jobs (lines 117-120)
- Middleware: CorrelationIdMiddleware, MetricsMiddleware (lines 123-124)
- DI pattern via kwargs on `app.state` (lines 76-115)
- **Key insight:** `StaticFiles` mount must come AFTER API routers to avoid catching `/api/*` requests. FastAPI matches routes in registration order, so mounting `/gui` after `/api` routers is correct.

**Conclusion:** Integration is straightforward. Add `StaticFiles` mount after existing routers. Add WebSocket endpoint as a new router. No structural issues.

## Investigation 3: Repository Protocol for Count (R-4, BL-034)

**Goal:** Confirm pagination fix scope and identify all files needing changes.

**Query:** Read `src/stoat_ferret/db/repository.py` (393 lines) and `src/stoat_ferret/db/async_repository.py` (380 lines)

**Findings:**
- Sync protocol: `VideoRepository` (line 16) — methods: add, get, get_by_path, list_videos, search, update, delete
- Sync implementations: `SQLiteVideoRepository` (line 109), `InMemoryVideoRepository` (line 312)
- Async protocol: `AsyncVideoRepository` (line 19) — same method set, async
- Async implementations: `AsyncSQLiteVideoRepository` (line 112), `AsyncInMemoryVideoRepository` (line 286)
- **No `count()` method exists** on any protocol or implementation
- `list_videos()` returns `list[Video]` with LIMIT/OFFSET but no total
- `search()` returns `list[Video]` with LIMIT but no total

**Files needing changes for BL-034:**
1. `src/stoat_ferret/db/async_repository.py` — add `count()` to protocol and both implementations
2. `src/stoat_ferret/db/repository.py` — add `count()` to sync protocol and implementations (for consistency)
3. `src/stoat_ferret/api/routers/videos.py` — call `count()` for `total` field instead of `len(videos)`
4. `src/stoat_ferret/api/schemas/video.py` — no change needed (already has `total: int`)

**Conclusion:** Well-scoped change. 3-4 files. No schema migration needed. SQLite: `SELECT COUNT(*) FROM videos`. InMemory: `return len(self._videos)`.

## Investigation 4: Video Response Schema (BL-034)

**Goal:** Confirm response schema already supports total count.

**Query:** Read `src/stoat_ferret/api/schemas/video.py` (first 70 lines)

**Findings:**
- `VideoListResponse` (line 34): has `videos`, `total`, `limit`, `offset` fields
- `VideoSearchResponse` (line 46): has `videos`, `total`, `query` fields
- Both already have `total: int` — the field exists but is populated incorrectly

**Query:** Grep `total|limit|offset|pagination` in `videos.py` router

**Findings:**
- Line 70: `total=len(videos)` — BUG: returns page size, not true total
- Line 96: `total=len(videos)` — BUG: same issue for search results

**Conclusion:** Schema is correct. Only the router logic needs fixing to call a new `count()` method.

## Investigation 5: FFmpeg Executor Pattern (R-4, BL-032)

**Goal:** Confirm thumbnail pipeline can reuse existing FFmpeg executor pattern.

**Query:** Read `src/stoat_ferret/ffmpeg/executor.py` (261 lines)

**Findings:**
- `FFmpegExecutor` protocol (line 38): `run(args, *, stdin, timeout) -> ExecutionResult`
- `RealFFmpegExecutor` (line 61): subprocess-based, production implementation
- `RecordingFFmpegExecutor` (line 114): wraps another executor, records interactions
- `FakeFFmpegExecutor` (line 171): replays recordings for testing
- `ExecutionResult` (line 19): has returncode, stdout, stderr, command, duration_seconds

**Conclusion:** Thumbnail generation fits perfectly. Call `executor.run(["-ss", "5", "-i", video_path, "-frames:v", "1", "-s", "320x180", output_path])`. Use `RecordingFFmpegExecutor` to capture, `FakeFFmpegExecutor` to replay in tests. Matches BL-032 AC#5.

## Investigation 6: CI Pipeline Structure (R-5)

**Goal:** Understand CI impact of adding frontend tooling.

**Query:** Read `.github/workflows/ci.yml` (157 lines)

**Findings:**
- Path filter (lines 22-34): triggers on `src/`, `rust/`, `tests/`, etc. — does NOT include `gui/`
- Test matrix: 3 OS x 3 Python = 9 jobs (lines 39-44)
- `ci-status` job (line 141): required check for branch protection, aggregates test + rust-coverage
- No Node.js setup, no npm/Vite steps currently

**CI changes needed:**
1. Add `gui/**` to path filter
2. Add new `frontend` job: Node.js setup, npm ci, Vite build, Vitest (ubuntu-latest only)
3. Add new `e2e` job: Node.js + Python + Playwright (ubuntu-latest only)
4. Update `ci-status` to include frontend + e2e job results
5. Add caching for `node_modules/` and Playwright browsers

**Conclusion:** Manageable. Frontend/E2E jobs run parallel to existing test matrix, on ubuntu-latest only, so they don't multiply the 9-entry matrix.

## Investigation 7: Settings Structure (Theme 01)

**Goal:** Confirm new settings fields fit existing pattern.

**Query:** Read `src/stoat_ferret/api/settings.py` (82 lines)

**Findings:**
- Uses `pydantic_settings.BaseSettings` with `env_prefix="STOAT_"` (line 23)
- Existing fields: database_path, api_host, api_port, debug, log_level, allowed_scan_roots
- Pattern: `Field(default=..., description=...)` with type annotations

**New fields needed (Theme 01, Feature 003):**
- `thumbnail_dir: str = Field(default="data/thumbnails", ...)` — env: `STOAT_THUMBNAIL_DIR`
- `gui_static_path: str = Field(default="gui/dist", ...)` — env: `STOAT_GUI_STATIC_PATH`
- `ws_heartbeat_interval: int = Field(default=30, ...)` — env: `STOAT_WS_HEARTBEAT_INTERVAL`

**Conclusion:** Clean addition. Follows existing pattern exactly.
