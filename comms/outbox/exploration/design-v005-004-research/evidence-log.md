# Evidence Log — v005 Research

## Concrete Values

| Parameter | Value | Source | Rationale |
|-----------|-------|--------|-----------|
| Thumbnail width | 320px | BL-032 AC#3: "default 320x180" | 320x180 matches 16:9 aspect ratio; `-vf scale=320:-1` auto-calculates height |
| Thumbnail height | 180px (16:9) or auto | Design spec + FFmpeg `-1` auto-scale | Auto-scale preserves source aspect ratio; 180px is default for 16:9 |
| Thumbnail JPEG quality | 5 (`-q:v 5`) | FFmpeg docs; range 2-31 | Balance of file size (~15-30KB) and visual quality |
| Thumbnail seek position | 5 seconds (`-ss 5`) | Common practice for representative frames | Avoids black frames at video start; fast seeking with `-ss` before `-i` |
| Thumbnail storage dir | `data/thumbnails/` | Impact analysis #12; follows `data/stoat.db` pattern | Configurable via `STOAT_THUMBNAIL_DIR` env var |
| WebSocket heartbeat interval | 30 seconds | Industry standard for keepalive | TBD — start with 30s; browser WebSocket API sends ping/pong automatically |
| WebSocket max connections | None (unlimited) | Single-user local app | Not needed for v005 scope; add limit if scaling to multi-user |
| GUI static path | `gui/dist` | Vite default `outDir`; design doc convention | Standard Vite output; configurable via `STOAT_GUI_STATIC_PATH` |
| Vite base URL | `/gui/` | Design doc: `/gui/*` route mount | Trailing slash required for Vite sub-path deployment |
| Vite dev server port | 5173 (default) | Vite default | Standard; proxied to FastAPI via `server.proxy` |
| FastAPI backend port | 8000 | `src/stoat_ferret/api/settings.py:api_port` default | Already configured in Settings class |
| Debounce delay (search) | 300ms | UI best practice | Standard debounce for search-as-you-type |
| Dashboard refresh interval | 30 seconds | Configurable per BL-031 AC#4 | Balance between freshness and API load |
| Pagination default limit | 20 | `src/stoat_ferret/api/routers/videos.py:52` | Already set; consistent across list endpoints |
| Pagination max limit | 100 | `src/stoat_ferret/api/routers/videos.py:52` | Already set; `Query(ge=1, le=100)` |
| E2E test workers (CI) | 1 | Playwright best practice for CI | `workers: process.env.CI ? 1 : undefined` |
| Playwright retries (CI) | 2 | Playwright recommendation | `retries: process.env.CI ? 2 : 0` |

## Existing Values Verified

| Parameter | Current Value | File | Line | Status |
|-----------|--------------|------|------|--------|
| Video `thumbnail_path` field | `None` (exists but unpopulated) | `src/stoat_ferret/db/models.py` | ~line 138 | VERIFIED — ready for BL-032 |
| `list_videos` total | `len(videos)` (page count) | `src/stoat_ferret/api/routers/videos.py` | ~line 71 | VERIFIED — needs fix per BL-034 |
| `search` total | `len(videos)` (result count) | `src/stoat_ferret/api/routers/videos.py` | ~line 93 | VERIFIED — needs fix per BL-034 |
| Job timeout | 300s (5 min) | `src/stoat_ferret/jobs/queue.py:290` | DEFAULT_TIMEOUT | VERIFIED |
| Allowed scan roots | `list[str]` (empty = unrestricted) | `src/stoat_ferret/api/settings.py` | ~line 65 | VERIFIED (LRN-017) |
| Correlation ID middleware | Active | `src/stoat_ferret/api/app.py:124` | CorrelationIdMiddleware | VERIFIED — available for WebSocket |

## Learnings Verification

| Learning | Status | Evidence |
|----------|--------|----------|
| LRN-005: Constructor DI over dependency_overrides | VERIFIED | `create_app()` in `app.py` uses kwargs → `app.state`; all tests use this pattern |
| LRN-008: Record-Replay for external dependencies | VERIFIED | `executor.py` has Real/Recording/Fake trio; used for FFmpeg testing |
| LRN-009: Handler registration for job queue | VERIFIED | `queue.register_handler()` in `AsyncioJobQueue` and `InMemoryJobQueue` |
| LRN-010: stdlib asyncio.Queue over external deps | VERIFIED | `AsyncioJobQueue` uses `asyncio.Queue` with background worker |
| LRN-017: Empty allowlist = unrestricted | VERIFIED | `allowed_scan_roots` defaults to empty list in Settings |

No learnings are STALE. All patterns referenced by backlog items remain active in the codebase.
