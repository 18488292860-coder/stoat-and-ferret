# Exploration: logging-plan-vs-reality

Structured logging was designed in v002 and v003, with all components created as specified — `configure_logging()`, `settings.log_level`, `ObservableFFmpegExecutor`, and 10 structlog loggers. However, `configure_logging()` was never called at application startup and `settings.log_level` was never consumed by any code. As a result, all `logger.info()` calls across the codebase are silently dropped, and the application's only functioning log output comes from uvicorn's hardcoded `log_level="info"`. The gap persisted from v002 through v007 without being caught by completion reports, quality gates, or retrospectives.

## Ownership

| Version | Theme | Feature | Role |
|---------|-------|---------|------|
| v002 | 04-ffmpeg-integration | 004-ffmpeg-observability | Created `configure_logging()`, structlog dependency, `ObservableFFmpegExecutor` |
| v003 | 02-api-foundation | 002-externalized-settings | Created `settings.log_level` setting |
| v003 | 02-api-foundation | 004-middleware-stack | Created correlation ID middleware, FR-003 "Logging Integration" |

## Did Completion Reports Catch the Gap?

**No.** All three completion reports passed all acceptance criteria:

- v002/004-ffmpeg-observability: 4/4 PASS — verified that `ObservableFFmpegExecutor` emits structlog calls, not that the calls produce visible output
- v003/002-externalized-settings: 5/5 PASS — verified the setting field exists, not that it's consumed
- v003/004-middleware-stack: 3/3 PASS — "Request logs include correlation ID" verified code paths, not observable output

The only document that flagged the gap was the v003/002-externalized-settings **handoff-to-next.md**, which noted: "The `log_level` and `debug` settings are available but not yet wired to actual logging configuration" and suggested "Wire `log_level` to structlog configuration." This was never acted on.

## Root Causes

1. **Design gap:** The v002 implementation plan created `configure_logging()` but never specified calling it from any startup path
2. **Design gap:** The v003 settings requirements scoped `log_level` as "define the field" not "wire it to logging"
3. **Verification gap:** Acceptance criteria tested code paths (does the code emit log calls?) not observable behaviour (is log output visible?)
4. **Process gap:** Handoff suggestions have no mechanism to become tracked backlog items

## Files

- [planned-logging.md](planned-logging.md) — All logging requirements from design documents
- [implemented-logging.md](implemented-logging.md) — What was delivered and what completion reports said
- [gap-analysis.md](gap-analysis.md) — Specific gaps with root cause analysis
