# C4 Code Level: API Tests

## Overview

- **Name**: API Tests
- **Description**: Comprehensive pytest test suite for all REST API endpoints of the stoat-and-ferret video editor
- **Location**: `tests/test_api/`
- **Language**: Python (pytest, async)
- **Purpose**: Validates HTTP endpoint behavior, request/response contracts, middleware, DI wiring, settings, effects CRUD, transitions, and static file serving
- **Parent Component**: TBD

## Code Elements

### Test Inventory (180 tests across 14 files)

| File | Tests | Coverage |
|------|-------|----------|
| conftest.py | 0 (7 fixtures, 2 helper classes) | Shared test infrastructure |
| test_middleware.py | 9 | CorrelationIdMiddleware, MetricsMiddleware |
| test_clips.py | 13 | Clip CRUD (POST/GET/PATCH/DELETE /api/v1/projects/{id}/clips) |
| test_projects.py | 11 | Project CRUD (POST/GET/DELETE /api/v1/projects) |
| test_app.py | 7 | create_app(), FastAPI instance, OpenAPI docs |
| test_di_wiring.py | 5 | Dependency injection via create_app() kwargs |
| test_health.py | 6 | GET /health/live, GET /health/ready |
| test_factory_api.py | 5 | ProjectFactory, ApiFactory integration |
| test_jobs.py | 5 | GET /api/v1/jobs/{id}, async scan workflow |
| test_gui_static.py | 5 | /gui static file mount, index.html serving |
| test_settings.py | 17 | Settings class, env overrides, validation |
| test_thumbnail_endpoint.py | 5 | GET /api/v1/videos/{id}/thumbnail |
| test_videos.py | 28 | Video CRUD, search, scan (POST/GET/DELETE /api/v1/videos) |
| test_effects.py | 64 | Effect discovery, preview, CRUD, transitions |

### Key Fixtures (conftest.py)

- `video_repository() -> AsyncInMemoryVideoRepository` -- Empty in-memory video store
- `project_repository() -> AsyncInMemoryProjectRepository` -- Empty in-memory project store
- `clip_repository() -> AsyncInMemoryClipRepository` -- Empty in-memory clip store
- `job_queue() -> InMemoryJobQueue` -- Job queue with scan handler registered
- `app() -> FastAPI` -- App wired with all in-memory test doubles via create_app() kwargs
- `client() -> Generator[TestClient]` -- Synchronous test client with TestClient context manager
- `api_factory() -> ApiFactory` -- Builder for creating test data via HTTP

### Helper Classes (conftest.py)

- `ApiFactory(client, video_repository)` -- Provides `.project()` method returning builder
- `_ApiProjectBuilder(client, video_repository)` -- Fluent builder for projects and clips

### test_effects.py (~64 tests)

Tests the entire effects subsystem through the API:
- Effect discovery: GET /api/v1/effects lists all 9 built-in effects
- Effect preview: POST /api/v1/effects/preview with valid/invalid params
- Effect CRUD on clips: POST/PATCH/DELETE /api/v1/projects/{id}/clips/{id}/effects/{index}
- Transitions: POST /api/v1/projects/{id}/effects/transition with adjacency validation
- Error handling: unknown effect types, invalid parameters, non-adjacent clips
- Registry validation: all effect definitions have valid schemas and AI hints

## Dependencies

### Internal Dependencies

- `stoat_ferret.api.app` (create_app)
- `stoat_ferret.api.middleware.correlation` (CorrelationIdMiddleware)
- `stoat_ferret.api.settings` (Settings, get_settings)
- `stoat_ferret.api.services.scan` (SCAN_JOB_TYPE, make_scan_handler)
- `stoat_ferret.api.services.thumbnail` (ThumbnailService)
- `stoat_ferret.db.async_repository` (AsyncInMemoryVideoRepository)
- `stoat_ferret.db.clip_repository` (AsyncInMemoryClipRepository)
- `stoat_ferret.db.project_repository` (AsyncInMemoryProjectRepository)
- `stoat_ferret.db.models` (Clip, Project, Video)
- `stoat_ferret.effects.definitions` (all 9 built-in effects, create_default_registry)
- `stoat_ferret.effects.registry` (EffectRegistry)
- `stoat_ferret.jobs.queue` (InMemoryJobQueue, JobOutcome, JobStatus)
- `stoat_ferret.ffmpeg.probe` (VideoMetadata)
- `tests.factories` (make_test_video, ProjectFactory)

### External Dependencies

- `pytest`, `pytest-asyncio`
- `fastapi` (FastAPI, TestClient)
- `pydantic` (ValidationError)
- `unittest.mock` (patch, AsyncMock, MagicMock)

## Relationships

```mermaid
classDiagram
    namespace ApiTestSuite {
        class Conftest {
            +video_repository()
            +project_repository()
            +clip_repository()
            +job_queue()
            +app()
            +client()
            +api_factory()
        }
        class ApiFactory {
            -_client TestClient
            -_video_repository AsyncInMemoryVideoRepository
            +project() _ApiProjectBuilder
        }
    }
    class RestApi {
        GET /health/*
        CRUD /api/v1/projects
        CRUD .../clips
        CRUD .../effects
        POST .../effects/transition
        CRUD /api/v1/videos
        GET /api/v1/jobs/{id}
        GET /api/v1/effects
        POST /api/v1/effects/preview
    }

    Conftest --> ApiFactory : provides
    Conftest --> RestApi : test fixtures inject doubles
    ApiFactory --> RestApi : exercises via HTTP
```

## Notes

- All tests use in-memory test doubles for repositories and job queue for deterministic, fast execution
- The `app` fixture uses create_app() parameter injection (not dependency_overrides) to wire test doubles
- TestClient context manager handles lifespan events automatically
- ApiFactory pattern enables fluent API test data creation with automatic video seeding
- test_effects.py is the largest test file (64 tests) covering the full effects and transitions subsystem
