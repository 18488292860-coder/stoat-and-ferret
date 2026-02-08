# Impact Analysis — v004

## Dependencies: What Existing Code Changes

### BL-020 (InMemory Test Doubles)
- **Modify**: `src/stoat_ferret/db/project_repository.py` — add deepcopy isolation to `AsyncInMemoryProjectRepository`, add seed helpers
- **Create**: New job queue module (e.g., `src/stoat_ferret/jobs/queue.py`) with `InMemoryJobQueue`
- **Create**: Contract tests verifying InMemory matches SQLite behavior

### BL-021 (Dependency Injection)
- **Modify**: `src/stoat_ferret/api/app.py` — add optional params to `create_app()`
- **Modify**: `src/stoat_ferret/api/app.py` — update `lifespan()` to conditionally use injected vs production deps
- **Modify**: `tests/test_api/conftest.py` — replace 4 `dependency_overrides` with `create_app()` params
- **Impact**: All existing API tests use the `client` fixture from conftest; changes must be backwards compatible

### BL-022 (Fixture Factory)
- **Create**: New fixture factory module (e.g., `tests/factories.py`)
- **Modify**: Existing test files that construct test data inline
- **No breaking changes**: Additive — old patterns continue working until migrated

### BL-023 (Black Box Tests)
- **Create**: `tests/test_blackbox/` directory with conftest and test modules
- **Modify**: `pyproject.toml` — register `blackbox` marker
- **Modify**: `.github/workflows/ci.yml` — optionally separate marker-based runs
- **No breaking changes**: New test directory, existing tests unaffected

### BL-024 (Contract Tests)
- **Create**: `tests/test_contract/` directory with FFmpeg executor contract tests
- **Modify**: `pyproject.toml` — `requires_ffmpeg` marker already used but not registered; register it
- **Modify**: `.github/workflows/ci.yml` — contract tests need FFmpeg available (already installed)

### BL-025 (Security Audit)
- **Create**: `docs/security-audit.md` — new audit document
- **Potentially modify**: `rust/stoat_ferret_core/src/sanitize/mod.rs` — if gaps found
- **Potentially modify**: Python-side path validation — if audit identifies missing checks

### BL-026 (Performance Benchmark)
- **Create**: `benchmarks/` directory with benchmark scripts
- **Create**: Results documentation
- **Modify**: `pyproject.toml` — potentially add `benchmark` marker
- **No breaking changes**: Fully additive

### BL-027 (Async Job Queue)
- **Create**: Job queue infrastructure (protocol, async impl, in-memory impl)
- **Modify**: `src/stoat_ferret/api/services/scan.py` — refactor to use job queue
- **Modify**: `src/stoat_ferret/api/routers/videos.py` — scan endpoint returns job ID
- **Create**: New job status endpoint (GET /jobs/{id})
- **Modify**: `src/stoat_ferret/api/app.py` — lifespan starts/stops job worker
- **Modify**: All existing scan tests — async pattern

### BL-009 (Property Test Guidance)
- **Modify**: `docs/auto-dev/PROCESS/generic/02-REQUIREMENTS.md` — add property test section
- **Modify**: `docs/auto-dev/PROCESS/generic/03-IMPLEMENTATION-PLAN.md` — add proptest planning
- **Modify**: `pyproject.toml` — add `hypothesis` to dev dependencies

### BL-010 (Rust Coverage)
- **Modify**: `.github/workflows/ci.yml` — add llvm-cov step
- **Create**: Coverage configuration for Rust workspace
- **Modify**: `AGENTS.md` — document llvm-cov commands

### BL-012 (Coverage Gaps)
- **Review**: `src/stoat_ferret_core/__init__.py:83-121` — ImportError fallback block
- **Potentially modify**: Coverage exclusion lines in `pyproject.toml:75-78`

### BL-014 (Docker Testing)
- **Create**: `Dockerfile` and `docker-compose.yml`
- **Modify**: `AGENTS.md` — document Docker testing workflow

### BL-016 (Search Unification)
- **Modify**: `src/stoat_ferret/db/repository.py:329-337` — InMemoryVideoRepository search
- **Modify**: `src/stoat_ferret/db/async_repository.py:301-309` — AsyncInMemoryVideoRepository search
- **Create**: Tests verifying consistent behavior across implementations

## Breaking Changes

### API Breaking Change: Scan Endpoint (BL-027)
- **Before**: `POST /videos/scan` returns `ScanResponse` synchronously
- **After**: `POST /videos/scan` returns `{"job_id": "..."}` immediately
- **Impact**: Any client consuming the scan endpoint must be updated
- **Mitigation**: v004 is pre-v1.0; documented as expected API evolution
- **New endpoint**: `GET /jobs/{id}` for status polling

### No Other API Breaking Changes
- All other backlog items are additive (new tests, new docs, new tools)
- `create_app()` changes are backwards compatible (optional params default to None)

## Test Impact

### New Test Files
| Backlog | Test Location | Estimated Tests |
|---------|---------------|-----------------|
| BL-020 | `tests/test_db/test_inmemory_*.py` | 15-20 |
| BL-021 | `tests/test_api/test_di.py` | 5-10 |
| BL-022 | `tests/test_factories.py` | 10-15 |
| BL-023 | `tests/test_blackbox/` | 15-25 |
| BL-024 | `tests/test_contract/` | 10-15 |
| BL-025 | `tests/test_security/` | 10-15 |
| BL-027 | `tests/test_jobs/` | 15-20 |
| BL-016 | `tests/test_db/test_search_parity.py` | 5-10 |

**Estimated new tests**: 85-130 (on top of current 395)

### Modified Test Files
- `tests/test_api/conftest.py` — DI pattern migration (BL-021)
- All scan-related tests — async job pattern (BL-027)

## Documentation Updates Required

| Document | Change | Caused By |
|----------|--------|-----------|
| `docs/design/05-api-specification.md` | Rewrite scan endpoint, add job status endpoint | BL-027 |
| `docs/design/03-prototype-design.md` | Update scan section for async | BL-027 |
| `docs/design/02-architecture.md` | Update job queue references | BL-027 |
| `docs/design/04-technical-stack.md` | Update job queue implementation details | BL-027 |
| `docs/design/06-risk-assessment.md` | Verify job queue test examples | BL-027 |
| `AGENTS.md` | Add Docker, benchmark, llvm-cov commands | BL-014, BL-026, BL-010 |
| `docs/auto-dev/PROCESS/generic/02-REQUIREMENTS.md` | Property test section | BL-009 |
| `docs/auto-dev/PROCESS/generic/03-IMPLEMENTATION-PLAN.md` | Property test planning | BL-009 |
