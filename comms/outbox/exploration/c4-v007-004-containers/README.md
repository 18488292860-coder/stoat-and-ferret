# C4 Container Synthesis -- Task 004

Identified 5 containers (3 runtime, 1 embedded database, 1 filesystem storage) reflecting a monolithic single-process architecture with an embedded Rust native extension. The system deploys as a single FastAPI process serving both the REST/WebSocket API and the pre-built React SPA, with SQLite for persistence and local filesystem for media storage. One OpenAPI specification was updated to v0.7.0 covering all 24 REST endpoints including the full effects CRUD and transition API added in v007.

## Containers Identified: 5

1. **API Server** (runtime) -- FastAPI/uvicorn process hosting REST API, WebSocket, job worker, effects engine with 9 effects, and static GUI serving
2. **Web GUI** (build artifact) -- React SPA built with Vite, served as static files by the API Server at `/gui`; runs as separate Vite dev server during development
3. **Rust Core Library** (in-process library) -- PyO3 native extension compiled with maturin, loaded at import time by the API Server for timeline math, clip validation, FFmpeg command building, all 9 effect builders, transition builders, and input sanitization
4. **SQLite Database** (embedded) -- File-based database at `data/stoat.db` accessed in-process via aiosqlite; schema managed by Alembic with 5 migration versions
5. **File Storage** (filesystem) -- Local directories for source videos, thumbnails, and database file

## Infrastructure Files Found

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage build: Stage 1 compiles Rust extension with maturin, Stage 2 creates slim Python 3.12 runtime with uv for testing |
| `docker-compose.yml` | Single `test` service with read-only source mounts for fast iteration |
| `.github/workflows/ci.yml` | CI with 5 jobs: path-filtered changes detection, 9-matrix test (3 OS x 3 Python), Rust coverage, frontend build+test, Playwright E2E with accessibility audits |
| `pyproject.toml` | Python project config with maturin build settings, entry point `python -m stoat_ferret.api` |
| `gui/package.json` | Frontend dependencies and build scripts (`npm run build` to `gui/dist/`) |
| `gui/vite.config.ts` | Vite config with API proxy for development (`/api`, `/health`, `/metrics`, `/ws` to port 8000) |
| `alembic.ini` + `alembic/versions/` | 5 database migration scripts for schema management |

## API Specifications Generated

| File | Description |
|------|-------------|
| `docs/C4-Documentation/apis/api-server-api.yaml` | OpenAPI 3.1 spec v0.7.0 -- 24 REST endpoints across 6 tag groups (Health, Videos, Projects, Clips, Effects, Transitions, Jobs) with full request/response schemas |

## Inferred vs Explicit

| Container | Source | Evidence |
|-----------|--------|----------|
| API Server | Explicit | `Dockerfile`, `docker-compose.yml`, `pyproject.toml` entry point, `src/stoat_ferret/api/__main__.py` |
| Web GUI | Explicit | `gui/package.json` build scripts, `gui/vite.config.ts` dev proxy, Vite build output to `gui/dist/` |
| Rust Core Library | Explicit | `pyproject.toml` maturin config, `Dockerfile` Stage 1 maturin build, `rust/stoat_ferret_core/Cargo.toml` |
| SQLite Database | Explicit | `alembic.ini`, 5 migration scripts in `alembic/versions/`, CI migration verification step |
| File Storage | Inferred | No explicit storage config; inferred from `STOAT_DATABASE_PATH`, `STOAT_THUMBNAIL_DIR` settings, and scan path validation logic |

## External Systems

| System | Protocol | Required |
|--------|----------|----------|
| FFmpeg | subprocess | Yes -- video thumbnail generation, video processing |
| ffprobe | subprocess | Yes -- video metadata extraction (dimensions, duration, codecs, frame rate) |
| Prometheus | HTTP scrape | No -- optional metrics collection from `/metrics` endpoint |

## Key Architectural Observations

- **Single-process monolith**: The API Server is the only runtime process. The Rust core library, SQLite database, and job queue all run in-process. No separate services, message brokers, or external databases.
- **Docker is test-only**: The Dockerfile and docker-compose.yml are configured exclusively for running the test suite in a containerized environment, not for production deployment.
- **No production deployment config**: There are no Kubernetes manifests, Terraform files, cloud deployment configs, or production Dockerfiles. The system runs as a local development application.
- **Embedded scaling constraints**: SQLite single-writer and in-process asyncio job queue prevent horizontal scaling. This is appropriate for the current single-user desktop/local application use case.
