# Feature: logging-startup

## Goal

Call configure_logging() during application startup and wire settings.log_level so operators can control log verbosity.

## Background

**Backlog Item:** BL-056 (P1)

`configure_logging()` exists in `src/stoat_ferret/logging.py` and 9 modules emit structlog calls, but `configure_logging()` is never called at startup. `settings.log_level` (via `STOAT_LOG_LEVEL` env var) is defined but never consumed. All `logger.info()` calls are silently dropped (Python's `lastResort` handler only shows WARNING+), and the only functioning log output is uvicorn's hardcoded `log_level="info"`.

Critical finding from Task 006: `configure_logging()` is not idempotent — it unconditionally adds a StreamHandler to the root logger. Multiple calls accumulate duplicate handlers. This must be fixed before wiring into startup.

## Functional Requirements

**FR-001:** Application calls configure_logging() during startup
- Acceptance: configure_logging() is called during startup lifespan before any request handling

**FR-002:** settings.log_level controls root logger level
- Acceptance: settings.log_level value is passed to configure_logging() and controls the root logger level

**FR-003:** Log level environment variable produces visible output
- Acceptance: STOAT_LOG_LEVEL=DEBUG produces visible debug output on stdout

**FR-004:** Structured log output is visible at INFO level
- Acceptance: All existing logger.info() calls across the 9 structlog modules produce visible structured output at INFO level

**FR-005:** uvicorn respects configured log level
- Acceptance: uvicorn log_level uses settings.log_level.lower() instead of hardcoded "info"

**FR-006:** configure_logging() is idempotent
- Acceptance: Calling configure_logging() multiple times does not add duplicate handlers to the root logger

**FR-007:** Existing tests continue to pass
- Acceptance: Full pytest suite passes with logging active

## Non-Functional Requirements

**NFR-001:** Log level conversion is type-safe
- Metric: settings.log_level (Literal string) is correctly converted to logging int via getattr(logging, settings.log_level)

## Handler Pattern

Not applicable for v008 — no new handlers introduced.

## Out of Scope

- File-based logging (separate follow-on item)
- Log rotation or aggregation
- Structured logging format changes (JSON vs console)
- New structlog call sites

## Test Requirements

**Unit tests:**
- Test configure_logging() is called during lifespan startup (mock/spy verification)
- Test settings.log_level string is correctly converted to logging int
- Test uvicorn log_level reads from settings.log_level.lower() instead of hardcoded "info"
- Test configure_logging() idempotency — calling twice does not add duplicate handlers

**System tests:**
- Test STOAT_LOG_LEVEL=DEBUG produces visible debug output
- Test STOAT_LOG_LEVEL=INFO produces structured log output from at least one structlog logger
- Test default log level (INFO) works without explicit env var

**Existing test modifications:**
- test_logging.py needs root logger handler cleanup between tests (reset_defaults() only resets structlog, not stdlib logging handlers)
- Verify no existing tests break from newly-active logging output

## Reference

See `comms/outbox/versions/design/v008/004-research/` for supporting evidence.
