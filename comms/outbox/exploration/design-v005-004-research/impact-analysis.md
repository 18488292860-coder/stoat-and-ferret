# Impact Analysis — v005 Research

This supplements the task-003 impact assessment with research-informed specifics.

## Dependencies: Code Changes Required

### App Factory (`src/stoat_ferret/api/app.py`)

| Change | Backlog | Scope |
|--------|---------|-------|
| Add `ws_manager` parameter to `create_app()` | BL-029 | Small — follows existing pattern |
| Add `gui_static_path` parameter to `create_app()` | BL-003, BL-028 | Small |
| Mount `StaticFiles` at `/gui` (after routers) | BL-003, BL-028 | Small |
| Include WebSocket router | BL-029 | Small |
| Add WebSocket manager to lifespan startup/shutdown | BL-029 | Small |

### Settings (`src/stoat_ferret/api/settings.py`)

| Field | Default | Backlog |
|-------|---------|---------|
| `thumbnail_dir: str` | `"data/thumbnails"` | BL-032 |
| `gui_static_path: str` | `"gui/dist"` | BL-003 |
| `ws_heartbeat_interval: int` | `30` | BL-029 |

### Repository Protocol (`src/stoat_ferret/db/async_repository.py`)

| Change | Backlog | Scope |
|--------|---------|-------|
| Add `count()` method to `AsyncVideoRepository` protocol | BL-034 | Small |
| Implement `count()` in `AsyncSQLiteVideoRepository` | BL-034 | Small — `SELECT COUNT(*) FROM videos` |
| Implement `count()` in `AsyncInMemoryVideoRepository` | BL-034 | Small — `return len(self._videos)` |

### New Files Required

| File | Backlog | Description |
|------|---------|-------------|
| `src/stoat_ferret/api/routers/websocket.py` | BL-029 | WebSocket endpoint and ConnectionManager |
| `src/stoat_ferret/api/services/thumbnail.py` | BL-032 | Thumbnail generation service |
| `gui/` directory tree | BL-028 | Entire frontend project scaffold |
| `gui/e2e/` | BL-036 | Playwright E2E tests |

## Breaking Changes

| Area | Breaking? | Details |
|------|-----------|---------|
| API response schemas | **No** | Adding `total` field to list responses is additive |
| New endpoints (`/ws`, `/gui/*`, thumbnail) | **No** | All new routes; no existing routes change |
| Repository protocol | **Minor** | Adding `count()` breaks custom implementations of `AsyncVideoRepository` outside codebase — but none exist |
| Settings class | **No** | All new fields have defaults |
| `create_app()` signature | **No** | New kwargs are optional with `None` defaults |
| CI workflow | **No** | Adding jobs doesn't affect existing jobs |

**Verdict:** No breaking changes to existing APIs or workflows. All changes are additive.

## Test Infrastructure Needs

### New Test Categories

| Category | Tool | Location | Backlog |
|----------|------|----------|---------|
| Frontend unit tests | Vitest | `gui/src/**/*.test.tsx` | BL-030, BL-031, BL-033, BL-035 |
| Frontend E2E tests | Playwright | `gui/e2e/**/*.spec.ts` | BL-036 |
| Accessibility tests | @axe-core/playwright | `gui/e2e/accessibility.spec.ts` | BL-036 |
| WebSocket integration tests | pytest + httpx | `tests/test_api/test_websocket.py` | BL-029 |
| Thumbnail contract tests | pytest + FakeFFmpegExecutor | `tests/test_api/test_thumbnail.py` | BL-032 |

### Existing Test Updates

| Test File | Change | Backlog |
|-----------|--------|---------|
| `tests/test_api/test_videos.py` | Verify `total` field accuracy in list/search responses | BL-034 |
| `tests/test_api/conftest.py` | Add `ws_manager` fixture to `create_app()` call | BL-029 |

### CI Workflow Updates

| Step | Details | Backlog |
|------|---------|---------|
| Add Node.js setup | `actions/setup-node@v4` with node 20 | BL-028 |
| Add `gui/**` to path filters | Trigger CI on frontend changes | BL-028 |
| Add npm build step | `cd gui && npm ci && npm run build` | BL-028 |
| Add Vitest step | `cd gui && npm test` | BL-028 |
| Add Playwright job | Install browsers, build frontend, start server, run E2E | BL-036 |

## Documentation Updates Required

| Document | Change | When |
|----------|--------|------|
| `docs/design/02-architecture.md` | Remove "(Future)" from Web UI; add `/ws` and `/gui/*` | With first GUI theme |
| `docs/design/04-technical-stack.md` | Replace "React or Svelte" with "React 18+" | After BL-028 |
| `docs/design/05-api-specification.md` | Add `/ws`, `/api/videos/{id}/thumbnail`, verify `total` | With BL-029, BL-032, BL-034 |
| `docs/design/01-roadmap.md` | Update M1.10-M1.12 checklist | Post-v005 |
| `AGENTS.md` | Add frontend quality gate commands | With BL-028 |

## Sequencing Implications

Based on research findings, the recommended implementation order:

1. **Infrastructure first** (LRN-019): Frontend scaffolding + CI pipeline (BL-028, BL-003)
2. **Backend services**: WebSocket endpoint (BL-029), thumbnail pipeline (BL-032), pagination fix (BL-034)
3. **GUI components**: Application shell (BL-030), dashboard (BL-031), library browser (BL-033), project manager (BL-035)
4. **Quality**: E2E test infrastructure (BL-036)

This follows the "Build Infrastructure First" learning (LRN-019) — infrastructure themes before consumer themes.
