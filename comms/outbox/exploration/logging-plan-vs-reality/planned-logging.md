# Planned Logging

## Ownership

Logging was designed across two versions:

| Version | Theme | Feature | What it owned |
|---------|-------|---------|---------------|
| v002 | 04-ffmpeg-integration | 004-ffmpeg-observability | structlog dependency, `configure_logging()`, `ObservableFFmpegExecutor` |
| v003 | 02-api-foundation | 002-externalized-settings | `settings.log_level` field |
| v003 | 02-api-foundation | 004-middleware-stack | Correlation ID via contextvars, FR-003 "Logging Integration" |

## v002: FFmpeg Observability Requirements

**Source:** `comms/inbox/versions/execution/v002/04-ffmpeg-integration/004-ffmpeg-observability/requirements.md`

### FR-001: Structured Logging
> - Log every FFmpeg command with:
>   - correlation_id (from context or generated)
>   - command args (truncated for safety)
>   - duration
>   - return code
>   - success/failure status

### FR-004: Add Dependencies
> - structlog>=24.0
> - prometheus-client>=0.20

### Acceptance Criteria
> - [ ] Structured logs emitted for every execution
> - [ ] Prometheus metrics incremented correctly
> - [ ] Metrics include duration histogram
> - [ ] Can wrap any executor implementation

### Implementation Plan Step 4: Configure Structlog
The implementation plan explicitly called for creating `src/stoat_ferret/logging.py` with a `configure_logging()` function. However, the plan only specified creating the function — it did **not** specify calling it from `app.py`, `__main__.py`, or any startup path.

> Create `src/stoat_ferret/logging.py`:
> ```python
> def configure_logging(json_format: bool = True, level: int = logging.INFO):
>     """Configure structlog for the application."""
> ```

The plan's "Usage Example" in the completion report showed `configure_logging()` being called manually:
> ```python
> from stoat_ferret.logging import configure_logging
> configure_logging(json_format=True)
> ```

But no plan step wired this into application startup.

## v002: Theme-Level Design

**Source:** `comms/inbox/versions/execution/v002/04-ffmpeg-integration/THEME_DESIGN.md`

> ### AD-003: Observability Layer
> ObservableFFmpegExecutor wraps any executor and adds:
> - Structured logging with correlation ID
> - Prometheus metrics
> - Duration tracking

> ### Success Criteria
> - Metrics and logging for all executions

## v003: Externalized Settings Requirements

**Source:** `comms/inbox/versions/execution/v003/02-api-foundation/002-externalized-settings/requirements.md`

### FR-001: Settings Class
> Create `src/stoat_ferret/api/settings.py` with Settings class:
> - `log_level`: str = "INFO"

### FR-005: Validation
> - `log_level` must be valid Python log level

No requirement specified that `log_level` should be wired to `configure_logging()` or any other logging infrastructure.

## v003: Middleware Stack Requirements

**Source:** `comms/inbox/versions/execution/v003/02-api-foundation/004-middleware-stack/requirements.md`

### FR-003: Logging Integration
> - Log correlation ID with each request
> - Use structlog for structured logging

### Acceptance Criteria
> - [ ] Request logs include correlation ID

This acceptance criterion was marked PASS in the completion report, but the check-logging-output exploration found that structlog is unconfigured and most log calls are silently dropped.

## v003: App Setup Implementation Plan

**Source:** `comms/inbox/versions/execution/v003/02-api-foundation/001-fastapi-app-setup/implementation-plan.md`

The `__main__.py` entry point was designed with a hardcoded `log_level="info"`:
> ```python
> uvicorn.run(app, host=settings.api_host, port=settings.api_port, log_level="info")
> ```

No step in any v003 implementation plan called `configure_logging()` during app startup.

## Summary of Planned Items

1. **structlog dependency** — planned in v002, added
2. **`configure_logging()` function** — planned in v002, created
3. **`ObservableFFmpegExecutor` with structlog** — planned in v002, created
4. **`settings.log_level`** — planned in v003, created
5. **Correlation ID logging integration** — planned in v003, partially done
6. **Wiring `configure_logging()` to startup** — never explicitly planned
7. **Wiring `settings.log_level` to logging config** — never explicitly planned
