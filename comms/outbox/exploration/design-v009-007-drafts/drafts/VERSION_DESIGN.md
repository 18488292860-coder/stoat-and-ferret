# Version Design: v009

## Description

Complete the observability pipeline (FFmpeg metrics, audit logging, file-based logs) and fix GUI runtime gaps (SPA routing, pagination, WebSocket broadcasts).

All 6 items are wiring gaps — existing components and infrastructure built in earlier versions but never connected to the application runtime. No new functionality is being built.

## Design Artifacts

Full design analysis available at: `comms/outbox/versions/design/v009/`

## Constraints and Assumptions

- BL-057 depends on BL-056 (structured logging, completed in v008)
- AuditLogger requires a separate sync `sqlite3.Connection` — aiosqlite has no public API to access its underlying connection
- SPA fallback route replaces `StaticFiles` mount and must be conditional on `gui/dist/` existing (LRN-020)
- Catch-all route must serve both static files and SPA fallback (Route and Mount cannot coexist at same path)
- Frontend pagination must be added to Projects page to meet BL-064 AC3
- All referenced classes exist with expected interfaces (verified in Task 004)

See `001-environment/version-context.md` for full context.

## Key Design Decisions

- **SPA routing:** Replace `StaticFiles` mount with dual-purpose catch-all route (see `006-critical-thinking/risk-assessment.md`)
- **AuditLogger connection:** Use separate sync `sqlite3.Connection` instead of accessing aiosqlite internals (see `006-critical-thinking/risk-assessment.md`)
- **WebSocket broadcasts:** SCAN events fire from job handler, not videos router; PROJECT_CREATED fires from projects router (see `006-critical-thinking/investigation-log.md`)
- **Pagination scope increase:** BL-064 requires frontend pagination UI on Projects page, following Library page pattern (see `006-critical-thinking/risk-assessment.md`)
- **Theme ordering:** Theme 1 before Theme 2 to avoid concurrent `app.py` modifications (LRN-042)

## Theme Overview

| # | Theme | Goal | Features | Backlog Items |
|---|-------|------|----------|---------------|
| 1 | observability-pipeline | Wire observability components into DI chain and startup | 3 | BL-059, BL-060, BL-057 |
| 2 | gui-runtime-fixes | Fix GUI and API runtime gaps | 3 | BL-063, BL-064, BL-065 |

See THEME_INDEX.md for detailed theme and feature listing.
