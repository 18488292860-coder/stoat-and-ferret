# Exploration: Create User Documentation

## Summary

Created comprehensive user documentation for the stoat-and-ferret project, consisting of 16 markdown files organized into two documentation folders.

## Output

### docs/setup/ (6 files)

Setup and installation documentation for getting the development environment running.

| File | Description |
|------|-------------|
| [README.md](setup/README.md) | Overview and index of setup documentation |
| [prerequisites.md](setup/prerequisites.md) | Required tools: Python 3.10+, Rust, Node.js, FFmpeg, uv, maturin. OS-specific notes for Windows/Linux/macOS |
| [development-setup.md](setup/development-setup.md) | Step-by-step dev environment setup: clone, venv, Rust build, GUI build, database init, dev server |
| [docker-setup.md](setup/docker-setup.md) | Docker-based setup: multi-stage build, docker-compose usage, volume mounts |
| [configuration.md](setup/configuration.md) | All STOAT_ environment variables, .env file support, alembic.ini settings, API server options |
| [troubleshooting.md](setup/troubleshooting.md) | Common issues: Rust build failures, maturin path issues, FFmpeg not found, migration errors, Windows-specific issues |

### docs/manual/ (10 files)

User manual documentation covering the REST API, effects system, GUI, and AI integration.

| File | Description |
|------|-------------|
| [README.md](manual/README.md) | Overview and index of user manual |
| [getting-started.md](manual/getting-started.md) | Quick start: start server, access GUI, first API call, create project |
| [api-overview.md](manual/api-overview.md) | REST API conventions: base URL, error format, pagination, endpoint groups |
| [api-reference.md](manual/api-reference.md) | Detailed endpoint reference with curl examples for all routers (videos, projects, clips, effects, jobs, health, WebSocket) |
| [effects-guide.md](manual/effects-guide.md) | Effects system: 9 available effects, parameter schemas, applying/updating/deleting effects, transitions, Effect Workshop |
| [timeline-guide.md](manual/timeline-guide.md) | Timeline and clip management: frame-based coordinates, project creation, clip operations, validation |
| [gui-guide.md](manual/gui-guide.md) | Web GUI guide: Dashboard, Library, Projects, Effect Workshop. SPA routing notes |
| [rendering-guide.md](manual/rendering-guide.md) | Rendering and export [Planned]: planned API design for render jobs, quality options, hardware acceleration |
| [ai-integration.md](manual/ai-integration.md) | AI integration: OpenAPI discovery, effect AI hints, programmatic Python workflows, natural language translation examples |
| [glossary.md](manual/glossary.md) | Glossary of 30+ terms: clip, timeline, effect, filter string, FFmpeg, PyO3, maturin, and more |

## Documentation Approach

- All content is grounded in actual source code: router implementations, Pydantic schemas, settings, Cargo.toml, package.json, Dockerfile, CI workflow
- Planned but unimplemented features are consistently marked with `[Planned]` tags
- Concrete curl commands and Python code examples throughout
- Cross-references between manual and setup docs
- Written for a developer audience without assuming familiarity with the specific tech stack

## Files Created

```
docs/
  setup/
    README.md
    prerequisites.md
    development-setup.md
    docker-setup.md
    configuration.md
    troubleshooting.md
  manual/
    README.md
    getting-started.md
    api-overview.md
    api-reference.md
    effects-guide.md
    timeline-guide.md
    gui-guide.md
    rendering-guide.md
    ai-integration.md
    glossary.md
```
