# C4 Container Synthesis — v006

Five containers were identified for stoat-and-ferret: one API Server process, one browser-delivered Web GUI SPA, one in-process Rust native library, one embedded SQLite database, and one local File Storage directory. The deployment pattern is a single-host monolith with no network service boundaries between the Python backend, Rust extension, and database — all co-located in one OS process except for the SPA (delivered as static assets from the same server).

## Containers Identified

**Count: 5**

| Container | Type | Explicit or Inferred |
|-----------|------|----------------------|
| API Server | API / Web Application | Explicit (Dockerfile, pyproject.toml entry point) |
| Web GUI | Web Application (SPA) | Explicit (gui/package.json, npm run build to gui/dist/) |
| Rust Core Library | In-process native library | Explicit (Cargo.toml, maturin build, PyO3 extension) |
| SQLite Database | Embedded database | Explicit (aiosqlite dependency, Alembic migrations, data/stoat.db) |
| File Storage | Local filesystem storage | Inferred (scan roots + thumbnails dirs referenced in source, no dedicated config file) |

Note: The Rust Core Library is not a standalone container in the strict sense (it is loaded in-process as a `.so`/`.dll`/`.dylib`), but it is documented as a container because it is a distinct build artifact with its own toolchain, lifecycle, and deployment step (`maturin build`).

## Infrastructure Files Found

- `Dockerfile` -- Multi-stage build: Stage 1 compiles Rust with maturin; Stage 2 runtime image installs Python deps and pre-built wheel. Default CMD runs pytest.
- `docker-compose.yml` -- Single `test` service wrapping the Dockerfile. Source volumes mounted for iteration. No production services defined.
- `pyproject.toml` -- Python package metadata, maturin build config, uvicorn entry point implied via `stoat_ferret.api` module structure.
- `gui/package.json` -- Frontend build scripts (`npm run build` -> `gui/dist/`), dev server (`npm run dev` on port 5173), Vitest test runner.
- `alembic.ini` + `alembic/versions/` -- Four migration scripts covering initial schema, audit log, projects, and clips tables.
- `.github/workflows/ci.yml` -- CI matrix: 3 OS x 3 Python versions for backend; Node 22 frontend job; Playwright E2E job; Rust coverage job.

**No Kubernetes, Terraform, or cloud-provider configs found.** The project is currently single-host development infrastructure only.

## API Specifications Generated

- `docs/C4-Documentation/apis/api-server-api.yaml` -- OpenAPI 3.1 specification for the API Server (updated from v0.3.0 to v0.6.0 to add effects endpoints)

### v006 Additions to OpenAPI Spec

Three new endpoints added reflecting the Effects Engine work in v006:

- `GET /api/v1/effects` -- List all registered effects with parameter schemas and AI hints
- `GET /api/v1/effects/{effect_type}` -- Get single effect definition
- `POST /api/v1/effects/{effect_type}/apply` -- Apply effect to clip (returns job_id + filter_string)

New schemas added: `EffectDefinitionResponse`, `EffectListResponse`, `EffectApplyRequest`, `EffectApplyResponse`. Existing `ClipResponse` updated to include `effects_json` field.

## Inferred vs Explicit

| Container | Basis |
|-----------|-------|
| API Server | Explicit: Dockerfile Stage 2, `stoat_ferret.api` module, uvicorn dependency in pyproject.toml |
| Web GUI | Explicit: `gui/package.json` build scripts, `npm run build` output to `gui/dist/` |
| Rust Core Library | Explicit: `rust/stoat_ferret_core/Cargo.toml`, maturin build pipeline, PyO3 module |
| SQLite Database | Explicit: aiosqlite dependency, Alembic migration history, `STOAT_DATABASE_PATH` env var |
| File Storage | Inferred: no dedicated infrastructure config; directories implied by `STOAT_ALLOWED_SCAN_ROOTS` env var and thumbnail path constants in source |

## External Systems

| System | Integration | Required |
|--------|-------------|----------|
| FFmpeg | subprocess invocation by API Server for thumbnail generation and video processing | Yes |
| ffprobe | subprocess invocation by API Server for video metadata extraction (dimensions, duration, codecs) | Yes |
| Prometheus | HTTP scrape of `/metrics` endpoint on API Server | No (optional) |

## Files Written

- `docs/C4-Documentation/c4-container.md` -- Updated container documentation reflecting v006 Effects Engine addition (Effects Engine component now listed under API Server, Rust Core Library description updated to mention DrawtextBuilder/SpeedControl effect builders)
- `docs/C4-Documentation/apis/api-server-api.yaml` -- Updated OpenAPI 3.1 spec (version bumped to 0.6.0, three effects endpoints + four new schemas added)
