# Version Design: v008

## Description

**v008 — Startup Integrity & CI Stability**

Fix P0 blockers and critical startup wiring gaps so a fresh install starts cleanly with working logging, database, and settings. All changes wire existing infrastructure built in v002/v003 into the application startup sequence — no new feature development.

## Design Artifacts

Full design analysis available at: `comms/outbox/versions/design/v008/`

## Constraints and Assumptions

- All changes are wiring of existing infrastructure — no new features (see `001-environment/version-context.md`)
- Theme 1 features share `src/stoat_ferret/api/app.py` as a modification point and must execute sequentially
- CREATE TABLE IF NOT EXISTS is sufficient; Alembic migrations deferred until schema evolution is needed (LRN-029)
- configure_logging() must be made idempotent before wiring into startup (see `006-critical-thinking/risk-assessment.md`)
- BL-055 "10 consecutive CI runs" AC is a post-merge validation criterion, not a pre-merge gate

## Key Design Decisions

- **Database startup**: Use create_tables() with IF NOT EXISTS rather than Alembic — simpler for current scope (see `006-critical-thinking/risk-assessment.md`)
- **Logging placement**: configure_logging() placed before _deps_injected guard for universal observability (see `006-critical-thinking/investigation-log.md`)
- **Logging idempotency**: Guard addHandler() to prevent handler accumulation across test runs (see `006-critical-thinking/risk-assessment.md`)
- **E2E fix**: Explicit 10s timeout matching established project pattern rather than increasing global timeout (see `004-research/evidence-log.md`)
- **Settings wiring**: get_settings() at runtime (not import time) for ws_heartbeat_interval, following 4 existing callsite patterns (see `006-critical-thinking/investigation-log.md`)

## Theme Overview

| # | Theme | Backlog Items | Goal |
|---|-------|---------------|------|
| 1 | application-startup-wiring | BL-058, BL-056, BL-062 | Wire database, logging, and settings into FastAPI lifespan |
| 2 | ci-stability | BL-055 | Fix flaky E2E test blocking CI merges |
