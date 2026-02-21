# Theme: application-startup-wiring

## Goal

Wire existing but disconnected infrastructure — database schema creation, structured logging, and orphaned settings — into the FastAPI lifespan startup sequence, so a fresh application start produces a fully functional system without manual intervention. All three features share the same modification point (`src/stoat_ferret/api/app.py` lifespan) and represent the same class of work: connecting infrastructure built in v002/v003 that was never wired.

## Design Artifacts

See `comms/outbox/versions/design/v008/006-critical-thinking/` for full risk analysis.

## Features

| # | Feature | Backlog | Goal |
|---|---------|---------|------|
| 001 | database-startup | BL-058 | Wire create_tables() into lifespan so database schema is created automatically on startup |
| 002 | logging-startup | BL-056 | Call configure_logging() at startup and wire settings.log_level to control log verbosity |
| 003 | orphaned-settings | BL-062 | Wire settings.debug to FastAPI and settings.ws_heartbeat_interval to ws.py |

## Dependencies

- No external dependencies. All infrastructure already exists in the codebase.
- Internal feature dependencies: 001 must complete before 002 (shared app.py lifespan), 002 must complete before 003 (same reason).

## Technical Approach

- Add create_tables_async() to schema.py using existing aiosqlite connection; call from lifespan inside _deps_injected guard (see `004-research/codebase-patterns.md` for lifespan pattern)
- Make configure_logging() idempotent, then call before _deps_injected guard for universal observability (see `004-research/evidence-log.md` for type conversion details)
- Replace DEFAULT_HEARTBEAT_INTERVAL constant with get_settings().ws_heartbeat_interval at runtime (see `004-research/codebase-patterns.md` for settings access pattern)

## Risks

| Risk | Mitigation |
|------|------------|
| Logging activation breaks tests | Make configure_logging() idempotent; see `006-critical-thinking/risk-assessment.md` |
| Shared modification point (app.py) merge conflicts | Sequential feature execution (001 -> 002 -> 003); see `006-critical-thinking/risk-assessment.md` |
| configure_logging() placement relative to _deps_injected guard | Place before guard with idempotency fix; see `006-critical-thinking/investigation-log.md` |
