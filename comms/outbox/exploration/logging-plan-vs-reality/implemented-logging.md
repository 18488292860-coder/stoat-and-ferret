# Implemented Logging

## What Was Actually Delivered

Cross-referenced with the `check-logging-output` exploration (`comms/outbox/exploration/check-logging-output/`).

### Components Created

| Component | File | Status |
|-----------|------|--------|
| `configure_logging()` | `src/stoat_ferret/logging.py` | Created but **never called** |
| `settings.log_level` | `src/stoat_ferret/api/settings.py` | Created but **never consumed** |
| `ObservableFFmpegExecutor` | `src/stoat_ferret/ffmpeg/observable.py` | Created and functional (emits structlog calls) |
| FFmpeg Prometheus metrics | `src/stoat_ferret/ffmpeg/metrics.py` | Created and functional |
| Correlation ID middleware | `src/stoat_ferret/api/middleware/correlation.py` | Created and functional (headers work) |
| Prometheus metrics middleware | `src/stoat_ferret/api/middleware/metrics.py` | Created and functional |
| 10 structlog logger instances | Various modules | Created but output silently dropped (INFO level) |

### Effective Logging Behaviour

With `configure_logging()` never called:
- Structlog operates unconfigured, proxying to Python's stdlib logging
- No handlers are attached to the root logger
- Python's `lastResort` handler (stderr, WARNING+) is the only safety net
- All `logger.info()` calls across 10 modules are **silently dropped**
- Only `logger.warning()` and `logger.error()` might appear on stderr
- Uvicorn's own logging (access logs, startup) works independently via hardcoded `log_level="info"`

### Changelog Entries

**v002 CHANGELOG.md (project root):**
> - Observability infrastructure (structlog, Prometheus)

**v002 docs/CHANGELOG.md:**
> - `ObservableFFmpegExecutor` wrapper with structured logging (structlog)
> - Prometheus metrics for FFmpeg command execution (duration, success/failure counts)

**v003 CHANGELOG.md (project root):**
> - Correlation ID middleware for request tracing
> - Prometheus metrics middleware
> - Externalized configuration with pydantic-settings

**v003 docs/CHANGELOG.md:**
> - Middleware stack with correlation ID (`X-Correlation-ID` header) and Prometheus metrics

Neither changelog mentions `configure_logging()` or notes that logging needs to be wired to startup.

### Completion Reports

**v002/04-ffmpeg-integration/004-ffmpeg-observability (4/4 acceptance criteria PASS):**
- Lists `src/stoat_ferret/logging.py` as a created file with `configure_logging()`
- Shows a usage example that calls `configure_logging(json_format=True)` — but this example was never integrated into the actual application
- All quality gates passed (ruff, mypy, pytest)
- The acceptance criterion "Structured logs emitted for every execution" was marked PASS based on the `ObservableFFmpegExecutor` emitting structlog calls — not on whether those calls actually produce visible output

**v003/02-api-foundation/002-externalized-settings (5/5 acceptance criteria PASS):**
- Lists `log_level` as a new field
- Quality gates all passed

**v003/02-api-foundation/002-externalized-settings/handoff-to-next.md:**
This is the only document that flagged the gap:
> - The `log_level` and `debug` settings are available but not yet wired to actual logging configuration
>
> ## Next Feature Suggestions
> 1. Wire `log_level` to structlog configuration

**v003/02-api-foundation/004-middleware-stack (3/3 acceptance criteria PASS):**
- "Request logs include correlation ID" was marked PASS
- The handoff says to "Use `get_correlation_id()` in structured logging to correlate log entries"

### Quality Gates and Reviews

- All quality gates (ruff, mypy, pytest) passed for every logging-related feature
- No quality gate checks whether `configure_logging()` is called at startup
- No integration test verifies that application log output is actually visible
- Retrospectives for v002, v003, v004, v005, v006, and v007 contain no mention of `configure_logging` being uncalled or `log_level` being unused
