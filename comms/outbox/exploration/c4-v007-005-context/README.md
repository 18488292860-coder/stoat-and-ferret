# C4 Context Synthesis: stoat-and-ferret

Identified 3 personas, documented 12 system features, and created 3 user journeys for the stoat-and-ferret AI-driven video editor. The system serves human editors via a web GUI, AI agents via a discoverable REST API, and developers/maintainers via CLI tooling and CI pipelines.

## System Purpose

stoat-and-ferret is an AI-driven, non-destructive video editor providing a REST API and web GUI for managing video libraries, assembling editing projects, applying 9 built-in effects and transitions, and rendering output via FFmpeg.

## Personas Identified

3 personas:

1. **Editor User (Human)** -- Uses the web GUI to manage a video library, build editing projects, apply effects and transitions, and monitor system health via the dashboard.
2. **AI Agent (Programmatic)** -- Discovers the API via OpenAPI schema and AI hints, then drives editing workflows programmatically through REST calls. The API is deliberately designed for machine consumption.
3. **Developer / Maintainer (Human)** -- Builds, tests, and deploys the system across three technology stacks (Python, Rust, TypeScript) using CLI tools and GitHub Actions CI.

## Features Documented

12 features:

1. Video Library Management (scan, metadata extraction, thumbnails, full-text search)
2. Project Management (create projects, add/reorder clips, timeline calculation)
3. Effect Application (9 effects: text overlay, speed control, volume, audio fade, audio mix, audio ducking, video fade, crossfade with 59 transition types, audio crossfade)
4. Transition Application (between adjacent clips with adjacency validation)
5. Effect Discovery (parameter schemas, AI hints, filter previews)
6. Effect Workshop GUI (catalog, parameter forms, filter preview, clip selector, effect stack CRUD)
7. Web Dashboard (health cards, metrics, WebSocket activity log)
8. Video Library Browser GUI (search, sort, pagination, scan modal)
9. API Discovery (OpenAPI, Swagger UI, ReDoc)
10. Health Monitoring (liveness/readiness probes)
11. Observability (Prometheus metrics, structured logging, correlation IDs, audit trail)
12. Async Job Processing (background job queue with status polling)

## User Journeys Created

3 journeys (one per persona):

1. **Editor: Import Videos and Build a Project with Effects** -- 13 steps from opening the GUI through scanning, project creation, effect application, and transition configuration. Each step verified against actual route definitions.
2. **AI Agent: Discover Effects and Drive an Editing Workflow** -- 9 steps covering API discovery, effect catalog querying, project assembly, effect CRUD, and transition application.
3. **Developer: Quality Verification and Deployment** -- 8 steps covering toolchain setup, quality gates across all three stacks, CI pipeline, and merge workflow.

## External Dependencies

6 external dependencies:

1. **FFmpeg / ffprobe** (required) -- All video processing via subprocess
2. **Local Filesystem** (required) -- Source videos, database, thumbnails
3. **SQLite** (required, embedded) -- In-process database via aiosqlite with Alembic migrations
4. **Prometheus** (optional) -- Metrics collection via HTTP scrape
5. **GitHub Actions CI** (development) -- 9-matrix testing, Rust coverage, E2E tests
6. **Docker** (optional) -- Multi-stage containerized deployment

## Sources Used

- `docs/C4-Documentation/c4-container.md` -- Container deployment architecture
- `docs/C4-Documentation/c4-component.md` -- Component relationships
- `README.md` -- Project description and status (alpha)
- `AGENTS.md` -- Project structure, commands, coding standards, PR workflow
- `docs/design/01-roadmap.md` -- Implementation phases and target user
- `docs/design/02-architecture.md` -- System architecture, hybrid Python/Rust rationale
- `docs/design/05-api-specification.md` -- REST API endpoint reference, AI integration examples
- `docs/design/08-gui-architecture.md` -- Web GUI design, Effect Workshop, AI Theater Mode
- `pyproject.toml` -- Python dependencies and build configuration
- `gui/package.json` -- Frontend dependencies
- `rust/stoat_ferret_core/Cargo.toml` -- Rust dependencies
- `Dockerfile` -- Containerized deployment configuration
- `tests/**/*.py` -- 60+ Python test files revealing implemented features
- `gui/src/**/*.test.{ts,tsx}` -- 27 frontend component/hook tests
- `gui/e2e/*.spec.ts` -- 6 E2E test specs (navigation, scan, project creation, accessibility, effect workshop)
- API route definitions in `src/stoat_ferret/api/routers/` -- 23 registered endpoints verified
