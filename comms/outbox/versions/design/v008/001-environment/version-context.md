# Version Context — v008

## Version Goals

**v008 — Startup Integrity & CI Stability**

Fix P0 blockers and critical startup wiring gaps discovered by the wiring audit. After v008, a fresh install starts cleanly with working logging, database, and settings.

## Referenced Backlog Items

| ID | Priority | Summary |
|----|----------|---------|
| BL-055 | P0 | Fix flaky E2E test (`project-creation.spec.ts:31` toBeHidden timeout) |
| BL-056 | P1 | Call `configure_logging()` at startup, wire `settings.log_level` |
| BL-058 | P0 | Wire `create_tables()` into lifespan startup |
| BL-062 | P2 | Wire `settings.debug` to FastAPI and `settings.ws_heartbeat_interval` to ws.py |

Full backlog item details will be gathered in Task 002.

## Theme Structure

**Theme 1: application-startup-wiring** (3 features)
- 001-database-startup [BL-058, P0]
- 002-logging-startup [BL-056, P1]
- 003-orphaned-settings [BL-062, P2]

**Theme 2: ci-stability** (1 feature)
- 001-flaky-e2e-fix [BL-055, P0]

## Dependencies and Constraints

- **No inter-version dependencies.** All v008 items are independent startup/CI fixes.
- **No investigation prerequisites.** All items are well-understood wiring gaps.
- **No cross-theme dependencies.** Each feature can be implemented independently.

## Risk Factors

- **BL-055 (flaky E2E):** May require Playwright timing investigation. The test `project-creation.spec.ts:31` has a `toBeHidden` timeout flake, likely a race condition.
- **All other items:** Low risk — straightforward wiring of existing code into startup lifecycle.

## Deferred Items to Be Aware Of

| Item | Deferred From | To | Notes |
|------|---------------|----|-------|
| Phase 3: Composition Engine | v008 (original roadmap) | post-v010 | Wiring audit fixes take priority over new features |
| Drop-frame timecode support | M1.2 | TBD | Not relevant to v008 |

## Context from Prior Versions

- **v007** completed the Effect Workshop GUI (4 themes, 11 features). No items deferred to v008.
- **v006** completed the Effects Engine Foundation (3 themes, 8 features). No items deferred to v008.
- **v005** deferred SPA fallback routing (now BL-063, scheduled for v009) and WebSocket connection consolidation.

## Assumptions

1. The wiring audit backlog items (BL-055 through BL-068) were created during a comprehensive audit between v007 and v008.
2. v008 scope is intentionally small (4 items) to address the most critical P0/P1 startup issues first.
3. Less critical wiring gaps continue in v009 (observability, GUI runtime) and v010 (API cleanup).
