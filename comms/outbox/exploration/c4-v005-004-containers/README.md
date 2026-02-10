# C4 Container Synthesis — stoat-and-ferret v005

Identified 5 containers (2 runtime, 1 in-process library, 1 embedded database, 1 filesystem storage) forming a monolithic single-process architecture with a separate SPA frontend. The system deploys as a single uvicorn process serving both the REST/WebSocket API and pre-built React static assets, with Rust core loaded as an in-process PyO3 extension.

## Containers Identified: 5

1. **API Server** (runtime) — FastAPI/uvicorn process hosting REST API, WebSocket, background job worker, and static GUI serving
2. **Web GUI** (runtime/static) — React SPA built with Vite, served as static files at `/gui` by the API Server; separate Vite dev server in development
3. **Rust Core Library** (in-process) — PyO3 native extension for timeline math, clip validation, FFmpeg command building, sanitization
4. **SQLite Database** (embedded) — File-based database at `data/stoat.db` accessed via aiosqlite
5. **File Storage** (filesystem) — Local directories for source videos, thumbnails, and database file

## Infrastructure Files Found

- **Dockerfile** — Multi-stage build (Rust builder + Python runtime) for containerized testing
- **docker-compose.yml** — Single `test` service for running pytest in Docker
- **.github/workflows/ci.yml** — CI pipeline with 5 jobs: path filtering, 9-matrix Python test, Rust coverage, frontend build+test, E2E Playwright tests
- **alembic.ini + alembic/versions/** — Database migration framework (4 migrations)
- **pyproject.toml** — Python project config with maturin build settings
- **gui/package.json** — Frontend build scripts
- **gui/vite.config.ts** — Dev server with API proxy configuration

## API Specifications Generated

- `docs/C4-Documentation/apis/api-server-api.yaml` — OpenAPI 3.1 spec with 17 REST endpoints across 4 resource groups (Health, Videos, Projects/Clips, Jobs), complete schemas for all request/response types

## Inferred vs Explicit

| Container | Source |
|-----------|--------|
| API Server | **Explicit** — Dockerfile, docker-compose.yml, `__main__.py` entry point, `settings.py` |
| Web GUI | **Explicit** — `gui/package.json`, `gui/vite.config.ts`, build scripts |
| Rust Core Library | **Explicit** — `pyproject.toml` maturin config, `Cargo.toml`, Dockerfile builder stage |
| SQLite Database | **Explicit** — `alembic.ini`, migration scripts, `settings.py` database_path |
| File Storage | **Inferred from code** — `settings.py` thumbnail_dir and allowed_scan_roots, no dedicated infra config |

## External Systems

- **FFmpeg / ffprobe** — Video processing and metadata extraction, invoked via subprocess
- **Prometheus** (optional) — Metrics scraping from `/metrics` endpoint
