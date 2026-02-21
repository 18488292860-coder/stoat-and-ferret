# Gap Analysis

## Gap 1: `configure_logging()` Never Called at Startup

**Designed:** v002/04-ffmpeg-integration/004-ffmpeg-observability implementation plan Step 4 created the function.

**Implemented:** The function exists in `src/stoat_ferret/logging.py` exactly as specified.

**Missing:** No design document or implementation plan ever specified calling `configure_logging()` from `app.py`, `__main__.py`, or the lifespan context manager. The v002 implementation plan has 5 steps: add dependencies, create metrics module, create observable executor, configure structlog (create the file), and add tests. None of these steps say "call `configure_logging()` during application startup."

**Root cause: Design gap.** The implementation plan specified creating the function but not wiring it into the application lifecycle. The implementer followed the plan correctly — the plan was incomplete.

## Gap 2: `settings.log_level` Never Consumed

**Designed:** v003/02-api-foundation/002-externalized-settings FR-001 specified adding `log_level` to the Settings class.

**Implemented:** The field exists with `Literal["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]` validation and `STOAT_LOG_LEVEL` env var support.

**Missing:** No requirement or implementation plan specified passing `settings.log_level` to `configure_logging()` or to `uvicorn.run()`. The v003 `__main__.py` implementation plan hardcodes `log_level="info"` for uvicorn.

**Root cause: Design gap.** The settings feature was scoped to "define the setting" not "wire the setting to its consumer." The handoff document explicitly noted this: "The `log_level` and `debug` settings are available but not yet wired to actual logging configuration." However, no subsequent version picked up this TODO.

## Gap 3: Structlog INFO Calls Silently Dropped

**Designed:** v003 FR-003 specified "Use structlog for structured logging" and the acceptance criterion "Request logs include correlation ID" was marked PASS.

**Implemented:** 10 modules create `structlog.get_logger(__name__)` and emit `logger.info()`, `logger.warning()`, and `logger.error()` calls.

**Missing:** Without `configure_logging()`, structlog proxies to Python's stdlib logging with no handlers. Python's `lastResort` handler only shows WARNING+, so all `logger.info()` calls are silently dropped.

**Root cause: Verification gap.** The acceptance criterion "Request logs include correlation ID" was verified by checking that the code emits log calls with correlation ID bound — not by verifying that those log calls produce visible output. The distinction between "code emits a log call" and "log output is actually visible" was never tested.

## Gap 4: Handoff TODO Never Executed

**Source:** `comms/outbox/versions/execution/v003/02-api-foundation/002-externalized-settings/handoff-to-next.md`

> ## Next Feature Suggestions
> 1. Wire `log_level` to structlog configuration

This was noted by the v003 implementer but was never picked up by any subsequent version (v004 through v007). The suggestion was in a handoff document, which is informational — it does not create backlog items or requirements. No backlog item was created to track this work.

**Root cause: Process gap.** Handoff "suggestions" have no mechanism to become tracked work. They exist only as context for the next feature in the same theme, not as durable action items.

## Summary

| Gap | Type | Where it fell through |
|-----|------|-----------------------|
| `configure_logging()` not called | Design gap | v002 implementation plan omitted the wiring step |
| `settings.log_level` not consumed | Design gap | v003 requirements scoped to "define" not "wire" |
| INFO logs silently dropped | Verification gap | Acceptance tested code paths, not observable output |
| Handoff TODO ignored | Process gap | No mechanism to promote handoff suggestions to backlog |

All four completion reports passed all acceptance criteria. All quality gates (ruff, mypy, pytest) passed. No retrospective (v002 through v007) flagged the issue. The gap persisted for 6 versions (v002 through v007) and was only discovered by the `check-logging-output` exploration in v007.
