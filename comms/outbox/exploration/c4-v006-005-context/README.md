# C4 Context Documentation — v006 Task 005

This exploration produced the system-level C4 context documentation for stoat-and-ferret v006. The document identifies 3 personas, 10 system features, 4 user journeys, and 3 external dependencies. The system's core purpose is to provide a programmatically controllable, AI-friendly video editing API that any HTTP client — human-operated browser or AI agent — can use to manage a video library, build editing projects, apply FFmpeg-backed effects, and render output files.

## System Purpose

stoat-and-ferret is a self-hosted, single-machine video editor designed for programmatic control. Its defining characteristic is that AI agents can discover what the system can do — via the effects discovery endpoint — and then issue the same API calls a human would make. This makes the system usable by natural-language automation without custom integrations. The Rust core library provides compile-time safety guarantees for all path validation and FFmpeg filter generation.

## Personas

Three personas were identified:

1. **Video Editor (Human User)** — Uses the web GUI to manage the video library, build editing projects, apply effects, and trigger rendering. Interacts primarily through the React SPA served at `/gui`.

2. **AI Agent (Programmatic User)** — Uses the REST API directly, driven by natural language instructions. The effects endpoint returns machine-readable schemas and AI hints specifically to support this persona. This persona was inferred from the architecture documents (AI-First Design section) and the `ai_hints` field present in every effect definition.

3. **Operator / Developer** — Deploys and configures the system, monitors health checks and Prometheus metrics, and contributes to the codebase. Distinct from the Video Editor because their primary interaction is with `/health`, `/metrics`, and structured logs rather than the editing workflow.

A fourth possible persona — a Prometheus monitoring system — was intentionally merged into an external dependency rather than a persona, since it is a system rather than a user.

## Features

Ten system features were identified:

| Feature | Discovery source |
|---------|-----------------|
| Video Library Management | API spec, test_videos.py, container doc |
| Video Search | API spec, container doc (FTS5 index) |
| Project and Timeline Management | API spec, test_projects.py, test_core_workflow.py |
| Clip Editing | API spec, test_clips.py |
| Effect Application | API spec, test_effects.py, v006 architecture doc |
| Effect Discovery | API spec, test_effects.py (test_list_effects_returns_all_effects) |
| Asynchronous Job Queue | API spec, test_jobs.py, architecture doc |
| Real-Time Dashboard | Container doc (Web GUI section), GUI component doc |
| Health Checks | API spec, test_health.py |
| Metrics Endpoint | Container doc, architecture doc |

## User Journeys

Four journeys were created, one per persona interaction pattern:

1. **Video Editor: Import and Browse the Video Library** — covers the scan-then-browse workflow including async job polling.
2. **Video Editor: Build a Project and Apply an Effect** — covers project creation, clip addition with validation, and effect application with Rust-generated filter string.
3. **AI Agent: Discover Effects and Apply via REST API** — covers the effect discovery workflow that the API is specifically designed for.
4. **Operator: Monitor System Health** — covers health probes, metrics scraping, and log correlation.

All journeys are qualified as current behavior (alpha, single-machine deployment). The render-to-file journey was not included as a separate journey because the render infrastructure is documented in the design docs as partially specified (render service described in architecture but render endpoint not yet fully implemented in v006).

## External Dependencies

Three external dependencies were identified:

1. **FFmpeg / ffprobe** — subprocess, required. Verified by test_executor.py, test_ffprobe.py, and the readiness health check code.
2. **Prometheus** — optional HTTP scraper. Confirmed optional by container doc ("optional" label) and architecture doc.
3. **Local File Storage** — local filesystem. Confirmed by container doc (File Storage container) and `STOAT_ALLOWED_SCAN_ROOTS` environment variable.

No other external systems were found. There is no remote database, no message broker, no external authentication service, and no cloud storage in the current implementation.

## Sources Used

- `docs/C4-Documentation/c4-container.md`
- `docs/C4-Documentation/c4-component.md`
- `docs/design/02-architecture.md`
- `docs/design/03-prototype-design.md`
- `docs/design/05-api-specification.md`
- `docs/design/01-roadmap.md`
- `AGENTS.md`
- `README.md`
- `tests/test_api/test_effects.py`
- `tests/test_api/test_videos.py`
- `tests/test_api/test_projects.py`
- `tests/test_blackbox/test_core_workflow.py`
- `tests/test_security/` (directory listing)

## Output

Context documentation written to: `docs/C4-Documentation/c4-context.md`
