# Retrospective Insights — v008 to v009

## Source

v008 retrospective: `comms/outbox/versions/execution/v008/retrospective.md`
Theme retrospectives: `docs/versions/v008/01-application-startup-wiring_retrospective.md`, `docs/versions/v008/02-ci-stability_retrospective.md`

## What Worked Well (continue in v009)

### 1. Tight scoping of wiring tasks yields first-iteration success
v008 achieved 4/4 features passing on first iteration with zero quality gate failures. All tasks were "wire existing code" rather than "build new features." v009 follows the same pattern — all 6 items are wiring gaps, not new functionality. This precedent suggests high first-iteration success rates.

### 2. Grouping features by modification point
Theme 01 grouped three features that all modified `app.py` lifespan, reducing context-switching. v009 Theme 1 similarly groups three observability features that all modify the DI/startup chain. Theme 2 groups three GUI runtime fixes that all touch the API/frontend layer.

### 3. mypy as a safety net for cross-module wiring
mypy caught an invalid `uvicorn.run(debug=...)` kwarg before runtime. Since v009 is entirely wiring work across module boundaries, mypy should be run after every change to catch similar issues early.

### 4. Idempotent startup patterns
Database schema creation uses `IF NOT EXISTS`, logging configuration guards against duplicate handlers. BL-057 (file logging) must follow this same pattern — the RotatingFileHandler must be idempotent to avoid duplicate log entries in tests.

## What Didn't Work (avoid in v009)

### 1. Deferred wiring creates accumulating dead code
BL-059 and BL-060 were created in v002 but never wired — 6 versions of dead code. v009 should complete all wiring in-scope rather than deferring any items. The AGENTS.md incremental binding rule addresses this for Rust, but the same principle applies to Python DI wiring.

### 2. Incomplete implementation across parallel components
BL-064 (pagination) happened because `count()` was added to one repository but not the other in v005. v009 implementation should verify that any pattern applied to one component is applied to all parallel components.

## Tech Debt: Addressed vs Deferred

### Addressed by v008 (foundation for v009)
- Database schema creation wired into lifespan startup
- Structured logging wired with `settings.log_level` (prerequisite for BL-057)
- All 9 Settings fields wired to consumers
- Flaky E2E test fixed

### Carried forward from v008 (relevant to v009)
- **Lifespan complexity growing:** BL-057 adds another handler to `configure_logging()`, which is called from the lifespan. Monitor but don't over-engineer.
- **E2E timeout audit:** BL-063 (SPA routing) may require new E2E tests — ensure explicit timeouts per LRN-043.
- **Settings consumption lint:** Not automated yet. Not blocking for v009 but worth noting.

### Addressed by v009
- ObservableFFmpegExecutor dead code since v002 (BL-059)
- AuditLogger dead code since v002 (BL-060)
- SPA routing deferred since v005 (BL-063)
- WebSocket broadcast calls deferred since v005 (BL-065)

## Architectural Decisions Informing v009

1. **DI via `create_app()` kwargs** (LRN-005): BL-059 and BL-060 must follow the constructor DI pattern, injecting via `create_app()` kwargs stored on `app.state`.
2. **Conditional static mount** (LRN-020): BL-063 SPA fallback must coexist with the conditional GUI mounting pattern that only mounts StaticFiles when the directory exists.
3. **Test double availability** (BL-059 AC4): The recording test double pattern must remain injectable — don't break the existing test infrastructure when wiring the observable executor.
4. **Idempotent startup** (LRN-040): BL-057's RotatingFileHandler must use the same handler dedup guard pattern established for stdout logging in v008.
