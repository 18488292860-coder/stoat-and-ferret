# External Research — v004

## 1. FastAPI Testing with Dependency Injection

**Pattern**: FastAPI's `app.dependency_overrides` is the standard mechanism for test DI. However, `create_app()` accepting optional dependencies is a cleaner pattern that avoids global state mutation.

**Best practice**: Use `create_app(repository=InMemoryRepo())` over `dependency_overrides` when possible. This makes test setup explicit and avoids forgetting to clear overrides between tests.

**BL-021 recommendation**: Add optional params to `create_app()`. When params are None, use production implementations (via lifespan). When provided, skip lifespan DB setup and use injected instances directly.

**Source**: FastAPI documentation on testing, DeepWiki fastapi/fastapi

## 2. Black Box Testing for REST APIs

**Test organization**: Separate test directories or pytest markers for unit vs integration vs black box. Register markers in `pyproject.toml` to avoid warnings.

**Recommended structure**:
```
tests/
├── conftest.py          # Shared fixtures
├── test_api/            # Existing API tests
│   └── conftest.py      # API-specific fixtures
├── test_blackbox/       # Black box tests (BL-023)
│   └── conftest.py      # Black box fixtures with full DI
└── test_contract/       # Contract tests (BL-024)
```

**Marker pattern**: `@pytest.mark.blackbox` for CI selection. Run with `pytest -m blackbox` or exclude with `pytest -m "not blackbox"`.

**Key principle**: Black box tests should use `TestClient` with `create_app()` injecting test doubles, exercising the full HTTP path without mocking internal components.

**Sources**: pytest-with-eric.com, pythontutorials.net best practices

## 3. Property-Based Testing with Hypothesis

**Invariant-first approach**: Use `hypothesis.stateful.RuleBasedStateMachine` for stateful testing. Define `@invariant` methods checked after every operation.

**Application to v004**:
- Timeline math: `Position + Duration - Duration == Position` (round-trip)
- Filter escaping: `unescape(escape(text)) == text` (if unescape existed)
- Clip validation: Random valid clips always pass validation
- Repository operations: `add(item); get(item.id) == item` (round-trip)

**BL-009 template guidance should include**:
1. Identify properties/invariants before writing implementations
2. Use `@given` for pure functions, `RuleBasedStateMachine` for stateful systems
3. Document expected test count per feature

**Dependency**: `hypothesis` not currently in dev dependencies — needs adding to pyproject.toml

**Source**: DeepWiki HypothesisWorks/hypothesis

## 4. FFmpeg Contract Testing (Verified Fakes)

**Verified fakes pattern**: Run the same parametrized test suite against both the real implementation and the fake. If the fake passes all tests that the real implementation passes, the fake is verified.

**Application to BL-024**:
```python
@pytest.fixture(params=["real", "recording", "fake"])
def executor(request, sample_video_path):
    if request.param == "real":
        return RealFFmpegExecutor()
    elif request.param == "recording":
        return RecordingFFmpegExecutor(RealFFmpegExecutor(), recording_path)
    else:
        return FakeFFmpegExecutor.from_file(recording_path)
```

**Representative commands for contract testing** (min 5 per BL-024 AC):
1. Version check: `ffmpeg -version`
2. Probe metadata: `ffprobe -v quiet -print_format json -show_streams input.mp4`
3. Trim clip: `ffmpeg -i input.mp4 -ss 0 -t 1 -c copy output.mp4`
4. Scale video: `ffmpeg -i input.mp4 -vf scale=640:480 output.mp4`
5. Apply filter: `ffmpeg -i input.mp4 -af volume=0.5 output.mp4`

**Current gap**: `FakeFFmpegExecutor` replays by index position, not by matching args. Contract tests should verify args match between recording and replay.

**Sources**: pythonspeed.com (verified fakes), Google SWE book ch13

## 5. Async Job Queue Patterns

**Recommended: `asyncio.Queue` producer-consumer**

For an in-process queue without external dependencies:
```python
class AsyncJobQueue:
    def __init__(self):
        self._queue = asyncio.Queue()
        self._results: dict[str, JobResult] = {}

    async def submit(self, job_id: str, coro) -> str:
        await self._queue.put((job_id, coro))
        return job_id

    async def get_status(self, job_id: str) -> JobStatus:
        return self._results.get(job_id, JobStatus.PENDING)
```

**InMemoryJobQueue for tests**: Execute jobs synchronously (no queue, immediate completion) for deterministic test behavior. Matches the pattern from `07-quality-architecture.md:599`.

**Lifespan integration**: Start worker task(s) in `lifespan()` context manager, cancel on shutdown:
```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    worker = asyncio.create_task(process_jobs(app.state.job_queue))
    yield
    worker.cancel()
```

**API pattern**: POST /videos/scan returns `{"job_id": "..."}`, GET /jobs/{id} returns status/progress.

**Sources**: Python asyncio.Queue docs, superfastpython.com
