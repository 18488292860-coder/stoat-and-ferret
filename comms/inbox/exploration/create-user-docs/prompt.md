
# Create User Manual and Setup Documentation

## Objective
Create two documentation folders under `docs/` with comprehensive, detailed documentation for the stoat-and-ferret project:
1. `docs/manual/` — User manual documentation
2. `docs/setup/` — Setup and installation documentation

## Context
Stoat-and-ferret is a programmatic video editor with:
- **Python/FastAPI** HTTP layer with REST API as the primary interface
- **Rust core** (`stoat_ferret_core`) for filter generation, timeline calculations, FFmpeg command building, input sanitization — exposed via PyO3/maturin
- **Web GUI** (React/Vite) served as static files from FastAPI, including an Effect Workshop
- **SQLite database** via Alembic migrations
- **Docker support** for containerized deployment
- **Quality tooling**: ruff, mypy, pytest, vitest

Key config files at root: `pyproject.toml`, `alembic.ini`, `docker-compose.yml`, `Dockerfile`, `uv.lock`
Source code: `src/` (Python), `rust/` (Rust core), `gui/` (frontend), `tests/` (test suite), `scripts/` (utility scripts)

## Research Phase
Before writing documentation, thoroughly examine:

1. **For Setup docs**: Read `pyproject.toml` for Python dependencies and scripts, `rust/stoat_ferret_core/Cargo.toml` for Rust dependencies, `gui/package.json` for frontend deps, `Dockerfile` and `docker-compose.yml` for container setup, `alembic.ini` for database config, `.github/workflows/` for CI setup clues, `scripts/` for any setup scripts, and the project `README.md`.

2. **For User Manual**: Read `docs/design/05-api-specification.md` for API details, `docs/design/08-gui-architecture.md` for GUI details, `src/stoat_ferret/api/routers/` for actual endpoint definitions, `docs/C4-Documentation/c4-component-effects-engine.md` and `c4-component-web-gui.md` for component details, any existing API schemas in `src/stoat_ferret/api/schemas/`, the GUI source in `gui/src/` for feature understanding, and `docs/design/01-roadmap.md` through `04-technical-stack.md` for overall capabilities.

## Output Requirements

### docs/setup/ folder — must contain:

1. **README.md** — Overview and index of setup documentation
2. **prerequisites.md** — Detailed prerequisites: Python 3.12+, Rust toolchain, Node.js, FFmpeg, uv package manager, maturin. OS-specific notes for Windows/Linux/macOS.
3. **development-setup.md** — Step-by-step dev environment setup: cloning, Python venv with uv, Rust core compilation with maturin, GUI build with npm/vite, database initialization with alembic, environment variables, running the dev server.
4. **docker-setup.md** — Docker-based setup: building images, docker-compose usage, environment configuration, persistent volumes, common docker commands.
5. **configuration.md** — All configuration options: environment variables, alembic.ini settings, database paths, API server options (host, port, CORS), GUI static file serving, FFmpeg path configuration.
6. **troubleshooting.md** — Common setup issues and solutions: Rust compilation failures, PyO3 binding issues, maturin build problems, FFmpeg not found, database migration errors, Windows-specific issues (Smart App Control, path issues), GUI build failures.

### docs/manual/ folder — must contain:

1. **README.md** — Overview and index of user manual documentation
2. **getting-started.md** — Quick start guide: starting the server, accessing the GUI, making your first API call, creating a simple project.
3. **api-overview.md** — REST API guide: base URL, authentication (if any), content types, error format, pagination patterns. Overview of endpoint groups (projects, clips, effects, timeline, jobs, render).
4. **api-reference.md** — Detailed API endpoint reference organized by resource. For each endpoint: method, path, description, request body schema, response schema, example curl commands. Cover all routers found in the codebase.
5. **effects-guide.md** — Effects system guide: available effect types, effect parameters, applying effects to clips, previewing effects, the Effect Workshop GUI, creating custom effect chains, FFmpeg filter string transparency.
6. **timeline-guide.md** — Timeline and clip management: creating projects, importing clips, timeline operations (trim, split, reorder), frame-accurate editing, timeline math concepts.
7. **gui-guide.md** — Web GUI guide: navigating the interface, project management, clip browser, timeline editor, Effect Workshop usage, AI Theater Mode, keyboard shortcuts if any.
8. **rendering-guide.md** — Rendering and export: submitting render jobs, job queue system, monitoring job progress via API/WebSocket, output format options, quality settings.
9. **ai-integration.md** — AI integration guide: how the API is designed for AI-driven control, OpenAPI schema discoverability, programmatic workflow examples, AI Theater Mode for content viewing.
10. **glossary.md** — Terms and concepts: clip, timeline, effect, filter chain, render job, non-destructive editing, FFmpeg filter string, PyO3 binding, etc.

## Writing Guidelines
- Write for a developer audience but don't assume familiarity with the specific tech stack
- Include concrete code examples, curl commands, and sample API responses where applicable
- Use consistent markdown formatting with proper headings, code blocks, and cross-references between docs
- Mark any features that are planned but not yet implemented with a clear "[Planned]" tag
- Base all documentation on actual code found in the repository — don't invent features
- Cross-reference between manual and setup docs where appropriate

## Output Path
All output goes to: `comms/outbox/exploration/create-user-docs/`

Additionally, copy the final documentation to:
- `docs/manual/` (all manual files)
- `docs/setup/` (all setup files)

Create a `README.md` in `comms/outbox/exploration/create-user-docs/` summarizing what was created and listing all files.

## Commit Instructions
When complete, stage and commit all new files with message: "docs: add user manual and setup documentation"
Include all files under `docs/manual/`, `docs/setup/`, and `comms/outbox/exploration/create-user-docs/`.
