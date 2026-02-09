# Codebase Patterns — v005 Research

## App Factory and DI Pattern

**File:** `src/stoat_ferret/api/app.py` (lines 76-131)

The `create_app()` factory accepts optional dependencies. When provided, they're stored on `app.state` and the `_deps_injected` flag skips lifespan DB setup (for tests).

```python
def create_app(
    *,
    video_repository: AsyncVideoRepository | None = None,
    project_repository: AsyncProjectRepository | None = None,
    clip_repository: AsyncClipRepository | None = None,
    job_queue: AsyncioJobQueue | None = None,
) -> FastAPI:
```

**Extension point for v005:** Add `ws_manager` and `gui_static_path` parameters:
```python
ws_manager: ConnectionManager | None = None,
gui_static_path: str | None = None,
```

## Router Registration

**File:** `src/stoat_ferret/api/app.py` (lines 117-120)

```python
app.include_router(health.router)
app.include_router(videos.router)
app.include_router(projects.router)
app.include_router(jobs.router)
```

Router prefixes: `/health`, `/api/v1/videos`, `/api/v1/projects`, `/api/v1/jobs`.

**For v005:** Add WebSocket router after existing routers, mount StaticFiles last.

## Dependency Resolution via `Depends()`

**File:** `src/stoat_ferret/api/routers/videos.py` (lines 28-47)

```python
async def get_repository(request: Request) -> AsyncVideoRepository:
    repo = getattr(request.app.state, "video_repository", None)
    if repo is not None:
        return repo
    return AsyncSQLiteVideoRepository(request.app.state.db)

RepoDep = Annotated[AsyncVideoRepository, Depends(get_repository)]
```

Pattern: Check `app.state` first (injected), fall back to production construction.

## Middleware Stack

**File:** `src/stoat_ferret/api/app.py` (lines 122-124)

```python
app.add_middleware(MetricsMiddleware)    # Prometheus counters/histograms
app.add_middleware(CorrelationIdMiddleware)  # Request tracing
```

**File:** `src/stoat_ferret/api/middleware/correlation.py`
- Generates UUID per request, stores in context var
- Available for WebSocket correlation ID inclusion (BL-029 AC#5)

## Lifespan Management

**File:** `src/stoat_ferret/api/app.py` (lines 31-74)

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    if getattr(app.state, "_deps_injected", False):
        yield
        return
    # Opens DB, creates queue, registers scan handler, starts worker
    # Cleanup: cancels worker, closes DB
```

**For v005:** Add WebSocket manager init and cleanup in lifespan.

## Repository Protocol Pattern

**File:** `src/stoat_ferret/db/async_repository.py`

Protocol-based with three implementations per entity:
1. **Protocol** (`AsyncVideoRepository`) — interface definition
2. **SQLite** (`AsyncSQLiteVideoRepository`) — production
3. **InMemory** (`AsyncInMemoryVideoRepository`) — testing

**For v005 pagination fix (BL-034):** Add `count()` method to protocol:
```python
async def count(self) -> int: ...
```
Currently `list_videos` returns `total=len(videos)` which is page count, not true total.

## FFmpeg Executor Pattern

**File:** `src/stoat_ferret/ffmpeg/executor.py`

Three-tier pattern (LRN-008):
- `RealFFmpegExecutor` — calls FFmpeg subprocess
- `RecordingFFmpegExecutor` — wraps Real, captures interactions to JSON
- `FakeFFmpegExecutor` — replays recordings, optional strict arg matching

**For v005 thumbnails (BL-032):** Reuse this exact pattern. Thumbnail generation via `RealFFmpegExecutor.run(["-ss", "5", "-i", path, ...])`.

## Job Queue Pattern

**File:** `src/stoat_ferret/jobs/queue.py`

- `AsyncJobQueue` protocol with `submit()`, `get_status()`, `get_result()`
- `AsyncioJobQueue` — background worker consuming `asyncio.Queue`
- `InMemoryJobQueue` — synchronous execution for tests
- Handler registration: `queue.register_handler("scan", make_scan_handler(repo))`

**For v005:** WebSocket events can be emitted from job handlers to broadcast scan progress.

## Settings

**File:** `src/stoat_ferret/api/settings.py` (lines 13-71)

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="STOAT_", env_file=".env")
    database_path: str = Field(default="data/stoat.db")
    api_host: str = Field(default="127.0.0.1")
    api_port: int = Field(default=8000)
    debug: bool = Field(default=False)
    allowed_scan_roots: list[str] = Field(default_factory=list)
```

**For v005:** Add fields:
- `thumbnail_dir: str = Field(default="data/thumbnails")`
- `gui_static_path: str = Field(default="gui/dist")`
- `ws_heartbeat_interval: int = Field(default=30)`

## Video Model

**File:** `src/stoat_ferret/db/models.py`

The `Video` dataclass already has `thumbnail_path: str | None` field (line ~138), set to `None` during scan. BL-032 populates this field.

## Scan Service

**File:** `src/stoat_ferret/api/services/scan.py`

`scan_directory()` iterates files, probes with ffprobe, stores via repository. Sets `thumbnail_path=None` on line 138. Thumbnail generation can be added here or as a post-scan step.

## Health Endpoints

**File:** `src/stoat_ferret/api/routers/health.py`

- `/health/live` — always returns `{"status": "ok"}`
- `/health/ready` — checks database connectivity and FFmpeg availability

**For v005:** Dashboard health indicator (BL-030, BL-031) calls `/health/ready`.

## Metrics

**File:** `src/stoat_ferret/api/middleware/metrics.py`

Prometheus `Counter` (`http_requests_total`) and `Histogram` (`http_request_duration_seconds`) with method/path/status labels. Dashboard metrics cards (BL-031) consume `/metrics` endpoint.
