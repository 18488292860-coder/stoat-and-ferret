# Codebase Patterns — v004 Research

## Repository Protocol Pattern

All repositories follow the same dual sync/async protocol structure.

### Existing Protocols
| Protocol | File | Line |
|----------|------|------|
| `VideoRepository` (sync) | `src/stoat_ferret/db/repository.py` | 15 |
| `AsyncVideoRepository` | `src/stoat_ferret/db/async_repository.py` | 17 |
| `AsyncProjectRepository` | `src/stoat_ferret/db/project_repository.py` | 16 |
| `AsyncClipRepository` | `src/stoat_ferret/db/clip_repository.py` | 13 |
| `FFmpegExecutor` | `src/stoat_ferret/ffmpeg/executor.py` | 38 |

### Existing InMemory Implementations
| Class | File | Line | Notes |
|-------|------|------|-------|
| `InMemoryVideoRepository` (sync) | `src/stoat_ferret/db/repository.py` | 293 | Substring search |
| `AsyncInMemoryVideoRepository` | `src/stoat_ferret/db/async_repository.py` | 264 | Substring search |
| `AsyncInMemoryProjectRepository` | `src/stoat_ferret/db/project_repository.py` | 184 | No deepcopy, no seed helper |
| `AsyncInMemoryClipRepository` | `src/stoat_ferret/db/clip_repository.py` | — | Exists, basic impl |

### Missing (Required for BL-020)
- `InMemoryJobQueue` — no job queue concept exists at all
- Deepcopy isolation on `AsyncInMemoryProjectRepository` — stores direct references
- Seed helpers on all InMemory repos

## Dependency Injection Pattern

### Current: FastAPI Depends with Annotated aliases

```python
# src/stoat_ferret/api/routers/videos.py:27-40
async def get_repository(request: Request) -> AsyncVideoRepository:
    return AsyncSQLiteVideoRepository(request.app.state.db)

RepoDep = Annotated[AsyncVideoRepository, Depends(get_repository)]
```

Same pattern in `projects.py` for project, clip, and video repos (lines 32-73).

### Current Test Override Pattern

```python
# tests/test_api/conftest.py:129-137
app.dependency_overrides[get_repository] = lambda: video_repository
app.dependency_overrides[get_project_repository] = lambda: project_repository
app.dependency_overrides[get_clip_repository] = lambda: clip_repository
app.dependency_overrides[get_video_repository] = lambda: video_repository
```

**4 override points** that BL-021 should replace with `create_app()` parameters.

### create_app() Current Signature

```python
# src/stoat_ferret/api/app.py:43
def create_app() -> FastAPI:
```

Takes zero parameters. Uses hardcoded `lifespan` context manager that opens SQLite connection from settings.

## FFmpeg Executor Pattern

### Class Hierarchy
```
FFmpegExecutor (Protocol)
├── RealFFmpegExecutor      — subprocess.run
├── RecordingFFmpegExecutor  — wraps any executor, records to JSON
├── FakeFFmpegExecutor       — replays from JSON recordings
└── ObservableFFmpegExecutor — wraps any executor, adds logging/metrics
```

**Contract gap**: `FakeFFmpegExecutor.run()` ignores `args` parameter (line 209: "used for command reconstruction") and replays sequentially by index. No verification that args match the recording.

## Search Behavior Difference (BL-016)

### SQLite FTS5 (production)
```python
# src/stoat_ferret/db/async_repository.py:186-195
WHERE videos_fts MATCH ?  # Uses: f'"{query}"*'
```
Prefix match with FTS5 tokenization. Query `"test"` matches "testing", "tests", "test.mp4".

### InMemory (testing)
```python
# src/stoat_ferret/db/async_repository.py:301-308
if query_lower in v.filename.lower() or query_lower in v.path.lower()
```
Substring match. Query `"est"` matches "test.mp4" — FTS5 would NOT match this.

## Scan Service (BL-027 target)

```python
# src/stoat_ferret/api/services/scan.py:16
async def scan_directory(path, recursive, repository) -> ScanResponse:
```

Synchronous blocking within async context — iterates `root.glob(pattern)`, calls `ffprobe_video()` per file, writes to repo. No job queue, no progress reporting.

## Rust Sanitization (BL-025 target)

| Function | File:Line | Attack Vector | Coverage |
|----------|-----------|---------------|----------|
| `escape_filter_text` | `sanitize/mod.rs:181` | FFmpeg filter injection | 8 special chars escaped |
| `validate_path` | `sanitize/mod.rs:230` | Path safety | Empty + null byte only |
| `validate_crf` | `sanitize/mod.rs:267` | Bounds | 0-51 range |
| `validate_speed` | `sanitize/mod.rs:306` | Bounds | 0.25-4.0 range |
| `validate_volume` | `sanitize/mod.rs:345` | Bounds | 0.0-10.0 range |
| `validate_video_codec` | `sanitize/mod.rs:415` | Command injection | 6-item whitelist |
| `validate_audio_codec` | `sanitize/mod.rs:455` | Command injection | 4-item whitelist |
| `validate_preset` | `sanitize/mod.rs:496` | Command injection | 10-item whitelist |

**Gap**: `validate_path` does NOT check path traversal (`../`), symlinks, or directory allowlists. Comment at line 206: "Full directory allowlist validation ... deferred to Python layer."

## Coverage Exclusions (BL-012)

| Location | Exclusion | Reason |
|----------|-----------|--------|
| `src/stoat_ferret_core/__init__.py:83` | `except ImportError: # pragma: no cover` | Rust extension not built |
| `pyproject.toml:76` | `if TYPE_CHECKING:` | Type-only imports |
| Lines 86-121 | 30+ `type: ignore` comments | Stub assignments in fallback |

## Pytest Configuration

```toml
# pyproject.toml:59-67
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing"
asyncio_mode = "auto"
markers = ["slow", "api", "contract"]
```

**Missing markers**: `blackbox`, `requires_ffmpeg` (used in conftest but not registered), `benchmark`.

## CI Workflow Structure

Single `test` job with 3x3 matrix (OS × Python). FFmpeg installed via AnimMouse/setup-ffmpeg. No Rust coverage, no Docker job, no separate marker-based test runs.
